# PRD: Hệ Thống Challenge — Sáng Tạo Theo Chủ Đề

| Thuộc tính         | Giá trị            |
| ------------------ | ------------------ |
| Tác giả            | RIZ Product Team   |
| Ngày tạo           | 2026-04-08         |
| Cập nhật           | 2026-04-13         |
| Trạng thái         | Synced with BE     |
| Phiên bản          | 1.3                |
| Tính năng liên quan | Không              |

---

## 1. Tổng Quan

### 1.1. Bối Cảnh

RIZ hiện có hai hệ thống nội dung tách biệt:

- **Project Feed**: hiển thị project dạng masonry grid, phân loại theo category (ARCHITECTS, ARTISTS, DESIGNERS, PHOTOGRAPHERS) và subcategory (LIVING_ROOM, KITCHEN, BEDROOM). Feed lấy từ API `/trending/feed`, chi tiết/list project dùng API `/project`.
- **Community Feed**: hiển thị post dạng list, phân loại theo post category (HUMANS_OF_RIZ, FUTURE_TREND_LAB). Có workflow duyệt bài (PENDING_APPROVAL → PUBLISHED). Dữ liệu từ API `/posts`.

Hiện chưa có cơ chế để leader tổ chức các hoạt động có chủ đề, có thời hạn và có quy tắc rõ ràng dưới dạng một surface riêng biệt trong app.

### 1.2. Mục Tiêu

- Cho phép leader tạo các challenge sáng tạo theo chủ đề, có thời hạn và luật chơi cụ thể
- Bài dự thi sau khi duyệt xuất hiện đồng thời trên **Challenge** và **Project Feed**
- Leader quyết định feed category cho challenge, member không cần tự phân loại

### 1.3. Chỉ Số Thành Công

| Chỉ số                                         | Mục tiêu                        |
| ----------------------------------------------- | -------------------------------- |
| Số challenge được tạo / tháng                   | > 0 (launch)                     |
| Số bài dự thi / challenge                       | >= 5                             |
| Tỷ lệ bài được duyệt                           | >= 70%                           |
| Engagement trên bài dự thi (like + comment)     | >= trung bình project thường     |

---

## 2. Đối Tượng Người Dùng

| Role     | Platform     | Auth        | Mapping hệ thống | Mô tả                                                                  |
| -------- | ------------ | ----------- | ----------------- | ----------------------------------------------------------------------- |
| Leader   | Web (admin)  | Đăng nhập   | Role: ADMIN/SUPERADMIN | Quản lý challenge: tạo, chỉnh sửa, duyệt/từ chối bài dự thi. Add member bằng email. |
| Member   | App (mobile) | Đăng nhập   | ChallengeMember   | User đã được Leader add. Vào tab Challenge để xem challenge và nộp bài dự thi. |
| Audience | App (mobile) | Không cần   | User hoặc anonymous | Bất kỳ ai mở app. Xem challenge, xem bài dự thi đã duyệt, xin tham gia. |

**Lưu ý về Audience**: Audience không cần đăng nhập để xem challenge và bài dự thi trong tab Challenge. Tuy nhiên, cần đăng nhập nếu muốn bình luận.

**Lưu ý về Member**: Leader add member qua web admin bằng email hoặc domain rule. Nếu email không tồn tại trong hệ thống, profile sẽ được auto-create. Có thể add theo domain (ví dụ: @riz.vn) để cho phép tất cả user có email matching domain được nộp bài. Khi nộp bài, hệ thống check user là challenge member hoặc email match với domain rule.

**Lưu ý về scope**: MVP cần model `ChallengeMember` tối thiểu để phân biệt Member (được Leader add) với Audience (user đăng nhập chưa được add). `Challenge` là một top-level feature trong app, đứng ngang hàng với `Feed`, `Create`, `AI Chat`, `Community`. Challenge hoàn toàn tách biệt với Community — không liên quan tới post feed.

---

## 3. Challenge: Sáng Tạo Theo Chủ Đề

### 3.1. Mô Tả

Leader đặt một chủ đề cụ thể (ví dụ: "Thiết kế nhà bếp phong cách Bắc Âu"). Member nộp sáng tạo hoàn chỉnh theo đúng chủ đề, không cần xoay quanh một tác phẩm chủ thể duy nhất. Miễn là đáp ứng đúng chủ đề, mục tiêu và luật chơi của challenge.

