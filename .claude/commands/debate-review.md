---
description: 3-round cross-model code review giữa Claude và Codex (qua codex-plugin-cc). Output .debate-review/<run-id>/REPORT.md với findings phân loại AGREED / CONFIRMED / DISPUTED / RETRACTED.
---

Bạn là **debate-review orchestrator**. Bạn chạy phiên review 3 vòng giữa
Claude (qua subagent `debate-reviewer`) và Codex (qua plugin slash commands
`/codex:rescue` + `/codex:status` + `/codex:result`), rồi tổng hợp thành
REPORT.md.

Bạn KHÔNG tự review code. Bạn chỉ điều phối, render prompt, chờ job, parse
output, và viết REPORT.md cuối cùng. Mọi finding phải đến từ 1 trong 2 reviewer.

---

## 0. Parse args

`/debate-review $ARGUMENTS`

Các trường hợp:
- Không args → review `git diff` hiện tại (working tree, unstaged + staged)
- `--base <ref>` → review `git diff <ref>...HEAD` (vd: `--base main`)
- `--paths <glob>` → giới hạn review trong paths này (vd: `--paths "src/**"`)
- Free-form intent text (vd: `/debate-review focus on auth changes`)
  → dùng làm `intent_summary` cho prompt reviewer

Nếu ambiguous, hỏi user qua `AskUserQuestion`. ĐỪNG đoán.

---

## 1. Pre-flight checks (FAIL FAST)

### 1a. Plugin Codex ready
Gọi slash command:
```
/codex:setup
```
Parse output: nếu báo "Codex missing" hoặc "not logged in" → STOP với message:
```
debate-review yêu cầu codex-plugin-cc. Chạy:
  /plugin install codex@openai-codex
  /codex:setup
  !codex login    # nếu chưa login
```

### 1b. Trong git repo
```bash
git rev-parse --git-dir
```

### 1c. Có diff để review
```bash
git diff --stat     # hoặc git diff <base>...HEAD --stat theo args
```
Trống → STOP với "Không có changes để review."

---

## 2. Setup run directory

```bash
RUN_ID="$(date -u +%Y-%m-%d-%H%M)-$(openssl rand -hex 3)"
RUN_DIR=".debate-review/${RUN_ID}"
mkdir -p "${RUN_DIR}/round-1" "${RUN_DIR}/round-2" "${RUN_DIR}/round-3"
```

Lưu diff snapshot:
```bash
# tùy args:
git diff                     > "${RUN_DIR}/diff.patch"     # working tree
# hoặc:
git diff <base>...HEAD       > "${RUN_DIR}/diff.patch"     # branch compare
```

Resolve paths:
- `SCHEMA_PATH = <skill-root>/templates/finding-schema.md` (relative tới repo root; copy vào `${RUN_DIR}/finding-schema.md` để Codex subshell đọc được nếu cần)
- `INTENT` — từ free-form args nếu có, hoặc subject của commit gần nhất (`git log -1 --pretty=%s`), hoặc "no intent provided"
- `STACK_HINT` — auto-detect từ root files: `package.json` (Node/TS), `Cargo.toml` (Rust), `go.mod` (Go), `pyproject.toml` (Python), `Gemfile` (Ruby)... Đọc thêm `README.md` § Stack/Tech nếu có. Không tìm được → "auto-detect from diff"

Khởi tạo ledger `${RUN_DIR}/state.json`:
```json
{
  "run_id": "...",
  "started_at": "<ISO>",
  "diff_path": ".debate-review/<id>/diff.patch",
  "codex_task_id": null,
  "rounds_completed": []
}
```

---

## 3. Round 1 — Independent review (PARALLEL)

Spawn 2 reviewer ĐỒNG THỜI, blind (không thấy findings của bên kia).

### 3a. Claude reviewer

Render prompt từ `templates/round-prompts.md` § Round 1:
- `reviewer_name = Claude`
- `output_path = ${RUN_DIR}/round-1/claude.md`
- các path khác như step 2

Spawn subagent chạy nền:
```
Agent({
  subagent_type: "debate-reviewer",
  description: "Claude round 1 review",
  prompt: <rendered>,
  run_in_background: true
})
```

### 3b. Codex reviewer

Render prompt tương tự với `reviewer_name = Codex`, `output_path =
${RUN_DIR}/round-1/codex.md`.

**Invoke qua plugin** (slash command, chạy background để parallel với Claude):

```
/codex:rescue --model gpt-5.5 --effort medium --background --fresh <RENDERED_PROMPT>
```

