# Đặc Tả Mới: Framing Flat2D Cho `riz-app-v2`

## 1. Mục tiêu

Tài liệu này định nghĩa lại feature `Framing & Scene Mockup` theo hướng bám sát codebase hiện tại của `riz-app-v2` và các artifact thiết kế trong `cyberk-flow/changes/add-flat2d-framing/`.

Mục tiêu của v1:

- Cho phép user chọn ảnh artwork, chọn khung, chọn room scene trực diện.
- Render khung bằng `@shopify/react-native-skia` với cơ chế 9-slice.
- Đặt artwork đã đóng khung vào room scene trực diện bằng mô hình 2D orthogonal.
- Export ra 1 ảnh cuối cùng và publish bằng flow `TWO_D` hiện có.
- Không tạo thêm một loại project mới trong app.

## 2. Quyết định kiến trúc

### 2.1 Quyết định chính

- Published output của feature này vẫn là `TWO_D`.
- Không reuse `room-image` làm persistence chính cho production flow.
- Không nhét feature này vào model `SceneTemplate` 3D hiện có.
- Tạo một contract riêng cho `Flat2D scene template`.
- State editor dùng `zustand`.
- Placement dùng hệ toạ độ canonical pixel-only xuyên suốt; preview chỉ resolve từ canonical bằng `previewScale`, export dùng canonical trực tiếp.
- Snapshot/flatten chỉ thực hiện khi export hoặc khi cần cache có kiểm soát.

### 2.2 Vì sao chọn hướng này

Hướng này bám đúng các flow và abstraction đang tồn tại:

- `riz-app-v2/screens/create-project/index.tsx` đang là entry cho tạo project.
- `riz-app-v2/screens/upload-project/index.tsx` đã là production flow để publish `TWO_D`.
- `riz-app-v2/screens/create-panorama/components/composer-screen.tsx` đã có pattern export hidden view bằng `captureRef`.
- `riz-app-v2/screens/panorama/components/PlaceOnWallStep.tsx` cho thấy pattern tách placement math và render math, phù hợp để tái dùng tư duy resolver chung cho preview/export.
- `riz-app-v2/screens/debug/image-composer-screen.tsx` đã có pattern background + overlay composition.

## 3. Phạm vi

### 3.1 In scope

- 2D orthogonal scene mockup
- 9-slice framing bằng Skia
- Background và foreground occluder
- Auto-fit để tránh artwork vượt vùng an toàn
- Export ảnh cuối cùng sang flow publish `TWO_D`
- Chọn input theo thứ tự bất kỳ

### 3.2 Out of scope cho v1

- Perspective transform
- Drag tự do artwork ra ngoài anchor của template
- Rotation artwork
- Save draft lên backend
- Tạo một `ProjectType` mới
- Đồng bộ frame preset từ backend

## 4. Tích hợp với codebase hiện tại

### 4.1 Luồng publish

Feature mới sẽ không thay thế luồng `Create2DProject` hiện tại. V1 chỉ bổ sung thêm một nhánh tạo ảnh 2D có mockup:

1. User vào flow tạo project 2D.
2. User chọn nhánh `Framing Flat2D`.
3. App mở composer mới.
4. Composer export ra 1 local image.
5. App điều hướng sang `riz-app-v2/screens/upload-project/index.tsx`.
6. `UploadProjectScreen` tiếp tục upload và tạo project `TWO_D` như hiện tại.

Điểm quan trọng:

- Không đổi contract tạo project hiện tại.
- Không đổi `ProjectType`.
- Không tạo một entity publish riêng cho feature này ở app layer.

### 4.2 Những phần được reuse

- `riz-app-v2/hooks/media/use-crop-flow.ts`
- `riz-app-v2/screens/upload-project/index.tsx`
- `riz-app-v2/screens/create-panorama/components/composer-screen.tsx`
- `riz-app-v2/integrations/react-intl/locales/en.json`
- `riz-app-v2/integrations/react-intl/locales/vi.json`

