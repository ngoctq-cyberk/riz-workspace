# Plan Thực Thi Tính Năng Challenge

| Thuộc tính    | Giá trị                    |
| ------------- | -------------------------- |
| Ngày tạo      | 2026-04-14                 |
| Cập nhật      | 2026-04-16                 |
| Tác giả       | Claude Code                |
| Trạng thái    | Phase A + B DONE — Bugfix 2026-04-16 |
| Phiên bản     | 2.5                        |
| Nguồn tham khảo | challenge-PRD.md, challenge-user-stories.md |

---

## 1. Tổng Quan

### 1.1. Hiện Trạng

| Component | Trạng thái | Ghi chú |
|-----------|------------|---------|
| **Backend API** | ✅ Phase A done + projectId fix | App-specific fields đã thêm vào detail + project response; `ChallengeSubmissionEntity` có `projectId` (2026-04-15) |
| **Admin Web** | ✅ Đã implement | Đầy đủ CRUD + review |
| **Mobile App** | ✅ B.1-B.5 done | Foundation, list, detail, submit, feed badge, polish xong |

### 1.2. Backend APIs Hiện Có

**Public endpoints** (`/challenges`):
| Method | Endpoint | Mô tả | Đủ cho App? |
|--------|----------|-------|-------------|
| GET | `/challenges` | List challenges (filter: status, category, pagination) | ✅ |
| GET | `/challenges/:id` | Chi tiết challenge | ⚠️ Thiếu user-specific state |
| GET | `/challenges/:id/submissions` | List bài dự thi | ⚠️ Chỉ trả APPROVED, không có bài của current user |
| POST | `/challenges/:id/submissions` | Nộp bài (multipart) | ✅ |

### 1.3. API Gaps - Backend Cần Bổ Sung (BLOCKER)

| Gap | Mô tả | Giải pháp đề xuất |
|-----|-------|-------------------|
| **isMember** | App cần biết user có phải member không để hiện/ẩn nút "Nộp bài" | Thêm field `isMember: boolean` vào `GET /challenges/:id` response khi có auth |
| **mySubmission** | App cần lấy bài dự thi của current user (kể cả PENDING/REJECTED) | Thêm endpoint `GET /challenges/:id/my-submission` hoặc field `mySubmission` trong detail |
| **challengeSubmission trong Project** | Feed cần hiện badge "Challenge" trên project | Expose `challengeSubmission` relation trong `ProjectEntity` |

**Lưu ý quan trọng:** 
- `GET /challenges/:id/submissions` chỉ trả bài `APPROVED` cho non-admin (line 720 challenge.service.ts)
- `GetChallengesDto` không có field `search` - search là **post-MVP** hoặc client-side filter

### 1.4. Mục Tiêu

**Backend Phase (làm trước):**
- Bổ sung API endpoints còn thiếu
- Expose challenge relation trong Project

**App Phase (làm sau khi BE xong):**
- Tab Challenge trong navigation (top SliverAppBar, không phải bottom nav)
- Xem danh sách challenge (public)
- Xem chi tiết challenge + gallery bài dự thi (public)
- Nộp bài dự thi (member only)
- Xem trạng thái bài đã nộp (member)
- Badge "Challenge" trên Feed

### 1.5. Route Namespace (QUAN TRỌNG)

Chốt convention để tránh mâu thuẫn:

| Route (file path) | Runtime path | Auth | Mô tả |
|-------------------|--------------|------|-------|
| `app/challenges/index.tsx` | `/challenges` | Public | List challenges (đặt NGOÀI `(protected)`) |
| `app/challenges/[id].tsx` | `/challenges/[id]` | Public | Detail challenge |
| `app/(protected)/challenges/[id]/submit.tsx` | `/challenges/[id]/submit` | Required | Submit form (đặt TRONG `(protected)`) |

