Epic 1: Quản Lý Challenge (Leader — Web Admin)

## Backend API Reference

| Method | Path | Mô tả |
|--------|------|-------|
| POST | `/admin/challenges` | Tạo challenge (multipart/form-data) |
| GET | `/admin/challenges` | List challenges (admin view, bao gồm DRAFT) |
| GET | `/admin/challenges/users/search?q=keyword` | Search users để add member |
| GET | `/admin/challenges/:id` | Chi tiết challenge |
| PATCH | `/admin/challenges/:id` | Chỉnh sửa challenge |
| DELETE | `/admin/challenges/:id` | Soft delete challenge |
| GET | `/admin/challenges/:id/members` | List members |
| POST | `/admin/challenges/:id/members` | Add members (email hoặc domain rule) |
| DELETE | `/admin/challenges/:id/members/:memberId` | Remove member |
| GET | `/admin/challenges/:id/submissions` | List tất cả submissions |
| PATCH | `/admin/challenges/:id/submissions/:submissionId/review` | Review submission |

---

US-01 — Tạo challenge mới
Là Leader, Tôi muốn tạo một challenge sáng tạo theo chủ đề trên web admin, Để tổ chức hoạt động sáng tạo có chủ đề, có thời hạn và luật chơi rõ ràng cho cộng đồng.

Tiêu chí chấp nhận (đã implement trên web admin + BE):

- [x] Leader điền được: tiêu đề, mô tả, mục tiêu, yêu cầu, luật chơi
- [x] Leader upload được ảnh bìa challenge (multipart/form-data, field: coverImage)
- [x] Leader chọn được thời điểm bắt đầu (`startsAt`) và thời điểm kết thúc (`endsAt`) — ISO 8601 datetime (UTC); admin picks datetime theo locale thiết bị của mình, FE convert sang UTC khi gửi API
- [x] Leader chọn được danh mục feed (feedCategory → categoryId, kiểu int)
- [x] Leader chọn được danh mục con (subCategory → subcategoryId) — tùy chọn
- [x] Challenge mới được tạo với status mặc định là `DRAFT`
- [ ] Web admin hiện chưa có field chọn status trực tiếp ở form tạo
- [x] Leader tạo được các hạng mục giải thưởng (awardCategories: [{ name: "..." }])

US-02 — Chỉnh sửa challenge
Là Leader, Tôi muốn chỉnh sửa thông tin challenge sau khi tạo, Để cập nhật nội dung hoặc thay đổi thời gian khi cần thiết.

Tiêu chí chấp nhận (BE đã implement):

- [x] Leader sửa được tất cả các trường đã điền lúc tạo
- [x] Thay đổi được phản ánh ngay trên app sau khi lưu
- [x] API: `PATCH /admin/challenges/:id` (multipart/form-data)

US-03 — Duyệt bài dự thi đơn lẻ
Là Leader, Tôi muốn xem và duyệt từng bài dự thi đang chờ, Để kiểm soát chất lượng nội dung trước khi bài xuất hiện công khai.

Tiêu chí chấp nhận (BE đã implement):

- [x] Leader xem được danh sách tất cả submissions của một challenge (bao gồm PENDING)
- [x] API: `GET /admin/challenges/:id/submissions`
- [x] Leader duyệt bài → `{ decision: "APPROVED" }`
- [x] Leader từ chối bài → `{ decision: "REJECTED", rejectionReason: "..." }`
- [x] API: `PATCH /admin/challenges/:id/submissions/:submissionId/review`

US-04 — Bulk duyệt/từ chối bài dự thi
Là Leader, Tôi muốn duyệt hoặc từ chối nhiều bài dự thi cùng lúc, Để tiết kiệm thời gian khi có nhiều bài chờ duyệt.

Tiêu chí chấp nhận (đã implement trên web admin):

- [x] Leader chọn được nhiều bài cùng lúc trong danh sách chờ duyệt
- [x] Leader thực hiện được duyệt hoặc từ chối hàng loạt
- [x] Trạng thái các bài được cập nhật lại từ server sau khi thực hiện
- [x] Không có bulk API riêng; web admin hiện fan-out nhiều request tới `PATCH /admin/challenges/:id/submissions/:submissionId/review`

US-05 — Add member vào challenge
Là Leader, Tôi muốn thêm thành viên vào challenge bằng email hoặc domain rule. Để xác định ai được phép nộp bài dự thi.

Tiêu chí chấp nhận (đã implement):

- [x] Leader nhập email để tìm và add user vào challenge
- [x] Nếu email không tồn tại, profile được auto-create và add vào challenge
- [x] Member được add xuất hiện trong danh sách thành viên của challenge
- [x] Leader có thể add domain rule (ví dụ: @agilead.vn) để cho phép tất cả user có email matching domain
- [x] API: `POST /admin/challenges/:id/members` với body `{ members: [{ email: "member@example.com" }] }` hoặc `{ members: [{ email: "@agilead.vn" }] }` cho domain rule

US-06 — Xem và xóa danh sách member
Là Leader, Tôi muốn xem danh sách tất cả member trong challenge và xóa member khi cần, Để quản lý thành viên tham gia challenge.

Tiêu chí chấp nhận (BE đã implement):

- [x] Leader xem được danh sách member với thông tin: id, challengeId, name, email, avatar, hasAccount, isDomainRule, domainPattern, addedAt
- [x] Leader phân biệt được member trực tiếp vs domain rule
- [x] Leader xóa được member khỏi challenge
- [x] API: `GET /admin/challenges/:id/members`
- [x] API: `DELETE /admin/challenges/:id/members/:memberId`

---

## Entity Response Format

### ChallengeEntity
```json
{
  "id": "string",
  "title": "string",
  "description": "string",
  "objective": "string",
  "requirements": "string",
  "rules": "string",
  "coverImageUrl": "string",
  "startsAt": "datetime (ISO 8601, UTC)",
  "endsAt": "datetime (ISO 8601, UTC)",
  "feedCategory": 1,
  "subCategory": 3,
  "awardCategories": [{ "id": "string", "name": "string" }],
  "status": "DRAFT | ACTIVE | ARCHIVED",
  "submissionCount": 0,
  "memberCount": 0,
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

### ChallengeMemberEntity
```json
{
  "id": "string",
  "challengeId": "string",
  "name": "string",
  "email": "string",
  "avatar": "string | null",
  "hasAccount": true,
  "isDomainRule": false,
  "domainPattern": "string | null",
  "addedAt": "datetime"
}
```

### ChallengeSubmissionEntity
```json
{
  "id": "string",
  "challengeId": "string",
  "title": "string",
  "description": "string",
  "imageUrl": "string",
  "author": {
    "id": "string",
    "name": "string",
    "email": "string",
    "avatar": "string | null"
  },
  "status": "PENDING | APPROVED | REJECTED",
  "rejectionReason": "string | null",
  "submittedAt": "datetime",
  "reviewedAt": "datetime | null"
}
```
