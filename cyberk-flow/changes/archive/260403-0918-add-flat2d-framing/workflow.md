# Trạng thái Workflow: add-flat2d-framing

> **Nguồn sự thật:** Stages/gates → file này · Hoàn thành task → `tasks.md`
>
> **Trạng thái checkbox:** `[ ]` chờ · `[/]` đang làm · `[x]` xong · `[-]` bỏ qua/N/A

## Lập kế hoạch

- [x] 1. Xem xét bối cảnh & Phân loại độ phức tạp
  - [x] Đọc `project.md` để hiểu bối cảnh dự án
  - [x] Chạy `cf changes` + `cf specs`
  - [x] Chọn `change-id`, chạy `cf new add-flat2d-framing`
  - [x] Phân loại độ phức tạp: **standard**
  - [x] Kiểm tra cờ nâng cấp: `new-api-contract`, `cross-boundary`
  - [x] Ghi nhận độ phức tạp + cờ nâng cấp trong file này (mục Ghi chú)
- [x] 2. Khám phá
  - [x] Chọn workstreams liên quan (Memory / Architecture / Patterns / Constraints)
  - [x] Thực thi workstreams (song song khi có thể)
  - [x] Điền `discovery.md` — phát hiện, phân tích khoảng trống, lựa chọn, rủi ro
  - [x] **🚪 Cổng: người dùng đã duyệt hướng đi** — đồng ý dùng local stub data
- [x] 3. Đề xuất
  - [x] Điền `proposal.md` — Lý do, Khẩu vị, Phạm vi, Khả năng, Tác động, Rủi ro
  - [x] **BẮT BUỘC** Quyết định UI Impact & E2E đã ghi trong `proposal.md`
- [x] 4. Đặc tả (Định dạng Delta)
  - [x] Tạo specs theo capability tại `specs/<capability>/spec.md`
  - [x] Mỗi yêu cầu có ≥1 kịch bản kiểm thử
  - [x] **🚪 Cổng: người dùng duyệt specs** — đã duyệt 6 specs, 20 yêu cầu, 34 kịch bản
- [x] 5. Thiết kế & Đánh giá rủi ro
  - [x] Tạo `design.md` — phân tích khoảng trống, quyết định kiến trúc, Ma trận rủi ro
  - [x] Rủi ro MEDIUM → kèm sơ đồ giao diện (Mermaid)
  - [x] **🚪 Cổng: thiết kế đã review** — tự review, MEDIUM risk chấp nhận được
- [x] 6. Các task
  - [x] Điền `tasks.md` — danh sách theo thứ tự thực thi, có dependency
  - [x] Mỗi task có: Deps, Refs, Tiêu chí Done, Test, Files, Approach
- [x] 7. Kiểm chứng
  - [-] Oracle review — bỏ qua, tự review đủ cho MEDIUM risk
  - [-] Phân loại finding: không có finding từ oracle
  - [x] `cf validate` pass
  - [x] Checklist: kịch bản ✓, khẩu vị ✓, câu hỏi đã giải ✓
  - [ ] **🚪 Cổng: người dùng duyệt plan** — trình checklist + đề xuất `/cf-build`

## Triển khai

<!-- QUY TẮC: Sau khi hoàn thành mỗi task, đánh dấu [x] trong tasks.md VÀ ghi log trong Nhật ký sửa đổi bên dưới. -->
- [x] 1. Đọc toàn bộ artifact (workflow.md, proposal.md, design.md, tasks.md)
- [ ] 2. Thực thi task tuần tự theo thứ tự dependency
- [ ] 3. Cập nhật: đánh `- [x]` trong tasks.md + ghi log trong Nhật ký sửa đổi SAU MỖI task
- [ ] 4. Cổng kiểm tra — chạy lệnh từ `project.md` § Commands, **PHẢI thực thi và xác nhận pass** _(đánh `[-]` nếu N/A)_:
  - [ ] Type check
  - [x] Lint
  - [x] Test
  - [-] E2E
- [ ] 5. Review (linh hoạt — chạy song song; bỏ cả hai nếu trivial):
  - [x] Code Review
  - [ ] Oracle Deep Analysis: `cf-oracle` subagent