### 3.2. Giá Trị

- Tạo sân chơi sáng tạo mở cho member
- Giúp leader điều phối phong cách và hướng đi sáng tạo
- Giúp audience theo dõi nhiều cách diễn giải khác nhau trên cùng một chủ đề

### 3.3. Cấu Trúc Bài Dự Thi

| Trường                        | Bắt buộc | Mô tả                                 |
| ----------------------------- | -------- | -------------------------------------- |
| Bộ ảnh chính                 | Yes      | >= 1 ảnh                               |
| Tiêu đề                       | Yes      | Tên bài dự thi                          |
| Mô tả ý tưởng                 | Yes      | Giải thích ý tưởng sáng tạo            |
| Ghi chú hướng tiếp cận        | No       | Phương pháp, công cụ, cảm hứng         |
| Mô tả đáp ứng chủ đề          | No       | Giải thích cách bài đáp ứng đề bài     |

**Giới hạn MVP**: Chỉ hỗ trợ ảnh (`image-only`). Không hỗ trợ video hoặc loại media khác trong bài dự thi.

---

## 4. Quy Tắc Chung

### 4.1. Challenge

- Mỗi challenge phải có: thời gian bắt đầu/kết thúc, mục tiêu, yêu cầu, luật chơi
- Leader chọn feed category (và subcategory nếu có) khi tạo challenge

### 4.2. Bài Dự Thi

- Chỉ member mới được nộp bài
- Bài phải qua leader duyệt trước khi hiển thị
- Sau khi duyệt, bài xuất hiện đồng thời trên Challenge và Feed
- Bài bị từ chối chỉ hiển thị cho member (kèm lý do nếu có)

### 4.3. Ma Trận Quyền

| Hành động               | Leader (ADMIN) | Member (ChallengeMember) | Audience (no auth) | Audience (logged in, chưa member) |
| ----------------------- | -------------- | ------------------------ | ------------------ | --------------------------------- |
| Tạo challenge           | Yes            | No                       | No                 | No                                |
| Duyệt / từ chối         | Yes            | No                       | No                 | No                                |
| Add member (by email)   | Yes            | No                       | No                 | No                                |
| Nộp bài dự thi          | No             | Yes                      | No                 | No                                |
| Xem challenge           | Yes            | Yes                      | Yes                | Yes                               |
| Xem bài đã duyệt        | Yes            | Yes                      | Yes                | Yes                               |
| Bình luận (template)    | —              | Yes                      | No (prompt login)  | Yes                               |
| Bookmark                | —              | Yes                      | No (prompt login)  | Yes                               |

---

## 5. Thiết Kế Kỹ Thuật

### 5.1. Nguyên Tắc: Bài Dự Thi = Project Mở Rộng

Bài dự thi là một **Project** có thêm metadata liên kết tới challenge. Lý do:

- Tận dụng toàn bộ hạ tầng Project Feed hiện tại (masonry grid, category filtering, bookmark, engagement)
- Bài dự thi sáng tạo theo chủ đề tạo ra sản phẩm phù hợp hiển thị trên feed
- Không tạo data model trùng lặp

### 5.2. Data Model

