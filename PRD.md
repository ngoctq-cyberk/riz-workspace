# PRD — Nền tảng RIZ

> Tài liệu Yêu cầu Sản phẩm cho nền tảng RIZ.
>
> **Đối tượng đọc:** Product, Design, Marketing, Vận hành và Phát triển sản phẩm.
> **Phạm vi:** mô tả các nghiệp vụ đã thực sự có trong sản phẩm hiện tại. Những phần chưa hoàn thiện được nêu rõ ở mục riêng.
> **Cập nhật:** 2026-05-04

---

## Mục lục

- [1. Giới thiệu](#1-giới-thiệu)
- [2. Vai trò người dùng](#2-vai-trò-người-dùng)
- [3. Đăng ký & Đăng nhập](#3-đăng-ký--đăng-nhập)
- [4. Hồ sơ cá nhân](#4-hồ-sơ-cá-nhân)
- [5. Trang khám phá tác phẩm](#5-trang-khám-phá-tác-phẩm)
- [6. Cộng đồng — Bài viết & Bình luận](#6-cộng-đồng--bài-viết--bình-luận)
- [7. Tương tác xã hội](#7-tương-tác-xã-hội)
- [8. Tác phẩm sáng tạo](#8-tác-phẩm-sáng-tạo)
- [9. Tác phẩm 2D](#9-tác-phẩm-2d)
- [10. Tác phẩm 3D](#10-tác-phẩm-3d)
- [11. Tác phẩm Panorama 360°](#11-tác-phẩm-panorama-360)
- [12. Cuộc thi](#12-cuộc-thi)
- [13. Bảng xếp hạng Trending](#13-bảng-xếp-hạng-trending)
- [14. Tin nhắn riêng](#14-tin-nhắn-riêng)
- [15. Thông báo](#15-thông-báo)
- [16. Trợ lý AI — Ước lượng chi phí](#16-trợ-lý-ai--ước-lượng-chi-phí)
- [17. Gói hội viên Premium](#17-gói-hội-viên-premium)
- [18. Cài đặt & Đa ngôn ngữ](#18-cài-đặt--đa-ngôn-ngữ)
- [19. Khu vực Quản trị](#19-khu-vực-quản-trị)
- [20. Ai được làm gì](#20-ai-được-làm-gì)
- [21. Giới hạn sử dụng](#21-giới-hạn-sử-dụng)
- [22. Phần đang hoàn thiện](#22-phần-đang-hoàn-thiện)

---

## 1. Giới thiệu

**RIZ** là nền tảng xã hội sáng tạo dành cho **nhiếp ảnh gia, kiến trúc sư và designer**. Người dùng có thể:

- Đăng tải tác phẩm dưới ba định dạng: **2D**, **3D** và **Panorama 360°**.
- Tham gia các **cuộc thi** chủ đề do đội ngũ vận hành RIZ phát động.
- Tương tác với cộng đồng: thích, lưu, theo dõi, bình luận, nhắn tin riêng.
- Sử dụng **trợ lý AI** để ước lượng chi phí thi công nội thất.
- Đăng ký gói **hội viên Premium** để mở khoá hạn mức trợ lý AI.

Sản phẩm gồm **2 ứng dụng**:

| Sản phẩm                       | Đối tượng        | Mục đích                                                                |
| ------------------------------ | ---------------- | ----------------------------------------------------------------------- |
| **RIZ Mobile** (iOS & Android) | Người dùng cuối  | Sáng tạo, khám phá, tương tác                                           |
| **RIZ Admin** (web)            | Đội ngũ vận hành | Kiểm duyệt nội dung, tổ chức cuộc thi, biên tập trending, gửi thông báo |

---

## 2. Vai trò người dùng

| Vai trò                   | Mô tả                                                                                             | Sử dụng          |
| ------------------------- | ------------------------------------------------------------------------------------------------- | ---------------- |
| **Người dùng**            | Người dùng cuối: sáng tạo và tương tác                                                            | Ứng dụng di động |
| **Quản trị viên**         | Đội vận hành: kiểm duyệt, tổ chức cuộc thi, biên tập trending, gửi thông báo                      | Khu vực quản trị |
| **Quản trị viên cấp cao** | Có toàn bộ quyền của Quản trị viên, đồng thời được cấp hoặc thu hồi quyền quản trị cho người khác | Khu vực quản trị |

> Người dùng thường không thể tự nâng quyền quản trị. Việc cấp quyền chỉ do Quản trị viên cấp cao thực hiện.

---

## 3. Đăng ký & Đăng nhập

### 3.1 Trên ứng dụng di động

**Đăng nhập bằng email + mã xác thực một lần (OTP)**

1. Người dùng nhập email.
2. Hệ thống gửi mã xác thực gồm **6 chữ số** đến hộp thư của người dùng. Mã có hiệu lực **10 phút**.
3. Người dùng nhập mã để đăng nhập. Hệ thống tự tạo hồ sơ nếu là lần đầu.
4. Người dùng có thể nhấn **Bỏ qua** ở màn hình nhập email để vào trực tiếp trang khám phá mà không cần đăng nhập. Khi đó các thao tác cần đăng nhập (như đăng tác phẩm, thích, lưu, bình luận, nhắn tin) sẽ không khả dụng.

**Quy tắc gửi mã xác thực để chống lạm dụng:**

- Mỗi email được nhận tối đa **1 mã/phút** và **10 mã/giờ**.
- Mỗi địa điểm truy cập (theo địa chỉ mạng) gửi tối đa **100 mã/ngày**.

### 3.2 Trên khu vực quản trị

- Đăng nhập bằng **email + mật khẩu** (mật khẩu tối thiểu 6 ký tự).
- Chỉ chấp nhận tài khoản có quyền Quản trị viên hoặc Quản trị viên cấp cao. Tài khoản người dùng thường bị từ chối.
- Hỗ trợ **gửi lại email xác nhận tài khoản** và **đặt lại mật khẩu** qua email.

### 3.3 Phiên đăng nhập

- Hệ thống tự duy trì trạng thái đăng nhập, người dùng không cần đăng nhập lại mỗi lần mở ứng dụng.
- Khi người dùng đổi mật khẩu, mọi thiết bị đang đăng nhập trước đó sẽ bị đăng xuất.
- Hỗ trợ **đăng xuất** và **xoá tài khoản**. Khi xoá tài khoản, hồ sơ và nội dung cá nhân được vô hiệu hoá.

---

## 4. Hồ sơ cá nhân

### 4.1 Thông tin hồ sơ

Mỗi người dùng có một hồ sơ với các thông tin:

- Ảnh đại diện và ảnh bìa.
- Tên hiển thị, email liên hệ, số điện thoại.
- Phần "Giới thiệu bản thân" và tệp đính kèm portfolio.
- Số người theo dõi và số người đang được theo dõi.

### 4.2 Trang hồ sơ trong ứng dụng

**Khi xem hồ sơ của chính mình** — có 3 mục:

- **Tác phẩm** — toàn bộ tác phẩm đã đăng, hiển thị dạng lưới.
- **Giới thiệu** — phần giới thiệu bản thân, đường dẫn liên kết, thông tin liên hệ.
- **Đã lưu** — các bài viết và tác phẩm đã được người dùng đánh dấu lưu lại. Mục này chỉ chính chủ hồ sơ thấy.

**Khi xem hồ sơ người khác** — có 2 mục: Tác phẩm và Giới thiệu. Có thêm nút **Theo dõi / Bỏ theo dõi** và **Nhắn tin**.

### 4.3 Chỉnh sửa hồ sơ

Người dùng có thể cập nhật:

- Ảnh đại diện, ảnh bìa, tệp đính kèm portfolio (mỗi loại ở một màn riêng).
- Tên, số điện thoại, địa chỉ.
- Phần giới thiệu chi tiết (About).

### 4.4 Xoá hồ sơ

- Người dùng chủ động xoá hồ sơ trong **Cài đặt → Xoá tài khoản**.
- Hệ thống yêu cầu xác nhận. Sau khi xác nhận, hồ sơ và toàn bộ nội dung liên quan bị vô hiệu hoá.

---

## 5. Trang khám phá tác phẩm

### 5.1 Bố cục

- Hiển thị dạng **lưới linh hoạt** — chiều cao mỗi thẻ tác phẩm tự co giãn theo ảnh.
- Đầu trang luôn cố định: thanh danh mục, ô tìm kiếm và bộ lọc phụ.
- Khi cuộn xuống đủ xa, xuất hiện nút **quay về đầu trang**.

### 5.2 Danh mục chính

Có ba nhóm cố định:

- **Nhiếp ảnh** — tác phẩm nhiếp ảnh.
- **Kiến trúc & Nội thất** — tác phẩm kiến trúc, thiết kế nội thất.
- **Người mẫu** — chân dung, mẫu, nhân vật.

### 5.3 Bộ lọc phụ

- Trong nhóm **Kiến trúc & Nội thất** có thêm bộ lọc theo **loại không gian** (ví dụ: phòng khách, phòng ngủ, văn phòng…).
- **Tìm kiếm**: gõ từ khoá để lọc theo tên hoặc mô tả tác phẩm.

### 5.4 Thẻ tác phẩm

Mỗi thẻ tác phẩm hiển thị:

- Ảnh tác phẩm.
- Ảnh đại diện và tên tác giả.
- Số lượt thích và số lượt bình luận.
- Nút lưu tác phaẩm.

### 5.5 Cuộc thi nổi bật

Quản trị viên có thể chọn ghim một cuộc thi nổi bật ở đầu trang khám phá để người dùng dễ thấy.

### 5.6 Trang nghệ thuật (Art Feed)

Bên cạnh trang khám phá chính, còn có một trang phụ dành riêng để giới thiệu các tác phẩm nghệ thuật, sử dụng cùng cơ chế hiển thị.

---

## 6. Cộng đồng — Bài viết & Bình luận

### 6.1 Bài viết

Khu vực Cộng đồng cho phép người dùng đăng các bài viết ngắn gồm **văn bản kèm ảnh hoặc video**.

**Đăng bài:**

- Có ô đăng bài nhanh ngay đầu trang Cộng đồng.
- Có thể đính kèm tới **5 ảnh hoặc video**.
- Người dùng chọn **chủ đề bài viết** (ví dụ: chia sẻ kinh nghiệm, hỏi đáp, giới thiệu dự án…).

**Hiển thị bài viết:**

- Mỗi bài viết hiện ảnh đại diện, tên người đăng, nội dung, các tệp đính kèm (1 đến 5 tệp với 5 cách bố trí khác nhau tuỳ số lượng).
- Hiển thị số lượt thích và số bình luận.
- Có **bộ lọc theo chủ đề** để người đọc xem nội dung mình quan tâm.

**Đăng bài khi mất mạng:**

- Khi không có kết nối, bài viết được lưu tạm trên thiết bị và **tự động đăng lại** ngay khi mạng phục hồi.
- Bài viết đang chờ đăng được hiển thị trạng thái rõ ràng.

**Vòng đời kiểm duyệt bài viết:**

```
Người dùng đăng bài  →  CHỜ DUYỆT
                          ├─ Quản trị viên duyệt và gắn chủ đề  →  ĐÃ XUẤT BẢN
                          └─ Quản trị viên từ chối                →  BỊ TỪ CHỐI
ĐÃ XUẤT BẢN  →  Quản trị viên có thể chuyển sang LƯU TRỮ hoặc CẤM HIỂN THỊ
```

### 6.2 Bình luận

**Điểm khác biệt quan trọng của RIZ:** người dùng **không tự gõ bình luận**. Hệ thống cung cấp một **thư viện câu bình luận mẫu** do đội vận hành biên soạn (ví dụ: "Tuyệt vời!", "Bố cục rất ấn tượng"). Người dùng chọn câu phù hợp để bình luận.

- Mỗi câu bình luận mẫu có nội dung, thứ tự hiển thị và trạng thái bật/tắt do đội vận hành quản lý.
- Có thể bình luận trên bài viết, tác phẩm, hoặc từng ảnh trong tác phẩm.
- **Bình luận chỉ một cấp** — không có chức năng trả lời bình luận.

> Cách làm này giúp giữ không khí cộng đồng tích cực, hạn chế tranh cãi và bình luận tiêu cực.

---

## 7. Tương tác xã hội

### 7.1 Thích

- Chỉ có **một loại biểu cảm: Thích**.
- Mỗi người chỉ thích được 1 lần cho cùng một nội dung; nhấn thích lần nữa sẽ bỏ thích.
- Có thể xem **danh sách những người đã thích** một bài viết hoặc tác phẩm.

### 7.2 Lưu

- Lưu lại bài viết, tác phẩm hoặc từng ảnh yêu thích để xem lại sau.
- Trong trang hồ sơ cá nhân có mục **Đã lưu** hiển thị toàn bộ nội dung đã lưu.
- Có thể lọc nội dung đã lưu theo loại (bài viết, tác phẩm, ảnh).

### 7.3 Theo dõi

- Theo dõi hoặc bỏ theo dõi người dùng khác.
- Trang hồ sơ hiển thị **số người theo dõi** và **số người đang theo dõi**.
- Có thể xem **danh sách người theo dõi** và **danh sách đang theo dõi** của bất kỳ người dùng nào.

---

## 8. Tác phẩm sáng tạo

Tác phẩm là **đơn vị nội dung chính** của RIZ. Có 3 loại:

- **2D** — bộ ảnh phẳng (nhiếp ảnh, tranh minh hoạ…).
- **3D** — không gian 3 chiều có thể tương tác.
- **Panorama 360°** — không gian 360° với ảnh dán lên 6 mặt tường ảo.

### 8.1 Thông tin chung của một tác phẩm

- Tên tác phẩm, lời giới thiệu, danh mục và danh mục con.
- Tệp đính kèm phụ trợ (ví dụ tệp PDF portfolio).
- Trạng thái kiểm duyệt.
- **Tuỳ chọn cho phép người khác dùng làm khuôn mẫu** — nếu tác giả bật, các người dùng khác có thể đánh dấu lưu tác phẩm vào kho khuôn mẫu cá nhân để tham khảo và lấy cảm hứng.

### 8.2 Vòng đời & kiểm duyệt

Tác phẩm **không cần duyệt trước**. Khi tác giả tạo xong, tác phẩm được **xuất bản ngay** và xuất hiện trên trang khám phá lập tức.

Đội vận hành **kiểm duyệt sau** — chỉ can thiệp khi nội dung vi phạm:

```
Tác giả tạo  →  ĐÃ XUẤT BẢN  (mặc định, hiển thị ngay)

Quản trị viên có thể chuyển trạng thái:
  ĐÃ XUẤT BẢN  →  BỊ TỪ CHỐI (kèm lý do)
  ĐÃ XUẤT BẢN  →  LƯU TRỮ
  ĐÃ XUẤT BẢN  →  CẤM HIỂN THỊ
```

> **Phân biệt với Bài viết Cộng đồng (mục 6.1):** Bài viết Cộng đồng **phải qua duyệt trước khi hiển thị**, còn Tác phẩm thì **xuất bản ngay rồi mới kiểm duyệt sau**.

### 8.3 Quản lý tác phẩm

Trong **Khu vực sáng tạo** trên ứng dụng, tác giả có thể:

- Xem danh sách tác phẩm của mình theo từng nhóm: **2D / 3D / Panorama 360°**.
- Chỉnh sửa thông tin tác phẩm.
- Sắp xếp lại thứ tự ảnh, thêm hoặc xoá ảnh.
- Xoá tác phẩm.

### 8.4 Khuôn mẫu đã lưu

Người dùng có thể truy cập:

- **Khuôn mẫu đã lưu** — các tác phẩm được tác giả gốc cho phép dùng làm khuôn mẫu mà người dùng đã đánh dấu lưu.
- **Ảnh khuôn mẫu đã lưu** — danh sách từng ảnh khuôn mẫu đã lưu.
- **Khuôn mẫu Panorama** — các mẫu Panorama do đội vận hành chuẩn bị sẵn.

---

## 9. Tác phẩm 2D

### 9.1 Quy trình tạo

1. Mở thư viện ảnh trên thiết bị, **chọn tối đa 5 ảnh**.
2. Xem trước theo dạng trượt, có thể loại bỏ ảnh không cần.
3. **Cắt khung** từng ảnh.
4. Điền thông tin: tên, mô tả, danh mục, **vật liệu sử dụng** (chất liệu, vật liệu xuất hiện trong tác phẩm).
5. Đăng tải → tác phẩm **được xuất bản ngay** và hiển thị trên trang khám phá.

### 9.2 Chỉnh sửa sau khi đăng

- Chỉnh sửa tên, mô tả, danh mục, vật liệu.
- Thêm, xoá hoặc sắp xếp lại ảnh.

---

## 10. Tác phẩm 3D

### 10.1 Khái niệm

Tác phẩm 3D dựa trên các **không gian mẫu** do đội vận hành chuẩn bị sẵn (ví dụ: phòng khách, phòng ngủ, văn phòng). Mỗi không gian mẫu có:

- Khung cảnh 3D nền.
- Các **vị trí trống** đã định sẵn để người dùng "thả" nội dung của mình vào. Có hai loại:
  - **Vị trí dán ảnh** — chẳng hạn ô tranh treo tường.
  - **Vị trí đặt vật thể** — chẳng hạn chỗ đặt ghế, đèn.

Đội vận hành cũng quản lý **thư viện vật thể 3D** — mỗi vật thể có ảnh đại diện, danh mục, kích thước.

### 10.2 Quy trình tạo

Quy trình gồm 3 bước:

1. **Chọn không gian mẫu** — duyệt và chọn 1 không gian.
2. **Trang trí không gian:**
   - Chạm vào vị trí dán ảnh để tải ảnh từ thiết bị lên.
   - Chạm vào vị trí đặt vật thể để mở thư viện và chọn vật thể phù hợp.
   - Có thể xoá vật thể đã đặt.
   - Người dùng có thể xoay và phóng to camera trong khung cảnh để xem từ nhiều góc.
3. **Lưu lại** — hệ thống tự chụp ảnh đại diện cho tác phẩm. Người dùng nhập tên và lời giới thiệu để hoàn tất.

### 10.3 Hiển thị

Tác phẩm 3D đã lưu giữ lại đầy đủ không gian mẫu gốc cùng với mọi điều chỉnh của người dùng (vật thể đã đặt, ảnh đã dán). Có thể mở lại để xem bất cứ lúc nào.

---

## 11. Tác phẩm Panorama 360°

### 11.1 Quy trình tạo (4 bước)

1. **Chọn không gian mẫu** — đội vận hành chuẩn bị sẵn các mẫu Panorama.
2. **Chọn ảnh** từ thư viện thiết bị.
3. **Đặt ảnh lên tường ảo** — kéo thả và phóng to để đặt ảnh đúng vị trí, đúng tỉ lệ trên một mặt tường.
4. **Điền thông tin** — tên, mô tả, danh mục → hoàn tất tạo tác phẩm Panorama 360°.

Hệ thống lưu **6 mặt tường** theo quy ước: trước, sau, trái, phải, trần và sàn. Mỗi mặt có thể gắn 1 ảnh.

### 11.2 Xem panorama

Trong ứng dụng đã có chế độ xem Panorama 360°. Trình xem nâng cao đang được hoàn thiện (xem mục 22).

### 11.3 Quản lý từ khu vực quản trị

Đội vận hành có thể tạo, sửa, xoá các tác phẩm Panorama mẫu để dùng làm khuôn cho người dùng hoặc làm nội dung biên tập (xem mục 19.4).

---

## 12. Cuộc thi

### 12.1 Vòng đời cuộc thi

```
Đội vận hành tạo (BẢN NHÁP)  →  Đội vận hành kích hoạt (ĐANG DIỄN RA)  →  Đội vận hành lưu trữ (ĐÃ KẾT THÚC)
```

Khi cuộc thi đang diễn ra, người đủ điều kiện có thể nộp bài, đội vận hành duyệt từng bài dự thi.

### 12.2 Thông tin một cuộc thi

- Tiêu đề, mô tả, **mục tiêu**, **yêu cầu**, **luật lệ**.
- Ảnh bìa.
- Ngày bắt đầu, ngày kết thúc.
- Danh mục và danh mục con.
- **Các hạng mục giải thưởng** — đội vận hành có thể tạo nhiều giải khác nhau cho một cuộc thi.

### 12.3 Quyền tham gia

Một cuộc thi có thể giới hạn người tham gia bằng **2 cách**:

- **Danh sách thành viên cụ thể** — đội vận hành thêm trực tiếp từng email vào danh sách.
- **Quy tắc theo tên miền email** — đội vận hành khai báo tên miền (ví dụ `@company.com`). Mọi người dùng có email thuộc tên miền này đều tự động đủ điều kiện nộp bài.

### 12.4 Trải nghiệm người dùng

**Danh sách cuộc thi:**

- Có hai mục: **Đang diễn ra** và **Đã kết thúc**.
- Mỗi cuộc thi hiển thị: ảnh bìa, tiêu đề, hạn nộp, trạng thái.

**Trang chi tiết cuộc thi:**

- Banner và tiêu đề ở đầu trang.
- Mục **Thông tin** — mục tiêu, yêu cầu, luật lệ, giải thưởng.
- Mục **Bài dự thi** — lưới các bài dự thi đã được duyệt.
- Hiển thị bài dự thi của chính người dùng nếu họ đã nộp.

**Nộp bài dự thi:**

- Biểu mẫu gồm: tiêu đề, mô tả, **ghi chú về cách tiếp cận đề tài**, **phần phản hồi chủ đề**.
- Tệp đính kèm:
  - Tối đa **5 ảnh** (có thể cắt khung).
  - Tối đa **5 video**, mỗi video không quá **120 giây** và **250MB**.
- Sau khi nộp, hiển thị thông báo nộp bài thành công.
- Có thể **chỉnh sửa lại bài dự thi** trước khi đội vận hành duyệt.

### 12.5 Vòng đời bài dự thi

```
Người dùng nộp  →  CHỜ DUYỆT
                   ├─ Đội vận hành duyệt   →  ĐÃ DUYỆT  (hiện trên trang cuộc thi và Cộng đồng)
                   └─ Đội vận hành từ chối →  BỊ TỪ CHỐI  (kèm lý do)
```

---

## 13. Bảng xếp hạng Trending

### 13.1 Khái niệm

Trending là **bảng xếp hạng các tác phẩm nổi bật** do đội vận hành biên tập theo từng cặp **danh mục — danh mục con**.

Mỗi cặp danh mục có thể tồn tại **nhiều phiên bản bảng xếp hạng**. Tại mỗi thời điểm chỉ có **1 phiên bản đang được công bố** — đây là phiên bản người dùng nhìn thấy trong ứng dụng.

### 13.2 Đội vận hành làm gì

Đội vận hành có thể:

- Xem **trang tổng quan** với số liệu thống kê từng danh mục.
- Trong trang chi tiết một danh mục: xem các phiên bản đã có, tạo phiên bản mới, **sao chép** một phiên bản để tạo phiên bản mới từ bản cũ, **xoá phiên bản**.
- Mở chi tiết một phiên bản để **chỉnh sửa danh sách tác phẩm**:
  - Thêm hoặc xoá tác phẩm.
  - **Kéo thả để sắp xếp lại thứ tự**.
- **Công bố** một phiên bản → phiên bản đó trở thành bản người dùng thấy. Các phiên bản còn lại tự chuyển sang trạng thái không hoạt động.

### 13.3 Người dùng thấy gì

- Trang khám phá ở mỗi danh mục hiển thị danh sách lấy từ phiên bản đang được công bố.
- Người dùng không thấy khái niệm "phiên bản" — họ chỉ thấy danh sách tác phẩm.

---

## 14. Tin nhắn riêng

### 14.1 Phạm vi

- Nhắn tin **một-đối-một** (chưa có nhóm chat).
- Có thể bắt đầu cuộc trò chuyện từ trang hồ sơ của người dùng khác.

### 14.2 Loại tin nhắn

| Loại                   | Mô tả                                                                  |
| ---------------------- | ---------------------------------------------------------------------- |
| **Văn bản**            | Tin nhắn dạng chữ                                                      |
| **Ảnh**                | Đính kèm ảnh                                                           |
| **Video**              | Đính kèm video                                                         |
| **Tệp**                | Đính kèm tài liệu                                                      |
| **Trích dẫn tác phẩm** | Gửi card xem trước của một tác phẩm trong cuộc trò chuyện              |
| **Tin nhắn ghi âm**    | Trình phát đã sẵn sàng; chức năng ghi âm trên ứng dụng đang hoàn thiện |

### 14.3 Trải nghiệm

- **Hộp thư** — danh sách các cuộc trò chuyện kèm ảnh đại diện, tên, tin nhắn cuối, số tin nhắn chưa đọc.
- **Màn hình trò chuyện:**
  - Mỗi loại tin nhắn có cách hiển thị riêng.
  - Khi đối phương đang gõ, người nhận thấy chỉ báo **"đang gõ"**.
  - Chạm vào ảnh hoặc video sẽ mở chế độ xem toàn màn hình.
  - Tự động cuộn xuống tin nhắn mới nhất; cuộn lên trên sẽ tải các tin nhắn cũ hơn.
- **Tức thời:** tin nhắn mới và trạng thái "đang gõ" được đẩy đến đối phương ngay lập tức.
- **Tắt thông báo** một cuộc trò chuyện để ngừng nhận thông báo đẩy từ cuộc đó.
- **Đánh dấu đã đọc** một cuộc trò chuyện. Trên hộp thư có hiển thị **tổng số tin chưa đọc**.

### 14.4 Giới hạn chống spam

- Mỗi người dùng được gửi tối đa **60 tin nhắn/phút** và **500 tin nhắn/giờ**.

---

## 15. Thông báo

### 15.1 Kênh thông báo

| Kênh               | Mô tả                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| **Trong ứng dụng** | Trung tâm thông báo trong ứng dụng, kèm chỉ báo số chưa đọc                                     |
| **Thông báo đẩy**  | Đẩy đến điện thoại iOS / Android                                                                |
| **Email**          | Email cho các sự kiện liên quan tài khoản (xác nhận tài khoản, đặt lại mật khẩu) và các bản tin |

### 15.2 Loại thông báo

- **Hệ thống** — thông báo chung từ đội vận hành.
- **Bài viết / tác phẩm mới** từ người mà bạn đang theo dõi.
- **Có người thích, bình luận, hoặc theo dõi bạn**.
- **Có tin nhắn mới**.
- **Bài viết của bạn được duyệt hoặc bị từ chối**.

### 15.3 Cài đặt thông báo

Người dùng có thể bật/tắt từng nhóm:

- **Trong ứng dụng:** tương tác (thích, bình luận), người theo dõi mới, tin nhắn, hoạt động cộng đồng, đơn hàng / hội viên.
- **Email:** cập nhật mua hàng, bản tin định kỳ.
- **Ngôn ngữ thông báo:** Tiếng Việt hoặc Tiếng Anh.

Ứng dụng cũng kiểm tra quyền nhận thông báo đẩy của hệ điều hành. Nếu người dùng đã tắt ở cấp hệ điều hành, ứng dụng sẽ hướng dẫn vào phần Cài đặt của điện thoại để bật lại.

### 15.4 Đa ngôn ngữ

Mỗi thông báo có **hai ngôn ngữ song hành** (Tiếng Việt và Tiếng Anh) cùng các phần điền động (tên người thích, tiêu đề bài viết…). Khi gửi, hệ thống tự động chọn ngôn ngữ phù hợp với từng người nhận.

### 15.5 Thông báo hàng loạt

Đội vận hành có thể gửi thông báo hàng loạt **đa ngôn ngữ** đến toàn bộ người dùng hoặc một danh sách cụ thể. Hệ thống vừa lưu thông báo trong ứng dụng vừa đẩy thông báo đến điện thoại, đồng thời tôn trọng cài đặt riêng của từng người nhận.

---

## 16. Trợ lý AI — Ước lượng chi phí

### 16.1 Bộ chọn trợ lý

Trong ứng dụng có màn hình **Chọn trợ lý AI** dạng trượt ngang. Hiện có:

- **Trợ lý ước lượng chi phí** — đã hoạt động.
- **Trợ lý phong cách** — đã giới thiệu trong giao diện chọn trợ lý nhưng **chưa mở để sử dụng**.

### 16.2 Trợ lý ước lượng chi phí

**Cách dùng:**

1. Người dùng mở phòng chat với trợ lý.
2. Tải lên ảnh **mặt bằng** hoặc ảnh **panorama** không gian.
3. AI phân tích và trả về số liệu: số phòng, số cửa, số cửa sổ, diện tích.
4. Người dùng xem **chi tiết ước lượng:**
   - Bảng phân tích chi phí.
   - Tuỳ chỉnh được: **mức chất lượng**, **cấp vật liệu**, **địa điểm thi công**, **tỉ lệ chi phí phát sinh**.
   - Bấm **Tính lại** để cập nhật kết quả.

### 16.3 Hạn mức theo gói

Trợ lý có hạn mức số lần tương tác mỗi ngày. Hạn mức được hiển thị trực tiếp trên giao diện chat.

| Gói      | Hạn mức                              |
| -------- | ------------------------------------ |
| Miễn phí | Có hạn mức (hiển thị trên giao diện) |
| Premium  | Không giới hạn                       |

Khi người dùng dùng hết hạn mức, ứng dụng hiển thị **lời mời nâng cấp Premium**.

### 16.4 Lịch sử

Người dùng có **lịch sử các cuộc trò chuyện** với trợ lý — có thể mở lại các phiên cũ để xem hoặc tiếp tục.

---

## 17. Gói hội viên Premium

### 17.1 Hình thức

- Hiện chỉ có một gói: **Premium hàng tháng**.
- Mua thông qua **mua trong ứng dụng** trên iOS (App Store) hoặc Android (Google Play).
- Không có cổng thanh toán riêng ngoài hai cửa hàng ứng dụng nói trên.

### 17.2 Quyền lợi (đã triển khai)

- **Tính lại trợ lý ước lượng chi phí không giới hạn.**
- Mở khoá các tính năng cao cấp xuất hiện trong các luồng trợ lý AI.

### 17.3 Trạng thái hội viên

Hệ thống lưu cho mỗi người dùng:

- Nền tảng đã mua (iOS hay Android), tên gói.
- Trạng thái: **đang hoạt động / đã hết hạn / đã huỷ**.
- Ngày mua, ngày hết hạn, có gia hạn tự động hay không.
- Ngày huỷ và lý do huỷ (nếu có).

### 17.4 Xác thực giao dịch

Khi người dùng thanh toán, ứng dụng gửi thông tin giao dịch về máy chủ:

- iOS được xác thực qua App Store của Apple.
- Android được xác thực qua Google Play.
- Hệ thống đảm bảo **một giao dịch chỉ được ghi nhận một lần** — không cấp hội viên trùng dù người dùng có gửi lại nhiều lần.
- Để tránh lạm dụng, mỗi người dùng chỉ gửi xác thực tối đa **5 lần/phút**.

---

## 18. Cài đặt & Đa ngôn ngữ

### 18.1 Trang Cài đặt trên ứng dụng

- **Thông tin tài khoản** — mở màn hình cập nhật hồ sơ.
- **Ngôn ngữ** — chọn Tiếng Việt hoặc Tiếng Anh.
- **Cài đặt thông báo** — vào trang cài đặt thông báo (mục 15.3).
- **Đăng xuất**.
- **Xoá tài khoản** — yêu cầu xác nhận trước khi thực hiện.

### 18.2 Đa ngôn ngữ

- Hai ngôn ngữ được hỗ trợ: **Tiếng Việt** và **Tiếng Anh**.
- Lần đầu mở ứng dụng, hệ thống tự đặt theo ngôn ngữ điện thoại. Sau khi người dùng chọn thủ công, hệ thống luôn ưu tiên lựa chọn của người dùng.
- Áp dụng cho: giao diện ứng dụng, nội dung thông báo, email tự động.

### 18.3 Video chào mừng

Trong luồng giới thiệu khi mở ứng dụng lần đầu, có một **video chào mừng** giới thiệu sản phẩm cho người dùng mới.

---

## 19. Khu vực Quản trị

### 19.1 Đăng nhập & Bố cục chung

- Đăng nhập bằng email và mật khẩu (xem mục 3.2).
- Sau khi đăng nhập, mặc định mở trang **Trending**.
- Thanh điều hướng bên trái có thể thu gọn, gồm các mục: **Trending, Cộng đồng, Cuộc thi, Trình chỉnh sửa ảnh, Tác phẩm, Panorama 360°, Thông báo**.
- Ở thanh phía trên có **menu lệnh nhanh (mở bằng Cmd+K)** và **chuyển nền sáng/tối**.
- Có các trang lỗi chuẩn cho các tình huống không có quyền, không tìm thấy, hoặc lỗi hệ thống.

### 19.2 Kiểm duyệt Bài viết Cộng đồng

- Danh sách bài viết được phân theo trạng thái: **Chờ duyệt / Đã xuất bản / Bị từ chối**.
- Mỗi dòng hiển thị: tác giả, nội dung, ảnh đại diện, ngày tạo, trạng thái.
- **Tìm kiếm** theo nội dung; phân trang 10 dòng/trang.
- **Từ chối hàng loạt** nhiều bài viết cùng lúc.
- **Duyệt một bài** kèm chọn chủ đề bài viết.

### 19.3 Kiểm duyệt Tác phẩm (kiểm duyệt sau)

Tác phẩm xuất hiện trên trang khám phá ngay khi tác giả đăng (xem mục 8.2). Đội vận hành **gỡ hoặc can thiệp về sau** khi cần:

- Danh sách tác phẩm có **bộ lọc**: trạng thái, danh mục, danh mục con, từ khoá (theo tên tác phẩm hoặc tên tác giả).
- Hỗ trợ **chọn nhiều và xử lý hàng loạt** — trong một thao tác có thể vừa khôi phục một số tác phẩm về Đã xuất bản, vừa từ chối một số khác.
- Mở cửa sổ xem chi tiết ảnh tác phẩm trước khi quyết định.
- Phân trang 10 dòng/trang.

### 19.4 Quản lý Panorama 360°

- Danh sách các tác phẩm Panorama (riêng cho đội vận hành).
- **Tạo mới**: biểu mẫu có công cụ tải ảnh, cho phép gắn ảnh vào 6 mặt tường.
- **Chỉnh sửa**, **xem trước**, **xoá**.
- Tìm kiếm theo tên tác phẩm hoặc tên tác giả; phân trang 10 dòng/trang.

### 19.5 Quản lý Cuộc thi

**Danh sách cuộc thi:**

- Các cột: ảnh bìa, tiêu đề, trạng thái, danh mục, ngày, số bài dự thi, số thành viên.
- Tìm kiếm theo tiêu đề; phân trang 20 dòng/trang.

**Tạo và sửa cuộc thi:**

- Các trường: tiêu đề, mô tả, mục tiêu, yêu cầu, luật, ảnh bìa, ngày bắt đầu, ngày kết thúc, trạng thái, danh mục, **các hạng mục giải thưởng**.

**Trang chi tiết một cuộc thi — có 3 mục:**

- **Thông tin** — toàn bộ thông tin cuộc thi.
- **Bài dự thi** — chia làm Chờ duyệt / Đã duyệt / Bị từ chối:
  - Mở cửa sổ chi tiết để xem ảnh và video bài dự thi.
  - Hành động **Duyệt** hoặc **Từ chối** (mở hộp thoại nhập lý do từ chối).
- **Thành viên:**
  - Liệt kê thành viên kèm thông tin ai đã thêm và thời điểm.
  - Thêm thành viên: tìm theo email hoặc nhập email trực tiếp; hỗ trợ cả **quy tắc theo tên miền email**.
  - Xoá thành viên.

### 19.6 Biên tập Trending

Cấu trúc đã mô tả ở mục 13. Khu vực quản trị cung cấp:

- **Trang tổng quan** — số liệu thống kê các danh mục.
- **Trang danh mục / danh mục con** — danh sách tác phẩm có thể đưa vào bảng xếp hạng và danh sách các phiên bản.
- **Trang chi tiết phiên bản** — kéo thả sắp xếp tác phẩm, thêm hoặc bớt tác phẩm.
- Các thao tác: **Tạo, Sửa, Xoá, Sao chép, Công bố** một phiên bản.

### 19.7 Trình chỉnh sửa ảnh

Khu vực quản trị có **trình chỉnh sửa ảnh** dùng để biên tập ảnh phục vụ nội dung trên nền tảng:

- Tải ảnh lên (PNG, JPG, WebP, GIF, tối đa 50MB) hoặc thay ảnh khác.
- Công cụ chú thích: bút vẽ, mũi tên, đường thẳng, hình tròn, hình chữ nhật, làm mờ, chèn chữ, chèn biểu tượng cảm xúc.
- Tuỳ biến nền (màu đặc, gradient, ảnh nền hoặc trong suốt), bo góc, đổ bóng, đệm, viền.
- Đổi tỉ lệ ảnh (auto hoặc các tỉ lệ chuẩn).
- Cắt ảnh.
- Lưu lại bộ thiết lập làm **mẫu định dạng** (preset) để dùng lại.
- Xuất kết quả: **sao chép** vào bộ nhớ tạm hoặc **tải xuống** dưới dạng PNG.
- Hỗ trợ phím tắt (P, A, L, C, R, B, T cho các công cụ; Cmd/Ctrl+Z để hoàn tác, Cmd/Ctrl+Shift+Z để làm lại; Delete để xoá).

### 19.8 Thông báo (gửi thông báo hàng loạt)

- Soạn thông báo **đa ngôn ngữ** (Tiếng Việt và Tiếng Anh) bằng **trình soạn thảo văn bản phong phú** (định dạng đậm, in nghiêng, danh sách, đường dẫn…).
- Chọn người nhận (toàn bộ người dùng hoặc một danh sách cụ thể).
- Gửi đồng thời qua **kênh trong ứng dụng** và **thông báo đẩy** đến điện thoại.

### 19.9 Các nghiệp vụ chỉ có ở phần lõi

Hai nhóm nghiệp vụ sau đã sẵn sàng ở phần lõi của hệ thống nhưng **chưa có giao diện trên khu vực quản trị**:

- **Quản lý người dùng** — danh sách người dùng, cấp hoặc thu hồi quyền quản trị (do Quản trị viên cấp cao thực hiện).
- **Cấu hình ứng dụng** — chỉnh các tham số chạy thực tế cho ứng dụng di động (ví dụ bật/tắt từng tính năng cho từng nhóm người dùng).

---

## 20. Ai được làm gì

| Hành động                                          | Người dùng | Quản trị viên | Quản trị viên cấp cao |
| -------------------------------------------------- | :--------: | :-----------: | :-------------------: |
| Đăng nhập ứng dụng di động                         |     ✅     |       –       |           –           |
| Đăng nhập khu vực quản trị                         |     –      |      ✅       |          ✅           |
| Đăng bài, tạo tác phẩm, nộp bài cuộc thi           |     ✅     |      ✅       |          ✅           |
| Sửa, xoá nội dung của mình                         |     ✅     |      ✅       |          ✅           |
| Thích, bình luận, lưu, theo dõi                    |     ✅     |      ✅       |          ✅           |
| Nhắn tin một-đối-một                               |     ✅     |      ✅       |          ✅           |
| Mua gói Premium                                    |     ✅     |      ✅       |          ✅           |
| Duyệt bài viết Cộng đồng (kiểm duyệt trước)        |     ❌     |      ✅       |          ✅           |
| Gỡ hoặc khôi phục tác phẩm (kiểm duyệt sau)        |     ❌     |      ✅       |          ✅           |
| Tạo, sửa, lưu trữ cuộc thi                         |     ❌     |      ✅       |          ✅           |
| Duyệt bài dự thi                                   |     ❌     |      ✅       |          ✅           |
| Quản lý thành viên & quy tắc tên miền của cuộc thi |     ❌     |      ✅       |          ✅           |
| Biên tập Trending (tạo, sửa, công bố phiên bản)    |     ❌     |      ✅       |          ✅           |
| Quản lý Panorama 360° (tạo, sửa, xoá)              |     ❌     |      ✅       |          ✅           |
| Sử dụng Trình chỉnh sửa ảnh                        |     ❌     |      ✅       |          ✅           |
| Gửi thông báo hàng loạt                            |     ❌     |      ✅       |          ✅           |
| Cấp / thu hồi quyền quản trị                       |     ❌     |      ❌       |          ✅           |

---

## 21. Giới hạn sử dụng

| Đối tượng                           | Giới hạn                                                             |
| ----------------------------------- | -------------------------------------------------------------------- |
| Yêu cầu mã xác thực OTP             | 100 lần/ngày/địa điểm truy cập · 1 lần/phút/email · 10 lần/giờ/email |
| Hiệu lực mã xác thực OTP            | 10 phút sau khi gửi                                                  |
| Gửi tin nhắn riêng                  | 60 tin/phút/người dùng · 500 tin/giờ/người dùng                      |
| Xác thực giao dịch hội viên         | 5 lần/phút/người dùng                                                |
| Trợ lý ước lượng chi phí (Miễn phí) | Có hạn mức ngày, hiển thị trên giao diện                             |
| Trợ lý ước lượng chi phí (Premium)  | Không giới hạn                                                       |
| Ảnh khi tạo tác phẩm 2D             | Tối đa 5 ảnh                                                         |
| Ảnh khi nộp bài dự thi              | Tối đa 5 ảnh                                                         |
| Video khi nộp bài dự thi            | Tối đa 5 video, mỗi video không quá 120 giây và 250MB                |
| Ảnh tải lên Trình chỉnh sửa ảnh     | Tối đa 50MB, định dạng PNG / JPG / WebP / GIF                        |
| Mật khẩu tài khoản quản trị         | Tối thiểu 6 ký tự                                                    |

---

## 22. Phần đang hoàn thiện

Các tính năng dưới đây đã có dấu vết trong sản phẩm nhưng **chưa hoàn thiện**:

| Tính năng                                       | Trạng thái hiện tại                                                              |
| ----------------------------------------------- | -------------------------------------------------------------------------------- |
| Trình xem Panorama 360° nâng cao                | Phiên bản hiện tại đã hoạt động ở mức cơ bản; phiên bản nâng cao đang phát triển |
| Trợ lý AI phong cách                            | Đã giới thiệu trong giao diện chọn trợ lý nhưng chưa mở để sử dụng               |
| Tin nhắn ghi âm                                 | Trình phát đã có sẵn; chức năng ghi âm trên ứng dụng đang hoàn thiện             |
| Giao diện quản lý người dùng (khu vực quản trị) | Phần lõi đã sẵn sàng; giao diện chưa làm                                         |
| Giao diện cấu hình ứng dụng (khu vực quản trị)  | Phần lõi đã sẵn sàng; giao diện chưa làm                                         |
| Trang Cài đặt và Trợ giúp (khu vực quản trị)    | Có lối vào trên thanh điều hướng nhưng chưa dẫn tới trang nội dung               |

### Những điều RIZ KHÔNG hỗ trợ

Để tránh hiểu nhầm, dưới đây là các tính năng **không có** trong sản phẩm hiện tại:

- **Trả lời bình luận** — bình luận chỉ một cấp.
- **Nhiều loại biểu cảm** — chỉ có một loại "Thích".
- **Đăng nhập qua mạng xã hội** (Google, Apple, Facebook) — chỉ đăng nhập bằng email + OTP trên ứng dụng và email + mật khẩu trên khu vực quản trị.
- **Mua bán tác phẩm trên nền tảng** — RIZ chưa có cổng thanh toán hay quy trình đặt hàng cho tác phẩm. Hệ thống thanh toán duy nhất hiện tại là gói hội viên Premium qua App Store / Google Play.
- **Báo cáo nội dung vi phạm**.
- **Phát trực tiếp (livestream)**.
- **Chat nhóm** — chỉ nhắn tin một-đối-một.

---

**Hết tài liệu.**
