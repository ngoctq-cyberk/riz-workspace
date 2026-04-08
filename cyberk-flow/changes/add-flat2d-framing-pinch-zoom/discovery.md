# Khám phá: add-flat2d-framing-pinch-zoom

## Các Workstream

| # | Workstream | Trạng thái | Phát hiện chính |
|---|---|---|---|
| 1 | Truy hồi change hiện có | ✅ Xong | `add-flat2d-framing` đã triển khai xong phần core/editor/export; pinch zoom phù hợp hơn khi tách thành follow-up change riêng |
| 2 | Ảnh chụp kiến trúc | ✅ Xong | `create-flat2d-framing` đã có route, store, preview Skia, snapshot renderer, hidden exporter |
| 3 | Patterns nội bộ | ✅ Xong | Repo đã có `GestureDetector`, Reanimated, pattern commit state sau gesture end |
| 4 | Nghiên cứu bên ngoài | ⏭ Bỏ qua | Tất cả dependency cần thiết đã có sẵn trong app |
| 5 | Kiểm tra ràng buộc | ✅ Xong | Preview hiện nằm trong `ScrollView`; state nguồn hiện là `printSizeCm`; snapshot cache chưa tính zoom scale |

## Phát hiện chính

### Vì sao tách change riêng

- `add-flat2d-framing` đã là một change gần hoàn tất, chủ yếu còn gate E2E và sync docs
- Pinch zoom là enhancement mới, đụng vào hành vi UI và contract preview/export sau khi feature chính đã xong
- Nếu tiếp tục nhét vào change cũ, workflow sẽ khó phân biệt đâu là phần shipped core flow và đâu là follow-up enhancement

### Dependencies (tất cả đã xác nhận có trong app)

| Dependency | Phiên bản | Trạng thái |
|---|---|---|
| `react-native-gesture-handler` | ~2.28.0 | ✅ Đã cài |
| `react-native-reanimated` | ~4.1.1 | ✅ Đã cài |
| `@shopify/react-native-skia` | 2.2.12 | ✅ Đã cài |
| `zustand` | ^5.0.9 | ✅ Đã cài |
| `react-native-view-shot` | 4.0.3 | ✅ Đã cài |

### Code paths liên quan

- `riz-app-v2/screens/create-flat2d-framing/components/flat2d-preview.tsx`
  Preview hiện render background/foreground bằng `expo-image`, framed artwork bằng Skia, nhưng chưa có gesture layer.
- `riz-app-v2/store/framing-flat2d-store.ts`
  Store hiện chưa có state riêng cho zoom composition; source-of-truth hình học vẫn là `printSizeCm`, `matWidthCm`, `framePresetId`.
- `riz-app-v2/screens/create-flat2d-framing/lib/layout-contract.ts`
  Placement contract hiện dùng `baseScale` + `autoFitScale`; chưa có lớp scale commit từ pinch zoom.
- `riz-app-v2/screens/create-flat2d-framing/components/artwork-snapshot.tsx`
  Snapshot dùng `resolveFlat2DLayout(... baseScale: 1)` và cache signature chưa bao gồm zoom scale.
- `riz-app-v2/screens/create-flat2d-framing/components/exporter.tsx`
  Hidden exporter đang compose scene cuối từ snapshot framed artwork + background/foreground, nên nếu zoom là một phần của composition thì snapshot/export phải cùng nhận một contract mới.

### Patterns nội bộ có thể tái sử dụng

#### 1. Gesture + commit sau khi tương tác kết thúc

- `components/ui/slider.tsx` dùng `GestureDetector` + `Gesture.Pan()`
- `screens/panorama/components/PlaceOnWallStep.tsx` dùng `runOnJS()` để commit state sau `.onEnd()`
- Pattern này phù hợp cho pinch: giữ state tạm trong phiên gesture, chỉ commit về store khi kết thúc

#### 2. Preview/export parity contract

- `screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts` đã khóa một phần parity giữa preview và export
- Đây là điểm phù hợp để mở rộng thêm case `compositionScale != 1`

#### 3. Snapshot cache theo signature

- `getArtworkSnapshotSignature()` đang là điểm gom input ảnh hưởng tới snapshot
- Nếu pinch zoom là behavior exportable, signature bắt buộc phải thêm zoom scale để tránh cache hit sai

## Phân tích khoảng trống

| Đã có | Cần thêm | Khoảng trống |
|---|---|---|
| Preview Flat2D live bằng Skia | Pinch zoom trực tiếp trên preview | Thiếu `Gesture.Pinch()` + phối hợp với `ScrollView` cha |
| Placement contract dùng chung preview/export | Scale riêng cho composition | Thiếu `compositionScale` tách khỏi `printSizeCm` |
| Snapshot renderer/exporter ổn định | Export phản ánh pinch zoom | Thiếu nối `compositionScale` vào snapshot signature và canonical layout |
| Store draft Flat2D | Reset/feedback cho zoom | Thiếu state/action/reset UI cho zoom |
| Unit parity test | Case pinch zoom | Thiếu test cho `compositionScale` và clamp bounds |

## Rủi ro

| Rủi ro | Mức độ | Giảm thiểu |
|---|---|---|
| Xung đột pinch với cuộn trang | MEDIUM | Khóa `ScrollView` trong phiên pinch; bật lại ở `onEnd/onFinalize` |
| Preview/export lệch scale | MEDIUM | Dùng một `compositionScale` chung cho preview, snapshot, export |
| Gesture spam re-render | LOW | Dùng transient state trong phiên gesture, chỉ commit store khi kết thúc |
| Cache snapshot sai | LOW | Nối `compositionScale` vào snapshot signature và thêm test cache invalidation |

## Đề xuất

Tạo change riêng `add-flat2d-framing-pinch-zoom` với phạm vi hẹp:

1. Thêm `compositionScale` vào contract Flat2D
2. Triển khai pinch zoom + reset UI trên preview
3. Đồng bộ snapshot/export/tests/E2E cho behavior mới

**Cách tiếp cận khuyến nghị**: Không ghi đè `printSizeCm` khi pinch. Dùng `compositionScale` riêng để giữ nghĩa của kích thước in vật lý và giúp feature này đứng độc lập với change gốc.