### 4.3 Những phần không nên reuse trực tiếp

- `riz-app-v2/lib/entities/scene-template/model/index.ts`

Lý do:

- Model này đang gắn mạnh với 3D qua `camera`, `dimensions`, `slots`.
- Nếu gắn thêm 2D orthogonal vào cùng model, layer domain sẽ khó hiểu và dễ sinh logic phân nhánh.

- `riz-app-v2/lib/api/room-image/types.ts`
- `riz-app-v2/hooks/use-save-room-image-mutation.ts`

Lý do:

- `room-image` hiện chỉ là nơi lưu một ảnh PNG đơn giản.
- Nó không gắn với flow publish project.
- Nó phù hợp cho debug hơn là production path.

## 5. Đề xuất cấu trúc file

### 5.1 Route và screen

- `riz-app-v2/app/(protected)/create-flat2d-framing.tsx`
- `riz-app-v2/screens/create-flat2d-framing/index.tsx`

### 5.2 UI component

- `riz-app-v2/screens/create-flat2d-framing/components/artwork-picker.tsx`
- `riz-app-v2/screens/create-flat2d-framing/components/frame-selector.tsx`
- `riz-app-v2/screens/create-flat2d-framing/components/scene-selector.tsx`
- `riz-app-v2/screens/create-flat2d-framing/components/flat2d-preview.tsx`
- `riz-app-v2/screens/create-flat2d-framing/components/exporter.tsx`

### 5.3 Domain và API

- `riz-app-v2/lib/entities/flat2d-scene-template/model/index.ts`
- `riz-app-v2/lib/entities/flat2d-scene-template/api/get-all.ts`
- `riz-app-v2/lib/entities/flat2d-scene-template/hooks/use-all.ts`

### 5.4 Store

- `riz-app-v2/store/framing-flat2d-store.ts`

### 5.5 Local frame preset registry

- `riz-app-v2/screens/create-flat2d-framing/constants/frame-presets.ts`
- `riz-app-v2/assets/images/frames/...`

### 5.6 Toán học và helper

- `riz-app-v2/screens/create-flat2d-framing/lib/placement.ts`
- `riz-app-v2/screens/create-flat2d-framing/lib/frame-sizing.ts`
- `riz-app-v2/screens/create-flat2d-framing/lib/nine-slice.ts`
- `riz-app-v2/screens/create-flat2d-framing/lib/layer-order.ts`

## 6. User flow

### 6.1 Entry point

Tại `riz-app-v2/screens/create-project/components/create-2d.tsx`, bổ sung thêm CTA mở composer `Flat2D Framing`.

Luồng cũ chọn nhiều ảnh để upload giữ nguyên.

### 6.2 Flow mới

User có thể thao tác theo bất kỳ thứ tự nào:

- Chọn artwork
- Crop artwork
- Chọn frame preset
- Chọn room scene
- Nhập kích thước vật lý của artwork
- Xem preview
- Export sang flow publish

UI có thể trình bày theo dạng 3 khối:

- Artwork
- Frame
- Scene

Preview luôn nằm ở nửa trên hoặc vùng trung tâm và phản ứng theo state hiện tại.

## 7. Domain model

## 7.1 Scene template 2D

Backend trả về scene template theo pixel canonical và app giữ nguyên hệ canonical này làm source-of-truth.

Contract đề xuất:

```ts
export interface Flat2DSceneTemplate {
  id: string;
  name: string;
  thumbnailUrl?: string;
  assets: {
    backgroundUrl: string;
    foregroundUrl?: string | null;
  };
  sceneSizePx: {
    width: number;
    height: number;
  };
  placement: {
    anchorPx: {
      x: number;
      y: number;
    };
    pixelsPerCm: number;
    safeRectPx: {
      x: number;
      y: number;
      width: number;
      height: number;
    };
    shadow: {
      opacity: number;
      offsetXPx: number;
      offsetYPx: number;
      blurRadiusPx?: number;
      scale?: number;
    };
  };
}
```

