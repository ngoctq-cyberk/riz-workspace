<!-- Các task được thực thi tuần tự theo phase gate. -->
<!-- QUY TẮC PHASE GATE: Chỉ bắt đầu phase kế tiếp khi toàn bộ Exit Criteria của phase hiện tại đã pass. -->
<!-- Giữ nguyên task ID để dễ đối chiếu review/findings trước đó. -->

## Phase 1 — Foundation

Mục tiêu: dựng nền tảng domain/store/route/localization để có thể mở screen shell ổn định trước khi làm math và UI chi tiết.

- [x] 1_1 Tạo domain entity `Flat2DSceneTemplate` với model, stub data, và react-query hook
  - **Refs**: specs/scene-selection/spec.md#flat2d-scene-template-entity, specs/scene-selection/spec.md#local-stub-data, specs/scene-selection/spec.md#scene-template-coordinate-system
  - **Xong khi**: `useAllFlat2DSceneTemplates()` trả về danh sách stub templates với toạ độ canonical pixel; TypeScript compile pass
  - **Test**: N/A — stub data + hook wiring, không có logic cần test
  - **Files**: `riz-app-v2/lib/entities/flat2d-scene-template/**`, `riz-app-v2/screens/create-flat2d-framing/constants/stub-scenes.ts`
  - **Cách làm**: Tạo thư mục `flat2d-scene-template/` theo cấu trúc FSD hiện có (`model/index.ts`, `api/get-all.ts`, `hooks/use-all.ts`). Model theo interface `Flat2DSceneTemplate` trong spec. Stub data đặt trong `screens/create-flat2d-framing/constants/stub-scenes.ts`, sau đó `api/get-all.ts` import lại constants này để trả qua `queryFn`. Hook bọc `useQuery` với `queryKey: ['flat2d-scene-templates']`. **KHÔNG normalize sang ratio** — giữ nguyên `anchorPx`, `safeRectPx` dạng pixel canonical (theo QĐ-7 trong design.md).

- [x] 1_2 Tạo zustand store `framing-flat2d-store.ts` cho editor state
  - **Refs**: specs/artwork-input/spec.md, specs/auto-fit/spec.md#auto-fit-calculation, design.md#QĐ-2
  - **Xong khi**: Store export đầy đủ state + actions cho artwork, frame, scene, printSize, mat, autoFit, export; TypeScript compile pass
  - **Test**: `store/__tests__/framing-flat2d-store.test.ts` (unit) — test actions: setArtwork, setFrame, setScene, setPrintSize, setMat, reset
  - **Files**: `riz-app-v2/store/framing-flat2d-store.ts`
  - **Cách làm**: Theo pattern `auth-store.ts` nhưng KHÔNG dùng `persist` middleware. State interface theo `FramingFlat2DDraftState` trong spec (section 7.3). Actions: `setArtwork`, `setFramePresetId`, `setSceneTemplateId`, `setPrintSizeCm`, `setMatWidthCm`, `setAutoFit`, `setExportImage`, `setIsExporting`, `reset`. Derived values (outerSize, autoFitScale) tính bằng selectors, không lưu trùng.

- [x] 1_3 Tạo route mới và screen shell `create-flat2d-framing`
  - **Deps**: 1_2
  - **Refs**: design.md#3.2, specs/artwork-input/spec.md
  - **Xong khi**: Route `/create-flat2d-framing` accessible; screen shell render được với header + placeholder sections
  - **Test**: N/A — screen shell, kiểm tra bằng navigation thủ công
  - **Files**: `riz-app-v2/app/(protected)/create-flat2d-framing.tsx`, `riz-app-v2/screens/create-flat2d-framing/index.tsx`
  - **Cách làm**: Route: thin wrapper import screen (theo pattern `create-panorama.tsx`). Screen: `ScrollView` với header (`HeaderWithTitle`), 3 section placeholders (artwork/frame/scene), preview area, export button. Import zustand store. Dùng `AppSafeAreaView` + dark theme (`bg-black`).