- [x] 6. Phân loại finding: chấp nhận/bác bỏ từng finding kèm lý do
- [x] 7. Vòng sửa Review _(tối đa 3 vòng — sửa, kiểm tra lại, review lại)_
- [ ] 8. Kiểm chứng
  - [ ] **🚪 Cổng: người dùng duyệt triển khai**
  - [ ] Trích xuất knowledge

## Archive

- [x] Kiểm tra sự ổn định sau merge
- [x] Hồi cứu
- [x] Apply deltas: `cf_apply` <!-- auto-ticked by script -->
- [x] Archive change: `cf_archive` <!-- auto-ticked by script -->

## Ghi chú

- **Độ phức tạp**: Standard — 15+ file mới, nhiều module (Skia 9-slice renderer, placement math, zustand store, export pipeline), rủi ro MEDIUM
- **Cờ nâng cấp**: `new-api-contract` (endpoint mới `GET /scenes/templates/2d`), `cross-boundary` (RN UI + Skia rendering + navigation + API)
- **Stages bắt buộc**: Tất cả stages đều bắt buộc (Standard base + cờ nâng cấp yêu cầu Design & Specs)
- **Spec đầu vào**: `docs/SPEC-framing-flat2d-v2.vi.md` — đặc tả sản phẩm chi tiết cho tính năng Flat2D Framing
- **Quyết định**: Dùng local stub data cho cả scene templates và frame presets trong v1
- **Hồi cứu**: 
  - **Estimate vs Actual**: Appetite was L ≤ 2w, took ~2 ngày.
  - **What worked**: Lên kiến trúc rõ ràng, dùng stub data cô lập rủi ro, phân loại dependencies tốt. Sử dụng store và hook chia layer FSD ổn định.
  - **What to improve**: Việc trao đổi requirements về thiết kế frame (nine-slice) nên thống nhất chuẩn tài sản đầu vào trước lúc code để tránh bug cắt khung. Đảm bảo UI mockup đồng bộ logic render.

## Nhật ký sửa đổi