### 7.1.1 Quy tắc ingest (không normalize)

Ngay sau khi load template:

- App giữ nguyên `anchorPx` và `safeRectPx` theo pixel canonical.
- App giữ nguyên `sceneSizePx` và `pixelsPerCm` làm canonical input.
- Preview chỉ resolve toạ độ tại thời điểm render bằng `previewScale`.
- Export dùng trực tiếp toạ độ canonical (scale = `1.0`).

Như vậy:

- Backend có thể author scene bằng pixel.
- App không bị khóa vào preview size cụ thể.
- Tránh round-trip convert ratio gây sai số floating-point.
- Preview và export vẫn khớp nhau vì dùng chung placement resolver.

## 7.2 Frame preset

V1 dùng frame preset local trong app, không phụ thuộc backend.

Contract đề xuất:

```ts
export interface FramePreset {
  id: string;
  label: string;
  previewColor?: string;
  asset: number;
  nineSlice: {
    left: number;
    top: number;
    right: number;
    bottom: number;
  };
  frameFaceWidthCm: number;
  minInnerWidthPx: number;
  minInnerHeightPx: number;
}
```

Ghi chú:

- `asset` có thể là local `require(...)`.
- `frameFaceWidthCm` là độ rộng phần khung nhìn thấy.
- `nineSlice` là cấu hình bắt buộc để render bằng Skia.

## 7.3 Store draft

Store đề xuất:

```ts
export interface FramingFlat2DDraftState {
  artwork: {
    uri: string;
    width: number;
    height: number;
    aspectRatio: number;
  } | null;
  framePresetId: string | null;
  sceneTemplateId: string | null;
  printSizeCm: {
    width: number;
    height: number;
  } | null;
  hasPrintSizeValidationError: boolean;
  matWidthCm: number;
  autoFitScale: number;
  autoFitApplied: boolean;
  flatArtworkSnapshotUri: string | null;
  flatArtworkSnapshotSignature: string | null;
  exportImage: {
    uri: string;
    width: number;
    height: number;
  } | null;
  isExporting: boolean;
}
```

### 7.3.1 Quy tắc state

- User chọn input theo bất kỳ thứ tự nào.
- Không có bước nào ép user phải hoàn tất bước trước mới chọn bước sau.
- Derived state được tính từ store, không lưu trùng lặp nếu không cần.
- `autoFitScale` chỉ ảnh hưởng hiển thị scene, không ghi đè lựa chọn kích thước vật lý gốc của user.
- Khi input kích thước in đang invalid ở UI, store cần có cờ validation để khóa export dù giá trị hợp lệ gần nhất vẫn còn trong draft.

## 8. Quy tắc kích thước

### 8.1 Artwork size

`printSizeCm` là kích thước phần ảnh in sau crop, chưa bao gồm mat và frame.

### 8.2 Outer size

Outer size dùng cho placement được tính như sau:

```ts
outerWidthCm = printWidthCm + 2 * (matWidthCm + frameFaceWidthCm)
outerHeightCm = printHeightCm + 2 * (matWidthCm + frameFaceWidthCm)
```

Placement trên scene luôn dùng `outerWidthCm` và `outerHeightCm`.

## 9. Rendering pipeline

## 9.1 Preview pipeline

Preview gồm 4 lớp:

1. Background scene
2. Shadow
3. Framed artwork
4. Foreground occluder

Render order này phải được cố định cho cả preview lẫn export.

## 9.2 Framed artwork renderer

Framed artwork được render bằng Skia:

- Input: cropped artwork, frame preset, mat width, target inner size
- Output: live preview node và snapshot on-demand

Yêu cầu:

