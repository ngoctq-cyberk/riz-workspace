# Đặc tả: auto-fit

## ADDED Requirements

### Requirement: auto-fit-calculation

App SHALL tự động scale placement artwork khi nó vượt safe rect của scene.

- Auto-fit SHALL kích hoạt khi canonical outer rect vượt ra ngoài `safeRectPx`
- Khoảng trống từ `anchorPx` đến từng cạnh của `safeRectPx` SHALL được tính riêng:
  - `leftClearancePx = max(0, anchorPx.x - safeRectPx.x)`
  - `rightClearancePx = max(0, safeRectPx.x + safeRectPx.width - anchorPx.x)`
  - `topClearancePx = max(0, anchorPx.y - safeRectPx.y)`
  - `bottomClearancePx = max(0, safeRectPx.y + safeRectPx.height - anchorPx.y)`
- Kích thước lớn nhất có thể fit quanh anchor SHALL được tính:
  - `maxFitWidthPx = 2 * min(leftClearancePx, rightClearancePx)`
  - `maxFitHeightPx = 2 * min(topClearancePx, bottomClearancePx)`
- Hệ số scale: `min(1, maxFitWidthPx / outerWidthPx, maxFitHeightPx / outerHeightPx)`
- Scale SHALL được áp dụng vào rect hiển thị trong khi giữ tâm tại anchor
- `autoFitApplied` SHALL được đặt thành `true` trong store khi scale < 1
- `autoFitScale` SHALL lưu hệ số scale đã tính
- Auto-fit SHALL NOT thay đổi giá trị `printSizeCm` gốc của người dùng

#### Scenario: Artwork nằm trong safe rect

- CHO rằng kích thước ngoài 400px × 300px và safeRect 800px × 600px
- THÌ autoFitScale = 1.0 và autoFitApplied = false
- VÀ không áp dụng scaling

#### Scenario: Artwork vượt safe rect theo chiều rộng

- CHO rằng kích thước ngoài 1200px × 600px và safeRect 800px × 600px
- THÌ scaleFactor = min(800/1200, 600/600) = 0.667
- VÀ autoFitScale = 0.667, autoFitApplied = true
- VÀ rect hiển thị được scale 0.667, căn giữa tại anchor

#### Scenario: Artwork vượt safe rect cả hai chiều

- CHO rằng kích thước ngoài 1200px × 900px và safeRect 800px × 600px
- THÌ scaleFactor = min(800/1200, 600/900) = min(0.667, 0.667) = 0.667
- VÀ rect hiển thị được scale 0.667

#### Scenario: Safe rect lệch tâm quanh anchor

- CHO rằng `safeRectPx = { x: 700, y: 400, width: 600, height: 800 }`
- VÀ `anchorPx = { x: 850, y: 800 }`
- VÀ kích thước ngoài là `500px × 700px`
- KHI auto-fit được tính
- THÌ `leftClearancePx = 150`, `rightClearancePx = 450` nên `maxFitWidthPx = 300`
- VÀ `topClearancePx = 400`, `bottomClearancePx = 400` nên `maxFitHeightPx = 800`
- VÀ `autoFitScale = min(1, 300/500, 800/700) = 0.6`
- VÀ rect sau scale vẫn nằm trọn trong `safeRectPx` dù anchor không nằm ở tâm safe rect

### Requirement: auto-fit-toast

App SHALL hiển thị thông báo toast khi auto-fit được áp dụng.

- Toast SHALL dùng `Toast.show()` (không phải `Alert`)
- Toast SHALL only xuất hiện khi `autoFitScale < 1`
- Toast SHALL NOT lặp lại cho cùng trạng thái auto-fit (debounce trên cùng scale)
- Message IDs: `flat2d_autofit_applied_title`, `flat2d_autofit_applied_message`

#### Scenario: Toast hiện khi auto-fit

- CHO rằng artwork được đặt và auto-fit kích hoạt (scale < 1)
- KHI auto-fit được áp dụng lần đầu
- THÌ thông báo toast xuất hiện ngắn gọn
- VÀ toast không xuất hiện lại cho đến khi giá trị scale thay đổi

#### Scenario: Không toast khi artwork vừa khít

- CHO rằng artwork nằm trong safe rect (scale = 1)
- THÌ không hiển thị toast auto-fit
