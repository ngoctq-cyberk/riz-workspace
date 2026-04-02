# Khám phá: add-flat2d-framing

## Các Workstream

| # | Workstream | Trạng thái | Phát hiện chính |
|---|---|---|---|
| 1 | Truy hồi bộ nhớ | ✅ Xong | Không có change nào liên quan flat2d framing trong bộ nhớ cyberk-flow |
| 2 | Ảnh chụp kiến trúc | ✅ Xong | Expo Router file-based routing; entities theo FSD tại `lib/entities/`; screens tại `screens/<feature>/` |
| 3 | Patterns nội bộ | ✅ Xong | 5 pattern tái sử dụng chính đã xác định (xem bên dưới) |
| 4 | Nghiên cứu bên ngoài | ⏭ Bỏ qua | Tất cả deps đã có trong project — không cần nghiên cứu thêm |
| 5 | Kiểm tra ràng buộc | ✅ Xong | 3 deps cốt lõi đã xác nhận có sẵn (xem bên dưới) |

## Phát hiện chính

### Dependencies (tất cả đã xác nhận có trong package.json)

| Dependency | Phiên bản | Trạng thái |
|---|---|---|
| `@shopify/react-native-skia` | 2.2.12 | ✅ Đã cài |
| `zustand` | ^5.0.9 | ✅ Đã cài |
| `react-native-view-shot` | 4.0.3 | ✅ Đã cài |

### Sử dụng Skia hiện tại

Skia hiện đang được dùng ở 2 nơi:
- `components/image-crop-view/crop-overlay.tsx` — `Canvas`, `Path`, `Skia`, `Line`, `vec`
- `components/gradient/gradient-text.tsx` — gradient rendering

**Lưu ý**: Không nơi nào dùng 9-slice hoặc image-based rendering. 9-slice frame rendering sẽ là **pattern Skia mới** trong codebase này.

### Patterns nội bộ có thể tái sử dụng

#### 1. Pattern Route (Expo Router)
- Routes tại `app/(protected)/<route-name>.tsx` — wrapper mỏng import screen component
- Ví dụ: `create-panorama.tsx` → `CreatePanorama360FromTemplateScreen`
- Navigation qua `router.navigate({ pathname, params })`

#### 2. Luồng Upload/Publish (`screens/upload-project/index.tsx`)
- `UploadProjectScreen` nhận param `images` dạng JSON-stringify `ImageData[]`
- Mỗi image: `{ id, uri, width, height }`
- Xử lý URI `ph://` qua `MediaLibrary.getAssetInfoAsync`
- Tạo project `TWO_D` qua hook `useProjectUpload`
- **Đây chính là điểm bàn giao** cho flat2d framing export

#### 3. Luồng Crop (`hooks/media/use-crop-flow.ts`)
- `useCropFlow()` trả về `{ startCropFlow, isProcessing }`
- Xử lý chuyển đổi `ph://` → URI thực
- Config: `{ images, aspectRatio, lockedRatio, onComplete, onCancel }`
- Trả `CroppedImage[]` qua callback `onComplete`

#### 4. Hidden Export View (`screens/create-panorama/components/composer-screen.tsx`)
- Pattern: `View` ngoài màn hình tại `left: -9999`, kích thước pixel cố định
- Dùng `react-native` `Image` (KHÔNG phải `expo-image`) để `captureRef` ổn định
- `captureRef()` từ `react-native-view-shot` → PNG local → nén WebP
- Bao gồm pattern `waitForRenderReady()` polling cho trạng thái tải ảnh
- Exporter dùng `forwardRef` + `useImperativeHandle` để component cha điều khiển

#### 5. Placement chuẩn hóa (`screens/panorama/components/PlaceOnWallStep.tsx`)
- Lưu vị trí overlay dạng giá trị chuẩn hóa: `offsetX/Y ∈ [0,1]`, `scale ∈ (0, ?)`, `scaleW`
- Kích thước preview tính qua callback `onLayout`
- `syncToState()` chuyển vị trí pixel → giá trị chuẩn hóa
- Kéo thả dùng `react-native-reanimated` shared values + `Gesture.Pan()`

### Model Scene Template 3D hiện tại

```ts
interface SceneTemplate {
  id: number;
  name?: string;
  config?: { dimensions: {x,y,z}, camera: {position, target, fov} };
  slots?: SceneTemplateSlot[];
}
```