- [x] 1_5 Thêm message IDs localization cho en/vi
  - **Refs**: specs/artwork-input/spec.md, specs/auto-fit/spec.md#auto-fit-toast, specs/export-pipeline/spec.md#export-error-handling
  - **Xong khi**: Tất cả message IDs mới có trong cả `en.json` và `vi.json`; không có key trùng
  - **Test**: N/A — config only
  - **Files**: `riz-app-v2/integrations/react-intl/locales/en.json`, `riz-app-v2/integrations/react-intl/locales/vi.json`
  - **Cách làm**: Thêm các nhóm key prefix `flat2d_`: screen title, field labels, empty states, export button, auto-fit toast, export errors, template loading errors. Ví dụ: `flat2d_screen_title`, `flat2d_artwork_label`, `flat2d_frame_label`, `flat2d_scene_label`, `flat2d_export_btn`, `flat2d_autofit_applied_title`, `flat2d_autofit_applied_message`, `flat2d_export_failed`, `flat2d_no_artwork_selected`, `flat2d_no_scene_selected`.

### Exit Criteria — Phase 1

- Route `/create-flat2d-framing` mở được và screen shell không crash
- `useAllFlat2DSceneTemplates()` trả đúng shape canonical pixel theo spec
- Store actions cơ bản hoạt động đúng
- Locale keys mới không thiếu ở cả `en.json` và `vi.json`
- Kiểm tra tối thiểu: `npx tsc --noEmit`, `bun lint`
- Chỉ sang Phase 2 khi toàn bộ điều kiện trên đã pass

## Phase 2 — Core Math & Renderer

Mục tiêu: hoàn tất frame preset registry, 9-slice renderer, placement math và auto-fit bằng pure logic có thể test độc lập.