```
Challenge
├── id                    BigInt
├── title                 String
├── description           String @db.Text    // Mô tả chung
├── objective             String @db.Text    // Mục tiêu
├── requirements          String @db.Text    // Yêu cầu
├── rules                 String @db.Text    // Luật chơi
├── coverImageUrl         String?
├── startsAt              DateTime
├── endsAt                DateTime
├── status                ChallengeStatus    // DRAFT | ACTIVE | ARCHIVED (phase "ended" là derived từ endsAt < now, không lưu DB)
├── categoryId            Int               // FK → ProjectCategory
├── subcategoryId         Int?              // FK → ProjectSubcategory
├── createdByUserId       BigInt            // FK → User (admin đã tạo)
├── createdAt             DateTime
├── updatedAt             DateTime
└── deletedAt             DateTime?         // Soft delete

ChallengeSubmission (bài dự thi — model riêng)
├── id                    BigInt
├── challengeId           BigInt            // FK → Challenge
├── projectId             BigInt @unique    // FK → Project (1-1 relationship)
├── authorId              BigInt            // FK → Profile
├── status                ChallengeSubmissionStatus  // PENDING | APPROVED | REJECTED
├── approachNote          String? @db.Text  // Ghi chú hướng tiếp cận
├── themeResponse         String? @db.Text  // Mô tả đáp ứng chủ đề
├── rejectionReason       String? @db.Text  // Lý do từ chối
├── reviewedBy            BigInt?           // FK → Profile (admin đã review)
├── submittedAt           DateTime
├── reviewedAt            DateTime?
├── createdAt             DateTime
└── updatedAt             DateTime
└── @@unique([challengeId, authorId])       // Mỗi member chỉ được nộp 1 bài/challenge

ChallengeMember
├── id                    BigInt
├── challengeId           BigInt            // FK → Challenge
├── profileId             BigInt            // FK → Profile
├── addedByUserId         BigInt?           // FK → User (admin đã add)
├── addedAt               DateTime
└── @@unique([challengeId, profileId])

ChallengeDomainRule (cho phép add member theo domain email)
├── id                    BigInt
├── challengeId           BigInt            // FK → Challenge
├── domainPattern         String            // Ví dụ: @agilead.vn
├── addedByUserId         BigInt?           // FK → User
├── addedAt               DateTime
└── @@unique([challengeId, domainPattern])

ChallengeAwardCategory (hạng mục giải thưởng/dự thi)
├── id                    BigInt
├── challengeId           BigInt            // FK → Challenge
├── name                  String            // Ví dụ: "Best Minimalist Approach"
├── order                 Int               // Thứ tự hiển thị
├── createdAt             DateTime
└── updatedAt             DateTime
```

**Lưu ý về ChallengeSubmission**: Bài dự thi được lưu trong model riêng `ChallengeSubmission`, link với `Project` qua `projectId` (1-1 relationship). Khi member nộp bài, hệ thống tạo Project + ChallengeSubmission cùng lúc. Status riêng biệt: `PENDING` → `APPROVED`/`REJECTED`.

**Lưu ý về ChallengeDomainRule**: Cho phép admin add member theo domain email (ví dụ: @riz.vn). Tất cả user có email matching domain pattern sẽ được phép nộp bài mà không cần add từng người.

**Lưu ý về scope**: Tất cả challenge thuộc hệ thống RIZ chung, không phân chia theo nhóm.

### 5.3. Mapping Bài Dự Thi Sang Project + ChallengeSubmission

Khi member nộp bài dự thi, hệ thống tạo **Project** và **ChallengeSubmission** với mapping sau:

| Bài dự thi (input)       | Target field                     | Nguồn giá trị               |
| ------------------------ | -------------------------------- | --------------------------- |
| Bộ ảnh chính             | Project.images (Media[])         | Member upload               |
| Tiêu đề                  | Project.name                     | Member nhập                 |
| Mô tả ý tưởng            | Project.introduction             | Member nhập                 |
| Ghi chú hướng tiếp cận   | ChallengeSubmission.approachNote | Member nhập (optional)      |
| Mô tả đáp ứng chủ đề     | ChallengeSubmission.themeResponse| Member nhập (optional)      |
| —                        | ChallengeSubmission.challengeId  | Từ challenge                |
| —                        | ChallengeSubmission.projectId    | Project vừa tạo             |
| —                        | ChallengeSubmission.status       | PENDING                     |
| —                        | Project.categoryId               | Từ challenge.categoryId     |
| —                        | Project.subcategoryId            | Từ challenge.subcategoryId  |
| —                        | Project.status                   | PUBLISHED                   |
| —                        | Project.type                     | TWO_D (default)             |
| —                        | Project.authorId                 | Member's profileId          |

**Upload Flow cho ảnh bài dự thi**: Hỗ trợ 2 cách:

**Cách 1 - Multipart/form-data** (recommended cho mobile):
```
POST /challenges/:id/submissions
Content-Type: multipart/form-data

title: "Tiêu đề"
description: "Mô tả ý tưởng"
approachNote: "Ghi chú (optional)"
themeResponse: "Mô tả đáp ứng chủ đề (optional)"
images: [file1, file2, ...]
```