Trong prompt, yêu cầu Codex:
- Read-only review (không edit code production)
- Ghi findings theo schema vào đúng `output_path`
- Tuân thủ mọi quy tắc trong `finding-schema.md` (copy nội dung schema vào prompt vì Codex session có thể không có tool context nhìn thấy file này cùng cách Claude thấy)

Plugin trả về `task-id` (vd `task-abc123`). Lưu vào `state.json.codex_task_id`
— sẽ dùng `--resume` ở round 2/3.

### 3c. Wait both

Poll Codex:
```
/codex:status <task-id>
```
Loop đến khi status = `done` / `failed`. Trong lúc poll có thể check
status của Claude subagent (background task).

Khi Codex `done`:
```
/codex:result <task-id>
```
→ verify file `${RUN_DIR}/round-1/codex.md` tồn tại + có `### F-` blocks
(hoặc `_No findings._`). Nếu Codex không ghi file (vd chỉ output ra final
message), fallback: grab final message từ `/codex:result` và SAVE thủ công
vào `output_path`.

Verify Claude tương tự: `${RUN_DIR}/round-1/claude.md` tồn tại + đúng schema.

Nếu 1 bên fail: retry 1 lần với prompt kèm error message. Vẫn fail → STOP.

Update `state.json.rounds_completed = ["1"]`.

---

## 4. Round 2 — Cross-rebuttal (PARALLEL)

### 4a. Claude rebuts Codex

Render Round 2 prompt:
- `reviewer_name = Claude`, `opponent_name = Codex`
- `opponent_findings_path = ${RUN_DIR}/round-1/codex.md`
- `own_findings_path = ${RUN_DIR}/round-1/claude.md`
- `output_path = ${RUN_DIR}/round-2/claude-rebuttals.md`

Spawn `debate-reviewer` subagent background.

### 4b. Codex rebuts Claude

Render Round 2 prompt (đảo vai), `output_path = ${RUN_DIR}/round-2/codex-rebuttals.md`.

**Invoke với `--resume`** để Codex giữ nguyên session context (diff đã đọc,
lập luận round 1):

```
/codex:rescue --model gpt-5.5 --effort medium --background --resume <task-id-từ-state.json> <RENDERED_PROMPT>
```

> Resume tiết kiệm token đáng kể ở round 2+. Không cần re-feed diff.

Trong prompt Round 2 cho Codex, **attach nội dung file `claude.md`** inline
(Codex không tự động đọc file mà Claude subagent tạo). Dùng heredoc hoặc
trích nguyên văn markdown của `${RUN_DIR}/round-1/claude.md` vào prompt.

### 4c. Wait both + verify

Poll như 3c. Verify:
- Mỗi finding trong `opponent_findings_path` có 1 `R-` block trong
  rebuttals file tương ứng (đếm so sánh).
- Thiếu finding → retry 1 lần với prompt liệt kê explicit findings còn thiếu.

**Short-circuit**: nếu cả 2 file rebuttal đều `agree` 100% → skip Round 3,
jump thẳng Step 6.

Update `state.json.rounds_completed = ["1", "2"]`.

---

## 5. Round 3 — Final positions (PARALLEL, nếu cần)

### 5a. Claude final
- `opponent_rebuttals_path = ${RUN_DIR}/round-2/codex-rebuttals.md`
- `own_findings_path = ${RUN_DIR}/round-1/claude.md`
- `output_path = ${RUN_DIR}/round-3/claude-final.md`

Spawn `debate-reviewer` subagent.

### 5b. Codex final

Render Round 3 prompt với `output_path = ${RUN_DIR}/round-3/codex-final.md`.

Invoke với `--resume`:
```
/codex:rescue --model gpt-5.5 --effort medium --background --resume <task-id> <RENDERED_PROMPT_R3>
```

Attach nội dung `claude-rebuttals.md` inline trong prompt (giống 4b).

### 5c. Wait + verify

Mỗi finding bị refute/escalate ở round 2 phải có 1 `P-` block ở round 3.
Thiếu → tự động đánh dấu "weak hold" trong synthesis (không retry, đó là
signal về chất lượng defense).

Update `state.json.rounds_completed = ["1", "2", "3"]`.

---

## 6. Synthesis — Viết REPORT.md

Orchestrator (bạn) đọc tất cả 6 file (hoặc 4 nếu skip round 3). Không gọi
Codex nữa — đây là phần bạn tự làm.

### 6a. Match findings hai bên cùng raise

Gộp findings cross-side khi:
- Cùng `file:line` ± 5 dòng
- Cùng `Category`
- Overlap key terms trong Claim > 50%

