# artwork-input Specification

## Purpose
TBD
## Requirements

### Requirement: artwork-picker

Màn hình editor SHALL cung cấp bộ chọn artwork sử dụng `useCropFlow` hiện có để chọn và crop ảnh đơn.

- Bộ chọn SHALL sử dụng `expo-image-picker` để cho phép người dùng chọn ảnh từ thư viện thiết bị
- Sau khi chọn ảnh, bộ chọn SHALL gọi `useCropFlow` hiện có để crop ảnh
- Bộ chọn SHALL xử lý tự động chuyển đổi URI `ph://` (qua `useCropFlow` hiện có)
- Sau khi crop, store SHALL ghi nhận `artwork.uri`, `artwork.width`, `artwork.height`, và `artwork.aspectRatio`
- Tỉ lệ khung hình SHALL được suy ra từ kích thước ảnh đã crop

#### Scenario: Người dùng chọn và crop artwork

- CHO rằng màn hình editor đang mở và chưa có artwork nào
- KHI người dùng chạm vào khu vực chọn artwork
- THÌ `expo-image-picker` mở thư viện ảnh thiết bị
- VÀ sau khi người dùng chọn một ảnh, `useCropFlow` khởi chạy ở chế độ crop
- VÀ sau khi crop, artwork xuất hiện trong preview
- VÀ store cập nhật với dữ liệu ảnh đã crop

#### Scenario: Người dùng thay thế artwork hiện có

- CHO rằng artwork đã được chọn
- KHI người dùng chạm lại vào khu vực artwork
- THÌ `expo-image-picker` mở để chọn ảnh mới, rồi chuyển qua `useCropFlow`
- VÀ artwork cũ bị thay thế trong store

### Requirement: print-size-input

Editor SHALL cho phép người dùng nhập kích thước in vật lý của artwork tính bằng cm.

- Editor SHALL cung cấp ô nhập số cho một chiều (chiều rộng hoặc chiều cao)
- Chiều còn lại SHALL được tự động tính từ tỉ lệ crop của artwork
- `printSizeCm` trong store SHALL đại diện cho vùng in thuần (không bao gồm mat và khung)
- Ô nhập SHALL chấp nhận giá trị thập phân với độ chính xác 0.1cm
- Giá trị nhập SHALL phải lớn hơn 0cm
- Giá trị không hợp lệ (`<= 0`, rỗng, hoặc không parse được) SHALL NOT cập nhật `printSizeCm`
- Chiều rộng in mặc định SHALL là 40cm nếu chưa có input

#### Scenario: Người dùng nhập chiều rộng in

- CHO rằng artwork có tỉ lệ 3:4 đã được chọn
- KHI người dùng nhập width = 60cm
- THÌ height được tự động tính là 80cm (60 * 4/3)
- VÀ `printSizeCm` cập nhật thành `{ width: 60, height: 80 }`

#### Scenario: Kích thước in cập nhật khi artwork thay đổi

- CHO rằng width in đã đặt 60cm cho artwork 3:4
- KHI người dùng chọn artwork mới với tỉ lệ 1:1
- THÌ height tự tính lại thành 60cm (giữ nguyên width)
- VÀ `printSizeCm` cập nhật thành `{ width: 60, height: 60 }`

#### Scenario: Giá trị kích thước in không hợp lệ bị từ chối

- CHO rằng artwork đã được chọn và `printSizeCm` hiện tại là `{ width: 40, height: 60 }`
- KHI người dùng nhập `0`, giá trị âm, hoặc chuỗi không hợp lệ vào ô width
- THÌ editor hiển thị trạng thái validation lỗi
- VÀ `printSizeCm` giữ nguyên giá trị hợp lệ gần nhất

### Requirement: mat-width-input

Editor SHALL cho phép người dùng điều chỉnh độ rộng mat (passe-partout) tính bằng cm.

- Độ rộng mat mặc định SHALL là 5cm
- Độ rộng mat SHALL nhận giá trị từ 0 đến 15cm
- Thay đổi mat SHALL phản ánh ngay trong preview
- Độ rộng mat SHALL được lưu dạng `matWidthCm` trong store

#### Scenario: Người dùng điều chỉnh độ rộng mat

- CHO rằng artwork và frame đã được chọn
- KHI người dùng đặt mat width = 3cm
- THÌ preview hiển thị viền mat 3cm giữa artwork và khung
- VÀ `matWidthCm` cập nhật thành 3

