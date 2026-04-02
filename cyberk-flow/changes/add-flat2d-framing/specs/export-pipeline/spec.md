# Đặc tả: export-pipeline

## ADDED Requirements

### Requirement: skia-artwork-snapshot

Pipeline export SHALL tạo snapshot PNG của framed artwork bằng Skia.

- Snapshot SHALL bao gồm: artwork đã crop + mat + frame (tất cả render qua Skia)
- Snapshot SHALL được render ở độ phân giải pixel canonical (không phải preview scale)
- Snapshot SHALL được cache theo chữ ký: `${artworkUri}:${framePresetId}:${printWidthCm}:${printHeightCm}:${matWidthCm}`
- Nếu chữ ký khớp snapshot trước đó, URI đã cache SHALL được tái sử dụng
- Output snapshot SHALL được lưu dạng `flatArtworkSnapshotUri` trong store
- Khi snapshot mới thay thế snapshot cũ, file tạm cũ SHALL được xoá ngay

#### Scenario: Snapshot tạo khi export

- CHO rằng artwork, frame, và print size đã đặt
- KHI người dùng khởi động export
- THÌ Skia render framed artwork ở độ phân giải đầy đủ
- VÀ lưu PNG vào temp storage cục bộ
- VÀ `flatArtworkSnapshotUri` cập nhật trong store

#### Scenario: Cache hit snapshot

- CHO rằng snapshot đã tạo trước đó với cùng chữ ký
- KHI người dùng export lại mà không thay đổi
- THÌ URI snapshot trước được tái sử dụng mà không cần render lại

#### Scenario: Snapshot cũ bị thay thế

- CHO rằng snapshot A đã tồn tại cho chữ ký cũ
- KHI người dùng thay đổi input và tạo snapshot B cho chữ ký mới
- THÌ file tạm của snapshot A bị xoá
- VÀ store chỉ còn giữ `flatArtworkSnapshotUri` trỏ đến snapshot B

### Requirement: hidden-scene-exporter

Pipeline export SHALL tổng hợp ảnh scene cuối cùng bằng hidden export view.

- Exporter SHALL dùng `react-native-view-shot` (`captureRef`) — theo pattern `composer-screen.tsx`
- Exporter SHALL dùng `react-native` `Image` (KHÔNG phải `expo-image`) cho tất cả lớp ảnh
- Hidden view SHALL được đặt ngoài màn hình (`left: -9999`)
- View SHALL render ở độ phân giải `sceneSizePx` (canonical)
- Thứ tự lớp trong export SHALL khớp preview: background → shadow → framed artwork snapshot → foreground
- Exporter SHALL đợi tất cả ảnh tải xong trước khi capture (pattern `waitForRenderReady`)

#### Scenario: Export tạo output đúng

- CHO rằng tất cả input (artwork, frame, scene) đã đặt
- KHI export được kích hoạt
- THÌ hidden view render ở độ phân giải canonical của scene
- VÀ `captureRef` capture ảnh tổng hợp dạng PNG
- VÀ output chứa `{ uri, width, height }`

#### Scenario: Export xử lý lỗi tải ảnh

- CHO rằng background scene không tải được
- KHI export được kích hoạt và tải ảnh timeout
- THÌ export ném lỗi
- VÀ `isExporting` reset về false
- VÀ bản nháp của người dùng KHÔNG bị xoá

### Requirement: upload-handoff

Sau export thành công, app SHALL điều hướng đến `UploadProjectScreen` với ảnh đã export.

- Navigation SHALL dùng: `router.navigate({ pathname: "/upload-project", params: { images: JSON.stringify([{ id, uri, width, height }]) } })`
- Bàn giao SHALL theo đúng contract mà `UploadProjectScreen` yêu cầu
- `isExporting` SHALL đặt thành false sau navigation
- `id` của ảnh export SHALL là `"flat2d-export"`

#### Scenario: Export thành công điều hướng đến upload

- CHO rằng export hoàn thành với `{ uri, width, height }`
- KHI handoff thực thi
- THÌ app điều hướng đến `/upload-project`
- VÀ params chứa `images` với data ảnh đã export
- VÀ `UploadProjectScreen` parse và hiển thị ảnh đúng

### Requirement: export-error-handling

Pipeline export SHALL xử lý lỗi một cách graceful.

- Nếu export thất bại, `isExporting` SHALL reset về false
- Bản nháp của người dùng SHALL NOT xoá khi thất bại
- Toast lỗi SHALL hiển thị với message ID `flat2d_export_failed`
- Người dùng SHALL có thể thử lại export

#### Scenario: Export thất bại cho phép thử lại

- CHO rằng export thất bại do lỗi `captureRef`
- THÌ `isExporting` reset về false
- VÀ toast lỗi xuất hiện
- VÀ nút export trở lại có thể chạm
- VÀ người dùng có thể thử lại mà không mất lựa chọn

### Requirement: snapshot-policy

App SHALL NOT tạo snapshot trên mỗi thay đổi input.

- Preview SHALL dùng Skia rendering trực tiếp (không snapshot trong khi chỉnh sửa)
- Snapshot SHALL only be tạo khi người dùng kích hoạt export rõ ràng
- Tối đa một snapshot SHALL được giữ tại một thời điểm (không lưu history)
- Snapshot tạm dùng cho composition SHALL được dọn sau khi export thành công và scene cuối đã được bàn giao cho `UploadProjectScreen`
- Các file snapshot/outdated temp khác SHALL NOT tích luỹ qua nhiều lần export

#### Scenario: Không snapshot trong khi chỉnh sửa

- CHO rằng người dùng đang điều chỉnh slider mat width
- THÌ không tạo Skia snapshot nào
- VÀ preview cập nhật trực tiếp qua Skia rendering
- VÀ `flatArtworkSnapshotUri` vẫn là null cho đến khi export

#### Scenario: Dọn file tạm sau handoff thành công

- CHO rằng scene cuối đã export thành công và app đã điều hướng sang `UploadProjectScreen`
- KHI handoff hoàn tất
- THÌ snapshot tạm dùng cho composition được xoá
- VÀ không còn file snapshot cũ bị bỏ lại ngoài file ảnh cuối cùng cần cho upload
