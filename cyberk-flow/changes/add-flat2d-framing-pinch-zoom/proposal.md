# Đề xuất: add-flat2d-framing-pinch-zoom

## Tại sao

`add-flat2d-framing` đã hoàn tất phần lớn implementation của editor Flat2D. Pinch zoom là một enhancement mới cho trải nghiệm chỉnh bố cục, ảnh hưởng trực tiếp đến preview, snapshot, export, và E2E. Tách nó thành change riêng giúp giữ change gốc gọn, tránh mở lại source-of-truth của feature đã gần hoàn tất, và giúp follow-up này có phạm vi, review, và rollout riêng.

## Khẩu vị (Appetite)

**S ≤ 3 ngày** — Mở rộng một flow hiện có, không thêm màn hình mới, không thêm backend hay dependency mới. Rủi ro chính nằm ở gesture coordination và parity preview/export.

## Phạm vi

### Trong phạm vi

- Pinch zoom 2 ngón trên preview `create-flat2d-framing`
- Scale đồng thời artwork, mat, frame, và shadow quanh anchor của scene
- State mới `compositionScale` tách khỏi `printSizeCm`
- Reset về `100%` và hiển thị feedback scale hiện tại
- Đồng bộ snapshot/export/cache signature với scale đã commit
- Unit tests cho bounds/parity/signature
- E2E cover pinch zoom + reset trong flow Flat2D

### Ngoài phạm vi

- Pan framed mockup
- Rotate artwork hoặc mockup
- Multi-touch edit phức tạp kiểu pinch + pan đồng thời
- Thay đổi backend, API, hoặc shape dữ liệu scene template
- Chỉnh semantics của `printSizeCm`

## Khả năng (Capabilities)

1. **pinch-zoom** — Pinch 2 ngón để scale framed mockup trong preview
2. **pinch-feedback** — Hiển thị phần trăm scale hiện tại và reset về mặc định
3. **export-parity** — Snapshot/export phản ánh chính xác scale đã commit

## Tác động

### Tác động người dùng

- Preview Flat2D trở thành vùng tương tác trực tiếp
- Người dùng có thể tinh chỉnh kích thước tổng thể của mockup bằng pinch mà không phải đổi thông số in
- Có thể quay về `100%` nhanh khi muốn trở lại bố cục mặc định

### Tác động nhà phát triển

- Mở rộng `framing-flat2d-store.ts` và `layout-contract.ts`
- Chỉnh `flat2d-preview.tsx`, `artwork-snapshot.tsx`, `exporter.tsx`, `index.tsx`
- Mở rộng parity/unit tests và E2E flow hiện có

### Tác động hệ thống

- Không cần dependency mới
- Không cần backend
- Không đổi route hay project type

## Rủi ro

**MEDIUM**

| Rủi ro | Mức độ | Giảm thiểu |
|---|---|---|
| Pinch xung đột với cuộn màn hình | MEDIUM | Khóa `ScrollView` trong phiên pinch |
| Preview/export lệch scale | MEDIUM | Một contract `compositionScale` dùng chung |
| Cache snapshot hit sai | LOW | Signature thêm `compositionScale` |
| Gesture re-render nhiều | LOW | Chỉ commit store ở cuối gesture |

## UI Impact & E2E

- **User-visible UI behavior affected?** YES
- **E2E required?** REQUIRED

Đây là thay đổi giao diện người dùng trực tiếp trong một flow đã có:

- Preview nhận gesture pinch mới
- Có affordance reset và feedback scale
- Export phải giữ đúng scale đã tương tác

Test E2E cần cover:

- Mở flow Flat2D đã có sẵn
- Chọn artwork/frame/scene
- Pinch zoom trên preview
- Reset về `100%`
- Export và verify điều hướng sang upload screen vẫn thành công
