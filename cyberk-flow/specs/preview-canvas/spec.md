# preview-canvas Specification

## Purpose
TBD
## Requirements

### Requirement: preview-layer-order

Canvas preview SHALL render đúng 4 lớp theo thứ tự cố định:

1. Ảnh background scene
2. Shadow (dưới framed artwork)
3. Framed artwork (khung + mat + artwork đã crop)
4. Foreground occluder (nếu có trong template)

- Thứ tự lớp này SHALL giống hệt cho cả preview và export
- Preview SHALL dùng `expo-image` cho hiệu năng background/foreground
- Lớp framed artwork SHALL dùng Skia rendering trực tiếp (không phải snapshot)

#### Scenario: Preview render đủ 4 lớp

- CHO rằng artwork, frame, và scene có foreground đều đã được chọn
- KHI preview render
- THÌ background ở dưới cùng, shadow xuất hiện dưới framed artwork, framed artwork nằm giữa anchor, và foreground che một phần artwork

### Requirement: preview-aspect-ratio

Canvas preview SHALL giữ đúng tỉ lệ khung hình của `sceneSizePx`.

- Canvas SHALL NOT kéo giãn để lấp toàn bộ không gian màn hình
- Preview scale SHALL là: `min(availableWidth / sceneWidthPx, availableHeight / sceneHeightPx)`
- Tất cả vị trí placement SHALL được resolve từ toạ độ scene canonical dùng cùng preview scale

#### Scenario: Preview giữ đúng tỉ lệ

- CHO rằng scene có `sceneSizePx: { width: 2048, height: 1536 }` (tỉ lệ 4:3)
- KHI hiển thị trên màn hình rộng 390px
- THÌ canvas preview là 390px × 292.5px (giữ đúng 4:3)
- VÀ toạ độ placement scale tỉ lệ tương ứng

### Requirement: placement-calculation

Preview SHALL đặt framed artwork dùng phép tính placement canonical.

- Kích thước ngoài: `outerWidthCm = printWidthCm + 2 * (matWidthCm + frameFaceWidthCm)`
- Pixel ngoài: `outerWidthPx = outerWidthCm * pixelsPerCm`
- Rect canonical: căn giữa tại `anchorPx` với kích thước ngoài
- Rect preview: rect canonical nhân với `previewScale`

#### Scenario: Artwork đặt tại tâm anchor

- CHO rằng scene có `anchorPx: { x: 1024, y: 768 }`, `pixelsPerCm: 10`
- VÀ artwork `printSizeCm: { width: 60, height: 80 }`, mat 5cm, frame face 2cm
- THÌ kích thước ngoài = `{ width: 74cm, height: 94cm }` → `{ width: 740px, height: 940px }`
- VÀ rect canonical căn giữa tại (1024, 768) với kích thước đó