→ Tạo 1 finding AGREED, gộp claim (chọn bản chi tiết hơn), severity = max.

### 6b. Bucket findings còn lại

Theo logic trong `templates/final-report.md` § Synthesis logic:

```
opponent_stance (round 2) × final_position (round 3) → bucket
```

- stance=agree → AGREED
- stance=refute + position=hold → DISPUTED
- stance=refute + position=concede → RETRACTED
- stance=refute + position=partial → DISPUTED (+ Updated claim)
- stance=refute + no position → DISPUTED (weak hold warning)
- stance=escalate + position=hold/partial → DISPUTED
- stance=escalate + position=concede → RETRACTED

### 6c. Render REPORT.md

Theo template `templates/final-report.md` → ghi vào `${RUN_DIR}/REPORT.md`.

### 6d. Metrics

- Convergence rate = 1 - DISPUTED / total_raised
- Concedes per side
- Wall time từ `state.json.started_at`

Đưa vào section "Process notes".

---

## 7. Output cho user

In tóm tắt (≤15 dòng):

```
✅ Debate review xong.

Run: <RUN_DIR>
Reviewers: Claude (debate-reviewer) vs Codex (task-<id>)
Rounds: 3 (hoặc 2 nếu short-circuit)

Kết quả:
  AGREED:    <n>  ← fix trước merge
  CONFIRMED: <n>  ← fix (1 bên đã concede sau debate)
  DISPUTED:  <n>  ⚠️  cần human quyết
  RETRACTED: <n>  (đã loại)

Convergence: <pct>%   Wall time: <X>m<Y>s

Full report: <RUN_DIR>/REPORT.md
Codex session có thể resume lại:  codex resume <session-id>    (xem REPORT.md)
```

Nếu DISPUTED > 0, list 1 dòng/finding (file:line + tiêu đề) — phần đáng đọc nhất.

---

## Quy tắc quan trọng

1. **KHÔNG tự review.** Mọi finding đến từ 1 trong 2 reviewer.
2. **KHÔNG sửa code.** Report-only skill.
3. **KHÔNG mediate.** Disputed = disputed. Orchestrator được phép ghi
   `Recommend:` 1 dòng trong DISPUTED nhưng không tự bucket.
4. **PARALLEL trong round, sequential giữa round.**
5. **Snapshot diff EARLY.** Cả 2 reviewer review CÙNG state.
6. **`--resume` cho Codex ở round 2/3.** Giữ session context, tiết kiệm token.
7. **Attach file content inline vào Codex prompt.** Codex subshell không
   thấy files do Claude subagent tạo theo cách Claude main thấy. Phải copy
   nội dung vào prompt.
8. **Retry 1 lần** khi output sai schema; lần 2 fail → STOP.
9. **Lock**: `${RUN_DIR}/.lock` với run_id + timestamp. Khác run_id + <30 phút
   → halt. Release ở cuối.

---

## Failure modes

| Tình huống | Hành động |
|---|---|
| `/codex:setup` báo missing/not-logged-in | STOP + hướng dẫn user install/login |
| Codex task fail (status=failed) | Đọc `/codex:result` để lấy error, retry 1 lần, vẫn fail → STOP |
| Codex hết quota giữa round | STOP; state.json giữ lại để resume sau |
| Diff > 5000 dòng | Cảnh báo user, offer split theo `--paths` |
| Round 1: 1 bên xong, 1 bên fail | STOP — không có debate nếu thiếu 1 reviewer |
| Round 2: bên kia bỏ qua 1 finding | Retry với prompt liệt kê findings còn thiếu |
| Round 3: thiếu position cho finding bị refute | "Weak hold" warning trong REPORT, không retry |
| Codex không ghi file, chỉ trả final message | Fallback: orchestrator save final message (từ `/codex:result`) vào output_path |

---

## Ghi chú về plugin Codex

- `/codex:rescue` là lựa chọn duy nhất cho phép custom prompt theo schema.
  `/codex:review` và `/codex:adversarial-review` không steerable đủ cho
  debate-review format.
- `--background` để parallel với Claude subagent; `--wait` sẽ block.
- `--resume` reuse Codex session; `--fresh` bắt đầu session mới (chỉ dùng
  ở round 1).
- `/codex:status` check trạng thái; `/codex:result <task-id>` lấy output cuối.
- `/codex:cancel <task-id>` nếu cần abort.
- Session ID của Codex (khác với task-id) được `/codex:result` trả về —
  ghi vào REPORT.md để user có thể `codex resume <session-id>` nếu muốn
  tiếp tục điều tra findings sau này.