**Cách 2 - Pre-uploaded URLs** (nếu đã upload ảnh trước):
```
POST /challenges/:id/submissions
{
  "title": "Tiêu đề",
  "description": "Mô tả ý tưởng",
  "imageUrls": ["https://cdn.../img1.jpg", "https://cdn.../img2.jpg"],
  "mediaMetadata": [
    { "width": 1920, "height": 1080, "thumbnailUrl": "...", "mediumUrl": "..." }
  ],
  "approachNote": "Ghi chú (optional)",
  "themeResponse": "Mô tả đáp ứng chủ đề (optional)"
}
```

**Flow nộp bài là flow mới**, không reuse form tạo Post hay form tạo Project. Cần tạo screen riêng trên app và endpoint riêng trên backend.

**Giới hạn MVP**: Project bài dự thi luôn là `TWO_D` và chỉ nhận ảnh.

### 5.4. Feed Category Do Leader Quyết Định

Khi tạo challenge, leader chọn `feedCategory` và `feedSubcategory` (tùy chọn). Khi member nộp bài:

- `project.category` = `challenge.feedCategory`
- `project.subcategory` = `challenge.feedSubcategory`
- Member không cần chọn — tự động gán từ challenge

**Ví dụ**:

| Challenge                                    | feedCategory  | feedSubcategory |
| -------------------------------------------- | ------------- | --------------- |
| "Tranh trừu tượng trong không gian sống"      | ARTISTS       | —               |
| "Thiết kế phòng khách Bắc Âu"                | ARCHITECTS    | LIVING_ROOM     |
| "Chụp ảnh kiến trúc đương đại"               | PHOTOGRAPHERS | —               |

### 5.5. Hiển Thị Trên Challenge Surface

Hiện tại Community chỉ hiển thị Post (fetch `/posts`, render dạng text + media carousel). Bài dự thi là Project, nên Challenge cần là một **surface riêng** trong app:

- Challenge là menu item top-level, ngang hàng với `Feed`, `Create`, `AI Chat`, `Community`
- Challenge có list screen riêng hiển thị danh sách challenge đang active
- Challenge detail hiển thị thông tin challenge + danh sách bài dự thi
- Bài dự thi fetch từ `GET /challenges/:id/submissions`, **không** từ `/posts`
- Render dạng gallery grid (khác với post list), hiển thị: ảnh chính, tiêu đề, tên tác giả
- Nhấn vào bài → mở chi tiết project (reuse project detail screen hiện có)

Nói cách khác: Challenge **không** reuse post list của Community và cũng **không** convert Project sang Post. Đây là một route/view riêng query trực tiếp project data của challenge submissions.

### 5.6. Hiển Thị Trên Project Feed

Bài dự thi đã duyệt (`Project.status = PUBLISHED`, có relation `project.challengeSubmission`) xuất hiện trên feed như project bình thường, với bổ sung:

- Badge "Challenge" trên feed item
- Chi tiết project hiển thị thêm: tên challenge, link navigate về tab/route Challenge
- Engagement (like, comment, bookmark) hoạt động như project bình thường

### 5.7. API Endpoints (Đã Implement)

**Backend — Public/User endpoints** (challenge.controller.ts):

| Method | Path                              | Auth               | Mô tả                                         |
| ------ | --------------------------------- | ------------------- | ---------------------------------------------- |
| GET    | `/challenges`                     | JwtOptionalGuard    | List challenges (filter: status, pagination)   |
| GET    | `/challenges/:id`                 | JwtOptionalGuard    | Chi tiết challenge                             |
| GET    | `/challenges/:id/submissions`     | JwtOptionalGuard    | List bài dự thi đã duyệt (APPROVED) của challenge |
| POST   | `/challenges/:id/submissions`     | JwtGuard + ChallengeSubmissionGuard | Nộp bài dự thi (multipart/form-data hoặc JSON với imageUrls) |

**Backend — Admin endpoints** (admin-challenge.controller.ts, `@Roles('ADMIN', 'SUPERADMIN')`):

| Method | Path                                                  | Mô tả                                  |
| ------ | ----------------------------------------------------- | --------------------------------------- |
| POST   | `/admin/challenges`                                   | Tạo challenge (multipart/form-data)    |
| GET    | `/admin/challenges`                                   | List challenges (admin view, với DRAFT)|
| GET    | `/admin/challenges/users/search`                      | Search users để add member             |
| GET    | `/admin/challenges/:id`                               | Chi tiết challenge (admin view)        |
| PATCH  | `/admin/challenges/:id`                               | Chỉnh sửa challenge                    |
| DELETE | `/admin/challenges/:id`                               | Soft delete challenge                  |
| GET    | `/admin/challenges/:id/members`                       | List challenge members                 |
| POST   | `/admin/challenges/:id/members`                       | Add members by email hoặc domain rule  |
| DELETE | `/admin/challenges/:id/members/:memberId`             | Remove member                          |
| GET    | `/admin/challenges/:id/submissions`                   | List tất cả submissions (bao gồm PENDING) |
| PATCH  | `/admin/challenges/:id/submissions/:submissionId/review` | Duyệt/từ chối 1 bài (APPROVED/REJECTED) |

