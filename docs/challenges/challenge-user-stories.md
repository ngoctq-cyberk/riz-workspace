# User Stories: Hệ Thống Challenge — Sáng Tạo Theo Chủ Đề

| Thuộc tính  | Giá trị          |
| ----------- | ---------------- |
| Nguồn       | challenge-PRD.md |
| Ngày tạo    | 2026-04-08       |
| Cập nhật    | 2026-04-20       |
| Phiên bản   | 1.4              |
| Trạng thái  | Synced with current code (Round 7 reviewer fixes) |

---

## Epic 1: Quản Lý Challenge (Leader — Web Admin)

### US-01 — Tạo challenge mới

**Là** Leader,  
**Tôi muốn** tạo một challenge sáng tạo theo chủ đề trên web admin,  
**Để** tổ chức hoạt động sáng tạo có chủ đề, có thời hạn và luật chơi rõ ràng cho cộng đồng.

**Tiêu chí chấp nhận** (đã implement):
- [x] Leader điền được: tiêu đề, mô tả, mục tiêu, yêu cầu, luật chơi
- [x] Leader upload được ảnh bìa challenge (multipart/form-data)
- [x] Leader chọn được thời điểm bắt đầu (`startsAt`) và thời điểm kết thúc (`endsAt`) dưới dạng datetime theo locale của thiết bị admin. FE convert sang ISO 8601 UTC khi gửi API. User ở múi giờ khác sẽ thấy timeline dịch theo device TZ của họ.
- [x] Leader chọn được danh mục feed (categoryId → ProjectCategory)
- [x] Leader chọn được danh mục con (subcategoryId → ProjectSubcategory) — tùy chọn
- [x] Leader có thể tạo các hạng mục giải thưởng (awardCategories)
- [x] API: `POST /admin/challenges` (multipart/form-data)

---

### US-02 — Chỉnh sửa challenge

**Là** Leader,  
**Tôi muốn** chỉnh sửa thông tin challenge sau khi tạo,  
**Để** cập nhật nội dung hoặc thay đổi thời gian khi cần thiết.

**Tiêu chí chấp nhận** (đã implement):
- [x] Leader sửa được tất cả các trường đã điền lúc tạo
- [x] Thay đổi được phản ánh ngay trên app sau khi lưu
- [x] API: `PATCH /admin/challenges/:id` (multipart/form-data)

---

### US-03 — Duyệt bài dự thi đơn lẻ

**Là** Leader,  
**Tôi muốn** xem và duyệt từng bài dự thi đang chờ,  
**Để** kiểm soát chất lượng nội dung trước khi bài xuất hiện công khai.

**Tiêu chí chấp nhận** (đã implement):
- [x] Leader xem được danh sách tất cả submissions (bao gồm PENDING) của một challenge
- [x] Leader xem được preview chi tiết từng bài (ảnh, tiêu đề, mô tả)
- [x] Leader duyệt bài → bài xuất hiện trên Challenge và Feed
- [x] Leader từ chối bài → có thể nhập lý do từ chối (rejectionReason)
- [x] API: `GET /admin/challenges/:id/submissions` để xem danh sách
- [x] API: `PATCH /admin/challenges/:id/submissions/:submissionId/review` với body `{ decision: "APPROVED" }` hoặc `{ decision: "REJECTED", rejectionReason: "..." }`

---

### US-04 — Bulk duyệt/từ chối bài dự thi

**Là** Leader,  
**Tôi muốn** duyệt hoặc từ chối nhiều bài dự thi cùng lúc,  
**Để** tiết kiệm thời gian khi có nhiều bài chờ duyệt.

**Tiêu chí chấp nhận** (đã implement trên web admin):
- [x] Leader chọn được nhiều bài cùng lúc trong danh sách chờ duyệt
- [x] Leader thực hiện được duyệt hoặc từ chối hàng loạt
- [x] Trạng thái các bài được cập nhật lại từ server sau khi thực hiện
- [x] Không có bulk API riêng; web admin hiện fan-out nhiều request tới `PATCH /admin/challenges/:id/submissions/:submissionId/review`

---

### US-05 — Add member vào challenge

**Là** Leader,  
**Tôi muốn** thêm thành viên vào challenge bằng email hoặc domain rule,  
**Để** xác định ai được phép nộp bài dự thi.

