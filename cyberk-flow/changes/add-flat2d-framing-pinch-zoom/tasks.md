## Phase 1 — Contract & Bounds

Mục tiêu: thêm contract `compositionScale` vào store/layout/export theo cách không phá semantics hiện tại của Flat2D.

- [ ] 1_1 Mở rộng store Flat2D với `compositionScale`
  - **Refs**: specs/pinch-zoom/spec.md#pinch-zoom-state-contract, design.md#QĐ-1
  - **Done**: Store có `compositionScale` mặc định `1`, `setCompositionScale`, `resetCompositionScale`; selector/export guard hiện tại không bị vỡ
  - **Test**: `riz-app-v2/store/__tests__/framing-flat2d-store.test.ts` (unit) — cập nhật nếu đã có; nếu chưa có thì thêm case cho state mới
  - **Files**: `riz-app-v2/store/framing-flat2d-store.ts`, `riz-app-v2/store/__tests__/**`
  - **Approach**: Mở rộng state hiện có thay vì tạo store mới. Giữ `printSizeCm` là source dữ liệu vật lý, chỉ thêm một lớp scale riêng cho composition.

- [ ] 1_2 Mở rộng layout contract và snapshot signature
  - **Deps**: 1_1
  - **Refs**: specs/pinch-zoom/spec.md#pinch-zoom-bounds, specs/export-pipeline/spec.md#pinch-zoom-export-parity, design.md#QĐ-3, design.md#QĐ-4
  - **Done**: `resolveFlat2DLayout()` nhận `compositionScale`; có helper clamp/max bound; snapshot signature nối thêm `compositionScale`
  - **Test**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/placement.test.ts` (unit), `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts` (integration)
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/lib/layout-contract.ts`, `riz-app-v2/screens/create-flat2d-framing/lib/placement.ts`, `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/**`, `riz-app-v2/screens/create-flat2d-framing/components/artwork-snapshot.tsx`
  - **Approach**: Áp `compositionScale` vào outer size trước auto-fit, sau đó giữ preview/export cùng dùng một resolver. Signature snapshot phải đổi khi pinch zoom đổi scale đã commit.

## Phase 2 — Preview Interaction

Mục tiêu: thêm pinch zoom mượt, không xung đột với cuộn trang, và có affordance reset rõ ràng.

- [ ] 2_1 Triển khai pinch zoom cho `Flat2DPreview`
  - **Deps**: 1_2
  - **Refs**: specs/pinch-zoom/spec.md#pinch-zoom-preview, specs/pinch-zoom/spec.md#pinch-zoom-scroll-coordination, design.md#QĐ-2, design.md#QĐ-5
  - **Done**: Người dùng pinch 2 ngón để zoom mockup; preview dùng transient scale khi tương tác; commit `compositionScale` khi gesture kết thúc
  - **Test**: Manual verification trên device/emulator; unit test helper clamp nếu cần
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/flat2d-preview.tsx`, `riz-app-v2/screens/create-flat2d-framing/index.tsx`
  - **Approach**: Bọc vùng preview bằng `GestureDetector` + `Gesture.Pinch()`. Dùng callback để screen cha khóa/mở `ScrollView`. Không hỗ trợ pan/rotate trong change này.

- [ ] 2_2 Thêm feedback scale và action reset
  - **Deps**: 2_1
  - **Refs**: specs/pinch-zoom/spec.md#pinch-zoom-feedback
  - **Done**: Khi `compositionScale != 1`, UI hiển thị badge/chip phần trăm scale và action reset về `100%`
  - **Test**: N/A — UI behavior, verify thủ công
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/index.tsx`, `riz-app-v2/integrations/react-intl/locales/en.json`, `riz-app-v2/integrations/react-intl/locales/vi.json`
  - **Approach**: Đặt affordance gần preview hoặc toolbar edit hiện có. Wording ngắn gọn, không chen vào semantics của print size settings.

## Phase 3 — Export Parity & Quality Gate

Mục tiêu: đảm bảo zoom đã commit phản ánh đúng ở snapshot/export/test automation.

- [ ] 3_1 Đồng bộ artwork snapshot và exporter với `compositionScale`
  - **Deps**: 2_2
  - **Refs**: specs/export-pipeline/spec.md#pinch-zoom-export-parity, design.md#QĐ-4
  - **Done**: Snapshot/export đọc đúng `compositionScale`; thay đổi scale tạo cache miss; output upload giữ đúng bố cục đã zoom
  - **Test**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts` (integration)
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/components/artwork-snapshot.tsx`, `riz-app-v2/screens/create-flat2d-framing/components/exporter.tsx`, `riz-app-v2/screens/create-flat2d-framing/index.tsx`
  - **Approach**: Chỉ dùng scale đã commit cho snapshot/export. Preview transient không được rò vào export flow.

- [ ] 3_2 Bổ sung automated tests cho parity, bounds, và cache signature
  - **Deps**: 3_1
  - **Refs**: specs/pinch-zoom/spec.md, specs/export-pipeline/spec.md
  - **Done**: Có test cover case `compositionScale != 1`, clamp bounds, reset về `1`, và cache signature đổi theo scale
  - **Test**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/placement.test.ts` (unit), `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/preview-export-parity.test.ts` (integration)
  - **Files**: `riz-app-v2/screens/create-flat2d-framing/lib/__tests__/**`
  - **Approach**: Mở rộng test suite hiện có thay vì tạo test file rời nếu cùng concern. Tập trung khóa regression ở contract thay vì snapshot pixel test nặng.

- [ ] 3_3 Viết E2E cho pinch zoom + reset trong flow Flat2D
  - **Deps**: 3_2
  - **Refs**: proposal.md#UI-Impact-E2E, specs/pinch-zoom/spec.md#pinch-zoom-feedback
  - **Done**: E2E pass cho flow: mở Flat2D → chọn artwork/frame/scene → pinch zoom → reset → export → vào upload screen
  - **Test**: `riz-app-v2/__tests__/e2e/flat2d-framing-pinch-zoom.test.ts` (e2e)
  - **Files**: `riz-app-v2/__tests__/e2e/flat2d-framing-pinch-zoom.test.ts`
  - **Approach**: Tách case pinch zoom thành test riêng để không làm flow core của change gốc thêm nặng. Nếu framework e2e hiện tại chưa cover gesture pinch tốt, ghi rõ workaround/harness trong task này.