**Backend — ChallengeSubmission Model**:

| Thay đổi                              | Mô tả                                                     |
| ------------------------------------- | ---------------------------------------------------------- |
| Schema: `ChallengeSubmission`         | Model riêng link với Project qua `projectId` (1-1)         |
| ChallengeSubmissionStatus             | `PENDING`, `APPROVED`, `REJECTED`                          |
| `approachNote`, `themeResponse`       | Metadata bài dự thi trong ChallengeSubmission              |
| `rejectionReason`                     | Lý do từ chối, admin điền khi reject                       |
| `reviewedBy`, `reviewedAt`            | Tracking ai đã review và khi nào                           |
| Project.challengeSubmission           | Relation 1-1 để lấy submission info từ project            |

**Mobile — Screen mới**:

| Screen                        | Mô tả                                                     |
| ----------------------------- | ---------------------------------------------------------- |
| Challenge List                | Danh sách challenge đang active, public access             |
| Challenge Detail              | Thông tin challenge + gallery bài dự thi đã duyệt         |
| Challenge Submit              | Form nộp bài: upload ảnh + tiêu đề + mô tả + fields optional |
| My Submissions                | Danh sách project challenge của current member + trạng thái + `rejectionReason` khi bị từ chối |

**Mobile — Điều hướng / sửa đổi screen hiện có**:

| Screen              | Thay đổi                                                     |
| ------------------- | ------------------------------------------------------------ |
| Bottom navigation / tab bar | Thêm menu item `Challenge`, đứng ngang hàng với `Feed`, `Create`, `AI Chat`, `Community` |
| Feed Item           | Hiển thị badge "Challenge" khi `project.challengeSubmission != null` |
| Project Detail      | Hiển thị tên challenge + link khi `project.challengeSubmission != null` |
| Community           | Không thay đổi flow post hiện tại                            |

**Admin Web — Screen mới**:

| Screen                 | Mô tả                                              |
| ---------------------- | --------------------------------------------------- |
| Challenge Management   | CRUD challenge, chọn category, thời gian            |
| Entry Review           | Danh sách bài chờ duyệt, xem chi tiết, duyệt/từ chối (đơn lẻ + bulk) |
| Challenge Members      | Add member bằng email, list members, remove member  |

---

## 6. Luồng Người Dùng

### 6.1. Leader Tạo Challenge (Web)

```
Leader mở trang quản lý Challenge trên web admin
  → Tạo Challenge mới
  → Nhập: tiêu đề, mô tả, mục tiêu, yêu cầu, luật chơi
  → Chọn feed category (+ subcategory nếu có)
  → Upload ảnh bìa
  → Chọn thời gian: ngày bắt đầu — ngày kết thúc
  → Xác nhận tạo
```

### 6.2. Member Nộp Bài (App)

```
Member mở app → Vào menu Challenge
  → Chọn challenge đang mở
  → Nhấn "Nộp bài" → Mở screen Challenge Submit (flow mới)
  → Upload bộ ảnh
  → Nhập tiêu đề (→ project.name)
  → Nhập mô tả ý tưởng (→ project.introduction)
  → (Tùy chọn) Ghi chú hướng tiếp cận (→ challengeSubmission.approachNote)
  → (Tùy chọn) Mô tả đáp ứng chủ đề (→ challengeSubmission.themeResponse)
  → Xác nhận nộp
  → API: POST /challenges/:id/submissions (multipart/form-data)
  → Hệ thống tạo Project + ChallengeSubmission (status: PENDING)
  → Hiển thị "Chờ duyệt"
```

### 6.3. Leader Duyệt Bài (Web)

