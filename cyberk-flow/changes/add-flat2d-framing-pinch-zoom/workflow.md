# Trạng thái Workflow: add-flat2d-framing-pinch-zoom

> **Nguồn sự thật:** Stages/gates → file này · Hoàn thành task → `tasks.md`
>
> **Trạng thái checkbox:** `[ ]` chờ · `[/]` đang làm · `[x]` xong · `[-]` bỏ qua/N/A

## Lập kế hoạch

- [x] 1. Xem xét bối cảnh & Phân loại độ phức tạp
  - [x] Đọc `project.md` để hiểu bối cảnh dự án
  - [x] Chạy `cf changes` + `cf specs`
  - [x] Chọn `change-id`, chạy `cf new add-flat2d-framing-pinch-zoom`
  - [x] Phân loại độ phức tạp: **small**
  - [x] Kiểm tra cờ nâng cấp: `cross-boundary`
  - [x] Ghi nhận độ phức tạp + cờ nâng cấp trong file này (mục Ghi chú)
- [x] 2. Khám phá
  - [x] Chọn workstreams liên quan (Memory / Architecture / Patterns / Constraints)
  - [x] Thực thi workstreams
  - [x] Điền `discovery.md` — phát hiện, phân tích khoảng trống, lựa chọn, rủi ro
  - [x] **🚪 Cổng: người dùng đã duyệt hướng đi** — tách pinch zoom thành change follow-up riêng
- [x] 3. Đề xuất
  - [x] Điền `proposal.md` — Lý do, Khẩu vị, Phạm vi, Khả năng, Tác động, Rủi ro
  - [x] **BẮT BUỘC** Quyết định UI Impact & E2E đã ghi trong `proposal.md`
- [x] 4. Đặc tả (Định dạng Delta)
  - [x] Tạo specs theo capability tại `specs/<capability>/spec.md`
  - [x] Mỗi yêu cầu có ≥1 kịch bản kiểm thử
  - [ ] **🚪 Cổng: người dùng duyệt specs** — chờ duyệt `pinch-zoom`, `export-pipeline`
- [x] 5. Thiết kế & Đánh giá rủi ro
  - [x] Tạo `design.md` — phân tích khoảng trống, quyết định kiến trúc, Ma trận rủi ro
  - [x] Rủi ro MEDIUM → kèm sơ đồ giao diện (Mermaid)
  - [x] **🚪 Cổng: thiết kế đã review** — tự review, MEDIUM risk chấp nhận được
- [x] 6. Các task
  - [x] Điền `tasks.md` — danh sách theo thứ tự thực thi, có dependency
  - [x] Mỗi task có: Deps, Refs, Tiêu chí Done, Test, Files, Approach
- [x] 7. Kiểm chứng
  - [-] Oracle review — bỏ qua ở bước plan, tự review đủ cho change nhỏ
  - [-] Phân loại finding: không có finding từ oracle
  - [x] `cf validate` pass
  - [x] Checklist: kịch bản ✓, khẩu vị ✓, câu hỏi đã giải ✓
  - [ ] **🚪 Cổng: người dùng duyệt plan** — trình checklist + đề xuất `/cf-build`

## Triển khai

<!-- QUY TẮC: Sau khi hoàn thành mỗi task, đánh dấu [x] trong tasks.md VÀ ghi log trong Nhật ký sửa đổi bên dưới. -->
- [ ] 1. Đọc toàn bộ artifact (workflow.md, proposal.md, design.md, tasks.md)
- [ ] 2. Thực thi task tuần tự theo thứ tự dependency
- [ ] 3. Cập nhật: đánh `- [x]` trong tasks.md + ghi log trong Nhật ký sửa đổi SAU MỖI task
- [ ] 4. Cổng kiểm tra — chạy lệnh từ `project.md` § Commands, **PHẢI thực thi và xác nhận pass** _(đánh `[-]` nếu N/A)_:
  - [ ] Type check
  - [ ] Lint
  - [ ] Test
  - [ ] E2E
- [ ] 5. Review (linh hoạt — chạy song song; bỏ cả hai nếu trivial):
  - [ ] Code Review
  - [ ] Oracle Deep Analysis: `cf-oracle` subagent
- [ ] 6. Phân loại finding: chấp nhận/bác bỏ từng finding kèm lý do
- [ ] 7. Vòng sửa Review _(tối đa 3 vòng — sửa, kiểm tra lại, review lại)_
- [ ] 8. Kiểm chứng
  - [ ] **🚪 Cổng: người dùng duyệt triển khai**
  - [ ] Trích xuất knowledge

## Lưu trữ

- [ ] Kiểm tra sự ổn định sau merge
- [ ] Hồi cứu
- [ ] Áp dụng delta: `cf_apply` <!-- auto-ticked by script -->
- [ ] Lưu trữ change: `cf_archive` <!-- auto-ticked by script -->

## Ghi chú

- **Độ phức tạp**: Small — follow-up enhancement trên flow Flat2D hiện có, chạm vào gesture + store + export parity
- **Cờ nâng cấp**: `cross-boundary` (RN gesture + Skia preview + export snapshot)
- **Quyết định chính**: Tách pinch zoom sang change riêng vì `add-flat2d-framing` đã gần hoàn tất và không nên mở rộng scope của change gốc
- **Quyết định kỹ thuật**: Dùng `compositionScale` riêng, không ghi đè `printSizeCm`

## Nhật ký sửa đổi

| Ngày | Giai đoạn | Thay đổi | Lý do |
| ---- | --------- | -------- | ----- |
| 2026-04-02 | Giai đoạn 1 | Tạo change `add-flat2d-framing-pinch-zoom` và phân loại Small complexity | Tách follow-up enhancement khỏi `add-flat2d-framing` đã gần hoàn tất |
| 2026-04-02 | Giai đoạn 2 | Khám phá xong preview/store/export hiện có | Xác định các điểm cần mở rộng cho pinch zoom mà không phá semantics `printSizeCm` |
| 2026-04-02 | Giai đoạn 3 | Proposal hoàn thành | Chốt phạm vi pinch zoom là enhancement UI/export parity riêng |
| 2026-04-02 | Giai đoạn 4 | Tạo 2 specs: `pinch-zoom`, `export-pipeline` | Đặc tả behavior pinch, reset, cache parity và export contract |
| 2026-04-02 | Giai đoạn 5 | Design hoàn thành | Chốt `compositionScale`, clamp bounds, và khóa `ScrollView` khi pinch |
| 2026-04-02 | Giai đoạn 6 | Tasks hoàn thành | Tách thành 3 phase: contract, preview interaction, export parity |
| 2026-04-02 | Giai đoạn 7 | Validation pass | `cf validate add-flat2d-framing-pinch-zoom` pass |