**Tiêu chí chấp nhận**:
- [x] Leader nhập email để tìm và add user vào challenge
- [x] Nếu email không tồn tại trong hệ thống, profile được auto-create và add vào challenge
- [x] Leader có thể add domain rule (ví dụ: @riz.vn) để cho phép tất cả user có email matching domain
- [x] Member được add xuất hiện trong danh sách thành viên của challenge
- [x] API: `POST /admin/challenges/:id/members` với body `{ members: [{ email: "..." }] }`

---

### US-06 — Xem và xóa danh sách member

**Là** Leader,  
**Tôi muốn** xem danh sách tất cả member trong challenge và xóa member khi cần,  
**Để** quản lý thành viên tham gia challenge.

**Tiêu chí chấp nhận** (đã implement):
- [x] Leader xem được danh sách member với thông tin cơ bản (tên, email, avatar, ngày add)
- [x] Leader xem được phân biệt member trực tiếp vs domain rule
- [x] Leader xóa được member khỏi challenge
- [x] API: `GET /admin/challenges/:id/members`
- [x] API: `DELETE /admin/challenges/:id/members/:memberId`

---

## Epic 2: Xem Challenge (Audience — App Mobile)

### US-07 — Xem danh sách challenge đang mở

**Là** Audience (không cần đăng nhập),  
**Tôi muốn** xem danh sách các challenge đang active trong tab Challenge,  
**Để** khám phá các hoạt động sáng tạo đang diễn ra.

**Tiêu chí chấp nhận** (BE đã implement):
- [ ] Tab Challenge hiển thị như menu item top-level, ngang hàng Feed / Create / AI Chat / Community (FE cần làm)
- [x] Danh sách challenge hiển thị: ảnh bìa, tiêu đề, thời gian, số submission, số member
- [x] Không cần đăng nhập để xem danh sách
- [x] API: `GET /challenges` với pagination, filter `status` (`DRAFT`/`ACTIVE`/`ARCHIVED`) và `phase` (`upcoming`/`ongoing`/`ended` — derived từ `startsAt`/`endsAt` so với `now`, không phải status lưu DB)

---

### US-08 — Xem chi tiết challenge và gallery bài dự thi

**Là** Audience (không cần đăng nhập),  
**Tôi muốn** xem chi tiết một challenge và các bài dự thi đã được duyệt,  
**Để** theo dõi sáng tạo của nhiều người theo cùng một chủ đề.

**Tiêu chí chấp nhận** (BE đã implement):
- [x] Màn hình hiển thị: tiêu đề, mô tả, mục tiêu, yêu cầu, luật chơi, thời gian, awardCategories
- [x] Danh sách bài dự thi đã duyệt (APPROVED) có thể fetch được
- [ ] FE cần làm: gallery grid hiển thị ảnh chính, tiêu đề, tên tác giả
- [x] API: `GET /challenges/:id` cho chi tiết
- [x] API: `GET /challenges/:id/submissions` cho danh sách submissions đã duyệt

---

### US-09 — Xem chi tiết bài dự thi

**Là** Audience (không cần đăng nhập),  
**Tôi muốn** nhấn vào bài dự thi để xem chi tiết,  
**Để** xem toàn bộ nội dung sáng tạo của bài đó.

**Tiêu chí chấp nhận**:
- [ ] Nhấn vào bài trong gallery → mở màn hình chi tiết project
- [ ] Chi tiết project hiển thị thêm: tên challenge + link navigate về tab Challenge
- [ ] Tương tác (like, comment, bookmark) hoạt động như project thường

---

### US-10 — Bình luận bài dự thi

**Là** Audience đã đăng nhập,  
**Tôi muốn** bình luận trên bài dự thi,  
**Để** tương tác và chia sẻ nhận xét với tác giả.

**Tiêu chí chấp nhận**:
- [ ] Audience đã đăng nhập bình luận được trên bài dự thi
- [ ] Audience chưa đăng nhập → hiển thị yêu cầu đăng nhập khi nhấn bình luận
- [ ] Bookmark cũng yêu cầu đăng nhập tương tự

---

## Epic 3: Nộp Bài Dự Thi (Member — App Mobile)