**Lưu ý quan trọng về navigation:**
- `(protected)` là **route group** của Expo Router — KHÔNG xuất hiện trong runtime URL
- Tất cả `router.push/navigate` đều dùng **plain path** (không có `(protected)` segment), giống pattern hiện có ở [screens/feed/index.tsx:92-102](riz-app-v2/screens/feed/index.tsx#L92-L102)
- Auth redirect do layout xử lý ở [app/(protected)/_layout.tsx:33-34](riz-app-v2/app/(protected)/_layout.tsx#L33-L34)
- ✅ Đúng: `router.push('/challenges/123/submit')`
- ❌ Sai: `router.push('/(protected)/challenges/123/submit')`

---

## 2. User Flow

### 2.1. Flow Xem Challenge (Audience - Không cần auth)

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FEED SCREEN                                  │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌───────────┐ ┌───────────┐            │
│  │ Feed │ │Create│ │AI Chat│ │ Challenge │ │ Community │            │
│  │  ●   │ │      │ │       │ │     ○     │ │           │            │
│  └──────┘ └──────┘ └──────┘ └─────┬─────┘ └───────────┘            │
└───────────────────────────────────┼─────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    CHALLENGE LIST SCREEN                             │
│                                                                      │
│  [Đang diễn ra]  [Đã kết thúc]                                      │
│   ────────────                                                       │
│  (BE enum: ACTIVE | ENDED — KHÔNG có UPCOMING, xem schema.prisma)    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░││
│  │░░░░░░░░░░░░░░ COVER IMAGE 16:9 (Full Width) ░░░░░░░░░░░░░░░░░░░░││
│  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓││
│  │▓▓ Challenge Title                       📅 Apr 10-30  👥12 📝5 ▓││
│  └────────────────────────────────┬────────────────────────────────┘│
│                                   │                                  │
│  ┌────────────────────────────────┼────────────────────────────────┐│
│  │░░░░░░░░░░░░░░ COVER IMAGE 16:9 │(Full Width) ░░░░░░░░░░░░░░░░░░░││
│  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓││
│  │▓▓ Another Challenge                     📅 Apr 15-May5 👥8 📝3 ▓││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────┼───────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   CHALLENGE DETAIL SCREEN                            │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                      [Cover Image]                               ││
│  │                      Full Width Hero                             ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Challenge Title                                                     │
│  📅 Apr 10, 2026 - Apr 30, 2026                                     │
│  👥 12 members  |  📝 5 submissions                                 │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ [Thông tin] [Bài dự thi]                                        ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  === Tab: Thông tin ===                                             │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ 📋 Mô tả                                                        ││
│  │ Lorem ipsum dolor sit amet...                                   ││
│  │                                                                 ││
│  │ 🎯 Mục tiêu                                                     ││
│  │ Thiết kế không gian phòng khách...                              ││
│  │                                                                 ││
│  │ 📝 Yêu cầu                                                      ││
│  │ - Tối thiểu 3 ảnh render                                        ││
│  │ - Phong cách Bắc Âu                                             ││
│  │                                                                 ││
│  │ ⚖️ Luật chơi                                                    ││
│  │ - Mỗi member chỉ được nộp 1 bài                                 ││
│  │ - Không sử dụng AI generate                                     ││
│  │                                                                 ││
│  │ 🏆 Hạng mục giải thưởng                                         ││
│  │ • Best Minimalist Approach                                      ││
│  │ • Most Creative Solution                                        ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  === Tab: Bài dự thi (Gallery Grid) ===                             │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ ┌───────┐ ┌───────┐ ┌───────┐                                   ││
│  │ │[Image]│ │[Image]│ │[Image]│                                   ││
│  │ │       │ │       │ │       │                                   ││
│  │ │Title  │ │Title  │ │Title  │                                   ││
│  │ │Author │ │Author │ │Author │                                   ││
│  │ └───┬───┘ └───────┘ └───────┘                                   ││
│  └─────┼───────────────────────────────────────────────────────────┘│
│        │                                                             │
│  ┌─────┴─────────────────────────────────────────────────────────┐  │
│  │        [Nộp bài dự thi]  (hiện khi là member + challenge ACTIVE)│  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
              │
              ▼ (Tap vào bài dự thi)
┌─────────────────────────────────────────────────────────────────────┐
│                   PROJECT DETAIL SCREEN                              │
│  (Reuse existing screen, thêm challenge info section)               │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ 🏆 Bài dự thi cho challenge:                                    ││
│  │    "Thiết kế phòng khách Bắc Âu"  [→]                           ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  [Existing project detail content...]                               │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2. Flow Nộp Bài (Member - Cần auth + member check)

**Logic kiểm tra quyền nộp bài:**
1. Kiểm tra user đã đăng nhập chưa
2. Nếu đã đăng nhập → Đọc trực tiếp `challenge.isMember` từ API response
3. Chỉ cho phép navigate sang Submit Screen khi cả 2 điều kiện đều thỏa mãn

> **QUAN TRỌNG:** Backend là **single source of truth** cho membership. App KHÔNG tự check email match domain rule — BE đã gộp cả `ChallengeMember` và `ChallengeDomainRule` vào field `isMember` (xem Task A.1.1). App chỉ consume boolean.

```
┌─────────────────────────────────────────────────────────────────────┐
│               CHALLENGE DETAIL SCREEN                                │
│                                                                      │
│  [... challenge info ...]                                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │              [Nộp bài dự thi]                                   ││
│  └─────────────────────────────────┬───────────────────────────────┘│
└────────────────────────────────────┼────────────────────────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
    (1. Chưa đăng nhập)    (2. Đã đăng nhập,        (3. Đã đăng nhập
              │             KHÔNG phải member)       + LÀ member)
              │                      │                      │
              ▼                      ▼                      ▼
┌─────────────────────┐  ┌─────────────────────────┐  ┌───────────────┐
│    LOGIN SCREEN     │  │     BOTTOM SHEET        │  │ SUBMIT SCREEN │
│ "Đăng nhập để       │  │     NOT A MEMBER        │  │               │
│  nộp bài"           │  │                         │  │  (xem 2.2.1)  │
└─────────────────────┘  │  ┌───────────────────┐  │  └───────────────┘
                         │  │        🔒         │  │
                         │  │                   │  │
                         │  │  Bạn chưa được    │  │
                         │  │  mời tham gia     │  │
                         │  │  challenge này    │  │
                         │  │                   │  │
                         │  │  Liên hệ admin    │  │
                         │  │  để được thêm vào │  │
                         │  │  danh sách thành  │  │
                         │  │  viên.            │  │
                         │  │                   │  │
                         │  │     [Đã hiểu]     │  │
                         │  └───────────────────┘  │
                         └─────────────────────────┘
```

**Lưu ý về Button "Nộp bài dự thi":**

| Trạng thái user | Trạng thái challenge | Hiển thị button | Hành vi khi tap |
|-----------------|---------------------|-----------------|-----------------|
| Chưa đăng nhập | ACTIVE | ✅ Hiện | → Login Screen |
| Đã đăng nhập, không phải member | ACTIVE | ✅ Hiện | → Bottom Sheet "Not a member" |
| Đã đăng nhập, là member, chưa nộp | ACTIVE | ✅ Hiện | → Submit Screen |
| Đã đăng nhập, là member, đã nộp | ACTIVE | ❌ Ẩn | Hiện "Bài đã nộp" section thay thế |
| Bất kỳ | ENDED/DRAFT | ❌ Ẩn | Không cho nộp |

### 2.2.1. Submit Screen Flow (Chỉ khi đã xác nhận là member)

```
                                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  CHALLENGE SUBMIT SCREEN                             │
│                                                                      │
│  ← Nộp bài dự thi                                                   │
│                                                                      │
│  Challenge: "Thiết kế phòng khách Bắc Âu"                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                        UPLOAD ẢNH                                ││
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                                ││
│  │  │ [+] │ │[img]│ │[img]│ │[img]│                                ││
│  │  │     │ │  ×  │ │  ×  │ │  ×  │                                ││
│  │  └─────┘ └─────┘ └─────┘ └─────┘                                ││
│  │  Tối thiểu 1 ảnh (chỉ hỗ trợ ảnh)                               ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Tiêu đề *                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Nhập tiêu đề bài dự thi...                                      ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Mô tả ý tưởng *                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Giải thích ý tưởng sáng tạo của bạn...                          ││
│  │                                                                 ││
│  │                                                                 ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Ghi chú hướng tiếp cận (tùy chọn)                                  │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Phương pháp, công cụ, cảm hứng...                               ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Mô tả đáp ứng chủ đề (tùy chọn)                                    │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Giải thích cách bài đáp ứng đề bài...                           ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │                      [Nộp bài]                                   ││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼ (Submit thành công)
┌─────────────────────────────────────────────────────────────────────┐
│                  SUCCESS MODAL/SCREEN                                │
│                                                                      │
│                         ✅                                           │
│                                                                      │
│            Nộp bài thành công!                                       │
│                                                                      │
│    Bài dự thi của bạn đang chờ duyệt.                               │
│    Bạn sẽ nhận được thông báo khi bài được                          │
│    duyệt hoặc có phản hồi.                                          │
│                                                                      │
│              [Xem bài đã nộp]                                        │
│              [Quay lại challenge]                                    │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3. Flow Xem Trạng Thái Bài Đã Nộp (Member)

```
┌─────────────────────────────────────────────────────────────────────┐
│              CHALLENGE DETAIL SCREEN (Member đã nộp bài)             │
│                                                                      │
│  [... challenge info ...]                                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  📄 Bài dự thi của bạn                                          ││
│  │  ┌───────┐                                                      ││
│  │  │[Image]│  "Tiêu đề bài"                                       ││
│  │  │       │  Trạng thái: [🟡 Chờ duyệt]                          ││
│  │  └───────┘                     hoặc [🟢 Đã duyệt]               ││
│  │                                hoặc [🔴 Từ chối]                ││
│  │                                                                 ││
│  │  (Nếu bị từ chối, hiện lý do:)                                  ││
│  │  💬 Lý do: "Bài chưa đáp ứng yêu cầu về..."                     ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  [Bạn đã nộp bài cho challenge này]  (disabled button)          ││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────┘
```

### 2.4. Flow Badge Challenge Trên Feed

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FEED SCREEN                                  │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  ┌──────────────────┐  ┌──────────────────┐                     ││
│  │  │    [Project]     │  │    [Project]     │                     ││
│  │  │   ┌──────────┐   │  │   ┌──────────┐   │                     ││
│  │  │   │🏆Challenge│   │  │   │          │   │                     ││
│  │  │   └──────────┘   │  │   │          │   │                     ││
│  │  │                  │  │   │          │   │                     ││
│  │  │   Project Title  │  │   Project Title  │                     ││
│  │  │   Author Name    │  │   Author Name    │                     ││
│  │  └──────────────────┘  └──────────────────┘                     ││
│  │                                                                 ││
│  │  Badge "Challenge" hiện ở góc trên của project card             ││
│  │  khi project.challengeSubmission != null                        ││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. UI Draft - Chi Tiết Màn Hình

> **Navigation Note:** Challenge tab nằm trong **top SliverAppBar** (cùng Feed, Create, AI Chat, Community),
> KHÔNG phải bottom navigation. Ref: `screens/feed/components/sliver-app-bar/tab-button.tsx`
>
> Wireframes dưới đây hiện footer navigation để minh họa vị trí tab, nhưng implementation thực tế
> sẽ nằm ở top bar theo pattern hiện tại của app.

### 3.1. Challenge List Screen

**Design notes:**
- Cover image dạng **landscape banner** (aspect ratio 16:9 hoặc 2:1)
- Mỗi challenge card chiếm **full-width** (1 card = 1 row)
- Text overlay trên ảnh với gradient tối ở bottom
- Style tham khảo: Pexels Challenges
- **Không có search** - chỉ filter tabs (API không support search)

```
┌──────────────────────────────────────────────────┐
│ ←  Challenge                              🔔     │
├──────────────────────────────────────────────────┤
│                                                  │
│ [Đang diễn ra]  [Đã kết thúc]                   │
│  ─────────────                                   │
│                                                  │
│ ┌──────────────────────────────────────────────┐ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░ COVER IMAGE (16:9) ░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ │
│ │▓▓ Thiết kế phòng khách Bắc Âu          ▓▓▓▓▓▓│ │
│ │▓▓ 📅 10/04 - 30/04  👥 12  📝 5        ▓▓▓▓▓▓│ │
│ └──────────────────────────────────────────────┘ │
│                                                  │
│ ┌──────────────────────────────────────────────┐ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░ COVER IMAGE (16:9) ░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ │
│ │▓▓ Chụp ảnh kiến trúc đương đại         ▓▓▓▓▓▓│ │
│ │▓▓ 📅 15/04 - 15/05  👥 8   📝 3        ▓▓▓▓▓▓│ │
│ └──────────────────────────────────────────────┘ │
│                                                  │
│ ┌──────────────────────────────────────────────┐ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░ COVER IMAGE (16:9) ░░░░░░░░░░░░░░░│ │
│ │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ │
│ │▓▓ Tranh trừu tượng trong không gian    ▓▓▓▓▓▓│ │
│ └──────────────────────────────────────────────┘ │
│                     ...                          │
├──────────────────────────────────────────────────┤
│  Feed | Create | AI | Challenge | Community      │
└──────────────────────────────────────────────────┘

Legend:
░░░ = Cover image area
▓▓▓ = Gradient overlay (dark) với text trắng
```

**Challenge Card Component specs:**
- Aspect ratio: **16:9** (hoặc 2:1 nếu muốn compact hơn)
- Border radius: 12px
- Gradient overlay: linear-gradient từ transparent → rgba(0,0,0,0.7)
- Text color: white
- Spacing giữa cards: 16px
- Horizontal padding: 16px

### 3.2. Challenge Detail Screen

**Design notes:**
- Cover image hero: **16:9 landscape**, full-width, có thể parallax scroll
- Back button overlay trên ảnh (transparent background)

```
┌────────────────────────────────────────┐
│ ← (overlay trên ảnh)           ⋮       │
│ ┌────────────────────────────────────┐ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░ COVER IMAGE HERO (16:9) ░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ │
│ └────────────────────────────────────┘ │
│                                        │
│ Thiết kế phòng khách Bắc Âu            │
│                                        │
│ 📅 10/04/2026 - 30/04/2026             │
│ 👥 12 thành viên  |  📝 5 bài dự thi   │
│                                        │
│ ┌────────────────┬───────────────────┐ │
│ │   Thông tin    │    Bài dự thi     │ │
│ │   ─────────    │                   │ │
│ └────────────────┴───────────────────┘ │
│                                        │
│ 📋 Mô tả                               │
│ Tạo không gian phòng khách theo        │
│ phong cách Bắc Âu, tối giản nhưng      │
│ ấm cúng...                             │
│                                        │
│ 🎯 Mục tiêu                            │
│ • Thể hiện sự sáng tạo trong thiết kế  │
│ • Tối ưu không gian sử dụng            │
│                                        │
│ 📝 Yêu cầu                             │
│ • Tối thiểu 3 góc chụp khác nhau       │
│ • Render chất lượng cao                │
│                                        │
│ ⚖️ Luật chơi                           │
│ • Mỗi thành viên chỉ được nộp 1 bài    │
│ • Không sử dụng ảnh từ nguồn khác      │
│                                        │
│ 🏆 Hạng mục                            │
│ • Best Minimalist Approach             │
│ • Most Creative Solution               │
│                                        │
├────────────────────────────────────────┤
│       ┌──────────────────────┐         │
│       │   Nộp bài dự thi     │         │
│       └──────────────────────┘         │
└────────────────────────────────────────┘
```

### 3.3. Challenge Detail - Tab Bài Dự Thi

```
┌────────────────────────────────────────┐
│ ←                              ⋮       │
├────────────────────────────────────────┤
│ [... header info ...]                  │
│                                        │
│ ┌────────────────┬───────────────────┐ │
│ │   Thông tin    │    Bài dự thi     │ │
│ │                │    ───────────    │ │
│ └────────────────┴───────────────────┘ │
│                                        │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │  [Image] │ │  [Image] │ │  [Image] │ │
│ │          │ │          │ │          │ │
│ │ Title    │ │ Title    │ │ Title    │ │
│ │ @author  │ │ @author  │ │ @author  │ │
│ └──────────┘ └──────────┘ └──────────┘ │
│                                        │
│ ┌──────────┐ ┌──────────┐              │
│ │  [Image] │ │  [Image] │              │
│ │          │ │          │              │
│ │ Title    │ │ Title    │              │
│ │ @author  │ │ @author  │              │
│ └──────────┘ └──────────┘              │
│                                        │
│         [Load more...]                 │
│                                        │
├────────────────────────────────────────┤
│       ┌──────────────────────┐         │
│       │   Nộp bài dự thi     │         │
│       └──────────────────────┘         │
└────────────────────────────────────────┘
```

### 3.4. Challenge Submit Screen

```
┌────────────────────────────────────────┐
│ ←  Nộp bài dự thi                      │
├────────────────────────────────────────┤
│                                        │
│ 🏆 Thiết kế phòng khách Bắc Âu         │
│                                        │
│ ── Ảnh bài dự thi * ──────────────────│
│                                        │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐   │
│ │  +   │ │[img] │ │[img] │ │[img] │   │
│ │ Add  │ │  ×   │ │  ×   │ │  ×   │   │
│ └──────┘ └──────┘ └──────┘ └──────┘   │
│ Tối thiểu 1 ảnh                        │
│                                        │
│ ── Tiêu đề * ─────────────────────────│
│ ┌────────────────────────────────────┐ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ── Mô tả ý tưởng * ───────────────────│
│ ┌────────────────────────────────────┐ │
│ │                                    │ │
│ │                                    │ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ── Ghi chú hướng tiếp cận ────────────│
│ ┌────────────────────────────────────┐ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│                                        │
│ ── Mô tả đáp ứng chủ đề ──────────────│
│ ┌────────────────────────────────────┐ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│                                        │
├────────────────────────────────────────┤
│       ┌──────────────────────┐         │
│       │      Nộp bài         │         │
│       └──────────────────────┘         │
└────────────────────────────────────────┘
```

---

## 4. Kế Hoạch Thực Thi

> **QUAN TRỌNG:** Thực hiện Backend Phase (A) hoàn tất trước khi bắt đầu App Phase (B).
> App cần API response đầy đủ để implement đúng các flow.

---

## PHASE A: BACKEND (riz-be) — ✅ HOÀN THÀNH (2026-04-14, hotfix 2026-04-15)

### A.1. Bổ sung API cho App (2-3 ngày) — Done

#### Task A.1.1: Enrich Challenge Detail Response — ✅ Done

**File:** `libs/challenge/src/challenge.service.ts`, `challenge.controller.ts`, `entities/challenge.entity.ts`

Khi user đã auth, `GET /challenges/:id` trả thêm các field dưới:

```typescript
{
  ...existingFields,

  // User-specific state (chỉ khi có auth — Absent cho unauthenticated)
  isMember: boolean,
  mySubmission: {
    id: string,
    projectId: string,
    title: string,
    imageUrl: string,
    status: 'PENDING' | 'APPROVED' | 'REJECTED',
    rejectionReason: string | null,
    submittedAt: string,
  } | null,
}
```

**Implementation:**
- [x] `getChallengeById(id, includeDraft, userProfileId?)` nhận optional `userProfileId` ([challenge.service.ts:594](riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L594))
- [x] Query `ChallengeMember` + `ChallengeDomainRule` (fallback theo domain email của profile) → set `isMember` trong helper `resolveUserChallengeState` ([challenge.service.ts:198](riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L198))
- [x] Query `ChallengeSubmission` của user kèm `project.global.attachments` cho imageUrl → set `mySubmission`
- [x] Thêm class `ChallengeMySubmissionEntity` và 2 field `isMember?`, `mySubmission?` vào `ChallengeEntity` ([challenge.entity.ts:15-42](riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L15-L42))
- [x] Controller pass `user?.profileId` từ `@CurUser()` vào service ([challenge.controller.ts:45-50](riz-be/apps/nest/libs/challenge/src/challenge.controller.ts#L45-L50))

#### Task A.1.2: Expose Challenge Relation trong Project — ✅ Done

**File:** `libs/project/src/entities/project.entity.ts`, `project.service.ts`, `helpers/project-mapper.helper.ts`, `admin-trending.service.ts`

Feed trả field `challengeSubmission` trên mọi ProjectEntity:

```typescript
{
  ...existingFields,

  challengeSubmission: {
    challengeId: string,
    challengeTitle: string,
    status: 'PENDING' | 'APPROVED' | 'REJECTED',
  } | null,
}
```

**Implementation:**
- [x] Thêm class `ProjectChallengeSubmissionEntity` + field `challengeSubmission?` vào `ProjectEntity` ([project.entity.ts:7-19](riz-be/apps/nest/libs/project/src/entities/project.entity.ts#L7-L19))
- [x] `ProjectMapperHelper.mapProjectData` map `project.challengeSubmission` → null-safe object với title lấy từ include `challenge.title` ([project-mapper.helper.ts](riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts))
- [x] Thêm `include.challengeSubmission.include.challenge.select.title` vào:
  - `getAllProjects` ([project.service.ts:159](riz-be/apps/nest/libs/project/src/project.service.ts#L159))
  - `getBookmarkedTemplates` ([project.service.ts:263](riz-be/apps/nest/libs/project/src/project.service.ts#L263))
  - `getProject` ([project.service.ts:380](riz-be/apps/nest/libs/project/src/project.service.ts#L380))
  - `AdminTrendingService.getTrendingFeed` (3 query blocks — trending/organic page 1, organic page 2+) ([admin-trending.service.ts](riz-be/apps/nest/libs/project/src/admin-trending.service.ts))
- [x] `mapToProjectEntity` trong `ProjectService` map challengeSubmission vào output ([project.service.ts:1093-1099](riz-be/apps/nest/libs/project/src/project.service.ts#L1093-L1099))

#### Task A.1.3: Viết Tests — ✅ Done

**File:** `libs/challenge/src/challenge.spec.ts` (mới)

- [x] Test `GET /challenges/:id` unauthenticated → `isMember`/`mySubmission` absent
- [x] Test authenticated non-member → `isMember=false`, `mySubmission=null`
- [x] Test `ChallengeMember` → `isMember=true`
- [x] Test `ChallengeDomainRule` → email match domain → `isMember=true`
- [x] Test `mySubmission` populated sau khi member submit (status PENDING, projectId, title)
- [x] Test project response có `challengeSubmission` khi project là submission
- [x] Test project thường → `challengeSubmission=null`

> Chạy tests (cần Postgres + Redis dev services): `cd riz-be/apps/nest && pnpm test -- --testPathPattern="challenge"`

### A.2. Summary — Changed Files (Phase A)

| File | Change |
|------|--------|
| `libs/challenge/src/challenge.controller.ts` | `getChallengeById` nhận `@CurUser() user?` |
| `libs/challenge/src/challenge.service.ts` | Thêm `resolveUserChallengeState`, extend `mapChallengeEntity`, `getChallengeById`; map `projectId` trong `mapSubmissionEntity` (hotfix 2026-04-15) |
| `libs/challenge/src/entities/challenge.entity.ts` | Thêm `ChallengeMySubmissionEntity`, `isMember?`, `mySubmission?`; thêm `projectId` vào `ChallengeSubmissionEntity` (hotfix 2026-04-15) |
| `libs/challenge/src/challenge.spec.ts` | Tests mới cho detail endpoint + project challengeSubmission |
| `libs/project/src/entities/project.entity.ts` | Thêm `ProjectChallengeSubmissionEntity` + field `challengeSubmission?` |
| `libs/project/src/helpers/project-mapper.helper.ts` | Extend type + mapping cho `challengeSubmission` |
| `libs/project/src/project.service.ts` | Add include trong 3 query + map field trong `mapToProjectEntity` |
| `libs/project/src/admin-trending.service.ts` | Add include vào 3 query block |

### A.4. Hotfix 2026-04-15 — `projectId` trong submission

**Vấn đề:** `GET /challenges/:id/submissions` không trả `projectId`, app không thể navigate sang `/art-feed?projectId=...` khi tap submission ở gallery.

**Fix:**
- [entities/challenge.entity.ts:240-242](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L240-L242) — thêm field `projectId: string` (sau `challengeId`)
- [challenge.service.ts:306](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L306) — `mapSubmissionEntity` set `projectId: row.project.id.toString()`

**Risk:** LOW (additive field, không breaking). gitnexus_impact + detect_changes confirmed no affected processes.

### A.5. Bugfix 2026-04-16 — Query invalidation + soft-delete guard

**Vấn đề 1 (CRITICAL):** Sau khi submit bài, `queryClient.invalidateQueries` dùng `CHALLENGE_QUERY_KEYS.detail(id)` → key `["challenges", "detail", id, undefined]` KHÔNG match cache key `["challenges", "detail", id, userId]` vì React Query partial matching thấy `undefined !== userId`. Kết quả: detail screen không refresh sau submit, user không thấy `mySubmission`.

**Vấn đề 2 (MEDIUM):** Thiếu invalidation cho challenge list → `submissionCount` trên card ở list screen stale sau submit.

**Vấn đề 3 (MEDIUM):** `resolveUserChallengeState` không check `project.deletedAt` trước khi map `mySubmission` → nếu project bị soft-delete, user thấy phantom submission.

**Fix:**
- [challenge-submit.tsx:88-90](../../riz-app-v2/screens/challenges/challenge-submit.tsx#L88-L90) — thay 2 invalidation riêng lẻ bằng `CHALLENGE_QUERY_KEYS.all` (prefix `["challenges"]` match tất cả challenge queries)
- [challenge.service.ts:242](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L242) — thêm guard `!submission.project.deletedAt` trước khi map mySubmission

**Risk:** LOW (fix-only, không thay đổi API contract hay data shape).

### A.3. Verification

- [x] TypeScript typecheck: no new errors introduced (chỉ còn pre-existing errors trong bookmark/post/trending/user spec files — không liên quan)
- [x] ESLint `--fix`: clean trên tất cả file đã sửa
- [x] `gitnexus_detect_changes`: scope đúng 8 file + 13 affected processes (tất cả là additive/optional — risk thực tế LOW)
- [x] Backward compatible: fields mới đều optional, không breaking change

---

## PHASE B: MOBILE APP (riz-app-v2)

> **Trạng thái:** B.1-B.5 ✅ Done (2026-04-16).

### B.1. Foundation (2-3 ngày) — ✅ Done

#### Task B.1.1: Setup API Layer

**Theo convention:** `lib/api/<feature>/index.ts + types.ts` với barrel export

```
lib/api/challenges/
├── index.ts          # API functions + query keys
└── types.ts          # TypeScript types
```

**File `lib/api/challenges/types.ts`:**
```typescript
export interface Challenge {
  id: string;
  title: string;
  description: string;
  objective: string;
  requirements: string;
  rules: string;
  coverImageUrl: string;
  startDate: string;
  endDate: string;
  feedCategory: number;
  subCategory?: number;
  awardCategories: { id: string; name: string }[];
  status: 'DRAFT' | 'ACTIVE' | 'ENDED' | 'ARCHIVED';
  submissionCount: number;
  memberCount: number;
  // User-specific (từ BE A.1.1)
  isMember?: boolean;
  mySubmission?: MySubmission | null;
}

export interface MySubmission {
  id: string;
  status: 'PENDING' | 'APPROVED' | 'REJECTED';
  rejectionReason: string | null;
  submittedAt: string;
  projectId: string;
}

// Public list API chỉ trả APPROVED (non-admin)
export interface PublicChallengeSubmission {
  id: string;
  title: string;
  description: string;
  imageUrl: string;
  author: { id: string; name: string; avatar?: string };
  status: 'APPROVED';
  submittedAt: string;
}

// Response khi submit() — map 1:1 với BE ChallengeSubmissionEntity
// Status có thể là PENDING ngay sau khi nộp
export interface ChallengeSubmissionResponse {
  id: string;
  challengeId: string;
  title: string;
  description: string;
  imageUrl: string;
  author: { id: string; name: string; avatar?: string };
  status: 'PENDING' | 'APPROVED' | 'REJECTED';
  rejectionReason?: string;
  submittedAt: string;
  reviewedAt?: string;
}

export interface GetChallengesParams {
  status?: 'ACTIVE' | 'ENDED';
  feedCategory?: number;
  page?: number;
  limit?: number;
}
```

**File `lib/api/challenges/index.ts`:**
```typescript
import { api } from "@/lib/axios";
import type {
  Challenge,
  PublicChallengeSubmission,
  ChallengeSubmissionResponse,
  GetChallengesParams,
} from "./types";

export const CHALLENGE_QUERY_KEYS = {
  all: ["challenges"] as const,
  list: (params?: GetChallengesParams) => [...CHALLENGE_QUERY_KEYS.all, "list", params] as const,
  detail: (id: string) => [...CHALLENGE_QUERY_KEYS.all, "detail", id] as const,
  submissions: (id: string) => [...CHALLENGE_QUERY_KEYS.all, "submissions", id] as const,
};

export const challengesApi = {
  list: (params?: GetChallengesParams) =>
    api.get<{ data: Challenge[]; pagination: any }>("/challenges", { params }).then(r => r.data),
  
  getById: (id: string) =>
    api.get<Challenge>(`/challenges/${id}`).then(r => r.data),
  
  getSubmissions: (id: string) =>
    api.get<PublicChallengeSubmission[]>(`/challenges/${id}/submissions`).then(r => r.data),
  
  submit: (id: string, formData: FormData) =>
    api.post<ChallengeSubmissionResponse>(`/challenges/${id}/submissions`, formData, {
      headers: { "Content-Type": "multipart/form-data" },
    }).then(r => r.data),
};

export * from "./types";
```

#### Task B.1.2: Setup Hooks

**Theo convention:** `hooks/challenges/` với barrel export

```
hooks/challenges/
├── index.ts
├── use-challenges.ts         # Query hooks
└── use-challenge-mutations.ts # Mutation hooks
```

#### Task B.1.3: Navigation Integration

**Files cần sửa:**
- `components/icons/challenge-icon.tsx` (new)
- `components/icons/index.ts` — **thêm `export { ChallengeIcon }` vào barrel** ([index.ts:1](riz-app-v2/components/icons/index.ts#L1))
- `lib/api/index.ts` — **thêm `export * from "./challenges"` vào barrel** ([index.ts:1](riz-app-v2/lib/api/index.ts#L1))
- `screens/feed/components/sliver-app-bar/tab-button.tsx` - thêm tab challenge
- `screens/feed/index.tsx` - thêm case "challenge" trong `handleTabPress`, navigate `router.push('/challenges')`
- `integrations/react-intl/locales/*.json` - thêm translations

**Convention note:**
- Pattern `lib/api/<feature>/` đã có tiền lệ ([lib/api/projects/](riz-app-v2/lib/api/projects/)).
- Pattern `hooks/<feature>/` là convention **đang chuyển đổi** — repo hiện còn các hook root-level như [hooks/use-projects.ts](riz-app-v2/hooks/use-projects.ts). Challenge feature follow convention mới (feature folder) để thống nhất về sau.

**Routes mới:**
- `app/challenges.tsx` (public - NGOÀI protected)
- `app/challenges/[id].tsx` (public - NGOÀI protected)
- `app/(protected)/challenges/[id]/submit.tsx` (protected)

### B.2. List & Detail Screens (3-4 ngày) — ✅ Done

#### Task B.2.1: Challenge List Screen — ✅ Done

**File:** [screens/challenges/index.tsx](../../riz-app-v2/screens/challenges/index.tsx)

- [x] Full-width card với cover image 16:9 ([challenge-card.tsx](../../riz-app-v2/screens/challenges/components/challenge-card.tsx))
- [x] Filter tabs: Đang diễn ra / Đã kết thúc ([challenge-filter-tabs.tsx](../../riz-app-v2/screens/challenges/components/challenge-filter-tabs.tsx))
- [x] Infinite scroll với React Query (`useChallenges` + page-based `getNextPageParam`)
- [x] Pull to refresh (`RefreshControl` + `refetch`)

#### Task B.2.2: Challenge Detail Screen — ✅ Done

**File:** [screens/challenges/challenge-detail.tsx](../../riz-app-v2/screens/challenges/challenge-detail.tsx)

- [x] Cover image hero 16:9 ([challenge-hero.tsx](../../riz-app-v2/screens/challenges/components/challenge-hero.tsx)) — parallax bỏ qua (post-MVP)
- [x] Tabs: Thông tin / Bài dự thi (state local, custom segmented control)
- [x] Gallery grid cho submissions ([challenge-submissions-tab.tsx](../../riz-app-v2/screens/challenges/components/challenge-submissions-tab.tsx)) — 2-column flex grid
- [x] "My Submission" section khi `mySubmission != null` ([my-submission-card.tsx](../../riz-app-v2/screens/challenges/components/my-submission-card.tsx))
- [x] Submit button hiển thị khi `status === ACTIVE` && không có `mySubmission`. Logic auth/member check tạm để TODO(B.3) — button navigate tới `/challenges/[id]/submit` (route chưa tồn tại)

**Lưu ý về Project Detail:**
App không có "project detail screen" riêng - dùng `art-feed` với `projectId` param.
Khi tap vào submission → navigate `/art-feed?projectId={projectId}`. Cần BE trả `projectId` trong `ChallengeSubmissionEntity` (đã fix 2026-04-15, xem A.4).

### B.3. Submit Flow (3-4 ngày) — ✅ Done

#### Task B.3.1: Submit Screen — ✅ Done

**File:** [screens/challenges/challenge-submit.tsx](../../riz-app-v2/screens/challenges/challenge-submit.tsx)

- [x] Form với title, description (required), approachNote, themeResponse (optional)
- [x] Image picker reusing `AddImagesBottomSheet` + `useCropFlow` (max 5 ảnh)
- [x] FormData multipart upload qua `useSubmitChallenge` mutation
- [x] Success alert + navigate back, error alert
- [x] Keyboard avoiding, validation (title + description + ≥1 image)
- [x] Route: `app/(protected)/challenges/[id]/submit.tsx`

#### Task B.3.2: Member Check & Auth Flow — ✅ Done

**File:** [hooks/challenges/use-can-submit.ts](../../riz-app-v2/hooks/challenges/use-can-submit.ts)

- [x] `useCanSubmit(challenge)` hook xử lý 3 case: unauthenticated → login, not member → bottom sheet, member → navigate submit
- [x] `NotMemberBottomSheet` component ([not-member-bottom-sheet.tsx](../../riz-app-v2/screens/challenges/components/not-member-bottom-sheet.tsx))
- [x] Tích hợp vào `ChallengeDetailScreen` thay thế TODO(B.3) cũ

#### Task B.3.3: My Submission Status UI — ✅ Done (trong B.2)

Đã implement trong B.2 với `MySubmissionCard` component.

### B.4. Feed Integration (2-3 ngày) — ✅ Done

#### Task B.4.1: Update Project Types — ✅ Done

**File:** [lib/api/projects/types.ts](../../riz-app-v2/lib/api/projects/types.ts)

- [x] Thêm `ProjectChallengeSubmission` interface
- [x] Thêm `challengeSubmission?` field vào `ProjectResponse`

#### Task B.4.2: Challenge Badge Component — ✅ Done

**File:** [components/challenge-badge.tsx](../../riz-app-v2/components/challenge-badge.tsx)

- [x] Trophy icon + "Challenge" text badge (amber color, rounded)

#### Task B.4.3: Feed Item Integration — ✅ Done

**Files:**
- [screens/feed/types/types.ts](../../riz-app-v2/screens/feed/types/types.ts) — thêm `challengeSubmission?` vào `MasonryItemData`
- [screens/feed/helper/transform.ts](../../riz-app-v2/screens/feed/helper/transform.ts) — map `challengeSubmission` từ `ProjectResponse`
- [screens/feed/components/main/feed-item.tsx](../../riz-app-v2/screens/feed/components/main/feed-item.tsx) — hiển thị `ChallengeBadge` ở góc trên-trái khi có challengeSubmission

### B.5. Polish & Testing (2-3 ngày) — ✅ Done

#### Task 5.1: Loading States & Error Handling — ✅ Done

- [x] Skeleton loaders cho list và detail (3 skeleton components: `ChallengeListSkeleton`, `ChallengeDetailSkeleton`, `SubmissionsTabSkeleton`)
- [x] Error states với icon, subtitle, và retry button (Trophy icon cho error, cải thiện layout)
- [x] Empty states với icon, title, subtitle (Trophy icon cho empty list, ImageOff cho empty submissions)

**Files mới:**
- [challenge-list-skeleton.tsx](../../riz-app-v2/screens/challenges/components/challenge-list-skeleton.tsx) — Skeleton 3 cards 16:9
- [challenge-detail-skeleton.tsx](../../riz-app-v2/screens/challenges/components/challenge-detail-skeleton.tsx) — Hero + meta + tabs + content skeleton
- [submissions-tab-skeleton.tsx](../../riz-app-v2/screens/challenges/components/submissions-tab-skeleton.tsx) — 2-column grid skeleton

#### Task 5.2: Animations & UX — ✅ Done

- [x] Tab switching animation — Reanimated `withTiming` sliding indicator (250ms) cho cả detail tabs và filter tabs
- [x] Image upload preview animation — `FadeIn`/`FadeOut` entering/exiting trên image previews
- [x] Submit success animation/modal — Custom `SubmitSuccessModal` với `ZoomIn` icon + `FadeIn` content (thay thế native Alert)

**Files mới:**
- [submit-success-modal.tsx](../../riz-app-v2/screens/challenges/components/submit-success-modal.tsx) — Animated success modal với CircleCheck icon

**Files sửa:**
- `challenge-detail.tsx` — Animated tab indicator, skeleton loading, improved error state
- `challenge-filter-tabs.tsx` — Animated tab indicator (full rewrite với Reanimated)
- `challenge-submit.tsx` — FadeIn/FadeOut image previews, success modal thay Alert
- `challenge-submissions-tab.tsx` — Skeleton loading, improved empty state
- `index.tsx` (list) — Skeleton loading, improved error/empty states

**i18n keys mới:**
- `challenges_error_subtitle`, `challenges_empty_subtitle`, `challenge_empty_submissions_subtitle`

#### Task 5.3: Testing
- [ ] Test các flow chính (manual testing required)
- [ ] Test edge cases (no challenges, empty submissions, etc.)
- [ ] Test responsive trên các device khác nhau

---

## 5. Cấu Trúc Thư Mục

### 5.1. Backend (riz-be)

```
riz-be/apps/nest/libs/challenge/src/
├── entities/
│   └── challenge.entity.ts    # Update: thêm isMember, mySubmission
├── challenge.service.ts       # Update: enrich getChallengeById
└── challenge.controller.ts    # Update: pass userId

riz-be/apps/nest/libs/project/src/
├── entities/
│   └── project.entity.ts      # Update: thêm challengeSubmission
└── project.service.ts         # Update: include relation
```

### 5.2. Mobile App (riz-app-v2)

**Theo convention:** `docs/conventions/folder-organize.md`

```
riz-app-v2/
├── app/
│   ├── challenges/
│   │   ├── index.tsx                      # Route: /challenges (PUBLIC)
│   │   └── [id].tsx                       # Route: /challenges/:id (PUBLIC)
│   └── (protected)/
│       └── challenges/
│           └── [id]/
│               └── submit.tsx             # Route: /(protected)/challenges/:id/submit
│
├── screens/
│   └── challenges/
│       ├── index.tsx                      # Challenge list screen
│       ├── challenge-detail.tsx           # Challenge detail screen
│       ├── challenge-submit.tsx           # Submit form screen
│       └── components/
│           ├── challenge-card.tsx
│           ├── challenge-filter-tabs.tsx
│           ├── challenge-hero.tsx
│           ├── challenge-info-tab.tsx
│           ├── challenge-submissions-tab.tsx
│           ├── submission-gallery-item.tsx
│           ├── my-submission-card.tsx
│           ├── not-member-bottom-sheet.tsx
│           ├── challenge-list-skeleton.tsx
│           ├── challenge-detail-skeleton.tsx
│           ├── submissions-tab-skeleton.tsx
│           └── submit-success-modal.tsx
│
├── lib/
│   └── api/
│       └── challenges/                    # Feature folder với barrel export
│           ├── index.ts                   # API functions + query keys
│           └── types.ts                   # TypeScript types
│
├── hooks/
│   └── challenges/                        # Feature folder với barrel export
│       ├── index.ts                       # Barrel export
│       ├── use-challenges.ts              # Query hooks
│       ├── use-challenge-mutations.ts     # Mutation hooks
│       └── use-can-submit.ts              # Submit permission logic
│
├── components/
│   ├── icons/
│   │   └── challenge-icon.tsx
│   └── challenge-badge.tsx                # Badge cho feed items
│
└── integrations/
    └── react-intl/
        └── locales/
            ├── en.json                    # Thêm challenge translations
            └── vi.json
```

**Lưu ý:**
- KHÔNG tạo `types/challenge.ts` riêng - đặt trong `lib/api/challenges/types.ts`
- KHÔNG tạo `challenge-search.tsx` - search là post-MVP
- Routes public đặt NGOÀI `(protected)/`

---

## 6. Ước Lượng Timeline

| Phase | Tasks | Thời gian | Phụ thuộc |
|-------|-------|-----------|-----------|
| **A.1** | BE: Enrich Challenge Detail | 1-2 ngày | - |
| **A.2** | BE: Project Challenge Relation | 1 ngày | - |
| **A.3** | BE: Tests | 0.5 ngày | A.1, A.2 |
| **--- BE Done ---** | | **2-3.5 ngày** | |
| **B.1** | App: Foundation (API, hooks, nav) | 2-3 ngày | A done |
| **B.2** | App: List & Detail | 3-4 ngày | B.1 |
| **B.3** | App: Submit Flow | 3-4 ngày | B.2 |
| **B.4** | App: Feed Integration | 2-3 ngày | A.2, B.2 |
| **B.5** | App: Polish & Testing | 2-3 ngày | B.1-B.4 |
| **--- App Done ---** | | **12-17 ngày** | |
| **Total** | | **14-20.5 ngày** | |

---

## 7. Dependencies & Risks

### 7.1. Dependencies

| Dependency | Status | Blocking |
|------------|--------|----------|
| BE: Challenge CRUD APIs | ✅ Done | - |
| BE: `isMember`, `mySubmission` trong detail | ✅ Done (A.1.1) | - |
| BE: `challengeSubmission` trong Project | ✅ Done (A.1.2) | - |
| App: `CameraRollGallery` component | ✅ Có sẵn | - |
| App: `useCropFlow` hook | ✅ Có sẵn | - |
| App: `art-feed` cho project detail | ✅ Có sẵn | - |

### 7.2. Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| BE changes delay | High | Làm BE trước, App sau - không làm song song |
| API response format không match | Medium | Verify với Swagger docs, test sớm |
| Image upload phức tạp | Low | Reuse `CameraRollGallery` + `useCropFlow` từ update-project |
| Navigation tab đông | Low | Có thể scroll horizontal hoặc redesign |
| Performance với nhiều images | Medium | Lazy loading, image optimization |

---

## 8. Acceptance Criteria

### 8.1. Backend Must Have
- [x] `GET /challenges/:id` trả `isMember` và `mySubmission` khi có auth
- [x] `ProjectEntity` có field `challengeSubmission`
- [x] Tests viết cho các endpoints mới (cần chạy integration: `pnpm test -- --testPathPattern="challenge"`)

### 8.2. App Must Have (MVP)
- [x] Tab Challenge trong top SliverAppBar
- [x] Xem danh sách challenge (full-width card, 16:9 cover)
- [x] Filter: Đang diễn ra / Đã kết thúc
- [x] Xem chi tiết challenge + gallery bài dự thi
- [x] Nộp bài dự thi (member only, với auth + member check) — B.3
- [x] Xem trạng thái bài đã nộp (PENDING/APPROVED/REJECTED + lý do)
- [x] Bottom sheet "Không phải member" khi không có quyền — B.3

### 8.3. App Should Have
- [x] Badge "Challenge" trên feed items — B.4
- [x] Navigate từ project → challenge detail (gallery → /art-feed)
- [x] Pull to refresh

### 8.4. Post-MVP (Out of Scope)
- [ ] Search challenges (API không support)
- [ ] Offline support
- [ ] Deep linking
- [ ] Share challenge

---

## 9. Notes Quan Trọng

### 9.1. UI Notes

**Navigation:**
- Challenge tab nằm trong **top SliverAppBar** (cùng với Feed, Create, AI Chat, Community)
- KHÔNG phải bottom navigation bar
- Tham khảo: `screens/feed/components/sliver-app-bar/tab-button.tsx`

**Project Detail:**
- App không có "project detail screen" riêng
- Dùng `art-feed` với `projectId` query param để focus item
- Khi tap submission → `router.push({ pathname: '/art-feed', params: { projectId } })`

### 9.2. API Notes

**Limitations:**
- `GET /challenges` không có `search` param - chỉ filter by `status`, `feedCategory`, `subCategory`
- `GET /challenges/:id/submissions` chỉ trả `APPROVED` submissions cho non-admin
- Cần BE changes để lấy `isMember`, `mySubmission` cho current user

**Upload format:**
```typescript
// Backend expects multipart field "images" as binary array
// Field name: "images" (plural)
// Ref: challenge.controller.ts:59-80
formData.append('images', { uri, type: 'image/jpeg', name } as any);
```

### 9.3. Convention Notes

**Folder structure:**
- API: `lib/api/challenges/index.ts + types.ts` (barrel export)
- Hooks: `hooks/challenges/index.ts + use-*.ts` (barrel export)
- KHÔNG đặt `types/challenge.ts` riêng

**Route naming:**
- Public routes: `/challenges`, `/challenges/[id]` (NGOÀI protected)
- Protected routes: `/(protected)/challenges/[id]/submit`

---

## 10. Checklist Trước Khi Bắt Đầu

### 10.1. Trước khi làm BE (Phase A)
- [ ] Đọc hiểu `challenge.service.ts` và `challenge.controller.ts`
- [ ] Hiểu flow `JwtOptionalGuard` để biết khi nào có userId
- [ ] Review `ProjectEntity` để biết cách thêm relation

### 10.2. Trước khi làm App (Phase B)
- [ ] Confirm BE Phase A đã deploy và test được
- [ ] Verify API response format qua Swagger/Postman
- [ ] Đọc `docs/conventions/folder-organize.md`
- [ ] Review `screens/update-project/components/add-images-modal.tsx` cho upload pattern
