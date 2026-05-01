# debate-review — Design notes

## Vì sao cần skill này

Single-model code review có 2 failure mode:

1. **False negatives** — model bỏ sót issue do bias huấn luyện hoặc đơn
   giản là không "nhìn thấy". Một model khác có training distribution khác
   sẽ catch những thứ này.
2. **False positives quá tự tin** — model raise findings sai nhưng
   confident. Người review tin theo, mất thời gian fix thứ không cần fix,
   hoặc tệ hơn là refactor sai hướng.

Cross-model debate xử lý cả hai:
- **False negatives** giảm vì 2 model với training khác nhau ít cùng miss.
- **False positives** giảm vì bên kia có cơ hội refute. Findings sống sót
  qua 3 rounds = signal mạnh.

DISPUTED bucket là **bonus quan trọng nhất**: 2 model thông minh không
đồng ý thường chỉ ra vùng tinh tế (architectural ambiguity, spec gap,
edge case không rõ behavior). Đây là nơi human attention cần focus.

## Vì sao 3 rounds, không phải 5 hay 1?

- **1 round (no debate)**: chỉ là parallel review. Không lọc false positive.
- **2 rounds (raise → rebut)**: bên raise không có cơ hội phản hồi rebuttal.
  Mọi rebuttal đều "win" tự động → bất công với findings đúng nhưng bị refute
  bằng counter-evidence sai.
- **3 rounds (raise → rebut → final position)**: cân bằng. Bên raise có
  đúng 1 cơ hội bảo vệ hoặc concede. Đủ để hội tụ ở đa số trường hợp.
- **5+ rounds**: diminishing returns. Nếu sau 3 rounds vẫn bất đồng → đó là
  DISPUTED thật, thêm round không giúp; cần human.

## Vì sao Claude + Codex, không phải 2 Claude khác nhau?

- 2 Claude khác temperature/prompt → vẫn cùng training distribution. Cùng
  blind spot.
- Claude (Anthropic) + Codex (OpenAI/GPT-based) → khác training data, khác
  RLHF. Blind spot ít overlap hơn.
- User đã có sẵn cả 2 (Claude Code + Codex plugin) → tận dụng setup hiện có.

Nếu sau này có model thứ 3 (vd Gemini) đáng kể, có thể mở rộng thành
3-way debate, nhưng synthesis logic phức tạp hơn (majority vote, weighted
consensus). Để đó.

## Vì sao cần schema bắt buộc?

Không có schema → orchestrator không thể:
- Match findings cùng issue giữa 2 bên
- Detect "weak hold" (round 3 không phản biện cụ thể)
- Bucket vào AGREED/CONFIRMED/DISPUTED/RETRACTED

Schema cũng ép reviewer chính xác (file:line + evidence bắt buộc) — chống
findings vague.

## Vì sao orchestrator KHÔNG được mediate?

Cám dỗ: "Tôi (Claude main agent) đọc cả 2 phía, tôi quyết bên nào đúng."

Nhưng:
- Main agent là Claude → bias về phía debate-reviewer (cũng Claude).
- Mediation rút ngắn DISPUTED → mất tín hiệu giá trị nhất.
- Human đọc 2 phía rồi quyết tốt hơn AI tự quyết — đặc biệt với spec/business
  context AI không có.

Orchestrator được phép `Recommend:` 1 dòng trong DISPUTED nếu thật sự nghiêng
về 1 bên, nhưng không được tự bucket thành CONFIRMED.

## Vì sao snapshot diff sớm?

Working tree có thể thay đổi giữa chừng (user edit file trong lúc skill chạy).
Nếu Claude review diff cũ, Codex review diff mới → debate vô nghĩa.

Snapshot `diff.patch` ở step 2 → cả 2 reviewer có cùng input, debate fair.

## Tradeoffs đã chấp nhận

| Tradeoff | Lý do chọn |
|---|---|
| 2x cost (chạy 2 model) | Đáng cho critical changes. User chọn khi nào dùng. |
| ~3-5x latency vs single review | Có thể parallel trong round, nhưng round phải sequential. Chấp nhận. |
| Codex CLI dependency | User đã có. Nếu thiếu → STOP rõ ràng, không degrade silently. |
| Schema rigidity | Cần để machine-parseable. Reviewer phải tuân, không "free-form". |
| Không tự fix | Skill chỉ produce report. Fix là việc khác (human, agent build, …). Tách concern. |

## So với single-model review thông thường

| Khía cạnh | Single-model review | debate-review |
|---|---|---|
| Reviewers | 1 model | 2 model khác hãng (Claude + Codex) |
| False negatives | Cao — bị giới hạn bởi blind spot của model | Thấp hơn — 2 model khác training distribution ít cùng miss |
| False positives | Khó phát hiện — không có ai phản biện | Lọc qua rebuttal round → finding sai bị refute với evidence |
| Confidence signal | Nhị phân (raise / không raise) | 4 buckets: AGREED / CONFIRMED / DISPUTED / RETRACTED |
| Cost | 1× | ~2× model + 3× round overhead |
| Khi dùng | Daily review | Critical changes (security, contract, migration, refactor lớn) |

debate-review không thay thế single-model review — bổ sung. Single-model
review dùng cho daily; debate-review dùng khi cần confidence calibration
cao hoặc khi single-model output gây nghi ngờ.

## Mở rộng tiềm năng (out of scope v0.1)

- **N-way debate** (3+ model)
- **Specialized reviewer per round** (vd Codex review security, Claude review architecture)
- **Auto-triage to fixer agent** (skill này hiện chỉ produce report)
- **Cost cap** (`--max-tokens` để giới hạn debate trên large diff)
- **Cross-language support** (test với Python, Rust, Go diff)
