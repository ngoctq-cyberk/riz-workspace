# Final Report Template

Main agent (orchestrator) ghi file `REPORT.md` theo schema này sau khi
đã đọc đủ 6 file (round 1/2/3 × Claude/Codex).

---

```markdown
# Debate Review Report

- **Run ID**: {{run_id}}
- **Started**: {{started_iso}}
- **Diff**: {{diff_summary}}  (vd: `12 files, +340 / -120`)
- **Intent**: {{intent_summary}}
- **Reviewers**: Claude ({{claude_model}}) vs Codex ({{codex_model}})

## Summary

| Bucket | Count | Action |
|---|---|---|
| AGREED   | {{n_agreed}}    | Fix trước khi merge |
| CONFIRMED | {{n_confirmed}} | Fix — bên kia đã concede sau debate |
| DISPUTED | {{n_disputed}}  | **Cần human quyết** — hai model bất đồng |
| RETRACTED | {{n_retracted}} | Bỏ qua — bên raise đã rút |

> **Đọc gì trước**: AGREED + DISPUTED. CONFIRMED đã chốt; RETRACTED đã loại.

---

## AGREED (cả 2 đều raise hoặc đều agree)

<repeat per finding>

### A-001 — <tiêu đề> [severity: ...]

- **File**: `path:line`
- **Claim**: <merged claim, lấy bản chi tiết hơn>
- **Raised by**: Claude (C-NNN) + Codex (X-MMM)  | hoặc: Claude (C-NNN), Codex agreed (R-C-NNN)
- **Suggested fix**: <merged>

</repeat>

---

## CONFIRMED (1 bên raise, bên kia concede sau debate)

<repeat>

### C-001 — <tiêu đề> [severity: ...]

- **File**: `path:line`
- **Raised by**: Claude (C-NNN)
- **Codex stance**: refute (round 2) → concede (round 3)
- **Why Codex conceded**: <quote ngắn từ codex-final.md>
- **Claim**: <claim cuối, có thể là Updated claim nếu partial>

</repeat>

---

## DISPUTED (sau 3 rounds vẫn bất đồng) ⚠️

> Đây là tín hiệu giá trị nhất. Hai model thông minh không đồng ý nghĩa
> là vấn đề tinh tế — đọc kỹ cả hai phía trước khi quyết.

<repeat>

### D-001 — <tiêu đề> [severity per side]

- **File**: `path:line`
- **Raised by**: <bên>
- **Position của Claude**: <hold | partial | refute>
  - Lập luận: <tóm tắt từ round-3 file của Claude>
- **Position của Codex**: <hold | partial | refute>
  - Lập luận: <tóm tắt từ round-3 file của Codex>
- **Decision needed**: <câu hỏi cụ thể human cần trả lời>
- **Recommend**: <nếu orchestrator có thiên hướng nghiêng về bên nào, ghi
  ngắn gọn — nhưng KHÔNG được tự quyết>

</repeat>

---

## RETRACTED

| ID | Raiser | Tiêu đề | Lý do rút |
|----|--------|---------|-----------|
| R-001 | Claude (C-005) | … | Conceded round 3 — Codex chỉ ra test đã cover case này |

---

## Process notes

- Round 1: Claude raised {{n_c1}} findings, Codex raised {{n_x1}}.
- Round 2: Claude refuted {{n_c2_refute}}/{{n_x1}}, Codex refuted {{n_x2_refute}}/{{n_c1}}.
- Round 3: {{n_concedes}} concedes total ({{n_claude_concedes}} Claude, {{n_codex_concedes}} Codex).
- Convergence rate: {{convergence_pct}}% (1 - DISPUTED / total raised).
- Wall time: {{duration}}.

## Files

- Diff snapshot: `{{run_dir}}/diff.patch`
- Round 1: `round-1/claude.md`, `round-1/codex.md`
- Round 2: `round-2/claude-rebuttals.md`, `round-2/codex-rebuttals.md`
- Round 3: `round-3/claude-final.md`, `round-3/codex-final.md`
```

---

## Synthesis logic (orchestrator phải apply)

Cho mỗi finding (Claude C-NNN hoặc Codex X-MMM):

```
let raised_by = bên emit ở round 1
let opponent_stance = stance từ round 2 của bên đối phương về finding này
let final_position = position từ round 3 của bên raise (nếu có)

if opponent_stance == "agree":
    bucket = AGREED
elif opponent_stance == "refute":
    if final_position == "hold":
        bucket = DISPUTED
    elif final_position == "concede":
        bucket = RETRACTED
    elif final_position == "partial":
        bucket = DISPUTED  (kèm Updated claim)
    else:
        bucket = DISPUTED  (no response = weak hold, cảnh báo)
elif opponent_stance == "escalate":
    if final_position in ("hold", "partial"):
        bucket = DISPUTED  (cần human để break tie)
    elif final_position == "concede":
        bucket = RETRACTED
```

**Đặc biệt — cùng finding hai bên cùng raise**: nếu Claude C-NNN và Codex
X-MMM cùng `file:line` ± 5 dòng và cùng category → tự động AGREED, gộp claim.

**Severity merge**: lấy max của hai bên.
