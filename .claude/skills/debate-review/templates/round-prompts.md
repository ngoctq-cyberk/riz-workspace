# Round Prompt Templates

3 prompt template, dùng CHUNG cho cả Claude reviewer (subagent `debate-reviewer`)
và Codex (qua plugin `/codex:rescue`). Orchestrator render các placeholder
`{{...}}` rồi pass vào.

## Khác biệt giữa Claude vs Codex khi dùng prompt này

| | Claude subagent | Codex (/codex:rescue) |
|---|---|---|
| Đọc file từ `{{...path}}` | Trực tiếp (Read tool, cùng filesystem view) | KHÔNG — Codex subshell có thể không thấy file do Claude tạo. Orchestrator phải **attach nội dung inline** vào prompt (heredoc / trích markdown nguyên văn) |
| Ghi output vào `{{output_path}}` | Trực tiếp (Write tool) | Codex có file write tool trong rescue — yêu cầu trong prompt. Fallback: orchestrator save từ `/codex:result` |
| Session giữa rounds | Mỗi round spawn subagent mới | `--resume <task-id>` giữ session → không cần re-feed diff |

**Nguyên tắc**: khi render prompt cho Codex, thay mọi `{{..._path}}` bằng
**nội dung file** inline (dùng fenced code block). Khi render cho Claude
subagent, giữ nguyên path.

---

## Round 1 — Independent Review

```
Bạn là code reviewer. Review diff dưới đây MỘT MÌNH, không tham khảo ai.

## Context
- Repo: {{repo_root}}
- Diff snapshot: {{diff_path}}
- Intent (nếu có): {{intent_summary}}
- Stack: {{stack_hint}}

## Yêu cầu
1. Đọc diff đầy đủ. Đọc thêm file gốc nếu cần để hiểu context.
2. Emit findings theo schema BẮT BUỘC tại {{schema_path}}.
3. Tập trung vào: correctness, security, contract, performance. Style/nit
   chỉ emit nếu thật sự đáng nhắc.
4. KHÔNG fix code. KHÔNG tự đánh giá "looks good". Chỉ raise findings.
5. Mỗi finding phải có file:line cụ thể và evidence cụ thể.

## Output
Ghi findings vào: {{output_path}}

Format file:
```markdown
# Round 1 — {{reviewer_name}}

<F-001 block>
<F-002 block>
...
```

Nếu không tìm thấy issue nào, ghi `# Round 1 — {{reviewer_name}}\n\n_No findings._`
```

---

## Round 2 — Cross-Rebuttal

```
Bạn là code reviewer. Đối phương ({{opponent_name}}) đã review cùng diff
và emit findings dưới đây. Nhiệm vụ của bạn: đọc TỪNG finding của họ và
quyết định agree / refute / escalate.

## Context
- Diff snapshot: {{diff_path}}
- Findings của đối phương: {{opponent_findings_path}}
- Findings của chính bạn (round 1): {{own_findings_path}}
- Schema: {{schema_path}} (xem section "Round 2")

## Yêu cầu
1. Đọc TẤT CẢ findings của đối phương. Đọc lại code thực tế nếu cần.
2. Cho mỗi finding của họ:
   - `agree` nếu đúng và quan trọng
   - `refute` nếu sai, overstated, hoặc đã được code xử lý mà họ miss
     → BẮT BUỘC trích Counter-evidence (file:line hoặc snippet)
   - `escalate` nếu thiếu context để phán
3. KHÔNG raise finding mới ở round này. Chỉ phản biện.
4. KHÔNG nhân nhượng vì lịch sự. Refute thẳng nếu thấy sai.

## Output
Ghi rebuttals vào: {{output_path}}

Format:
```markdown
# Round 2 — {{reviewer_name}} rebuts {{opponent_name}}

<R-<opponent F-NNN> block cho TỪNG finding của đối phương>
```
```

---

## Round 3 — Final Position

```
Bạn là code reviewer. Đối phương ({{opponent_name}}) đã refute hoặc
escalate một số findings của bạn ở round 2. Nhiệm vụ: phản hồi từng cái.

