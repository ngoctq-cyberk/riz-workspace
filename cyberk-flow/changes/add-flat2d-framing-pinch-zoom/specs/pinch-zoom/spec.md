# Đặc tả: pinch-zoom

## ADDED Requirements

### Requirement: pinch-zoom-preview

Editor SHALL cho phép người dùng pinch 2 ngón trực tiếp trên preview để scale toàn bộ framed mockup.

- Pinch zoom SHALL scale đồng thời artwork, mat, frame, và shadow quanh `anchorPx`
- Gesture SHALL chỉ hoạt động khi artwork, `printSizeCm`, và preview layout đều đã sẵn sàng
- V1 SHALL NOT hỗ trợ pan hoặc rotate framed mockup bằng gesture này

#### Scenario: Pinch out làm framed mockup lớn hơn

- CHO rằng artwork, frame, và scene đã được chọn
- KHI người dùng pinch out trên preview
- THÌ framed mockup lớn lên quanh anchor của scene
- VÀ background/foreground scene giữ nguyên

#### Scenario: Pinch in làm framed mockup nhỏ hơn

- CHO rằng người dùng đã zoom framed mockup lớn hơn mặc định
- KHI người dùng pinch in trên preview
- THÌ framed mockup nhỏ lại mượt mà
- VÀ anchor không bị dịch chuyển

### Requirement: pinch-zoom-state-contract

App SHALL quản lý pinch zoom bằng trạng thái riêng, tách khỏi `printSizeCm`.

- Store SHALL có `compositionScale` mặc định `1`
- Phiên pinch SHALL dùng `transientGestureScale` trong lúc tương tác và commit thành `compositionScale` khi gesture kết thúc
- `printSizeCm` SHALL NOT bị ghi đè bởi pinch zoom
- Preview, artwork snapshot, và hidden exporter SHALL cùng dùng `compositionScale` đã commit

#### Scenario: Pinch zoom được giữ tới lúc export

- CHO rằng người dùng pinch framed mockup tới `compositionScale = 1.25`
- KHI người dùng export
- THÌ snapshot artwork và ảnh export cuối đều phản ánh scale `1.25`
- VÀ không bị quay về scale mặc định `1.0`

### Requirement: pinch-zoom-bounds

App SHALL kẹp pinch zoom trong biên hợp lệ của scene.

- `compositionScale` hiệu lực SHALL NOT nhỏ hơn `0.25`
- `compositionScale` hiệu lực SHALL NOT vượt `maxCompositionScale` suy ra từ `safeRectPx`
- Khi người dùng pinch vượt biên, preview SHALL dừng tại giá trị biên thay vì tạo state không export được

#### Scenario: Pinch vượt safe rect bị chặn ở scale tối đa

- CHO rằng framed mockup đang ở gần giới hạn của `safeRectPx`
- KHI người dùng tiếp tục pinch out
- THÌ preview dừng ở `maxCompositionScale`
- VÀ export sau đó vẫn tạo ra layout hợp lệ trong scene

### Requirement: pinch-zoom-scroll-coordination

Editor SHALL phối hợp pinch zoom với cuộn màn hình để tránh xung đột gesture.

- Trong lúc pinch đang active, `ScrollView` cha SHALL tạm thời bị khóa cuộn
- Khi pinch kết thúc hoặc bị hủy, `ScrollView` cha SHALL được bật lại
- Một chạm hoặc kéo 1 ngón SHALL NOT làm framed mockup di chuyển

#### Scenario: Pinch không kéo theo cuộn trang

- CHO rằng người dùng đang ở màn hình edit Flat2D
- KHI người dùng pinch trên preview
- THÌ preview zoom theo gesture
- VÀ trang không bị cuộn trong thời gian pinch

### Requirement: pinch-zoom-feedback

Editor SHALL hiển thị trạng thái pinch zoom hiện tại và cung cấp cách quay về mặc định.

- Khi `compositionScale != 1`, UI SHALL hiển thị chip/badge phần trăm scale hiện tại
- UI SHALL cung cấp action reset để đưa `compositionScale` về `1`
- Reset SHALL cập nhật ngay preview và export contract

#### Scenario: Người dùng reset về 100%

- CHO rằng `compositionScale = 0.8`
- KHI người dùng chạm action reset
- THÌ `compositionScale` trở về `1`
- VÀ preview phản ánh ngay kích thước mặc định