- Không stretch 4 góc frame
- Chỉ stretch phần cạnh theo cấu hình 9-slice
- Nếu có mat, render mat như một lớp riêng trong cùng cây render
- Độ dày phần khung nhìn thấy trong preview/export phải bám theo `frameFaceWidthCm` sau khi quy đổi sang pixel canonical
- `nineSlice` chỉ quyết định cách cắt PNG nguồn thành 9 vùng, không phải nguồn sự thật cho độ dày mặt khung ngoài đời

## 9.3 Snapshot policy

Không gọi snapshot mỗi lần user đổi slider hoặc đổi scene.

Policy cho v1:

- Preview dùng live Skia render
- Chỉ tạo `flatArtworkSnapshotUri` khi export
- Khi snapshot mới thay snapshot cũ, xoá file tạm cũ ngay
- Sau handoff thành công sang `UploadProjectScreen`, dọn snapshot temp dùng cho composition
- Không để tích luỹ snapshot/outdated temp qua nhiều lần export
- Cho phép cache snapshot theo chữ ký:

```ts
snapshotKey = `${artworkUri}:${framePresetId}:${printWidthCm}:${printHeightCm}:${matWidthCm}`
```

Nếu chữ ký không đổi, có thể reuse snapshot cũ.

## 9.4 Scene preview canvas

Preview canvas phải giữ nguyên aspect ratio của `sceneSizePx`.

Không được dùng `stretch` toàn vùng preview theo kích thước màn hình, vì làm như vậy placement sẽ lệch.

Quy tắc:

```ts
previewScale = min(availableWidth / sceneWidthPx, availableHeight / sceneHeightPx)
previewCanvasWidth = sceneWidthPx * previewScale
previewCanvasHeight = sceneHeightPx * previewScale
```

Mọi vị trí preview đều resolve từ canonical scene sang preview canvas bằng cùng một scale.

## 10. Placement và auto-fit

## 10.1 Placement canonical

Khi đã có:

- `sceneSizePx`
- `pixelsPerCm`
- `outerWidthCm`
- `outerHeightCm`

App tính:

```ts
outerWidthPx = outerWidthCm * pixelsPerCm
outerHeightPx = outerHeightCm * pixelsPerCm
```

Rect canonical ban đầu:

```ts
left = anchorPx.x - outerWidthPx / 2
top = anchorPx.y - outerHeightPx / 2
width = outerWidthPx
height = outerHeightPx
```

## 10.2 Auto-fit

Không dùng mô hình `safe_area` chỉ có `max_width_px` và `max_height_px`.

V1 bắt buộc dùng `safeRectPx` đầy đủ:

```ts
{
  x: number;
  y: number;
  width: number;
  height: number;
}
```

Nếu canonical rect vượt ra ngoài `safeRectPx`, app tính theo khoảng cách từ `anchorPx` đến từng cạnh:

```ts
leftClearancePx = max(0, anchorPx.x - safeRectPx.x)
rightClearancePx = max(0, safeRectPx.x + safeRectPx.width - anchorPx.x)
topClearancePx = max(0, anchorPx.y - safeRectPx.y)
bottomClearancePx = max(0, safeRectPx.y + safeRectPx.height - anchorPx.y)

maxFitWidthPx = 2 * min(leftClearancePx, rightClearancePx)
maxFitHeightPx = 2 * min(topClearancePx, bottomClearancePx)

scaleFactor = min(1, maxFitWidthPx / outerWidthPx, maxFitHeightPx / outerHeightPx)
```

Sau đó:

- áp scale vào artwork display rect
- giữ tâm tại anchor
- set `autoFitApplied = true`
- đảm bảo rect sau scale vẫn nằm trọn trong `safeRectPx` cả khi anchor lệch tâm

### 10.2.1 Quy tắc UX

- Chỉ show thông báo auto-fit bằng `Toast.show`
- Không dùng `Alert`
- Thông báo chỉ hiện khi scale thực sự nhỏ hơn `1`
- Không spam toast lặp lại liên tục trong cùng một trạng thái