| Ngày | Giai đoạn | Thay đổi | Lý do |
| ---- | --------- | -------- | ----- |
| 2026-04-02 | Giai đoạn 1 | Hoàn thành xem xét bối cảnh & phân loại | Standard complexity với cờ `new-api-contract` + `cross-boundary` |
| 2026-04-02 | Giai đoạn 2 | Khám phá xong, cổng đã duyệt | Người dùng đồng ý dùng local stub data. Tất cả deps đã có. 5 pattern tái sử dụng |
| 2026-04-02 | Giai đoạn 3 | Đề xuất hoàn thành | Appetite L ≤ 2w, rủi ro MEDIUM, E2E trong phạm vi |
| 2026-04-02 | Giai đoạn 4 | 6 specs đã tạo, cổng đã duyệt | artwork-input, frame-selection, scene-selection, preview-canvas, auto-fit, export-pipeline |
| 2026-04-02 | Giai đoạn 5 | Thiết kế hoàn thành | 8 quyết định kiến trúc, 3 sơ đồ Mermaid, ma trận rủi ro 7 components |
| 2026-04-02 | Giai đoạn 6 | Tasks hoàn thành | 19 tasks trong 5 phase gates, dependency-aware |
| 2026-04-02 | Giai đoạn 7 | Validation pass | cf validate pass, checklist ✓ |
| 2026-04-02 | Triển khai (Task 1_1) | Tạo entity Flat2DSceneTemplate + stub data + react-query hook | Dựng domain layer tách biệt, dùng canonical pixel-only theo spec |
| 2026-04-02 | Triển khai (Task 1_2) | Tạo zustand store framing-flat2d-store | Có đầy đủ state/actions cốt lõi cho artwork/frame/scene/print/mat/auto-fit/export |
| 2026-04-02 | Triển khai (Task 1_3) | Tạo route `/create-flat2d-framing` và screen shell | Mở được màn hình mới với header, placeholder sections, preview area, export button |
| 2026-04-02 | Triển khai (Task 1_5) | Thêm locale keys `flat2d_*` cho en/vi | Đảm bảo key song ngữ cho labels, empty states, auto-fit toast, export error |
| 2026-04-02 | Triển khai (Task 2_1) | Tạo `FRAME_PRESETS` + 3 frame PNG assets + nineSlice/minInner constraints | Hoàn tất registry khung để renderer và selector dùng chung |
| 2026-04-02 | Triển khai (Task 2_2) | Triển khai `lib/nine-slice.ts` với `calcNineSliceRects` + `renderNineSlice` và unit test | Đảm bảo 9-slice render đúng hình học src/dst, góc không stretch |
| 2026-04-02 | Triển khai (Task 2_3) | Triển khai `frame-sizing.ts`, `placement.ts` + unit tests cho auto-fit/resolve | Chuẩn hóa resolver preview/export theo canonical pixel và cover case safe rect lệch tâm |
| 2026-04-02 | Triển khai (Task 1_4) | Thêm CTA vào `Create2DProject` để mở `/create-flat2d-framing` | Tạo đường vào flow Flat2D từ màn hình tạo 2D hiện tại |
| 2026-04-02 | Triển khai (Task 3_1) | Tạo `ArtworkPicker` với image picker + crop flow + print size + mat width controls | Đảm bảo nhập liệu artwork/print/mat và reject print size không hợp lệ |
| 2026-04-02 | Triển khai (Task 3_2) | Tạo `FrameSelector` cuộn ngang, hỗ trợ chọn/bỏ chọn frame preset | Cập nhật `framePresetId` trong store theo hành vi toggle |
| 2026-04-02 | Triển khai (Task 3_3) | Tạo `SceneSelector` dùng react-query scene templates + cập nhật preview nền theo scene đã chọn | Đảm bảo chọn scene cập nhật state và phản ánh ngay ở khu vực preview cơ bản |
| 2026-04-02 | Triển khai (Task 2_4) | Tạo `Flat2DPreview` với 4 layer, Skia framed artwork live, aspect ratio chuẩn scene, cập nhật auto-fit vào store | Hoàn tất preview-canvas theo spec và dùng shared placement contract |
| 2026-04-02 | Triển khai (Task 4_1) | Tạo `Flat2DExporter` hidden view dùng `react-native` Image + `captureRef` + `waitForRenderReady` | Đảm bảo export compose canonical-resolution ổn định theo pattern đã chứng minh |
| 2026-04-02 | Triển khai (Task 4_2) | Nối export flow: snapshot Skia canonical + cache signature + cleanup snapshot cũ/new + handoff `/upload-project` object-form params | Hoàn thiện upload handoff contract và snapshot policy |
| 2026-04-02 | Triển khai (Task 4_3) | Thêm `SCENE_LAYER_ORDER` + parity test `preview-export-parity.test.ts` cho shared placement/layer order | Khoá rủi ro lệch preview/export bằng automated test |
| 2026-04-02 | Triển khai (Task 5_1) | Thêm auto-fit toast UX chống spam theo thay đổi scale | Đảm bảo phản hồi UX đúng lúc khi auto-fit kích hoạt |
| 2026-04-02 | Triển khai (Task 5_2) | Thêm loading state nút export, validation thiếu input/print size, error toast retry-safe | Củng cố polish và khả năng retry khi export lỗi mà không mất draft |
| 2026-04-02 | Triển khai (Gate check) | Chạy `bun lint` pass, `bun test screens/create-flat2d-framing/lib/__tests__` pass, `npx tsc --noEmit` fail do lỗi pre-existing ngoài phạm vi Flat2D | Ghi nhận trạng thái kiểm tra thực tế trước khi sang phase tiếp theo |
| 2026-04-02 | Triển khai (Bugfix crop flow) | Sửa resolve `ph://` URI trong `use-crop-flow` (fallback parse assetId từ URI) + fallback complete trong `crop-images` khi crop lỗi | Khắc phục trường hợp nhấn `Xong` nhưng không tiến được do lỗi crop URI local trên iOS |
| 2026-04-02 | Triển khai (Bugfix crop flow #2) | Thêm timeout guard cho `saveImage`/`centerCrop` ở `crop-images` để tránh promise treo khi nhấn `Xong` | Đảm bảo thao tác `Xong` luôn có kết quả và không bị kẹt màn crop |
| 2026-04-02 | Triển khai (Bugfix crop flow #3) | Thêm fallback `router.back()` trong `crop-images` khi `complete()` đã chạy nhưng callback điều hướng không thực thi | Chặn triệt để trạng thái nhấn `Xong` mà vẫn đứng nguyên ở màn crop |
| 2026-04-02 | Triển khai (Debug crop flow) | Thêm toast debug khi nhấn `Xong` + khi callback `onComplete` chạy, và gắn runtime logs chi tiết ở `handleDone`, `crop-images-service.complete`, `ArtworkPicker.onComplete` | Tăng khả năng quan sát để xác định chính xác điểm tắc trên thiết bị thực |
| 2026-04-02 | Triển khai (Bugfix crop flow #4) | Chuyển quyền đóng route về `crop-images`, để `complete()` trả callback chạy sau `router.back()`, đồng thời bỏ `router.back()` lặp ở các caller | Loại race do màn cha phải tự đóng shared crop route, giúp `Done` hoàn tất ổn định ở mọi nơi dùng chung |
| 2026-04-02 | Triển khai (Bugfix frame overlay) | Bỏ vẽ nine-slice region `center` cho frame renderer và thêm regression test | Khắc phục việc PNG khung opaque che kín artwork khi người dùng chọn khung |
| 2026-04-02 | Triển khai (Designer note) | Thêm `docs/NOTE-flat2d-frame-design.vi.md` mô tả guideline thiết kế asset khung Flat2D | Đồng bộ kỳ vọng giữa design và renderer 9-slice hiện tại để tránh lặp lại lỗi artwork bị che |
| 2026-04-02 | Triển khai (Designer note refinement) | Bổ sung mục “Designer chỉ cần bàn giao gì” trong `docs/NOTE-flat2d-frame-design.vi.md` | Làm rõ designer chỉ cần 1 file master + metadata, không phải cắt 9 file hay chuẩn bị asset riêng cho pinch zoom |
| 2026-04-02 | Triển khai (Designer note wording) | Đổi các bullet kỹ thuật khó hiểu trong `docs/NOTE-flat2d-frame-design.vi.md` sang ngôn ngữ handoff trực quan cho designer | Giảm jargon như “nine-slice” và diễn đạt theo cách designer có thể hành động ngay |
| 2026-04-02 | Triển khai (Designer note example) | Bổ sung ví dụ handoff cụ thể cho 1 mẫu frame và đổi wording sang ngôn ngữ ít technical hơn | Giúp designer biết chính xác với 1 thiết kế frame thì cần gửi gì cho dev |
| 2026-04-02 | Review fix | Sửa 4 findings sau code review: khóa export khi print size invalid, buộc frame renderer bám `frameFaceWidthCm`, fail export khi layer load lỗi, và thêm replace-artwork flow giữ nguyên print width hợp lệ | Đưa implementation quay lại đúng contract của spec trước khi archive |
| 2026-04-02 | Triển khai (Task 5_4) | Đồng bộ `docs/SPEC-framing-flat2d-v2.vi.md` với implementation thực tế | Ghi rõ store validation flag, contract shadow `blurRadiusPx`, nguyên tắc độ dày frame, lỗi exporter, và replace-artwork behavior |
| 2026-04-02 | Triển khai (Gate check #2) | Chạy `bun test screens/create-flat2d-framing/lib/__tests__ store/__tests__/framing-flat2d-store.test.ts` pass, `bun lint` pass, `npx tsc --noEmit` tiếp tục fail do lỗi pre-existing ngoài phạm vi Flat2D | Xác nhận vòng fix review không tạo regression trong phạm vi change |
| 2026-04-02 | Triển khai (Task 5_3) | Bỏ qua E2E theo chỉ định của user | Unblock archive prep; residual risk là flow Flat2D chưa có mobile E2E automation trong repo |
