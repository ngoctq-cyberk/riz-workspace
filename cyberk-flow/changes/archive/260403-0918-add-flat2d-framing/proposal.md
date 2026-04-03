# Đề xuất: add-flat2d-framing

## Tại sao

Người dùng cần một cách để hình dung artwork 2D trong bối cảnh phòng thực tế với khung trang trí trước khi publish. Hiện tại, app chỉ hỗ trợ upload ảnh thô cho project `TWO_D`. Thêm trình soạn framing & scene mockup giúp tăng giá trị cảm nhận của project đã publish và giúp người xem/mua hiểu artwork trông như thế nào trong không gian thực.

## Khẩu vị (Appetite)

**L ≤ 2 tuần** — Màn hình editor mới với Skia rendering pipeline, zustand store, luồng export, và tích hợp entity/API mới. Độ phức tạp cao do 9-slice frame rendering và yêu cầu khớp preview/export.

## Phạm vi

### Trong phạm vi

- Route mới `create-flat2d-framing` và màn hình editor
- Chọn artwork qua `useCropFlow` hiện có (ảnh đơn)
- Registry frame preset cục bộ với 9-slice rendering qua `@shopify/react-native-skia`
- Entity scene template (`Flat2DSceneTemplate`) với stub data cục bộ (API backend sau)
- Quản lý state editor bằng `zustand` store (không persist)
- Canvas preview giữ đúng tỉ lệ scene với thứ tự render 4 lớp
- Logic auto-fit khi artwork vượt safe rect của scene
- Pipeline export: Skia snapshot → hidden view-shot scene composition
- Bàn giao sang `UploadProjectScreen` hiện có với project type `TWO_D`
- Đa ngôn ngữ (en/vi)
- Nhập kích thước in (cm) với tự động tính chiều thứ hai từ tỉ lệ crop
- Nhập độ rộng mat

### Ngoài phạm vi

- Perspective transform
- Kéo tự do artwork ra ngoài anchor template
- Xoay artwork
- Lưu bản nháp lên backend
- `ProjectType` mới
- Frame presets đồng bộ từ backend
- Triển khai API backend (frontend dùng stubs)

## Khả năng (Capabilities)

1. **artwork-input** — Chọn và crop ảnh artwork, nhập kích thước in (cm)
2. **frame-selection** — Chọn từ các frame preset cục bộ với 9-slice rendering và độ rộng mat
3. **scene-selection** — Chọn từ các scene template (stub data cục bộ trong v1)
4. **preview-canvas** — Preview trực tiếp 4 lớp (background → shadow → framed artwork → foreground)
5. **auto-fit** — Tự động scale artwork khi vượt safe rect của scene
6. **export-pipeline** — Export 2 bước (Skia snapshot + hidden view-shot) → điều hướng đến luồng upload

## Tác động

### Tác động người dùng
- CTA mới trong luồng Tạo 2D: "Tạo với Khung & Mockup Phòng"
- Trải nghiệm editor toàn màn hình mới
- Project sau publish hiển thị bình thường dạng `TWO_D` trong feed/creator studio

### Tác động nhà phát triển
- Entity mới: `flat2d-scene-template` (api, hooks, model)
- Store mới: `framing-flat2d-store.ts`
- Screen mới: `create-flat2d-framing/` với 5+ components
- Pattern Skia rendering mới: 9-slice frame renderer
- ~15-20 file mới

### Tác động hệ thống
- Không cần thay đổi backend cho v1 (stub data cục bộ)
- Không thay đổi database
- Không dependency bên ngoài mới
- Luồng upload/publish hiện có giữ nguyên

## Rủi ro

**MEDIUM**

| Rủi ro | Mức độ | Giảm thiểu |
|---|---|---|
| Độ chính xác 9-slice rendering Skia | MEDIUM | Unit test cô lập + visual regression; nghiên cứu API `drawImageNine` |
| Khớp hình ảnh preview/export | MEDIUM | Module tính toạ độ dùng chung; thứ tự layer cố định |
| Timing API backend | LOW | Stub data cục bộ tách rời frontend khỏi timeline backend |
| Navigation stack đúng đắn | LOW | Theo pattern panorama→upload hiện có (đã fix gần đây) |
| Áp lực bộ nhớ với ảnh lớn | LOW | Chính sách snapshot đơn; dọn file tạm sau export |

## UI Impact & E2E

- **User-visible UI behavior affected?** YES
- **E2E required?** REQUIRED

Đây là luồng giao diện người dùng mới với:
- Route và screen mới
- Editor tương tác với chọn artwork/frame/scene
- Pipeline export tạo ảnh
- Bàn giao điều hướng đến luồng upload hiện có

Test E2E cần cover:
- Mở luồng flat2d framing từ màn hình tạo project
- Hoàn thành luồng đầy đủ: chọn artwork → chọn frame → chọn scene → export → màn hình upload
- Hành vi auto-fit khi artwork vượt safe rect