Message id đề xuất:

- `flat2d_autofit_applied_title`
- `flat2d_autofit_applied_message`

## 10.3 Placement state trong app

Sau khi tính rect canonical và auto-fit, app convert sang preview rect theo scale của preview canvas.

State lưu trong editor ưu tiên canonical data + derived values, không lưu preview-pixel state và không normalize placement sang ratio.

## 11. Export pipeline

## 11.1 Mô hình export

Export theo 2 bước:

1. Tạo snapshot PNG của `FlatArtwork`
2. Dùng hidden export view để compose ảnh cuối cùng

### 11.1.1 Bước 1: snapshot artwork

Skia render framed artwork và export ra PNG local tạm thời.

### 11.1.2 Bước 2: compose scene cuối

Hidden export view dùng:

- `react-native-view-shot`
- `react-native` `Image`
- local/remote image sources

Không dùng `expo-image` trong hidden exporter vì pattern đang dùng ở `riz-app-v2/screens/create-panorama/components/composer-screen.tsx` cho thấy `react-native` `Image` an toàn hơn với `captureRef`.

Exporter chỉ được `captureRef()` sau khi tất cả layer bắt buộc đã load thành công:

- background scene
- artwork snapshot
- foreground occluder nếu scene có foreground

Nếu bất kỳ layer bắt buộc nào load lỗi, exporter phải throw để flow hiển thị lỗi và cho phép retry, không được capture ảnh thiếu lớp.

## 11.2 Export output

Exporter trả về:

```ts
{
  uri: string;
  width: number;
  height: number;
}
```

App điều hướng sang `UploadProjectScreen` với:

```ts
images = JSON.stringify([
  {
    id: "flat2d-export",
    uri,
    width,
    height
  }
])
```

Flow này bám đúng logic parse của `riz-app-v2/screens/upload-project/index.tsx`.

## 11.3 Publish

Sau khi điều hướng sang `UploadProjectScreen`, phần còn lại dùng nguyên flow hiện có:

- user nhập `name`
- user nhập `introduction`
- user chọn `category`
- app upload ảnh
- app tạo `TWO_D` project

## 12. Artwork input và crop

Artwork input reuse `useCropFlow`.

Quy tắc:

- Nếu source là `ph://`, giữ nguyên flow convert hiện có
- Crop hoàn tất rồi mới vào renderer
- Aspect ratio của artwork sau crop là nguồn sự thật cho `printSizeCm`
- Nếu user thay artwork khi đã có `printSizeCm.width` hợp lệ, app giữ nguyên width hiện tại và chỉ tính lại height theo aspect ratio mới

UX đề xuất:

- User nhập một chiều trước
- Chiều còn lại được suy ra theo aspect ratio của ảnh đã crop

Ví dụ:

- user nhập width 60 cm
- app tự suy ra height theo aspect ratio

## 13. API contract

## 13.1 Endpoint mới

Đề xuất endpoint riêng cho 2D scene template:

```txt
GET /scenes/templates/2d
```

Không overload:

- `/room-image`
- `/project/panorama-templates`
- `/scenes/templates`

## 13.2 Response ví dụ

```json
{
  "id": "living_room_straight_01",
  "name": "Orthogonal Living Room",
  "thumbnailUrl": "https://cdn.domain.com/flat2d/thumb.webp",
  "assets": {
    "backgroundUrl": "https://cdn.domain.com/flat2d/bg.webp",
    "foregroundUrl": "https://cdn.domain.com/flat2d/fg.png"
  },
  "sceneSizePx": {
    "width": 2048,
    "height": 1536
  },
  "placement": {
    "anchorPx": {
      "x": 1024,
      "y": 768
    },
    "pixelsPerCm": 10.5,
    "safeRectPx": {
      "x": 624,
      "y": 468,
      "width": 800,
      "height": 600
    },
    "shadow": {
      "opacity": 0.3,
      "offsetXPx": 0,
      "offsetYPx": 15,
      "blurRadiusPx": 24,
      "scale": 1
    }
  }
}
```