→ **Gắn chặt với 3D**. Spec đúng khi xác định KHÔNG NÊN tái sử dụng cho Flat2D.

### Pattern Store

Các store hiện có: `auth-store.ts`, `iap-store.ts`, `locale-store.ts`, `onboarding-store.ts`
- Tất cả dùng `zustand` với middleware `persist` và `AsyncStorage`
- `framing-flat2d-store.ts` mới sẽ theo cùng pattern nhưng **không persist** (chỉ draft, không cần lưu đĩa cho v1)

### Cấu trúc Entity

Entities hiện có tại `lib/entities/`:
- `3d/` — 3D rendering
- `model3d/` — quản lý model 3D
- `scene-template/` — scene template 3D (api/, hooks/, model/)

Mỗi entity theo cấu trúc: `api/` (hàm fetch), `hooks/` (wrapper react-query), `model/` (TypeScript interfaces)

### Điểm vào: Màn hình Tạo 2D

`screens/create-project/components/create-2d.tsx`:
- Component `Create2DProject` với gallery ảnh + carousel
- Nút "Tiếp" → `useCropFlow` → điều hướng đến `/upload-project`
- **CTA cho flat2d framing cần thêm tại đây** (hoặc ở component cha `create-project`)

## Phân tích khoảng trống

| Đã có | Cần thêm | Khoảng trống |
|---|---|---|
| Skia deps | 9-slice rendering | Code vẽ Skia mới (pipeline JSX với `drawImageRect` 9 vùng; `drawImageNine` là phương án imperative khả dụng) |
| Pattern hidden export view | Export scene Flat2D | Component exporter mới theo pattern hiện có |
| Luồng upload | Bàn giao từ flat2d | Tối thiểu — chỉ cần navigate với image data |
| Hook crop flow | Input artwork | Tái sử dụng trực tiếp — không cần thay đổi |
| Pattern placement chuẩn hóa | Placement trên scene | Đơn giản hóa (không drag trong v1, chỉ anchor-based) |
| Entity scene template 3D | Entity scene template 2D | Entity mới song song với cấu trúc hiện có |
| Pattern route | Route mới | File route mới theo convention hiện tại |
| Zustand stores | Store editor Flat2D | Store mới — không cần persistence |
| File localization | Message IDs mới | Chỉ bổ sung — không breaking changes |
| `GET /scenes/templates` (3D) | `GET /scenes/templates/2d` | **Endpoint API mới** — cần backend nhưng v1 dùng stub |

## Rủi ro

| Rủi ro | Mức độ | Giảm thiểu |
|---|---|---|
| Độ chính xác 9-slice rendering Skia | MEDIUM | Chốt contract 9-slice bằng test src/dst rects; giữ khả năng chuyển sang `drawImageNine` (imperative) nếu cần tối ưu |
| Khớp preview/export | MEDIUM | Dùng chung module tính toạ độ; cùng thứ tự layer |
| Backend API chưa sẵn sàng | LOW | Dùng stub data cục bộ, tách rời frontend khỏi timeline backend |
| Hiệu năng (ảnh scene lớn) | LOW | `expo-image` cho preview; RN `Image` chỉ trong export |
| Navigation stack sau export | LOW | Theo pattern panorama→upload hiện có (đã fix gần đây) |

## Câu hỏi mở

1. **Timeline backend**: Khi nào `GET /scenes/templates/2d` sẵn sàng? Có thể bắt đầu với stub data cục bộ không? → **Đã quyết định: dùng local stub data**
2. **Asset khung**: Ảnh PNG khung (cho 9-slice) đã thiết kế/sẵn sàng chưa, hay cần tạo mới?
3. **Asset scene**: Ảnh background/foreground cho room scenes có sẵn chưa, hay cần sản xuất?

## Đề xuất

Tiến hành với plan **Standard** complexity. Tất cả deps cốt lõi đã có. Công việc mới chính:

1. Skia 9-slice frame renderer (pattern Skia mới trong codebase)
2. Entity Flat2D scene template + tích hợp API
3. Màn hình editor với zustand store
4. Pipeline export theo pattern hidden-view-shot hiện có

**Cách tiếp cận đề xuất**: Bắt đầu với local stub data cho scene templates và frame presets trong khi backend phát triển API. Cho phép frontend phát triển độc lập.
