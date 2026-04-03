# Đặc tả: frame-selection

## ADDED Requirements

### Requirement: frame-preset-registry

App SHALL duy trì một registry frame preset cục bộ được đóng gói cùng app.

- Mỗi preset SHALL định nghĩa: `id`, `label`, `asset` (local require), cấu hình `nineSlice`, `frameFaceWidthCm`, `minInnerWidthPx`, `minInnerHeightPx`
- Cấu hình `nineSlice` SHALL chỉ định `{ left, top, right, bottom }` insets cho 9-slice rendering
- Presets SHALL được lưu tại `screens/create-flat2d-framing/constants/frame-presets.ts`
- Asset khung SHALL được lưu tại `assets/images/frames/`
- V1 SHALL bao gồm tối thiểu 3 frame preset (ví dụ: đen mỏng, vàng cổ điển, trắng hiện đại)

#### Scenario: App tải frame presets

- CHO rằng màn hình editor mở
- THÌ bộ chọn frame hiển thị tất cả preset có sẵn từ registry cục bộ
- VÀ mỗi preset hiển thị thumbnail trực quan

### Requirement: frame-selector-ui

Editor SHALL cung cấp UI chọn frame cho phép người dùng chọn frame preset.

- Bộ chọn SHALL hiển thị preview frame trong danh sách cuộn ngang
- Chọn frame SHALL cập nhật `framePresetId` trong store
- Người dùng SHALL có thể bỏ chọn frame (đặt thành "không có khung")
- Chọn frame SHALL cập nhật ngay canvas preview

#### Scenario: Người dùng chọn frame

- CHO rằng editor đang mở với artwork đã chọn
- KHI người dùng chạm thumbnail frame preset
- THÌ preview cập nhật hiển thị artwork với frame đã chọn
- VÀ `framePresetId` trong store cập nhật thành ID của preset đã chọn

#### Scenario: Người dùng bỏ chọn frame

- CHO rằng người dùng đã chọn một frame
- KHI người dùng chạm lại frame đang chọn (hoặc tùy chọn "không có khung")
- THÌ preview hiển thị artwork không có khung
- VÀ `framePresetId` trong store đặt thành null

### Requirement: nine-slice-frame-renderer

App SHALL render khung sử dụng kỹ thuật 9-slice (nine-patch) qua `@shopify/react-native-skia`.

- Renderer SHALL NOT kéo giãn 4 vùng góc của asset khung
- Renderer SHALL only be kéo giãn vùng cạnh theo trục tương ứng
- Renderer SHALL dùng `drawImageNine` hoặc API Skia tương đương
- V1 implementation SHALL dùng API tương đương (`drawImageRect` cho 9 vùng) để giữ tương thích pipeline JSX/declarative hiện tại
- Nếu mat width > 0, renderer SHALL render lớp mat màu đồng nhất giữa artwork và cạnh trong khung
- Kích thước ngoài khung SHALL được tính: `innerSize + 2 * frameFaceWidthCm` (chuyển sang pixel)

#### Scenario: Frame render đúng với 9-slice

- CHO rằng frame preset có nineSlice `{ left: 30, top: 30, right: 30, bottom: 30 }`
- KHI render quanh vùng inner 400x600px
- THÌ 4 góc là 30x30px mỗi góc, không bị scale
- VÀ cạnh trên/dưới kéo giãn ngang để lấp khoảng trống
- VÀ cạnh trái/phải kéo giãn dọc để lấp khoảng trống

#### Scenario: Frame render có mat

- CHO rằng frame đã chọn và matWidthCm = 5
- KHI framed artwork render
- THÌ vùng mat màu trắng/trung tính hiển thị giữa cạnh artwork và cạnh trong khung
- VÀ độ rộng mat đồng đều ở cả 4 cạnh