## 13.3 Tại sao contract này phù hợp hơn bản cũ

- Có `sceneSizePx` canonical rõ ràng
- Có `safeRectPx` đầy đủ vị trí và kích thước
- Có đủ data để preview và export khớp nhau
- Không khóa app vào raw preview pixel hiện tại

## 14. Điều hướng và i18n

### 14.1 Route

Thêm route:

- `riz-app-v2/app/(protected)/create-flat2d-framing.tsx`

### 14.2 Nút mở flow

Ở `riz-app-v2/screens/create-project/components/create-2d.tsx`, thêm CTA mới:

- `Create with Frame & Room Mockup`

Không bỏ flow cũ.

### 14.3 Localization

Thêm message ids vào:

- `riz-app-v2/integrations/react-intl/locales/en.json`
- `riz-app-v2/integrations/react-intl/locales/vi.json`

Các nhóm text cần có:

- screen title
- field labels
- empty states
- export button
- auto-fit toast
- export errors
- template loading errors

## 15. Quy tắc hiệu năng

- Không snapshot lại artwork trên mỗi thay đổi nhỏ
- Không upload ảnh trung gian lên server trước khi publish project
- Background và foreground preview có thể dùng `expo-image`
- Hidden exporter dùng `react-native` `Image`
- Chỉ giữ một snapshot artwork mới nhất nếu không cần history

## 16. Quy tắc lỗi

### 16.1 Lỗi có thể recover

- Chưa chọn artwork
- Chưa chọn scene
- Chưa chọn frame
- Template load fail
- Export fail

Các lỗi này nên dùng:

- inline validation
- toast
- alert chỉ khi thao tác không thể tiếp tục

### 16.2 Lỗi export

Nếu export fail:

- reset `isExporting`
- không clear draft
- cho phép user thử lại

## 17. Kế hoạch triển khai

### Phase 1: Data và navigation

- tạo route mới
- tạo screen shell
- tạo scene template entity mới
- load template list bằng `react-query`

### Phase 2: Editor state và UI

- tạo `zustand` store
- artwork picker
- frame selector
- scene selector
- preview canvas giữ đúng aspect ratio

### Phase 3: Renderer

- Skia 9-slice renderer cho frame
- tính outer size
- placement canonical
- auto-fit

### Phase 4: Export và publish

- snapshot artwork
- hidden scene exporter
- điều hướng sang `UploadProjectScreen`

### Phase 5: Hardening

- toast UX
- loading states
- retry logic
- preview/export parity testing

## 18. Checklist nghiệm thu

- Preview và ảnh export cuối giống nhau về vị trí và scale
- Auto-fit chỉ kích hoạt khi artwork vượt `safeRectPx`
- Foreground occluder nằm trên artwork trong cả preview lẫn export
- Ảnh từ `ph://` hoạt động đúng
- Route mới không phá flow tạo `TWO_D` cũ
- Project sau publish vẫn xuất hiện bình thường ở feed và creator studio dưới loại `TWO_D`

## 19. Kết luận

Spec mới này giữ đúng hướng sản phẩm của ý tưởng ban đầu, nhưng điều chỉnh lại theo reality của `riz-app-v2`:

- publish qua `TWO_D`
- canonical pixel-only placement trong app (resolver chung preview/export)
- tách entity 2D template khỏi 3D template
- dùng Skia cho frame, nhưng giữ pattern hidden export view cho ảnh scene cuối

Đây là phương án có xác suất triển khai nhanh, ít va chạm codebase hiện tại, và giảm rủi ro tạo thêm một flow debug hoặc một abstraction sai tầng trong app.