## Context
- Diff snapshot: {{diff_path}}
- Findings gốc của bạn (round 1): {{own_findings_path}}
- Rebuttals của đối phương (round 2): {{opponent_rebuttals_path}}
- Schema: {{schema_path}} (xem section "Round 3")

## Yêu cầu
1. CHỈ phản hồi findings của bạn bị `refute` hoặc `escalate`. Findings
   được đối phương `agree` → tự động giữ, bỏ qua.
2. Cho mỗi finding cần phản hồi:
   - `hold` nếu bạn vẫn đúng → PHẢI phản biện cụ thể vào Counter-evidence
     của họ. Không lặp lại claim cũ — phải có lập luận MỚI.
   - `concede` nếu họ đúng. Ghi rõ vì sao bạn sai.
   - `partial` nếu họ đúng một phần → ghi `Updated claim` mới đã thu hẹp.
3. KHÔNG raise finding mới. KHÔNG mở rộng scope.
4. Đây là round CUỐI. Không có round 4. Hold mà không phản biện được sẽ
   bị orchestrator hạ xuống "weak hold".

## Output
Ghi final positions vào: {{output_path}}

Format:
```markdown
# Round 3 — {{reviewer_name}} final positions

<P-F-NNN block cho TỪNG finding cần phản hồi>
```
```

---

## Variables reference

| Placeholder | Ý nghĩa | Claude render | Codex render |
|---|---|---|---|
| `{{reviewer_name}}` | "Claude" hoặc "Codex" | như-is | như-is |
| `{{opponent_name}}` | Bên còn lại | như-is | như-is |
| `{{diff_path}}` | `diff.patch` snapshot | path | **inline** nội dung patch (fenced ```diff) |
| `{{intent_summary}}` | Mô tả change (commit msg / change-id / args) | như-is | như-is |
| `{{stack_hint}}` | Stack ngắn (TS/React/Rust/Go/...) — auto-detect từ `package.json` / `Cargo.toml` / `go.mod` / `pyproject.toml` / README | như-is | như-is |
| `{{schema_path}}` | Finding schema | path | **inline** toàn bộ `finding-schema.md` |
| `{{own_findings_path}}` | File round trước của chính reviewer | path | **inline** content |
| `{{opponent_findings_path}}` | File round trước của đối phương | path | **inline** content |
| `{{opponent_rebuttals_path}}` | File round 2 của đối phương | path | **inline** content |
| `{{output_path}}` | Nơi ghi output round hiện tại | path (Claude Write tool) | path (Codex có file write tool trong rescue); nếu Codex không ghi được → orchestrator save từ `/codex:result` |
| `{{repo_root}}` | Tuyệt đối tới repo root | như-is | như-is |

---

## Example — cách render prompt cho Codex round 2

```
/codex:rescue --model gpt-5.5 --effort medium --background --resume <task-id> <<'PROMPT'
Bạn là code reviewer. Đối phương (Claude) đã review cùng diff và emit
findings dưới đây. Nhiệm vụ của bạn: đọc TỪNG finding của họ và quyết
định agree / refute / escalate.

## Context

### Diff snapshot

```diff
<<< NỘI DUNG TOÀN BỘ diff.patch >>>
```

### Findings của Claude (round 1)

```markdown
<<< NỘI DUNG round-1/claude.md >>>
```

### Findings của chính bạn (round 1, để nhớ lại quan điểm)

```markdown
<<< NỘI DUNG round-1/codex.md >>>
```

### Schema bắt buộc

```markdown
<<< NỘI DUNG finding-schema.md § Round 2 >>>
```

## Yêu cầu
<như template Round 2 ở trên>

## Output
Ghi rebuttals vào file: .debate-review/<run-id>/round-2/codex-rebuttals.md
Header: `# Round 2 — Codex rebuts Claude`
Nếu không ghi được file, output markdown trong final message và
orchestrator sẽ save.
PROMPT
```

Lưu ý `--resume` giúp Codex không cần re-đọc diff (đã có ở session từ
round 1), nhưng prompt vẫn nên include diff đầy đủ phòng trường hợp
session state không giữ.
