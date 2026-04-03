# scene-selection Specification

## Purpose
TBD
## Requirements

### Requirement: flat2d-scene-template-entity

App SHALL định nghĩa entity domain mới `Flat2DSceneTemplate` tách biệt khỏi `SceneTemplate` 3D hiện có.

- Entity SHALL đặt tại `lib/entities/flat2d-scene-template/`
- Model SHALL bao gồm: `id`, `name`, `thumbnailUrl`, `assets` (`backgroundUrl`, `foregroundUrl`), `sceneSizePx`, `placement` (`anchorPx`, `pixelsPerCm`, `safeRectPx`, `shadow`)
- Tầng API SHALL cung cấp hàm query để fetch tất cả template
- Tầng hooks SHALL bọc hàm query bằng `react-query`

#### Scenario: App tải scene templates

- CHO rằng màn hình editor mở
- KHI templates được yêu cầu
- THÌ hook trả về danh sách `Flat2DSceneTemplate` từ stub data cục bộ
- VÀ mỗi template chứa đầy đủ các trường bắt buộc

### Requirement: scene-template-coordinate-system

App SHALL dùng hệ toạ độ **canonical pixel-only** cho toàn bộ placement, auto-fit và export.

- Template SHALL lưu `anchorPx`, `safeRectPx`, `sceneSizePx`, `pixelsPerCm` dạng pixel canonical — KHÔNG normalize sang ratio
- Placement math SHALL hoạt động hoàn toàn trong không gian pixel canonical
- Preview SHALL resolve toạ độ canonical sang view bằng một hệ số `previewScale = min(viewWidth / sceneWidthPx, viewHeight / sceneHeightPx)`
- Export SHALL dùng toạ độ canonical pixel trực tiếp (vì export render ở `sceneSizePx`)
- Module `placement.ts` SHALL là resolver duy nhất — nhận input canonical pixel, trả output cho cả preview (nhân `previewScale`) và export (trực tiếp)
- App SHALL NOT tạo bước trung gian normalize sang ratio rồi resolve ngược — mỗi bước chuyển đổi thêm đều tạo rủi ro floating-point error

#### Scenario: Canonical pixel dùng cho preview

- CHO rằng template có `sceneSizePx: { width: 2048, height: 1536 }`, `anchorPx: { x: 1024, y: 768 }`
- VÀ preview view có kích thước 390px × 292.5px → `previewScale = 390 / 2048 ≈ 0.1904`
- KHI preview render
- THÌ anchor hiển thị tại `{ x: 1024 * 0.1904 ≈ 195, y: 768 * 0.1904 ≈ 146.25 }`
- VÀ tất cả kích thước outer rect cũng nhân cùng `previewScale`

#### Scenario: Canonical pixel dùng cho export

- CHO rằng cùng template trên
- KHI export render ở `sceneSizePx` (2048 × 1536)
- THÌ anchor đặt tại `{ x: 1024, y: 768 }` — dùng trực tiếp, không chuyển đổi
- VÀ kết quả export chính xác pixel-perfect

### Requirement: scene-selector-ui

Editor SHALL cung cấp UI chọn scene để chọn room scenes.

- Bộ chọn SHALL hiển thị thumbnail scene trong danh sách cuộn
- Chọn scene SHALL cập nhật `sceneTemplateId` trong store
- Preview SHALL cập nhật ngay lập tức hiển thị background scene đã chọn
- V1 SHALL bao gồm tối thiểu 2 stub scene cục bộ

#### Scenario: Người dùng chọn scene

- CHO rằng editor đang mở
- KHI người dùng chạm thumbnail scene
- THÌ preview background đổi sang scene đã chọn
- VÀ `sceneTemplateId` trong store cập nhật

### Requirement: local-stub-data

V1 SHALL dùng stub data đóng gói cục bộ cho scene templates thay vì API backend.

- Stub templates SHALL được định nghĩa trong `screens/create-flat2d-framing/constants/stub-scenes.ts`
- Ảnh background/foreground stub SHALL được đóng gói dạng asset cục bộ hoặc dùng URL placeholder
- Tầng entity SHALL được cấu trúc để chuyển sang API calls thực mà không cần thay đổi UI
- Hook query SHALL dùng cùng interface react-query bất kể nguồn dữ liệu

#### Scenario: Stub data phục vụ templates

- CHO rằng không có API backend
- KHI app yêu cầu scene templates
- THÌ hook trả về stub templates đã định nghĩa cục bộ
- VÀ UI hoạt động giống hệt như khi dùng API data thực

