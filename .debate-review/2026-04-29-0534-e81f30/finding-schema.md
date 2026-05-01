# Finding Schema (BẮT BUỘC)

Mọi finding ở mọi round PHẢI tuân theo schema này. Skill orchestrator dùng
`id` và `file:line` để so khớp findings giữa Claude và Codex.

## Format

Mỗi finding là một block markdown đứng riêng:

```markdown
### F-<NNN>: <tiêu đề ngắn>

- **Severity**: blocker | high | medium | low | nit
- **Category**: correctness | security | performance | contract | maintainability | style
- **File**: <path>:<line>  (hoặc `<path>:<line-start>-<line-end>`)
- **Status**: open                    <!-- round 1 only -->
- **Claim**: <1-3 câu mô tả vấn đề. Phải nêu RÕ điều gì sai và TẠI SAO.>
- **Evidence**: <code snippet, link tới spec, hoặc reference cụ thể>
- **Suggested fix**: <1 câu — không phải code, chỉ là hướng>
```

## Quy tắc

1. **ID format**: `F-001`, `F-002`, … đánh số tăng dần TRONG MỖI bên (Claude
   có F-001..F-NNN của Claude, Codex có F-001..F-MMM của Codex riêng). Skill
   sẽ rename thành `C-001` (Claude) và `X-001` (Codex) khi merge.
2. **File:line bắt buộc** — finding không có địa chỉ cụ thể bị skill loại.
3. **Claim phải falsifiable** — "code này tệ" KHÔNG hợp lệ. "Hàm `parseUser`
   tại line 42 không validate `email`, gây crash khi nhận `null`" hợp lệ.
4. **Evidence là yêu cầu** — khẳng định không kèm bằng chứng sẽ bị bên kia
   refute dễ dàng ở round 2.
5. **Không bịa file/line** — nếu không chắc, không emit finding.

## Round 2 (Rebuttal) — thêm field

```markdown
### R-<bên đối phương F-NNN>: <tiêu đề finding gốc>

- **Stance**: agree | refute | escalate
- **Reasoning**: <bằng chứng cụ thể vì sao agree/refute. Nếu refute, PHẢI
  trích code/spec/test chứng minh finding gốc sai hoặc overstated.>
- **Counter-evidence**: <file:line hoặc trích dẫn>
```

- `agree` — đồng ý finding của đối phương, không cần phản biện.
- `refute` — finding sai/overstated. Bắt buộc có Counter-evidence.
- `escalate` — không đủ context để phán; cần human hoặc thêm thông tin.

## Round 3 (Final position) — chỉ cho findings của CHÍNH MÌNH bị refute

```markdown
### P-<F-NNN của mình>: <tiêu đề>

- **Position**: hold | concede | partial
- **Reasoning**: <phản hồi rebuttal của đối phương. Hold = bảo vệ, concede
  = chấp nhận sai, partial = đúng một phần.>
- **Updated claim** (nếu partial): <claim mới sau khi điều chỉnh>
```

- Chỉ phản hồi nếu round 2 đối phương đã `refute` hoặc `escalate`.
- Findings được `agree` ở round 2 → tự động giữ nguyên, không cần round 3.

## Anti-patterns (sẽ bị orchestrator flag)

- ❌ "Có thể có vấn đề với …" → vague, không actionable
- ❌ Finding không có file:line
- ❌ Refute mà không có Counter-evidence
- ❌ Hold ở round 3 mà không phản hồi rebuttal cụ thể
- ❌ Findings trùng nhau trong cùng 1 bên (gộp lại)
