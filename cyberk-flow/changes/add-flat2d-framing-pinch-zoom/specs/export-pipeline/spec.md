# Đặc tả: export-pipeline

## ADDED Requirements

### Requirement: pinch-zoom-export-parity

Pipeline export SHALL phản ánh đúng `compositionScale` đã commit từ preview.

- Snapshot cache signature SHALL bao gồm `compositionScale`
- Khi `compositionScale` thay đổi, snapshot cũ SHALL NOT được tái sử dụng
- Hidden exporter SHALL nhận output từ snapshot đã render theo `compositionScale` mới

#### Scenario: Thay đổi pinch zoom làm snapshot cache miss

- CHO rằng snapshot đã tồn tại tại `compositionScale = 1`
- KHI người dùng pinch zoom và commit `compositionScale = 1.2`
- THÌ snapshot cũ SHALL NOT được tái sử dụng
- VÀ pipeline tạo snapshot mới phản ánh scale mới

#### Scenario: Export dùng scale đã commit

- CHO rằng preview đang hiển thị `compositionScale = 0.9`
- KHI người dùng export
- THÌ snapshot artwork và ảnh export cuối đều dùng `compositionScale = 0.9`
- VÀ ảnh upload không quay về layout mặc định