```
Leader mở trang quản lý Challenge trên web admin
  → Chọn challenge → Tab "Submissions"
  → Xem chi tiết bài dự thi (preview project)
  → Duyệt: API PATCH /admin/challenges/:id/submissions/:submissionId/review { decision: "APPROVED" }
    → Bài hiện trên Challenge + Feed (đúng category)
  → Từ chối: API PATCH /admin/challenges/:id/submissions/:submissionId/review { decision: "REJECTED", rejectionReason: "..." }
    → Chỉ member thấy (kèm lý do)
```

### 6.4. Audience Xem & Tương Tác (App)

```
Mở app → Vào menu Challenge (không cần đăng nhập) → Xem challenge
  → Xem danh sách bài dự thi đã duyệt (gallery grid)
  → Nhấn vào bài → Xem chi tiết project
  → Bình luận / Bookmark → Prompt đăng nhập nếu chưa auth

Xem feed → Thấy bài dự thi (badge "Challenge")
  → Nhấn vào → Xem chi tiết → Link navigate tới Challenge
```

---

## 7. Phạm Vi MVP

### 7.1. Bao Gồm

| #  | Chức năng                                             | Role     | Platform |
| -- | ----------------------------------------------------- | -------- | -------- |
| 1  | Tạo challenge sáng tạo theo chủ đề                    | Leader   | Web      |
| 2  | Thiết lập: thời gian, mục tiêu, yêu cầu, luật chơi   | Leader   | Web      |
| 3  | Chọn feed category + subcategory cho challenge         | Leader   | Web      |
| 4  | Duyệt hoặc từ chối bài dự thi (kèm lý do), bao gồm bulk review | Leader | Web |
| 5  | Add member bằng email hoặc domain rule; email chưa tồn tại sẽ auto-create profile | Leader | Web |
| 6  | Xem danh sách challenge đang mở (public, không cần auth) | All   | App      |
| 7  | Nộp bài dự thi (flow mới: hỗ trợ multipart images hoặc imageUrls + metadata + text) | Member | App |
| 8  | Bài dự thi tạo như Project (`Project.status = PUBLISHED`, `ChallengeSubmission.status = PENDING`) | System | — |
| 9  | Xem trạng thái bài (chờ duyệt / duyệt / từ chối)     | Member   | App      |
| 10 | Hiển thị bài đã duyệt trong challenge (gallery grid)   | System   | App      |
| 11 | Hiển thị bài đã duyệt trên Feed (đúng category, có badge) | System | App  |
| 12 | Bình luận bài dự thi (template comment, cần đăng nhập) | Member/Audience | App |
| 13 | Thêm menu item `Challenge` top-level, public access    | System   | App      |

**Lưu ý endpoint trong MVP**:

- Feed project public tiếp tục dùng `GET /trending/feed`
- Query project/detail project tiếp tục dùng `GET /project` và `GET /project/:id`
- Không dùng `/projects` trong API contract của MVP

### 7.2. Ngoài Phạm Vi MVP

| Chức năng                                          | Lý do hoãn                          |
| -------------------------------------------------- | ----------------------------------- |
| Loại A — Tác phẩm chủ thể trong nhiều ngữ cảnh     | Phụ thuộc tính năng 2D/360 Framing, triển khai sau |
| Giải thưởng / xếp hạng challenge                   | Phức tạp, cần validation thêm      |
| Nhiều vòng challenge                               | Chưa có nhu cầu rõ ràng             |
| Challenge liên nhóm                                | Cần thiết kế permission phức tạp    |
| Analytics cho leader                               | Nice-to-have, không blocking         |
| Member override feed category                      | Giữ đơn giản cho MVP                |
| Video / media khác trong bài dự thi               | MVP chỉ hỗ trợ ảnh                  |
| Notification khi challenge mở / bài được duyệt     | Phase 2                             |

---

## 8. Câu Hỏi Mở

| #  | Câu hỏi                                                              | Trạng thái |
| -- | --------------------------------------------------------------------- | ---------- |
| 1  | Mỗi member được nộp bao nhiêu bài / challenge?                        | Chưa quyết |
| 2  | Member có được chỉnh sửa bài sau khi nộp (trước khi duyệt)?          | Chưa quyết |
| 3  | Leader có thể chỉnh sửa challenge sau khi đã có bài dự thi?          | Chưa quyết |
| 4  | Khi challenge kết thúc, bài dự thi vẫn hiển thị trên feed?            | Chưa quyết |