### US-11 — Nộp bài dự thi

**Là** Member (đã được Leader add hoặc email match domain rule),  
**Tôi muốn** nộp bài dự thi cho một challenge đang mở,  
**Để** tham gia hoạt động sáng tạo và được leader xem xét.

**Tiêu chí chấp nhận** (BE đã implement):
- [x] Chỉ member (hoặc user có email match domain rule) mới được nộp bài (ChallengeSubmissionGuard)
- [x] Member upload được ít nhất 1 ảnh (multipart/form-data hoặc imageUrls)
- [x] Member nhập tiêu đề (bắt buộc, max 255 chars)
- [x] Member nhập mô tả ý tưởng (bắt buộc, max 5000 chars)
- [x] Member nhập ghi chú hướng tiếp cận (tùy chọn) → `approachNote`
- [x] Member nhập mô tả đáp ứng chủ đề (tùy chọn) → `themeResponse`
- [x] Mỗi member chỉ được nộp 1 bài/challenge (unique constraint)
- [x] API: `POST /challenges/:id/submissions` (multipart/form-data với images[])

---

### US-12 — Xem trạng thái bài dự thi của mình

**Là** Member,  
**Tôi muốn** xem danh sách bài đã nộp cùng trạng thái hiện tại,  
**Để** biết bài nào đã được duyệt, đang chờ hoặc bị từ chối.

**Tiêu chí chấp nhận**:
- [ ] Member xem được danh sách tất cả bài đã nộp cho challenge
- [ ] Mỗi bài hiển thị trạng thái: Chờ duyệt / Đã duyệt / Từ chối
- [ ] Bài bị từ chối hiển thị lý do (nếu leader có điền)

---

## Epic 4: Hiển Thị Trên Project Feed (System)

### US-13 — Badge "Challenge" trên Feed

**Là** người dùng xem Project Feed,  
**Tôi muốn** nhận biết được bài dự thi challenge ngay trên feed,  
**Để** phân biệt với project thông thường và khám phá nội dung từ các challenge.

**Tiêu chí chấp nhận**:
- [ ] Bài dự thi trên feed hiển thị badge "Challenge"
- [ ] Bài dự thi đã duyệt xuất hiện đúng danh mục mà leader đã chọn cho challenge

---

### US-14 — Navigate từ Feed về Challenge

**Là** người dùng xem chi tiết project bài dự thi trên Feed,  
**Tôi muốn** nhấn vào link challenge để đến trang challenge tương ứng,  
**Để** xem thêm các bài dự thi khác trong cùng challenge.

**Tiêu chí chấp nhận**:
- [ ] Chi tiết project của bài dự thi hiển thị tên challenge kèm link điều hướng
- [ ] Nhấn link → đến đúng màn hình chi tiết challenge

---

## Tóm Tắt User Stories

| US    | Mô tả ngắn                       | Role     | Platform | Epic   |
| ----- | --------------------------------- | -------- | -------- | ------ |
| US-01 | Tạo challenge mới                 | Leader   | Web      | Epic 1 |
| US-02 | Chỉnh sửa challenge               | Leader   | Web      | Epic 1 |
| US-03 | Duyệt bài đơn lẻ                  | Leader   | Web      | Epic 1 |
| US-04 | Bulk duyệt/từ chối bài            | Leader   | Web      | Epic 1 |
| US-05 | Add member bằng email             | Leader   | Web      | Epic 1 |
| US-06 | Xem và xóa member                 | Leader   | Web      | Epic 1 |
| US-07 | Xem danh sách challenge           | Audience | App      | Epic 2 |
| US-08 | Xem chi tiết challenge + gallery  | Audience | App      | Epic 2 |
| US-09 | Xem chi tiết bài dự thi           | Audience | App      | Epic 2 |
| US-10 | Bình luận bài dự thi              | Audience | App      | Epic 2 |
| US-11 | Nộp bài dự thi                    | Member   | App      | Epic 3 |
| US-12 | Xem trạng thái bài của mình       | Member   | App      | Epic 3 |
| US-13 | Badge "Challenge" trên Feed       | System   | App      | Epic 4 |
| US-14 | Navigate từ Feed về Challenge     | System   | App      | Epic 4 |