- [x] 2_1 Tạo frame preset registry và frame assets
  - **Refs**: specs/frame-selection/spec.md#frame-preset-registry
  - **Xong khi**: `FRAME_PRESETS` array export được với ≥3 presets; mỗi preset có `nineSlice` config hợp lệ, `minInnerWidthPx`, và `minInnerHeightPx`
  - **Test**: N/A — constants only
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/constants/frame-presets.ts`, `riz-app-v2/assets/images/frames/*`
  - **Cách làm**: Tạo 3 frame presets đơn giản: (1) mỏng đen 2cm, (2) classic gỗ 3cm, (3) trắng hiện đại 1.5cm. Mỗi preset cần ảnh PNG 9-sliceable (border rõ ràng). Dùng `require()` cho asset. NineSlice insets xác định bằng pixel thực tế trong ảnh PNG. Mỗi preset PHẢI khai báo thêm `minInnerWidthPx` và `minInnerHeightPx` để chặn trường hợp inner rect nhỏ hơn ngưỡng asset khung có thể render đúng.

- [x] 2_2 Triển khai module 9-slice rendering bằng Skia
  - **Deps**: 2_1
  - **Refs**: specs/frame-selection/spec.md#nine-slice-frame-renderer, design.md#QĐ-3, design.md#9-slice-rendering-flow
  - **Xong khi**: Function `renderNineSlice()` vẽ frame lên Skia canvas đúng — góc không bị stretch, cạnh chỉ stretch 1 chiều
  - **Test**: `screens/create-flat2d-framing/lib/__tests__/nine-slice.test.ts` (unit) — test tính toán src/dst rects
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/lib/nine-slice.ts`
  - **Cách làm**: Hàm nhận input: `skImage` (loaded Skia image), `innerRect` (vùng bên trong frame), `nineSlice` (insets). Tính 9 cặp `{srcRect, dstRect}`. Dùng Skia `canvas.drawImageRect(image, srcRect, dstRect, paint)` cho mỗi vùng. Export cả hàm tính rects riêng (cho unit test) và hàm draw (cho canvas). Lưu ý: load frame image bằng `useImage` hook từ `@shopify/react-native-skia`.

- [x] 2_3 Triển khai module placement và frame-sizing (resolver thống nhất)
  - **Refs**: specs/preview-canvas/spec.md#placement-calculation, specs/auto-fit/spec.md#auto-fit-calculation, design.md#QĐ-7, design.md#outer-size-calculation
  - **Xong khi**: Functions `calcOuterSize`, `calcCanonicalRect`, `calcAutoFit`, `resolveRect` pass unit tests; cùng hàm `resolveRect(rect, scale)` dùng cho cả preview (`previewScale`) và export (`1.0`); `calcAutoFit` xử lý đúng cả trường hợp safe rect lệch tâm quanh anchor
  - **Test**: `screens/create-flat2d-framing/lib/__tests__/placement.test.ts` (unit) — test các case: vừa khít, vượt 1 chiều, vượt 2 chiều, không có frame, safe rect lệch tâm; test preview scale và export scale = 1.0 cho cùng output
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/lib/placement.ts`, `riz-app-v2/screens/create-flat2d-framing/lib/frame-sizing.ts`
  - **Cách làm**: `frame-sizing.ts`: `calcOuterSizeCm(printSizeCm, matWidthCm, frameFaceWidthCm)` → `{ widthCm, heightCm }`. `placement.ts`: `calcCanonicalRect(anchorPx, outerSizePx)` → `{ left, top, width, height }` trong không gian canonical pixel. `calcAutoFit(canonicalRect, anchorPx, safeRectPx)` → `{ scale, applied }`, với scale tính từ khoảng cách `anchorPx` đến bốn cạnh của `safeRectPx` thay vì chỉ dùng `safeRectPx.width/height`. `resolveRect(rect, scale)` → scaled rect — đây là resolver duy nhất, preview gọi với `previewScale`, export gọi với `1.0`. Tất cả pure functions, dễ test.

### Exit Criteria — Phase 2

- `FRAME_PRESETS` có đủ mọi field bắt buộc, gồm `minInnerWidthPx` và `minInnerHeightPx`
- Unit test `nine-slice` pass
- Unit test `placement` pass, bao gồm preview scale và export scale = `1.0`
- Auto-fit logic đúng với các case boundary trong spec, gồm cả safe rect lệch tâm
- Kiểm tra tối thiểu: `npx tsc --noEmit`, `bun lint`
- Chỉ sang Phase 3 khi toàn bộ điều kiện trên đã pass

## Phase 3 — Input & Selection UI

Mục tiêu: hoàn tất đường vào flow mới và các bộ chọn artwork/frame/scene để người dùng thao tác được end-to-end ở mức UI cơ bản.

- [x] 1_4 Thêm CTA mở flow Flat2D Framing vào màn hình Tạo 2D
  - **Deps**: 1_3
  - **Refs**: design.md#3.1, specs/artwork-input/spec.md
  - **Xong khi**: Nút "Tạo với Khung & Mockup" hiển thị trên `Create2DProject`; chạm vào điều hướng đến `/create-flat2d-framing`
  - **Test**: N/A — UI wiring, verify bằng navigation thủ công
  - **Files**: `riz-app-v2/screens/create-project/components/create-2d.tsx`
  - **Cách làm**: Thêm `Pressable` mới phía trên hoặc dưới gallery hiện tại. Text dùng `FormattedMessage` với ID mới. `onPress` gọi `router.navigate("/create-flat2d-framing")`. Giữ nguyên flow cũ không bị ảnh hưởng.

- [x] 3_1 Triển khai component `ArtworkPicker`
  - **Deps**: 1_2, 1_5
  - **Refs**: specs/artwork-input/spec.md
  - **Xong khi**: Chọn artwork qua `expo-image-picker` rồi `useCropFlow`; store cập nhật; print size input hoạt động; giá trị `<= 0` bị từ chối; mat width slider hoạt động
  - **Test**: N/A — UI component, verify bằng thao tác thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/artwork-picker.tsx`
  - **Cách làm**: Section với: (1) ảnh thumbnail artwork hoặc placeholder empty state, (2) `Pressable` gọi `ImagePicker.launchImageLibraryAsync({ mediaTypes: ['images'] })`, lấy uri truyền vào `useCropFlow` với `lockedRatio: false`, (3) `TextInput` cho print width (cm), height auto-calc, chỉ chấp nhận giá trị `> 0` với precision 0.1cm, giữ nguyên giá trị hợp lệ gần nhất nếu user nhập `<= 0` hoặc input không parse được, (4) slider hoặc `TextInput` cho mat width (0-15cm). Update store qua actions. Theo dark theme `bg-black` / `text-white`.

- [x] 3_2 Triển khai component `FrameSelector`
  - **Deps**: 2_1, 1_2, 1_5
  - **Refs**: specs/frame-selection/spec.md#frame-selector-ui
  - **Xong khi**: Hiển thị presets cuộn ngang; chọn/bỏ chọn frame; store cập nhật
  - **Test**: N/A — UI component, verify bằng thao tác thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/frame-selector.tsx`
  - **Cách làm**: `FlatList` horizontal với `FRAME_PRESETS`. Mỗi item: ảnh thumbnail frame + label. Selected state dùng border highlight. Tap toggle: chọn nếu chưa chọn, bỏ nếu đang chọn (set `framePresetId` null). Thêm option đầu tiên "Không khung" với icon.

- [x] 3_3 Triển khai component `SceneSelector`
  - **Deps**: 1_1, 1_2, 1_5
  - **Refs**: specs/scene-selection/spec.md#scene-selector-ui
  - **Xong khi**: Hiển thị thumbnails scene; chọn scene; store cập nhật; preview background thay đổi
  - **Test**: N/A — UI component, verify bằng thao tác thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/scene-selector.tsx`
  - **Cách làm**: `FlatList` horizontal với templates từ `useAllFlat2DSceneTemplates()`. Mỗi item: thumbnail scene. Selected state dùng border. Tap set `sceneTemplateId` trong store. Loading state khi query đang fetch.

### Exit Criteria — Phase 3

- CTA mới mở đúng flow `/create-flat2d-framing`
- Chọn artwork xong crop thành công và store cập nhật đúng
- Chọn/bỏ chọn frame cập nhật state và preview cơ bản
- Chọn scene cập nhật background preview
- Print size và mat width thay đổi được từ UI
- Input kích thước in `<= 0` hoặc không hợp lệ bị reject và không ghi đè state hợp lệ gần nhất
- Smoke test thủ công trên device pass cho toàn bộ flow nhập liệu
- Chỉ sang Phase 4 khi toàn bộ điều kiện trên đã pass

## Phase 4 — Preview, Export & Polish

Mục tiêu: hoàn tất live preview, auto-fit UX, export pipeline canonical-resolution, và handoff sang upload flow.

- [x] 2_4 Triển khai component `Flat2DPreview` — canvas preview 4 lớp
  - **Deps**: 2_2, 2_3, 1_1, 1_2
  - **Refs**: specs/preview-canvas/spec.md, design.md#3.3
  - **Xong khi**: Preview render 4 lớp đúng thứ tự; giữ tỉ lệ scene; framed artwork live Skia; auto-fit hoạt động
  - **Test**: N/A — visual component, verify bằng thao tác thủ công trên device
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/flat2d-preview.tsx`
  - **Cách làm**: Component nhận state từ zustand store selector. Layout: `View` giữ aspect ratio (dùng `onLayout` + `sceneSizePx`). Lớp 1: `expo-image` cho background. Lớp 2: `View` absolute position với shadow style. Lớp 3: `Canvas` (Skia) render framed artwork live — gọi `renderNineSlice` + vẽ artwork bên trong. Lớp 4: `expo-image` cho foreground (nếu có). Calc position qua `placement.ts`.

- [x] 4_1 Triển khai component `Flat2DExporter` — hidden scene exporter
  - **Deps**: 2_4, 2_3
  - **Refs**: specs/export-pipeline/spec.md, design.md#3.3, design.md#QĐ-4
  - **Xong khi**: Exporter tạo PNG cuối cùng ở canonical resolution; handoff đến UploadProjectScreen thành công
  - **Test**: N/A — integration verification bằng export→upload thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/exporter.tsx`
  - **Cách làm**: Theo pattern `WallOverlayExporter`: `forwardRef` + `useImperativeHandle({ export })`. View offscreen `left: -9999` với kích thước `sceneSizePx`. 4 lớp dùng RN `Image` (KHÔNG expo-image). `waitForRenderReady()` poll cho onLoad states. `captureRef()` → PNG → trả `{ uri, width, height }`. Xử lý lỗi: try/catch, reset `isExporting`, toast lỗi.

- [x] 4_2 Kết nối export flow trong screen chính + Skia snapshot
  - **Deps**: 4_1, 2_2, 1_2
  - **Refs**: specs/export-pipeline/spec.md#skia-artwork-snapshot, specs/export-pipeline/spec.md#upload-handoff, specs/export-pipeline/spec.md#snapshot-policy
  - **Xong khi**: Nút export: tạo Skia snapshot ở canonical resolution (KHÔNG dùng preview-scale canvas) → compose scene → `router.navigate({ pathname: "/upload-project", params: { images } })` → project tạo thành công dạng `TWO_D`; snapshot temp cũ được xoá khi bị thay thế và snapshot composition temp được dọn sau handoff thành công
  - **Test**: N/A — full flow test thủ công trên device
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/index.tsx`
  - **Cách làm**: Nút export: validate (artwork + scene required). Bước 1: render framed artwork ở kích thước canonical pixel đầy đủ rồi dùng `makeImageSnapshotAsync` hoặc cơ chế tương đương để tạo snapshot; KHÔNG capture trực tiếp canvas preview đang scale theo màn hình. Cache theo signature. Khi signature đổi, xoá snapshot temp cũ trước khi ghi snapshot mới. Bước 2: gọi `exporterRef.current.export()`. Bước 3: `router.navigate({ pathname: "/upload-project", params: { images: JSON.stringify([{ id: "flat2d-export", uri, width, height }]) } })`. Sau handoff thành công, dọn snapshot temp không còn cần. Error toast dùng `Toast.show`. Auto-fit toast khi scale < 1.

- [x] 4_3 Viết automated parity test cho preview/export
  - **Deps**: 4_2
  - **Refs**: specs/preview-canvas/spec.md#preview-layer-order, specs/export-pipeline/spec.md#hidden-scene-exporter, design.md#4
  - **Xong khi**: Có automated test chứng minh preview và export dùng cùng placement contract và cùng layer order; parity test pass
  - **Test**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts` (integration/lightweight)
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts`, `riz-app-v2/screens/create-flat2d-framing/lib/layer-order.ts`, `riz-app-v2/screens/create-flat2d-framing/components/*`
  - **Cách làm**: Tách shared `SCENE_LAYER_ORDER` constant và shared placement contract để preview/export cùng dùng. Viết test assert cùng input canonical cho ra preview rect = `resolveRect(canonicalRect, previewScale)` và export rect = `resolveRect(canonicalRect, 1.0)`; đồng thời assert layer order giữ nguyên `background -> shadow -> artwork -> foreground`.

- [x] 5_1 Triển khai auto-fit toast UX
  - **Deps**: 2_3, 1_2
  - **Refs**: specs/auto-fit/spec.md#auto-fit-toast
  - **Xong khi**: Toast hiện khi auto-fit kích hoạt; không spam lặp; không hiện khi scale = 1
  - **Test**: N/A — UX behavior, verify bằng thao tác thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/index.tsx`
  - **Cách làm**: `useEffect` watch `autoFitScale` từ store. Nếu `autoFitScale < 1` và khác giá trị trước → `Toast.show({ text1: intl.formatMessage({ id: "flat2d_autofit_applied_title" }), text2: ... })`. Dùng `useRef` lưu scale cũ để so sánh, tránh spam.

- [x] 5_2 Thêm loading states, error handling, validation UI
  - **Deps**: 4_2
  - **Refs**: specs/export-pipeline/spec.md#export-error-handling, specs/artwork-input/spec.md#print-size-input
  - **Xong khi**: Loading spinner khi export; error toast khi fail; nút export disabled khi thiếu input hoặc `printSizeCm` không hợp lệ; retry hoạt động
  - **Test**: N/A — UI polish
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/index.tsx`, `riz-app-v2/screens/create-flat2d-framing/components/*.tsx`
  - **Cách làm**: Nút export: disabled + `ActivityIndicator` khi `isExporting`. Validation: check artwork + scene trước khi cho export; chặn `printSizeCm` không hợp lệ (`<= 0`, null, NaN). Error: try/catch trong export flow, reset `isExporting`, `Toast.show` error. Empty states cho mỗi section khi chưa chọn.

### Exit Criteria — Phase 4

- Preview giữ đúng aspect ratio, đúng layer order, và render live bằng Skia
- Auto-fit áp dụng đúng khi artwork vượt `safeRectPx`
- Toast auto-fit không spam lặp
- Snapshot được tạo ở canonical resolution, không dùng preview-scale canvas
- Snapshot temp cũ được cleanup đúng khi signature đổi và sau handoff thành công
- Export compose đúng và handoff sang `UploadProjectScreen` bằng object-form `router.navigate({ pathname, params })`
- Automated parity test cho preview/export pass
- Lỗi export không làm mất draft và có thể retry
- Smoke test thủ công export → upload pass trên device
- Kiểm tra tối thiểu: `npx tsc --noEmit`, `bun lint`
- Chỉ sang Phase 5 khi toàn bộ điều kiện trên đã pass

## Phase 5 — E2E Gate & Sync Docs

Mục tiêu: khóa chất lượng bằng E2E automation và đồng bộ tài liệu sau khi implementation ổn định.

- [ ] 5_3 Viết E2E test cho luồng Flat2D Framing
  - **Deps**: 5_2
  - **Refs**: proposal.md#UI-Impact-E2E, specs/export-pipeline/spec.md#upload-handoff, specs/auto-fit/spec.md#auto-fit-calculation
  - **Xong khi**: E2E test pass: mở flow → chọn artwork → chọn frame → chọn scene → export → verify navigation đến upload screen; có ít nhất 1 case cover auto-fit khi artwork vượt safe rect
  - **Test**: `riz-app-v2/__tests__/e2e/flat2d-framing.test.ts` (e2e)
  - **Files**: `riz-app-v2/__tests__/e2e/flat2d-framing.test.ts`
  - **Cách làm**: Dùng Detox hoặc Maestro. Nếu repo chưa có framework phù hợp, task này BAO GỒM việc dựng hạ tầng tối thiểu cần thiết để chạy E2E cho flow mới; KHÔNG thay thế bằng test plan manual. Test phải cover tối thiểu: (1) flow chính launch app → navigate đến create project → tap CTA "Tạo với Khung" → chọn artwork → chọn frame → chọn scene → tap export → verify navigate đến upload screen; (2) case artwork vượt `safeRectPx` để xác nhận auto-fit được áp dụng và flow vẫn export thành công. **Trạng thái hiện tại**: user chủ động waive task này để unblock archive prep; residual risk là không có mobile E2E automation cho flow này trong change gốc.

- [x] 5_4 Cập nhật docs dự án nếu design khác biệt
  - **Deps**: 4_2
  - **Refs**: design.md
  - **Xong khi**: `docs/SPEC-framing-flat2d-v2.vi.md` đồng bộ với quyết định thực tế (nếu có divergence)
  - **Test**: N/A — doc only
  - **Files**: `docs/SPEC-framing-flat2d-v2.vi.md`
  - **Cách làm**: Review lại spec gốc so với `design.md`. Nếu có quyết định khác (ví dụ: `drawImageRect` thay vì `drawImageNine`, hoặc canonical pixel-only thay vì normalize ratio), cập nhật spec gốc để đồng bộ. Ghi chú phần nào đã triển khai khác và lý do.

### Exit Criteria — Phase 5

- E2E flow chính pass
- E2E case auto-fit pass
- Tài liệu gốc được đồng bộ với quyết định thực tế
- Chỉ khi Gate 5 pass thì implementation mới được coi là hoàn chỉnh
