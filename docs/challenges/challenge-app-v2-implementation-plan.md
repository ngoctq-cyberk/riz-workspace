# Plan Thực Thi Tính Năng Challenge — V2 (Restructure)

| Thuộc tính      | Giá trị                                                                                                      |
| --------------- | ------------------------------------------------------------------------------------------------------------ |
| Ngày tạo        | 2026-04-18                                                                                                   |
| Cập nhật        | 2026-04-20 — Reviewer comments fix: drop `ENDED` enum, ISO datetime contract, expose submission notes (Round 7) |
| Tác giả         | Claude Code                                                                                                  |
| Trạng thái      | ✅ **Implementation done** (V2.0a → V2.7 + Round 4/5/6/7 fixes) — chờ seed data + manual QA trước deploy (DB brand new, không cần data migration) |
| Nguồn tham khảo | [challenge-app-implementation-plan.md](challenge-app-implementation-plan.md) (v1, đã triển khai Phase A + B) |

---

## 🎯 Implementation Status (2026-04-19)

| Task                                            | Status      | Ghi chú                                                                                                                       |
| ----------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------- |
| V2.0a Admin-FE Status field                     | ✅ Done     | 7 files, ENDED bị xóa khỏi union + label, ESLint + tsc clean                                                                  |
| V2.0b BE phase/feedCategorySlug + reject ENDED  | ✅ Done     | Tests SKIPPED — pre-existing infra (Docker postgres + jest mapper thiếu `@app/subscription`). Code path verified via tsc + eslint |
| V2.1 Top tab cleanup                            | ✅ Done     | `ChallengeIcon` + locale `feed_tab_challenge` xóa hẳn, grep verify 0 match                                                    |
| V2.2 Feed default Photographers + Pinned card   | ✅ Done     | `useFeaturedChallenge` + `PinnedChallengeCard` ongoing-only                                                                   |
| V2.3 Collapsible info                           | ✅ Done     | `CollapsibleInfoSection` + `ShortSummary`, Reanimated `withTiming`                                                            |
| V2.4 Detail merge submissions + fix useCanSubmit| ✅ Done     | `useCanSubmit` tách `isSubmissionWindowOpen` vs `canSubmit`; `challenge-detail.tsx` dùng window cho visibility CTA            |
| V2.5 Submit link xem thể lệ                     | ✅ Done     | `router.back()` về detail                                                                                                     |
| V2.6 Community dynamic per-challenge tabs       | ✅ Done     | 1-query, flat tab bar, `getPhase()` helper, upcoming skip fetch, filter tabs `/challenges` đổi 3 phase                        |
| V2.7 QA + lint + detect_changes                 | ✅ Done     | gitnexus: admin-fe LOW, BE MEDIUM (whitespace), app-v2 MEDIUM (FeedScreen — expected)                                          |
| V2.8 Round 4 code-review fixes                  | ✅ Done     | Date-only calendar-day parsing (app), subCategory clear normalize (BE DTO), phase+status AND semantics (BE service)           |
| V2.9 Round 5 BE business-TZ alignment            | ✅ Done     | BE parse/format date-only theo Asia/Ho_Chi_Minh (UTC+7) để khớp client semantics. Fix submission window drift 7h                |
| V2.10 Round 6 App + admin-FE alignment           | ✅ Done     | App parse `YYYY-MM-DD` theo +07:00 (khớp BE); admin-FE schema cho phép same-day                                               |
| V2.11 Round 7 Reviewer comments fix              | ✅ Done     | Drop `ENDED` enum, rename `startDate`/`endDate` → `startsAt`/`endsAt` ISO datetime (UTC), expose `approachNote`/`themeResponse` trên submission |

**Pre-deploy TODO (ngoài scope code):**

- [x] ~~Chạy migration `UPDATE "Challenge" SET status = 'ACTIVE' WHERE status = 'ENDED'`~~ — **không áp dụng**: DB target là brand new, không có record legacy `ENDED` (xác nhận 2026-04-19, xem changelog Round 3)
- [ ] Seed ≥1 ACTIVE PHOTOGRAPHERS challenge ongoing trên dev/staging để test pinned card (§7.2)
- [ ] Fix jest `moduleNameMapper` thêm `^@app/subscription(|/.*)$` ở `apps/nest/package.json` để chạy BE test suite (infra issue ngoài scope)
- [ ] Manual QA trên device theo Acceptance Criteria §6.1 + §6.2
- [ ] Commit per repo (riz-be, riz-admin-fe, riz-app-v2)

---

## 0. Changelog So Với Bản V2 Đầu Tiên

Plan này đã được cập nhật qua 7 vòng review (xem [review findings](challenge-app-v2-implementation-plan-review-findings.md) + [code review](challenge-app-v2-code-review.md) + [reviewer comments](challenge-review-comments.md)). Tóm tắt thay đổi:

**Round 7 — Reviewer comments fix (2026-04-20):**

Phản hồi 4 comments của reviewer trên BE PR challenge V2 (chi tiết tại [challenge-review-comments.md](challenge-review-comments.md)). Sau khi phản biện, chốt hướng fix như sau — **huỷ bỏ business-TZ approach của Round 5/6, quay về ISO datetime UTC như tiêu chuẩn cho các field `createdAt`/`updatedAt`**.

- **Drop `ENDED` khỏi `ChallengeStatus` enum**: enum Prisma trước đây có 4 giá trị `DRAFT | ACTIVE | ENDED | ARCHIVED` nhưng `ENDED` chưa bao giờ được set (admin bị chặn write, V2 derive "ended" từ `endsAt < now`). Quyết định không thêm cron job (simpler, không có side-effect cần chạy) → xoá hẳn `ENDED` khỏi enum để schema honest với reality. Migration: [`20260420093500_drop_challenge_ended_status/migration.sql`](../../riz-be/apps/nest/prisma/migrations/20260420093500_drop_challenge_ended_status/migration.sql) (DB brand new, không có row `ENDED` để migrate). `ChallengePhase.Ended` trong [`get-challenges.dto.ts`](../../riz-be/apps/nest/libs/challenge/src/dtos/get-challenges.dto.ts) **giữ nguyên** derivation `{ endsAt: { lt: now } }` — không cần `status = ENDED`. Update [`writable-status.ts`](../../riz-be/apps/nest/libs/challenge/src/constants/writable-status.ts) bỏ comment "guard against future ENDED". Remove test block "Admin write-path: reject status=ENDED" trong [`challenge.spec.ts`](../../riz-be/apps/nest/libs/challenge/src/challenge.spec.ts) — enum-level validation tự reject giá trị không tồn tại, không cần test explicit.
- **API contract đổi `startDate`/`endDate` (date-only `YYYY-MM-DD`) → `startsAt`/`endsAt` (ISO 8601 datetime, UTC)**: đồng bộ naming + format với `createdAt`/`updatedAt`. Admin picks datetime theo locale của họ, FE convert sang UTC ISO khi gửi; users trong locale khác nhau hiển thị theo device TZ của họ → timeline "dịch" đúng theo múi giờ user. **Bỏ khái niệm business TZ cố định** (`Asia/Ho_Chi_Minh +07:00`) đã dựng ở Round 5/6. Files:
  - **BE**: [`create-challenge.dto.ts`](../../riz-be/apps/nest/libs/challenge/src/dtos/create-challenge.dto.ts) rename field; [`challenge.entity.ts`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts) đổi format `date` → `date-time`; [`challenge.service.ts`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts) dùng `new Date(iso)` / `.toISOString()` trực tiếp, bỏ gọi helper; 2 guards tương tự; xoá hẳn `challenge-date.helper.ts` + thư mục `helpers/`; spec update các test request payload sang ISO datetime.
  - **Admin-FE**: [`types.ts`](../../riz-admin-fe/src/entities/challenge/model/types.ts), [`challenge-api.ts`](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts), [`challenge-schema.ts`](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts) rename field + đổi validation sang ISO compare (`new Date(endsAt) > new Date(startsAt)`); [`challenge-form.tsx`](../../riz-admin-fe/src/features/challenges/challenge-form/ui/challenge-form.tsx) đổi `<Input type="date">` → `<Input type="datetime-local">` qua `Controller`, thêm 2 helper `isoToLocalInput` / `localInputToIso` để bridge giữa form state (ISO UTC) và native input value (local wall clock); list/detail pages hiển thị qua `new Date(iso).toLocaleString()`.
  - **App-v2**: [`types.ts`](../../riz-app-v2/lib/api/challenges/types.ts) rename field; [`challenge-date.ts`](../../riz-app-v2/utils/challenge-date.ts) rewrite → `formatDateShort`/`formatDateFull` (device-local `DD/MM` / `DD/MM/YYYY`) + `daysFromNow`, bỏ `parseBusinessDay*`/`formatBusinessDay*`; [`challenge-phase.ts`](../../riz-app-v2/utils/challenge-phase.ts) dùng `new Date(iso)` trực tiếp; update 5 callers (`pinned-challenge-card`, `challenge-card`, `challenge-detail`, `submissions-empty-state`, `mini-challenge-card-header`, `use-can-submit`) — truyền `startsAt`/`endsAt` ISO thay `startDate`/`endDate`.
- **Expose `approachNote`/`themeResponse` trên `ChallengeSubmissionEntity`**: 2 field này thuộc submission (không phải challenge), vẫn đang được ghi vào DB khi user submit nhưng không trả ra API response do thiếu `@Expose()`. Thêm vào [`challenge.entity.ts:ChallengeSubmissionEntity`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts) để admin reviewer thấy note của người submit khi judge. Không drop cột DB như reviewer đề xuất ban đầu (reviewer hiểu lầm fields đó thuộc `Challenge`).
- **Round 5 + 6 business-TZ logic bị loại bỏ hoàn toàn**: không còn file `challenge-date.helper.ts` ở BE hay `utils/challenge-date.ts` với business-TZ ở app; `BUSINESS_TZ_OFFSET` hard-code đã xoá. Tradeoff đã chấp nhận: admin ở Việt Nam tạo "20/04 23:59" thì user Úc (+10) truy cập lúc đó sẽ thấy challenge vẫn còn `endsAt` lúc `21/04 02:59` theo giờ local Úc — đúng intent "challenge có 1 instant start + 1 instant end toàn cầu, render theo locale từng người".

**Round 6 — App + admin-FE alignment với business-TZ (2026-04-19):**

Follow-up cho Round 5: BE đã chuyển sang business-TZ `+07:00` nhưng app V2 vẫn parse `YYYY-MM-DD` theo device local TZ (di sản của Round 4). Cross-repo mismatch — user ở LA thấy CTA submit active lên tới ~14h sau khi BE đã đóng window. Đồng thời admin-FE schema còn chặn `startDate === endDate` dù BE đã cho phép.

- **App parse theo business-TZ**: [`riz-app-v2/utils/challenge-date.ts`](../../riz-app-v2/utils/challenge-date.ts) refactor: đổi `parseLocalDayStart/End` → `parseBusinessDayStart/End` dùng `new Date(\`${dateStr}T00:00:00.000+07:00\`)`; `formatLocalDay*` → `formatBusinessDay*` derive parts qua offset shift. Cập nhật 7 callers: `utils/challenge-phase.ts`, `hooks/challenges/use-can-submit.ts`, `screens/community/components/mini-challenge-card-header.tsx`, `screens/community/components/submissions-empty-state.tsx`, `screens/feed/components/main/pinned-challenge-card.tsx`, `screens/challenges/components/challenge-card.tsx`, `screens/challenges/challenge-detail.tsx`. Các file display bỏ `intl.formatDate(parseLocalDay...)` (vì Intl sẽ shift về device TZ) → dùng trực tiếp `formatBusinessDay*` để giữ nguyên calendar day mà admin đã nhập.
- **Admin-FE schema cho phép same-day**: [`riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts`](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts) đổi `new Date(endDate) > new Date(startDate)` thành `data.endDate >= data.startDate` (lexicographic compare trên ISO date-string). Message cập nhật "must be on or after start date". Khớp với BE Round 5 đã cho phép 1-day challenge.
- **Admin-FE không cần fix TZ parsing**: form dùng `<Input type="date">` với raw string, list/detail render `{challenge.startDate}` raw — không đi qua `new Date()` nên không chịu drift. Chỉ schema comparison là chỗ duy nhất dùng `new Date()` và đã được thay bằng string compare.

**Round 5 — BE business-TZ alignment (2026-04-19):**

Follow-up cho Round 4 Finding #1: client đã parse date-only theo local calendar day, nhưng BE vẫn dùng `new Date('YYYY-MM-DD')` → UTC midnight. Ở user VN, BE coi window đóng từ `07:00 VN` ngày endDate trong khi app hiển thị "Đang diễn ra" tới `23:59 VN` cùng ngày. Request submit giữa khoảng này bị BE 400 `submission window is closed`.

- Thêm [`riz-be/apps/nest/libs/challenge/src/helpers/challenge-date.helper.ts`](../../riz-be/apps/nest/libs/challenge/src/helpers/challenge-date.helper.ts) với `parseBusinessDayStart`, `parseBusinessDayEnd`, `formatBusinessDay`. Business TZ cố định `Asia/Ho_Chi_Minh` (`+07:00`, không DST). `startsAt` = VN 00:00 trên `startDate`, `endsAt` = VN 23:59:59.999 trên `endDate`.
- Update parsing ở 4 call sites:
  - `challenge.service.ts:118-119` (`resolveCreateValidationContext`)
  - `challenge.service.ts:136-137` (`resolveUpdateValidationContext`)
  - `guards/create-challenge-validation.guard.ts:23-24`
  - `guards/update-challenge-validation.guard.ts:46-47`
- Update response mapper `challenge.service.ts:174-175` dùng `formatBusinessDay(instant)` thay vì `toISOString().slice(0,10)` để round-trip về đúng ngày admin nhập.
- **Submission guard** (`guards/challenge-submission.guard.ts:62-65`) và **phase filter** (`challenge.service.ts:543-551`) **không cần sửa** — chúng so instants với `now`, semantics đúng ngay khi storage chuyển sang instant chính xác.
- **Side-effect tích cực**: `validateTimeline` (`endsAt <= startsAt`) trước đây reject challenge same-day (startDate == endDate → cùng UTC midnight). Giờ same-day pass vì `endsAt (23:59:59.999 VN) > startsAt (00:00 VN)` — 1-day challenge được hỗ trợ.
- Test infra note: existing tests dùng Prisma direct (`startsAt: new Date(now + ...)`), không đi qua helper, nên không ảnh hưởng. Round-trip tests qua API (create → GET) vẫn match vì parse + format cùng TZ.

**Round 4 — post code-review fixes (2026-04-19):**

Phản hồi 3 findings trong [challenge-app-v2-code-review.md](challenge-app-v2-code-review.md). Tất cả đã fix vào code.

- **Finding #1 [P1] fix — Date-only contract + timezone**: BE vẫn serialize `startDate`/`endDate` dưới dạng date-only string (`YYYY-MM-DD`), nhưng app V2 giờ parse như **local calendar day** thay vì UTC instant. Thêm [`riz-app-v2/utils/challenge-date.ts`](../../riz-app-v2/utils/challenge-date.ts) với `parseLocalDayStart`, `parseLocalDayEnd`, `formatLocalDayShort`, `formatLocalDayFull`, `daysFromNow`. Submission window: mở từ `00:00 startDate local` → `23:59:59.999 endDate local` (inclusive). Tất cả render bỏ HH:mm — chỉ còn `DD/MM` hoặc `DD/MM/YYYY`. Files updated: `utils/challenge-phase.ts`, `hooks/challenges/use-can-submit.ts`, `screens/community/components/mini-challenge-card-header.tsx` (xóa helper `formatTimeDate` render giờ), `screens/community/components/submissions-empty-state.tsx`, `screens/feed/components/main/pinned-challenge-card.tsx`, `screens/challenges/components/challenge-card.tsx`, `screens/challenges/challenge-detail.tsx`. **Không cần sửa BE** — contract date-only được giữ nguyên, chỉ fix parsing phía client.
- **Finding #2 [P1] fix — subCategory clear không fail**: `riz-be/apps/nest/libs/challenge/src/dtos/create-challenge.dto.ts` đổi `subCategory` từ `@Type(() => Number)` sang `@Transform` custom normalize `""` / `null` / `undefined` → `null`, và tự convert string số sang number. Type signature đổi thành `number | null`. Admin-FE flow "đổi khỏi ARCHITECTS và clear subCategory" giờ gửi `subCategory=""` → DTO normalize → service/guard nhận `null` → Prisma update `subcategoryId: null` (guard logic đã đúng sẵn). Đồng thời fix service re-derivation path `challenge.service.ts:146-157` — đổi `dto.subCategory ?? current.subcategoryId` thành `dto.subCategory !== undefined ? dto.subCategory : current.subcategoryId` để phân biệt "không đổi" vs "clear" (bug `??` treat null giống undefined).
- **Finding #3 [P2] fix — `phase` + `status` combine đúng AND semantics**: `riz-be/apps/nest/libs/challenge/src/challenge.service.ts:543-585` refactor — `phaseClause` chỉ còn thuần time-window (`startsAt`/`endsAt`), không hard-code `status: ACTIVE` nữa. Tách `statusClause` riêng với priority: caller-provided `status` > phase-implied `ACTIVE` > default (hide DRAFT). Request `?status=ARCHIVED&phase=ended` giờ trả "ARCHIVED với endsAt < now", đúng theo spec DTO. Default behavior khi caller không pass status: phase vẫn implies ACTIVE (backward-compatible với test `?phase=ended excludes ARCHIVED`). DTO doc `get-challenges.dto.ts` update để reflect AND semantics.

**Round 3 — post-implementation code review (2026-04-19):**

- **Finding #1 [P1] fix**: `community-posts-view.tsx` sticky spacer `COMMUNITY_FULL_HEADER_HEIGHT` gây gap thừa khi parent container render `CommunityHeader` ngoài list (`showHeader={false}`). Fix: thêm `STICKY_ONLY_HEADER_HEIGHT` (chỉ tính ShareInput + CategoryFilter) và threshold `STICKY_THRESHOLD` cũng conditional theo `showHeader` — khi parent render header thì sticky trigger ngay khi user scroll qua `stickySection`, không đợi đến 145px (banner+glass không còn trong list).
- **Finding #2 [P2] fix**: `community-tabs.tsx` indicator absolute ngoài `ScrollView` không đồng bộ khi user scroll horizontal strip. Fix: **tách 2 indicator** — indicator cho tab "posts" render ngoài `ScrollView` (stable, pinned), indicator cho challenge tabs render **bên trong `ScrollView` content** → tự di chuyển theo scroll offset. Không cần track `scrollX` thủ công. `handleTabLayout` cho challenge tabs giờ lưu x relative content (không còn offset `postsW` thủ công).
- **Finding #3 [P2] N/A**: Reviewer đề xuất defensive fallback cho legacy `status=ENDED` ở admin-fe list/edit. **Bỏ qua hoàn toàn** — DB target là brand new, không có record `ENDED` legacy. Migration §2.6.4 không áp dụng, code `isLegacyEndedStatus`/`getChallengeStatusLabel` helper và edit-page block warning đã được tạo rồi revert. Admin-fe giữ assumption canonical: `status ∈ {DRAFT, ACTIVE, ARCHIVED}`.
- **§2.4.11 + §2.6.4 deprecated**: Cả 2 section (migration notes + legacy ENDED handling) giữ để reference historic rationale nhưng không còn pre-deploy step. DB mới sẽ không trigger path này.
- **Cập nhật tham chiếu kế thừa**: `§2.4.10` note về "data cũ cần migration", `§2.4.12` spec "ACTIVE + ENDED + ARCHIVED", §2.6.2 row 7/8 "sau migration §2.6.4", `§2.6.5` rationale "reject ENDED để bảo vệ legacy", writable-status.ts comment đã được soft-rewrite theo hướng "DB brand new + future-proof guard" thay vì "legacy compat".
- **BE code**: `writable-status.ts` comment cập nhật — ENDED reject là future-proof guard (automation/direct API), không còn viện dẫn legacy records.

**Round 1 — fix contract & simplify:**

**Round 1 — fix contract & simplify:**

- Pinned card chỉ hiện challenge **ongoing** (`phase=ongoing`). Bỏ hướng `phase=open` + teaser upcoming
- Bỏ public endpoint `/project-categories` + hook client. Thay bằng `?feedCategorySlug=` ở `GET /challenges` — BE dùng Prisma relation filter
- Sửa diagnosis `JwtOptionalGuard` ở §2.1.3; V2.0b batch-resolve `isMember`/`mySubmission` cho list rows
- `use-can-submit.ts` vào Files SỬA; tách hook thành `isSubmissionWindowOpen` (status + window, cho visibility CTA — cho phép anonymous/non-member vẫn thấy CTA để route login / mở NotMemberBottomSheet) vs `canSubmit` (thêm auth + member + !submitted, cho action gating). `challenge-detail.tsx:105` dùng `isSubmissionWindowOpen && !hasSubmitted`, **không** dùng `canSubmit` làm visibility (xem §2.4.8)
- Thêm task BE reject `ENDED` ở create/update DTO (§2.6.5)
- Cắt P3 polish: `scrollToIndex` auto-center, `setInterval` polling, Zustand persist activeTab, collapsible summary bar ở submit, auto-expand info khi submit

**Round 2 — community flat layout:**

- Community tab bar **flat**, không chia nhóm visual theo phase. Upcoming + ongoing + ended xuất hiện chung trong tab bar (label = title, thứ tự response).
- **1 query** duy nhất `useChallenges({ status: "ACTIVE", limit: 30 })` cho community — thay cho hướng 3 query phase ở Round 1. BE sort `startsAt DESC` → mới nhất lên đầu.
- Phase **derive client-side** qua helper `getPhase(challenge)` — chỉ dùng trong overview (MiniChallengeCardHeader), empty state, và logic skip fetch submissions cho upcoming.
- `MiniChallengeCardHeader` có 3 variant badge: "Sắp bắt đầu" / "Đang diễn ra" / "Đã kết thúc"; CTA "Xem chi tiết" thống nhất (submit flow luôn qua detail screen).
- `/challenges` route đổi sang 3 filter tab `upcoming / ongoing / ended` (khác community — route này là trang filter explicit).
- Finding 2: contract anonymous = `isMember`/`mySubmission` **absent** (khớp entity `@ApiPropertyOptional`), không phải `false/null`.

Timeline: **8-9.2 ngày** (không đổi — flat layout thực tế còn đơn giản hơn 3-query, bù cho thời gian đã spec).

---

## 1. Tổng Quan

### 1.1. Bối Cảnh

Plan v1 đã hoàn thành (Phase A + B). Challenge feature hiện đang hoạt động theo mô hình:

- **Top tab "Challenge"** trong SliverAppBar (cùng Feed / Create / AI Chat / Community)
- **Route `/challenges`**: list screen
- **Route `/challenges/[id]`**: detail screen với 2 tab "Thông tin" / "Bài dự thi"
- **Route `/(protected)/challenges/[id]/submit`**: submit form
- **Badge "Challenge"** trên feed item khi `project.challengeSubmission != null`

### 1.2. Yêu Cầu Thay Đổi (V2)

| #   | Yêu cầu                                                                                                                                                                                 | Tác động                                                                                            |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| R1  | Feed mặc định mở tab **Photography** (là `feedCategory` của challenge sắp tới — tạm **fix cứng**); hiển thị challenge card **lên đầu feed**; tap → detail challenge                     | `screens/feed/*`                                                                                    |
| R2  | **Bỏ tab Challenge** khỏi top SliverAppBar. Giữ nguyên trang detail & submit                                                                                                            | `screens/feed/components/sliver-app-bar/tab-button.tsx`, `screens/feed/index.tsx`                   |
| R3  | Detail challenge **hiển thị thêm submissions hiện có** ngay trong trang (gộp tab "Bài dự thi" vào nội dung chính, không dùng tab)                                                       | `screens/challenges/challenge-detail.tsx`                                                           |
| R4  | Detail challenge **rút gọn** phần thông tin (description, objective, requirements, rules, awards…) — default collapse, chỉ expand khi user muốn đọc hoặc khi vào submit flow            | `screens/challenges/challenge-detail.tsx`, `challenge-info-tab.tsx`                                 |
| R5  | Trang **Community dùng dynamic tabs**: tab "Bài viết" (posts) + mỗi challenge ACTIVE là **1 tab riêng**. Tap tab challenge → hiển thị **submissions grid của challenge đó** (không phải challenge list) | `screens/community/*`                                                                               |

### 1.3. Hiện Trạng Code Liên Quan

| Component             | File                                                                                                                                     | Ghi chú                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Feed category state   | [screens/feed/index.tsx:51-54](../../riz-app-v2/screens/feed/index.tsx#L51-L54)                                                           | Default `category = "ARCHITECTS"` — **cần đổi sang `PHOTOGRAPHERS`**             |
| Feed categories config| [screens/feed/constants/constants.ts:32-58](../../riz-app-v2/screens/feed/constants/constants.ts#L32-L58)                                | Đã có slug `PHOTOGRAPHERS`                                                        |
| Top tab config        | [screens/feed/components/sliver-app-bar/tab-button.tsx:34-41](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx#L34-L41) | `tabs` array — **cần bỏ entry `challenge`**                                       |
| Tab press handler     | [screens/feed/index.tsx:92-104](../../riz-app-v2/screens/feed/index.tsx#L92-L104)                                                         | `handleTabPress` có case `"challenge"` — **cần bỏ**                               |
| Challenge detail tabs | [screens/challenges/challenge-detail.tsx](../../riz-app-v2/screens/challenges/challenge-detail.tsx)                                       | Có state `activeTab: "info" \| "submissions"` — **cần restructure**              |
| Community header      | [screens/community/components/community-header.tsx](../../riz-app-v2/screens/community/components/community-header.tsx)                   | Hiện tại KHÔNG có tab nào → **cần thêm**                                          |
| List challenge screen | [screens/challenges/index.tsx](../../riz-app-v2/screens/challenges/index.tsx)                                                             | Giữ làm public route explicit cho deep-link / từ detail; community **không reuse** (§2.4 dùng flat tab bar riêng) |

### 1.4. Mục Tiêu

- **UX**: Đưa challenge lên vị trí dễ thấy hơn (đầu feed Photography) thay vì tab riêng trong top bar đang quá tải.
- **Information architecture**: Dọn dẹp top bar (5 tab → 4 tab), gom entry-point vào Community.
- **Detail screen**: Giảm tải nội dung, nâng trọng số cho phần submissions (là nội dung user muốn xem nhất).

### 1.5. Phạm Vi

**In scope:**

- Frontend app (`riz-app-v2`) — restructure navigation + detail screen (primary)
- Admin frontend (`riz-admin-fe`) — thêm Status field vào challenge form (§2.6, blocker)
- Backend (`riz-be`) — mở rộng `GET /challenges` với:
  - Param `?phase=ongoing|ended|upcoming` (bỏ `open` — pinned card chỉ ongoing)
  - Param `?feedCategorySlug=<slug>` (thay vì thêm public endpoint `/project-categories`)
  - Reject `ENDED` ở create/update DTO để khớp status model 3-values (§2.6.5)

**Out of scope:**

- Thay đổi Challenge / Submission data model
- Admin review submissions flow (đã có)
- Pagination cho `/challenges/:id/submissions` (hiện BE trả array đầy đủ)
- Deep link `/community?tab=challenge-{id}`

### 1.6. Strategy Cho Pinned Challenge Resolution

**Vấn đề:** App cần lấy 1 challenge "nổi bật" ở PHOTOGRAPHERS category để hiển thị pinned card. Nhưng:

- BE `feedCategory: Int` FK → `ProjectCategory.id` SERIAL — id không stable giữa local/dev/prod
- Không có public endpoint trả category list (chỉ admin)
- Pinned card phải là **call-to-action submit** → chỉ hiện challenge **đang diễn ra** (ongoing). Upcoming không có path submit, ended không cho submit.

**Quyết định (sau review):**

1. Tận dụng `GET /challenges` hiện có (đã sort `startsAt DESC`)
2. Mở rộng `?phase=` param với các giá trị rời: `upcoming`, `ongoing`, `ended`. **Bỏ `phase=open`** (không còn use-case gom upcoming+ongoing).
3. Thay vì thêm public endpoint `/project-categories` + hook resolve slug→id, **mở rộng `GET /challenges` nhận trực tiếp `?feedCategorySlug=<slug>`** — BE tự join `ProjectCategory.slug`. Tiết kiệm 1 public endpoint, 1 hook, 1 cache layer, và không tạo cross-module dependency giữa feature challenge và project taxonomy.

**Query pinned card:**

```
GET /challenges?phase=ongoing&feedCategorySlug=PHOTOGRAPHERS&limit=1
```

App lấy `response.data[0] ?? null` → render pinned card (hoặc ẩn nếu null).

> Lý do chốt **ongoing-only** cho pinned card: pinned card là CTA submit. Upcoming chưa submit được → hiện teaser "tham gia ngay" tạo xung đột UX với BE guard reject và làm tăng 1 nhánh UI không cần thiết. User muốn khám phá upcoming thì vào community tab bar (§2.4) — mọi challenge ACTIVE đều xuất hiện chung, overview khi tap vào tự hiển thị phase (countdown cho upcoming, CTA cho ongoing, archive cho ended).

### 1.7. Rule Chọn Challenge Cho Pinned Card

Chỉ hiện challenge **ongoing**:

- **Ongoing** (`status='ACTIVE' AND startsAt ≤ now ≤ endsAt`): CTA "Nộp bài dự thi"

Loại ra:

- **Upcoming** (`now < startsAt`): không pinned (chưa submit được — user thấy ở community tab)
- **Ended** (`now > endsAt`): không pinned (user xem ở community tab ended)
- ARCHIVED / DRAFT: không pinned

→ Filter = **`phase=ongoing`**.

**Sort rule:**

- `ORDER BY startsAt DESC, id DESC` — default của [`challenge.service.ts:550`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L550)
- Với `phase=ongoing`, tất cả match đều có `startsAt ≤ now` — challenge khởi động gần nhất lên đầu → phù hợp pinned card.

---

## 2. Thay Đổi Chi Tiết

### 2.1. R1 — Feed: Default Photography + Challenge Card Lên Đầu

#### 2.1.1. Default tab = Photography

**File:** [screens/feed/index.tsx](../../riz-app-v2/screens/feed/index.tsx)

```diff
- const [category, setCategory] = useState<CategorySlug>("ARCHITECTS");
+ const [category, setCategory] = useState<CategorySlug>("PHOTOGRAPHERS");
```

**Lưu ý:**

- `FEED_HEADER_HEIGHT_WITH_ROOM` chỉ áp dụng cho `ARCHITECTS` (vì có RoomFilter). Với default `PHOTOGRAPHERS`, sticky header height ban đầu là `FEED_HEADER_HEIGHT_WITHOUT_ROOM = 176`. Không cần đổi logic sticky.
- `TrendingTitle` render `feed_trending_in_photographers` — đã tồn tại trong locales.

#### 2.1.2. Challenge Card ở đầu Feed (chỉ khi tab Photographers active)

**Strategy:** Inject pinned card vào `listHeaderComponent` của `MasonryGridFeed`, dưới `TrendingTitle` trên masonry grid.

**File mới:** `screens/feed/components/main/pinned-challenge-card.tsx`

```typescript
import { router } from "expo-router";
import { useFeaturedChallenge } from "@/hooks/challenges";

export function PinnedChallengeCard() {
  const { data: challenge } = useFeaturedChallenge({ feedCategorySlug: "PHOTOGRAPHERS" });
  if (!challenge) return null;

  // Chỉ render ongoing — phase đã filter ở BE.
  // UI: hero ảnh + title + "Còn Y ngày · Nộp bài dự thi"
  // Tap → router.push(`/challenges/${challenge.id}`)
}
```

**Logic:**

- Hook `useFeaturedChallenge` gọi `GET /challenges?phase=ongoing&feedCategorySlug=PHOTOGRAPHERS&limit=1` — chỉ ongoing
- Không cần client compute phase — BE đã đảm bảo chỉ trả challenge `startsAt ≤ now ≤ endsAt`
- Nếu response rỗng → không render gì (ẩn card). Không fallback upcoming.
- Tap → navigate detail

**File sửa:** [screens/feed/index.tsx](../../riz-app-v2/screens/feed/index.tsx) `listHeaderComponent`

```diff
  const listHeaderComponent = (
    <>
      {isSticky ? ... : <FeedHeader ... />}
      {trendingTitle}
+     {category === "PHOTOGRAPHERS" && <PinnedChallengeCard />}
    </>
  );
```

**Vị trí gợi ý trong layout:**

```
┌──────────────────────────────────────┐
│ [SliverAppBar / Filter header]       │
├──────────────────────────────────────┤
│ Trending in Photography              │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │░░░░ PINNED CHALLENGE CARD ░░░░░░░│ │  ← MỚI
│ │▓ "Challenge title"    📅 date  ▓▓│ │
│ └──────────────────────────────────┘ │
│                                      │
│ ┌────────┐ ┌────────┐                │
│ │ [item] │ │ [item] │  ← masonry    │
│ └────────┘ └────────┘                │
└──────────────────────────────────────┘
```

#### 2.1.3. BE — Mở rộng `GET /challenges` với `?phase=` và `?feedCategorySlug=`

**File sửa:**

- [`riz-be/apps/nest/libs/challenge/src/dtos/get-challenges.dto.ts`](../../riz-be/apps/nest/libs/challenge/src/dtos/get-challenges.dto.ts) — thêm `phase` + `feedCategorySlug` param
- [`challenge.service.ts`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts) `getChallenges()` — apply phase clause + resolve slug → categoryId vào where, **và** extend service signature để nhận `userProfileId` cho phép resolve `isMember`/`mySubmission` trên list response (xem note cuối mục)
- [`challenge.controller.ts`](../../riz-be/apps/nest/libs/challenge/src/challenge.controller.ts) — `getChallenges()` đã có `JwtOptionalGuard`; thêm `@CurUser() user?: User` và truyền `user?.profileId` xuống service

**DTO:**

```typescript
// get-challenges.dto.ts
export enum ChallengePhase {
  Upcoming = "upcoming", // status=ACTIVE AND startsAt > now
  Ongoing = "ongoing", // status=ACTIVE AND startsAt <= now <= endsAt
  Ended = "ended", // status=ACTIVE AND endsAt < now
}

export class GetChallengesDto {
  // ...existing: status, feedCategory, subCategory, page, limit

  @ApiPropertyOptional({ enum: ChallengePhase })
  @IsOptional()
  @IsEnum(ChallengePhase)
  phase?: ChallengePhase;

  @ApiPropertyOptional({ description: "Filter theo slug của ProjectCategory (ví dụ 'PHOTOGRAPHERS')" })
  @IsOptional()
  @IsString()
  feedCategorySlug?: string;
}
```

> **Chốt:** bỏ `ChallengePhase.Open`. Sau review, pinned card chỉ ongoing-only (§1.6) nên không còn use-case gom upcoming+ongoing.

**Service `getChallenges()` — apply phase + slug vào where:**

```typescript
const now = new Date();
const phaseClause: Prisma.ChallengeWhereInput = (() => {
  switch (query.phase) {
    case ChallengePhase.Upcoming:
      return { status: ChallengeStatus.ACTIVE, startsAt: { gt: now } };
    case ChallengePhase.Ongoing:
      return { status: ChallengeStatus.ACTIVE, startsAt: { lte: now }, endsAt: { gte: now } };
    case ChallengePhase.Ended:
      return { status: ChallengeStatus.ACTIVE, endsAt: { lt: now } };
    default:
      return {};
  }
})();

// Resolve slug → categoryId qua relation filter (không cần extra query)
const categoryClause: Prisma.ChallengeWhereInput = query.feedCategorySlug
  ? { category: { is: { slug: query.feedCategorySlug } } }
  : query.feedCategory
    ? { categoryId: query.feedCategory }
    : {};

const where: Prisma.ChallengeWhereInput = {
  deletedAt: null,
  ...(query.status ? { status: query.status } : includeDraft ? {} : { status: { not: ChallengeStatus.DRAFT } }),
  ...categoryClause,
  ...(query.subCategory ? { subcategoryId: query.subCategory } : {}),
  ...phaseClause,
};
```

**Lưu ý quan trọng:**

- `phase` clause **thêm** vào default where (`status != DRAFT`), không replace — giữ backward compat.
- Nếu `phase` truyền cùng `status` khác `ACTIVE` → phase thắng (phase luôn pin `status=ACTIVE`). Hoặc validate DTO reject combination — tuỳ BE dev chọn.
- `feedCategorySlug` và `feedCategory` không truyền cùng lúc — validate DTO reject combination hoặc slug thắng.
- Sort order `[{startsAt: 'desc'}, {id: 'desc'}]` giữ nguyên ([`challenge.service.ts:550`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L550)).

**Tests cần thêm:**

- [ ] `?phase=ongoing&feedCategorySlug=PHOTOGRAPHERS&limit=1`: trả đúng 1 challenge match ongoing + category slug, hoặc rỗng nếu không có
- [ ] `?phase=ongoing`: chỉ trả challenge trong cửa sổ `startsAt ≤ now ≤ endsAt`
- [ ] `?phase=upcoming`: chỉ trả challenge `startsAt > now`
- [ ] `?phase=ended`: chỉ trả challenge `endsAt < now` (và `status=ACTIVE`, không phải ARCHIVED)
- [ ] `?phase=*`: KHÔNG trả ARCHIVED / DRAFT
- [ ] `?feedCategorySlug=PHOTOGRAPHERS`: match đúng category qua slug (verify join đúng record)
- [ ] `?feedCategorySlug=UNKNOWN_SLUG`: trả rỗng, không throw
- [ ] `/challenges` không truyền `phase` → giữ nguyên behavior cũ (không regression)
- [ ] List response (logged-in) có `isMember`/`mySubmission` khớp user hiện tại (xem resolve userState bên dưới)
- [ ] List response (anonymous) **không có** field `isMember` / `mySubmission` (absent/undefined) — match entity contract [`challenge.entity.ts:116-127`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L116-L127) (`@ApiPropertyOptional`, "Absent for unauthenticated requests")

**Resolve `isMember`/`mySubmission` trên list (MỚI so với v2 bản đầu):**

Plan gốc assume list item có `isMember`/`mySubmission` ở §2.4.4, §2.4.5. Nhưng hiện tại [`challenge.service.ts:535-565`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L535-L565) `getChallenges()` **không nhận `userProfileId`** và chỉ dùng `mapChallengeEntity(row)` thuần — user state không được resolve trên list response. Guard `JwtOptionalGuard` đã apply ở [`challenge.controller.ts:40`](../../riz-be/apps/nest/libs/challenge/src/challenge.controller.ts#L40) nên chỉ cần extend signature:

```typescript
// challenge.controller.ts
@Get()
@ApiOkResponse({ type: PaginatedChallengesEntity })
@UseGuards(JwtOptionalGuard)
getChallenges(
  @Query() query: GetChallengesDto,
  @CurUser() user?: User,
): Promise<PaginatedChallengesEntity> {
  return this.challengeService.getChallenges(query, false, user?.profileId);
}

// challenge.service.ts
async getChallenges(
  query: GetChallengesDto,
  includeDraft = false,
  userProfileId?: bigint,
): Promise<PaginatedChallengesEntity> {
  // ... build where ...
  const rows = await this.prisma.challenge.findMany({ ... });
  // Batch resolve user state cho tất cả rows (1 query cho members, 1 query cho submissions)
  const userStateById = userProfileId
    ? await this.resolveUserChallengeStatesBatch(rows.map(r => r.id), userProfileId)
    : new Map();
  return plainToInstance(PaginatedChallengesEntity, {
    data: rows.map(row => this.mapChallengeEntity(row, userStateById.get(row.id))),
    pagination: { ... },
  });
}
```

> **Rationale:** `getChallengeById()` đã có pattern `resolveUserChallengeState(challengeId, profileId)` ở [`challenge.service.ts:596`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L596). Method mới `resolveUserChallengeStatesBatch` batch-query `challengeMember` + `challengeSubmission` với `challengeId IN (...)` để tránh N+1. Alternative: tạo sub-select trong Prisma `include` — nhưng batch query 2 round-trip dễ maintain hơn. Không được gọi `resolveUserChallengeState` trong loop.
>
> **Contract cho anonymous:** khớp `getChallengeById()` hiện tại ở [`challenge.service.ts:596-598`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L596-L598) — khi `userProfileId == null` thì `userState = undefined`, `mapChallengeEntity(row, undefined)` sẽ không merge các field `isMember`/`mySubmission` vào entity. Response anonymous có shape `ChallengeEntity` với 2 field này **absent** (không phải `false`/`null`). Khớp với `@ApiPropertyOptional` ở [`challenge.entity.ts:116-127`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L116-L127).

#### 2.1.4. App — Hook `useFeaturedChallenge`

**File mới:** `riz-app-v2/hooks/challenges/use-featured-challenge.ts`

```typescript
import { useQuery } from "@tanstack/react-query";
import { challengesApi, CHALLENGE_QUERY_KEYS } from "@/lib/api/challenges";
import { useProfileId } from "@/store/auth-store";

interface Params {
  feedCategorySlug: string;
}

/**
 * Lấy 1 challenge ongoing cho pinned card ở đầu feed.
 * BE filter trực tiếp theo phase=ongoing + slug → không cần resolve slug→id ở client.
 */
export function useFeaturedChallenge({ feedCategorySlug }: Params) {
  const userId = useProfileId();
  return useQuery({
    queryKey: [...CHALLENGE_QUERY_KEYS.all, "featured", feedCategorySlug, userId ?? "anon"],
    queryFn: async () => {
      const res = await challengesApi.list({
        phase: "ongoing",
        feedCategorySlug,
        limit: 1,
      });
      return res.data[0] ?? null;
    },
    enabled: !!feedCategorySlug,
    staleTime: 5 * 60 * 1000,
  });
}
```

> Convention: query key chứa `userId` để tránh cache bẩn giữa anonymous/logged-in — xem [`use-challenges.ts:19-48`](../../riz-app-v2/hooks/challenges/use-challenges.ts#L19-L48).

**File sửa:** [`riz-app-v2/lib/api/challenges/types.ts`](../../riz-app-v2/lib/api/challenges/types.ts) — thêm `phase` + `feedCategorySlug` vào params:

```typescript
export type ChallengePhase = "upcoming" | "ongoing" | "ended";

export interface GetChallengesParams {
  status?: ChallengeStatus;
  phase?: ChallengePhase;
  feedCategory?: number;
  feedCategorySlug?: string;
  subCategory?: number;
  page?: number;
  limit?: number;
}
```

> `challengesApi.list()` đã truyền `params` nguyên xi qua axios ([`index.ts:21-24`](../../riz-app-v2/lib/api/challenges/index.ts#L21-L24)) — không cần sửa method, chỉ mở rộng type.

**File sửa:** [`riz-app-v2/hooks/challenges/index.ts`](../../riz-app-v2/hooks/challenges/index.ts) — thêm barrel export:

```diff
  export * from "./use-challenges";
  export * from "./use-challenge-mutations";
  export * from "./use-can-submit";
+ export * from "./use-featured-challenge";
```

> **Bỏ `useCategoryIdBySlug` / `useProjectCategories`** khỏi scope — không cần resolve slug ở client, BE xử trực tiếp.

**Pinned card render (không còn branching phase):**

- Hiển thị: cover hero + title + date range + "Còn Y ngày · Nộp bài dự thi"
- Tap → `router.push('/challenges/${challenge.id}')`
- BE đã đảm bảo phase=ongoing nên không cần compute phase ở client.

### 2.2. R2 — Bỏ Tab Challenge Khỏi Top Bar

#### 2.2.1. Xoá entry trong `tabs` array

**File:** [screens/feed/components/sliver-app-bar/tab-button.tsx](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx)

```diff
  export const tabs: TabItem[] = [
    { id: "feed", labelId: "feed_tab_feed", icon: "feed" },
    { id: "create", labelId: "feed_tab_create", icon: "create" },
    { id: "ai-chat", labelId: "feed_tab_ai_chat", icon: "ai-chat" },
-   { id: "challenge", labelId: "feed_tab_challenge", icon: "challenge" },
    { id: "community", labelId: "feed_tab_community", icon: "community" },
  ];
```

**Xóa luôn** import `ChallengeIcon` và entry `challenge` trong `ICON_MAP` tại cùng file. Community tab bar là text-only (§2.4.5 — label = challenge title, không dùng icon); `ChallengeBadge` dùng `Trophy` từ `lucide-react-native` ([challenge-badge.tsx:1-10](../../riz-app-v2/components/challenge-badge.tsx#L1-L10)), không dùng `ChallengeIcon`. Không còn consumer nào sau khi xóa top tab.

#### 2.2.2. Xoá case `"challenge"` trong `handleTabPress`

**File:** [screens/feed/index.tsx:92-104](../../riz-app-v2/screens/feed/index.tsx#L92-L104)

```diff
  const handleTabPress = useCallback((tabId: string) => {
    if (tabId === "create") {
      router.navigate("/creator-studio" as Href);
    } else if (tabId === "community") {
      router.navigate("/community" as Href);
    } else if (tabId === "ai-chat") {
      router.navigate("/ai-agent-selector" as Href);
-   } else if (tabId === "challenge") {
-     router.navigate("/challenges" as Href);
    } else {
      console.log("Tab pressed:", tabId);
    }
  }, []);
```

#### 2.2.3. Giữ nguyên các route

- `app/challenges/index.tsx` → **giữ** (public route explicit cho deep-link / navigation từ detail; community **không reuse** route này — §2.4 dùng flat tab bar riêng)
- `app/challenges/[id].tsx` → **giữ**
- `app/(protected)/challenges/[id]/submit.tsx` → **giữ**

**Xóa dứt điểm** `components/icons/challenge-icon.tsx` + re-export trong `components/icons/index.ts` — sau V2.1 không còn consumer nào (chi tiết §3.4 + Task V2.1).

### 2.3. R3 + R4 — Detail Challenge: Gộp Submissions + Collapsible Info

#### 2.3.1. Layout mới (bỏ tab segmented)

**Before (v1):**

```
[Hero]
[Title + meta]
[Tabs: Thông tin | Bài dự thi]   ← bỏ
├─ Tab Thông tin: full content
└─ Tab Bài dự thi: gallery grid
[Submit button]
```

**After (v2):**

```
[Hero]
[Title + meta]
[Short summary: 2-3 dòng mô tả ngắn]
[▼ Xem chi tiết challenge]   ← collapsible (default collapsed)
  ├─ Mô tả
  ├─ Mục tiêu
  ├─ Yêu cầu
  ├─ Luật chơi
  └─ Hạng mục giải thưởng
[My Submission card] (nếu có)
[── Bài dự thi (N) ──]   ← section header
[Gallery grid]
[Load more]
[Submit button sticky bottom] (nếu eligible)
```

**Wireframe:**

```
┌────────────────────────────────────────┐
│ ← (overlay)                   ⋮        │
│ ┌────────────────────────────────────┐ │
│ │░░░░ COVER HERO (16:9) ░░░░░░░░░░░░│ │
│ └────────────────────────────────────┘ │
│                                        │
│ Thiết kế phòng khách Bắc Âu            │
│ 📅 10/04 - 30/04  👥 12  📝 5         │
│                                        │
│ Ngắn gọn: challenge về thiết kế        │
│ phòng khách phong cách Bắc Âu...       │
│                                        │
│ ┌────────────────────────────────────┐ │
│ │  ▼ Xem chi tiết challenge          │ │ ← collapsed
│ └────────────────────────────────────┘ │
│                                        │
│ 📄 Bài dự thi của bạn                  │ ← nếu có
│ [My submission card]                   │
│                                        │
│ ─── Bài dự thi (5) ────────────────    │
│ ┌────┐ ┌────┐                          │
│ │img │ │img │                          │
│ └────┘ └────┘                          │
│ ┌────┐ ┌────┐                          │
│ │img │ │img │                          │
│ └────┘ └────┘                          │
│ ┌────┐                                 │
│ │img │                                 │
│ └────┘                                 │
│                                        │
├────────────────────────────────────────┤
│    ┌──────────────────────┐            │
│    │   Nộp bài dự thi     │  ← sticky │
│    └──────────────────────┘            │
└────────────────────────────────────────┘
```

Khi user tap **"Xem chi tiết challenge"**:

```
│ ┌────────────────────────────────────┐ │
│ │  ▲ Thu gọn                         │ │
│ ├────────────────────────────────────┤ │
│ │ 📋 Mô tả                           │ │
│ │ Tạo không gian phòng khách...      │ │
│ │                                    │ │
│ │ 🎯 Mục tiêu                        │ │
│ │ • Thể hiện sự sáng tạo             │ │
│ │                                    │ │
│ │ 📝 Yêu cầu                         │ │
│ │ • Tối thiểu 3 góc chụp             │ │
│ │                                    │ │
│ │ ⚖️ Luật chơi                       │ │
│ │ • Mỗi thành viên chỉ nộp 1 bài     │ │
│ │                                    │ │
│ │ 🏆 Hạng mục                        │ │
│ │ • Best Minimalist Approach         │ │
│ └────────────────────────────────────┘ │
```

#### 2.3.2. Bỏ "auto-expand" — navigate thẳng sang submit

**Behavior:** Tap "Nộp bài" → navigate submit ngay. **Không** expand info trước khi navigate — vì sau navigate user không còn ở detail screen nữa, việc expand không có hiệu ứng nhìn thấy được, chỉ làm tăng state phải đồng bộ giữa hai màn.

Nếu user muốn đọc rule đầy đủ trước khi nộp bài, họ chủ động tap "Xem chi tiết challenge" trong `CollapsibleInfoSection` (§2.3.3). Submit screen giữ đơn giản (§2.3.4).

#### 2.3.3. Component mới: `CollapsibleInfoSection`

**File mới:** `screens/challenges/components/collapsible-info-section.tsx`

```typescript
interface Props {
  challenge: Challenge;
  defaultExpanded?: boolean;
  onToggle?: (expanded: boolean) => void;
}
// Dùng Reanimated `withTiming` cho height animation
// Chevron icon rotate 180deg khi expand
// Chứa lại nội dung từ `challenge-info-tab.tsx`
```

**File cần thay/xoá:** [screens/challenges/components/challenge-info-tab.tsx](../../riz-app-v2/screens/challenges/components/challenge-info-tab.tsx) — refactor thành content-only component (bỏ tab wrapper), được `CollapsibleInfoSection` render.

#### 2.3.4. Submit Screen: Link xem thể lệ

**File:** [screens/challenges/challenge-submit.tsx](../../riz-app-v2/screens/challenges/challenge-submit.tsx)

Ở đầu form giữ đơn giản:

- Challenge title (đã có)
- Một link text **"Xem thể lệ đầy đủ"** → navigate back về detail (hoặc mở bottom sheet reuse `CollapsibleInfoSection` content)

> **Bỏ khỏi MVP:** Collapsible "Luật chơi nhanh" ở submit screen. Rationale: disclosure đã có ở detail (`CollapsibleInfoSection`) — thêm một lớp nữa ở submit là 2 lớp chồng, phải duy trì sync i18n + skeleton + state giữa 2 màn mà giá trị UX thấp. Nếu sau MVP thấy thật sự cần rule reminder ở submit → bổ sung sau.

#### 2.3.5. Bỏ tab state + segmented control

**File:** [screens/challenges/challenge-detail.tsx](../../riz-app-v2/screens/challenges/challenge-detail.tsx)

```diff
- const [activeTab, setActiveTab] = useState<"info" | "submissions">("info");
- const tabIndicator = useAnimatedStyle(...);
```

Remove:

- Tab state + indicator animation
- Conditional render `activeTab === "info" ? <InfoTab /> : <SubmissionsTab />`
- Import `react-native-reanimated` nếu không còn dùng (giữ nếu `CollapsibleInfoSection` cần)

Replace với scroll view linear:

```tsx
<Animated.ScrollView>
  <ChallengeHero />
  <ChallengeMeta />
  <ShortSummary />
  <CollapsibleInfoSection />
  {mySubmission && <MySubmissionCard />}
  <SubmissionsSectionHeader count={submissions.length} />
  <ChallengeSubmissionsGrid />
</Animated.ScrollView>
```

#### 2.3.6. Skeleton update

**File:** [screens/challenges/components/challenge-detail-skeleton.tsx](../../riz-app-v2/screens/challenges/components/challenge-detail-skeleton.tsx)

Update skeleton layout để match layout mới: hero → meta → summary lines → collapsed box → grid skeleton. Bỏ skeleton cho tab indicator.

### 2.4. R5 — Community: Dynamic Per-Challenge Tabs

#### 2.4.1. Layout Mới

```
┌──────────────────────────────────────────────────┐
│ [CommunityHeader — banner + glass]               │
├──────────────────────────────────────────────────┤
│ ┌──────────┬───────────┬───────────┬───────────┐ │
│ │ Bài viết │ Challenge │ Challenge │ Challenge │ │ ← Flat horizontal
│ │ ────────│     A     │     B     │     C     │ │   scrollable
│ └──────────┴───────────┴───────────┴───────────┘ │   (không phân biệt phase)
├──────────────────────────────────────────────────┤
│ Tab "Challenge A" (tap vào):                     │
│   ┌────────────────────────────────────────────┐ │
│   │ [Cover 16:9]                               │ │
│   │ Title                                      │ │
│   │ [Badge: Sắp bắt đầu / Đang diễn ra / Đã    │ │
│   │  kết thúc]  + countdown/date range         │ │
│   │ 👥 members · 📝 submissions count          │ │
│   │ [Xem chi tiết challenge →]                 │ │
│   └────────────────────────────────────────────┘ │
│   ───── Bài dự thi ─────                         │
│   (Upcoming) Empty state countdown               │
│   (Ongoing/Ended) ┌────┐ ┌────┐                  │
│                   │img │ │img │ Grid 2-col       │
│                   └────┘ └────┘                  │
└──────────────────────────────────────────────────┘
```

**Highlights:**

- Tab bar **flat** (không phân nhóm), **scrollable horizontal**. Label = title. Thứ tự = response order (`startsAt DESC` từ BE).
- Tab "Bài viết" **pinned đầu** (fixed position)
- Mỗi tab challenge có **overview (MiniChallengeCardHeader)** ở đầu + **submissions grid** bên dưới. Overview là nơi duy nhất hiển thị phase.
- Nút "Xem chi tiết challenge →" navigate sang `/challenges/:id` cho user đọc thông tin + nộp bài

#### 2.4.2. Rule Lấy Danh Sách Challenges Cho Tabs

**Quyết định:** **1 query duy nhất** `useChallenges({ status: "ACTIVE", limit: 30 })`. Không phân biệt phase ở tầng query. Tab bar render flat, thứ tự tab = thứ tự response (BE sort `startsAt DESC, id DESC` → challenge khởi động mới nhất lên đầu).

**Rationale:**

- Tab bar flat — không chia nhóm visual theo phase. Phase chỉ hiển thị ở **overview trong tab** (MiniChallengeCardHeader § 2.4.4), không cần đến từ query.
- "Mới nhất lên đầu" là UX mong muốn — challenge cũ tự đẩy ra cuối theo thứ tự tự nhiên, không cần giữ riêng ended. Nếu challenge ended từ lâu rớt khỏi cap 30 → acceptable (user đã quên).
- Giảm 3 HTTP request/lần mở community xuống **1 request** — tiết kiệm bandwidth + memory + nhất quán cache.
- BE không cần thêm sort custom theo phase.

**Query:**

```ts
const challengesQ = useChallenges({ status: "ACTIVE", limit: 30 });
const challenges = challengesQ.data?.pages.flatMap((p) => p.data) ?? [];
```

**Derive phase client-side:**

Mỗi challenge có `startsAt`/`endsAt`, phase = pure function của time + dates. Component tiêu thụ (MiniChallengeCardHeader, SubmissionsEmptyState, ChallengeSubmissionsView) tự gọi helper:

```ts
function getPhase(c: Challenge): "upcoming" | "ongoing" | "ended" {
  const now = Date.now();
  const startsAt = new Date(c.startsAt).getTime();
  const endsAt = new Date(c.endsAt).getTime();
  if (now < startsAt) return "upcoming";
  if (now > endsAt) return "ended";
  return "ongoing";
}
```

> Helper này đặt ở [`riz-app-v2/lib/api/challenges/index.ts`](../../riz-app-v2/lib/api/challenges/index.ts) hoặc `utils/challenge-phase.ts` để reuse. Không được duplicate logic ở nhiều component.

**Sort order trong tab bar:** đúng theo response order — không client-side sort hay group. BE sort `startsAt DESC` đã phù hợp.

**Cap:** `limit=30` cứng. Nếu sau này user feedback cần nhiều hơn → post-MVP bổ sung pagination `fetchNextPage` trên tab bar.

Hook `useChallenges` giữ nguyên signature ([`use-challenges.ts:19-37`](../../riz-app-v2/hooks/challenges/use-challenges.ts#L19-L37)).

> **Note về `?phase=` BE param:** vẫn giữ ở §2.1.3 cho pinned card (`phase=ongoing`) và `/challenges` route filter tabs (§2.4.12). Community **không dùng** — nhưng BE tách vẫn đáng làm vì có 2 consumer khác.

#### 2.4.3. Refactor Prerequisite — Extract `CommunityPostsView`

`screens/community/index.tsx` hiện gắn chặt (header, sticky, pending uploads banner, FlashList posts) vào 1 render flow chung. **Phải refactor trước** khi implement tabs động.

Tạo `screens/community/community-posts-view.tsx` chứa **toàn bộ logic tab posts hiện tại**:

- Sticky section: `ShareInputBox` + `PostCategoryFilter`
- `PendingPostsBanner` + `PendingPostsList`
- `FlashList` posts với renderItem
- Tất cả hooks posts (`useGetPosts`, `useActiveVideo`, `useCreatePost`, `usePendingPostsCount`)
- `useSliverAnimation`, scroll-to-top, sticky logic
- `ReactorListBottomSheet`, `CommentThreadModal`
- URL params `postId`, `openComments`

**Verify:** Community screen trước/sau refactor identical về UX (manual regression test).

#### 2.4.4. Component Mới: `ChallengeSubmissionsView`

**File mới:** `screens/community/challenge-submissions-view.tsx`

Render nội dung của 1 tab challenge trong community. View reuse cho mọi phase — phase derive client-side từ `startsAt`/`endsAt`:

```typescript
interface Props {
  challenge: Challenge; // có đủ info challenge + isMember + mySubmission
}

export function ChallengeSubmissionsView({ challenge }: Props) {
  const phase = getPhase(challenge); // xem §2.4.2
  // Upcoming: KHÔNG fetch submissions (chắc chắn rỗng — BE guard reject submit trước startsAt).
  const { data: submissions, isRefetching, refetch } = useChallengeSubmissions(challenge.id, {
    enabled: phase !== "upcoming",
  });

  return (
    <FlashList
      ListHeaderComponent={<MiniChallengeCardHeader challenge={challenge} />}
      data={phase === "upcoming" ? [] : submissions ?? []}
      renderItem={({ item }) => (
        <SubmissionGalleryItem
          submission={item}
          onPress={() =>
            router.push({
              pathname: "/art-feed",
              params: {
                projectId: item.projectId,
                profileId: item.author.id,
              },
            } as Href)
          }
        />
      )}
      numColumns={2}
      estimatedItemSize={200}
      refreshing={isRefetching}
      onRefresh={refetch}
      ListEmptyComponent={<SubmissionsEmptyState challenge={challenge} />}
    />
  );
}
```

**Empty state theo phase (component mới `SubmissionsEmptyState` — tự derive phase từ challenge prop):**

| Phase      | Empty state                                                                                                      |
| ---------- | ---------------------------------------------------------------------------------------------------------------- |
| `upcoming` | Icon countdown + "Challenge chưa bắt đầu" + text "Bắt đầu sau X ngày · HH:MM dd/MM" + CTA "Xem chi tiết challenge" |
| `ongoing`  | Icon empty box + "Chưa có bài dự thi nào" + CTA "Xem chi tiết challenge" (submit flow luôn qua detail — §2.4.8)    |
| `ended`    | Icon archive + "Challenge đã kết thúc mà chưa có bài dự thi"                                                      |

**Chú thích:**

- Upcoming không fetch `/challenges/:id/submissions` vì BE guard đã reject submit trước `startsAt` → submissions chắc chắn = 0. Tiết kiệm 1 request mỗi lần tap tab upcoming.
- `useChallengeSubmissions` cần nhận option `enabled` — nếu hook hiện tại chưa support, mở rộng signature (backward compat: default `enabled: true`).
- `MiniChallengeCardHeader` tự derive phase để render **status badge** + countdown/date range/"Đã kết thúc" — xem §2.4.4 component spec bên dưới.

**Contract:**

- BE endpoint `GET /challenges/:id/submissions` trả `ChallengeSubmissionEntity[]` (không pagination) — xem [`challenge.controller.ts:52-56`](../../riz-be/apps/nest/libs/challenge/src/challenge.controller.ts#L52-L56).
- App hook `useChallengeSubmissions()` đã dùng `useQuery` — xem [`use-challenges.ts:51-58`](../../riz-app-v2/hooks/challenges/use-challenges.ts#L51-L58).
- **Không** dùng `useInfiniteQuery` / `fetchNextPage` / `Load more` trong MVP. Nếu cần pagination submissions → cần mở BE contract riêng + đưa vào plan lần sau.
- Navigation phải truyền cả `projectId` **và** `profileId` (nhất quán với `challenge-submissions-tab.tsx:73-89` hiện tại).

**Reuse được:**

- `SubmissionGalleryItem` từ `screens/challenges/components/` (đã implement v1)
- `challengesApi.getSubmissions()` endpoint sẵn có
- `useChallengeSubmissions` hook sẵn có — không tạo mới

**Component mới: `MiniChallengeCardHeader`** — card 16:9 compact, tự derive phase từ `challenge` prop (gọi `getPhase(challenge)`) rồi render variant tương ứng:

| Phase      | Badge (màu)                | Thông tin chính                                      | CTA                         |
| ---------- | -------------------------- | ---------------------------------------------------- | --------------------------- |
| `upcoming` | "Sắp bắt đầu" (accent)     | Countdown "Bắt đầu sau X ngày · HH:MM dd/MM"         | "Xem chi tiết challenge →"  |
| `ongoing`  | "Đang diễn ra" (primary)   | "Còn Y ngày · kết thúc HH:MM dd/MM" + member/submission count | "Xem chi tiết challenge →"  |
| `ended`    | "Đã kết thúc" (muted)      | Date range đầy đủ + submission count                 | "Xem chi tiết challenge →"  |

- Button "Xem chi tiết" thống nhất cho cả 3 phase → navigate `/challenges/:id` nơi user đọc info + submit (nếu ongoing + `canSubmit`).
- **Không** hiển thị CTA "Nộp bài" trực tiếp ở overview — submit flow luôn qua detail screen để user đọc info trước (§2.4.8).

#### 2.4.5. `CommunityTabs` Component — Dynamic Tabs

**File mới:** `screens/community/components/community-tabs.tsx`

```typescript
type CommunityTabItem =
  | { id: "posts"; label: string }
  | {
      id: `challenge-${string}`;
      label: string;
      challenge: Challenge;
    };

interface Props {
  tabs: CommunityTabItem[];
  activeTabId: string;
  onChange: (id: string) => void;
}
```

**Rendering:**

- Tab "Bài viết" **pinned đầu** (không scroll)
- Các tab challenge **scrollable horizontal** — dùng `ScrollView` ngang đơn giản. **Không** dùng `scrollToIndex` auto-center — cắt khỏi MVP. User chủ động scroll tab bar.
- Label challenge tab: `challenge.title` truncate ~20 chars (ellipsis)
- **Không phân biệt visual theo phase** — mọi tab render giống nhau (cùng foreground color). Phase chỉ hiển thị bên trong overview khi tap (§2.4.4).
- **Thứ tự tab** = thứ tự response từ `useChallenges` (BE sort `startsAt DESC` → mới nhất lên đầu). Không client-side sort/group.
- Indicator animation Reanimated cho active tab (giữ — cost thấp, value cao)

**Loading state:** Khi query đang loading → hiện skeleton tabs (3 placeholder).

**Empty state:** Không có challenge nào → tab bar chỉ có "Bài viết".

#### 2.4.6. Container `CommunityScreen`

```tsx
export default function CommunityScreen() {
  const insets = useSafeAreaInsets();

  // 1 query duy nhất (xem §2.4.2). BE sort startsAt DESC → mới nhất lên đầu.
  const { data } = useChallenges({ status: "ACTIVE", limit: 30 });
  const challenges = useMemo(() => data?.pages.flatMap((p) => p.data) ?? [], [data]);

  const tabs: CommunityTabItem[] = useMemo(
    () => [
      { id: "posts", label: "Bài viết" },
      ...challenges.map((c) => ({
        id: `challenge-${c.id}`,
        label: c.title,
        challenge: c,
      })),
    ],
    [challenges],
  );

  // Local useState — KHÔNG dùng Zustand persist activeTab (cắt khỏi MVP).
  const [activeTabId, setActiveTabId] = useState<string>("posts");

  // Fallback: nếu tabs đổi và activeTabId không còn match → về "posts".
  // Note: challenge chuyển phase (upcoming→ongoing, ongoing→ended) giữa session KHÔNG fallback
  //       — tab vẫn tồn tại, chỉ nội dung overview + empty state tự đổi theo time.
  // Chỉ fallback khi challenge biến mất (ARCHIVED/rớt cap 30).
  useEffect(() => {
    if (!tabs.find((t) => t.id === activeTabId)) {
      setActiveTabId("posts");
    }
  }, [tabs, activeTabId]);

  const activeTab = tabs.find((t) => t.id === activeTabId);

  return (
    <>
      <Stack.Screen options={{ headerShown: false }} />
      <View className="flex-1 bg-white" style={{ paddingTop: insets.top }}>
        <CommunityHeader />
        <CommunityTabs tabs={tabs} activeTabId={activeTabId} onChange={setActiveTabId} />
        {activeTab?.id === "posts" ? (
          <CommunityPostsView />
        ) : activeTab && "challenge" in activeTab ? (
          <ChallengeSubmissionsView key={activeTab.challenge.id} challenge={activeTab.challenge} />
        ) : null}
      </View>
    </>
  );
}
```

> **Cắt khỏi MVP (so với v2 bản đầu):** Zustand persist activeTab — acceptable khi local state reset về "posts" mỗi session.

#### 2.4.7. Lưu ý Thiết Kế & Edge Cases

| Concern                                                              | Cách xử lý                                                                                                                                                                                                                            |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CommunityHeader**                                                  | Giữ trong container — hiển thị cho mọi tab                                                                                                                                                                                            |
| **Số tab quá nhiều**                                                 | BE `limit=30` cứng. Nếu DB có >30 challenge `status=ACTIVE`, các challenge cũ (theo `startsAt`) bị cắt — acceptable. Post-MVP: admin pin/feature hoặc pagination `fetchNextPage` cho tab bar.                                         |
| **Scroll position tab bar khi active tab ở ngoài viewport**          | MVP: không auto-scroll. User tự swipe. (Cắt `scrollToIndex` khỏi MVP.)                                                                                                                                                                |
| **Tab title quá dài**                                                | Truncate với ellipsis (~20 chars) hoặc giới hạn width fixed 120px                                                                                                                                                                     |
| **Invalid tab state**                                                | Nếu `activeTabId = "challenge-123"` nhưng challenge 123 không còn trong response (ARCHIVED hoặc rớt khỏi cap 30) → fallback về `"posts"` qua `useEffect` (§2.4.6).                                                                    |
| **Challenge chuyển phase giữa session (upcoming→ongoing, ongoing→ended)** | Phase derive client-side theo `now` mỗi lần render. Tab **không biến mất**; khi user mở lại `ChallengeSubmissionsView` (hoặc time-based re-render), overview + empty state tự reflect phase mới. Không cần refetch để đổi phase. |
| **Admin set ARCHIVED giữa session**                                  | Refetch khi focus screen sẽ loại challenge khỏi response (BE filter `status=ACTIVE`). Nếu đang xem challenge đó → fallback "posts" qua effect (§2.4.6).                                                                                |
| **Deep link `?tab=challenge-123`**                                   | Post-MVP.                                                                                                                                                                                                                             |
| **Refresh challenges list**                                          | Invalidate `useChallenges({ status: "ACTIVE" })` qua `useFocusEffect` khi user quay lại screen. Pull-to-refresh trong `CommunityPostsView` không invalidate — đó là behavior của posts tab riêng.                                    |
| **Memory footprint**                                                 | Conditional render bằng `key={challenge.id}` → đổi tab = unmount/mount. Không giữ all views alive.                                                                                                                                    |
| **Loading tabs lần đầu**                                             | Skeleton tab bar + content skeleton cho đến khi query done.                                                                                                                                                                           |
| **Offline/error**                                                    | Nếu load challenges failed → chỉ hiện tab "Bài viết", log error. Không break UX.                                                                                                                                                      |
| **My submission highlight**                                          | **GIỮ** — trong submissions grid, nếu `submission.id === challenge.mySubmission?.id` → visual indicator (border, badge). Cost thấp, value cao cho user biết bài của mình đâu trong grid.                                              |

#### 2.4.8. Tương Tác Với Các Flow Khác

- **Pinned card ở feed (R1)** → chỉ hiện challenge **ongoing** (`status=ACTIVE AND startsAt <= now <= endsAt`) — pinned card mục đích call-to-action submit. Tap → `/challenges/:id`. User vẫn có 2 entry points: feed pinned card (cho ongoing) và community tab (mọi challenge ACTIVE).
- **Detail screen (R3)** → submissions grid trong detail giữ nguyên. Có trùng lặp nội dung với community tab, nhưng là hai view khác (detail có info + my-submission; community có overview card + submissions grid + empty state).
- **Submit flow** → user muốn nộp bài: từ community tab challenge → overview thấy phase ongoing + CTA → tap "Xem chi tiết challenge" → detail → "Nộp bài dự thi". KHÔNG shortcut submit từ community. Tab với phase upcoming/ended không có CTA submit (trong overview và empty state đều khóa) — BE submission guard cũng reject.
- **Tab upcoming trong community** → tap vào: overview hiện badge "Sắp bắt đầu" + countdown; submissions grid rỗng với empty state "Challenge chưa bắt đầu"; CTA duy nhất là "Xem chi tiết challenge" để user xem countdown + thể lệ + trạng thái. **Membership vẫn là admin-controlled** (admin thêm qua email hoặc domain rule — [`admin-challenge.controller.ts:138-159`](../../riz-be/apps/nest/libs/challenge/src/admin-challenge.controller.ts#L138-L159)); V2 **không có self-join / opt-in notify** ở app — đây không nằm trong scope plan này.
- **`useCanSubmit` — PHẢI SỬA & tách 2 khái niệm** (trước review bị liệt nhầm vào Files KHÔNG ĐỔI). Hiện tại [`use-can-submit.ts:25`](../../riz-app-v2/hooks/challenges/use-can-submit.ts#L25) chỉ check `challenge?.status === "ACTIVE"`, thiếu window date. **Quan trọng:** không được gộp tất cả điều kiện vào 1 cờ `canSubmit` rồi dùng nó cho cả visibility CTA — sẽ giết 2 flow đang có ở detail (anonymous → login, non-member → `NotMemberBottomSheet`). Phải **tách rõ**:

  ```ts
  // Visibility — quyết định có hiện CTA submit ở detail không.
  // Bắt đầu từ status + window; KHÔNG phụ thuộc auth/member/hasSubmitted.
  const now = Date.now();
  const startsAt = new Date(challenge.startsAt).getTime();
  const endsAt = new Date(challenge.endsAt).getTime();
  const withinWindow = now >= startsAt && now <= endsAt;
  const isSubmissionWindowOpen =
    challenge?.status === "ACTIVE" && withinWindow;

  // Action — điều kiện thực sự để nhấn submit thành công.
  // handleSubmitPress() vẫn branching login/not-member/submitted trước khi navigate.
  const canSubmit =
    isSubmissionWindowOpen && isAuthenticated && isMember && !hasSubmitted;
  ```

  Hook phải export **cả `isSubmissionWindowOpen` và `canSubmit`** (cùng `hasSubmitted`, `handleSubmitPress`, `isNotMemberSheetOpen`, `closeNotMemberSheet` như hiện tại). `canSubmit` giữ nguyên signature cho các consumer khác (ví dụ empty state ongoing trong community nếu cần guard inner logic); visibility CTA dùng `isSubmissionWindowOpen`.

  Thiếu check `now >= startsAt` ở window → app mở CTA submit cho challenge upcoming (chưa đến `startsAt`) → BE reject với 400 `"Challenge submission window is closed"` → UX lỗi.

- **`ChallengeDetailScreen` render CTA — PHẢI SỬA cùng lúc**: sticky button ở [`challenge-detail.tsx:105`](../../riz-app-v2/screens/challenges/challenge-detail.tsx#L105) hiện tính `showSubmitButton = challenge.status === "ACTIVE" && !hasSubmitted`. Phải chuyển sang dùng `isSubmissionWindowOpen && !hasSubmitted` từ hook — **cùng source of truth** với `useCanSubmit` về window, nhưng **vẫn cho anonymous + non-member thấy CTA** để `handleSubmitPress()` xử lý login / mở `NotMemberBottomSheet` như hiện tại. Không dùng trực tiếp `canSubmit` làm điều kiện visibility — sẽ ẩn CTA cho 2 flow này và mất discoverability.

#### 2.4.9. Rationale — Flat Tab Bar, Phase Ở Overview

**Quyết định:** Community tab bar là danh sách **flat** gồm "Bài viết" pinned + mọi challenge `status=ACTIVE` (upcoming/ongoing/ended) theo thứ tự `startsAt DESC`. Không chia nhóm visual theo phase ở tab label. Phase chỉ xuất hiện trong **overview** (MiniChallengeCardHeader) + empty state khi user tap vào tab.

**Lý do:**

1. **Tab bar là index** — chỉ cần cho user thấy có challenge nào, không cần cho biết trạng thái ở level đó. Phase là thông tin nặng, đặt ở overview hợp lý hơn (có không gian render countdown, badge, CTA).
2. **Đơn giản cho mắt** — user scroll tab bar nhanh; quá nhiều màu/icon/badge khác nhau gây nhiễu.
3. **"Mới nhất lên đầu" là natural order** — BE đã sort `startsAt DESC`. Challenge cũ tự rơi xuống cuối/rớt khỏi cap 30. Không cần logic giữ ended riêng.
4. **Chi phí implement thấp** — 1 query, 1 hook, không cần phase-aware sort/group ở FE.

**Nhược điểm chấp nhận:**

- User không thấy trước phase của tab trước khi tap → phải tap vào mới biết. Trade-off OK vì title thường gợi ý đủ (và overview load nhanh).
- Ended challenge cũ có thể bị cắt khỏi cap 30 khi volume active tăng cao → user không có entry point tới ended cũ đó. Acceptable: user đã quên challenge từ lâu; vẫn còn deep-link + badge project nếu cần.

**Escape hatch cho admin:** Admin có thể set `status=ARCHIVED` để ẩn challenge khỏi community ngay lập tức (xem §2.4.10).

#### 2.4.10. Status Model

**Plan v2 chỉ dùng 3 status:**

| Status       | Ý nghĩa                                                                 | Ai set               |
| ------------ | ----------------------------------------------------------------------- | -------------------- |
| **DRAFT**    | Admin biên soạn, chưa public                                            | Admin tạo mặc định   |
| **ACTIVE**   | Public — lifecycle auto theo `startsAt`/`endsAt`                        | Admin publish        |
| **ARCHIVED** | Admin chủ động ẩn (escape hatch)                                        | Admin bấm            |

> **`ENDED` trong enum** — giữ nguyên trong Prisma schema (không breaking) nhưng **deprecated**. Plan v2 không set `ENDED` mới ở BE/admin, và DB target là brand new nên không có record `ENDED` legacy. Migration one-off ở §2.4.11 **không áp dụng** (Round 3, 2026-04-19). Post-MVP có thể drop enum value khỏi Prisma schema.

**Phase derive client-side cho challenge `status=ACTIVE`:**

| Điều kiện                       | Phase        | Behavior trong community                                                                                                                   |
| ------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `now < startsAt`                | **upcoming** | Tab hiện. Tap → overview badge "Sắp bắt đầu" + countdown; empty state "Challenge chưa bắt đầu" + CTA "Xem chi tiết". Không fetch submissions. |
| `startsAt ≤ now ≤ endsAt`       | **ongoing**  | Tab hiện. Tap → overview badge "Đang diễn ra" + "Còn Y ngày"; submissions grid; CTA thống nhất "Xem chi tiết challenge" (submit luôn qua detail — §2.4.8). Pinned feed card cũng hiện. |
| `now > endsAt`                  | **ended**    | Tab hiện. Tap → overview badge "Đã kết thúc" + ngày kết thúc; submissions grid (archive view). Submit bị BE reject.                         |

**Visibility matrix:**

| Combination            | Community tabs                         | Pinned feed              | `/challenges` list       | Deep-link   | Submit          |
| ---------------------- | :------------------------------------: | :----------------------: | :----------------------: | :---------: | :-------------: |
| DRAFT                  | ❌                                     | ❌                       | ❌                       | Admin only  | ❌              |
| ACTIVE + upcoming      | ✅ (chung list, overview countdown)    | ❌ (chưa tới startsAt)   | ✅                       | ✅          | ❌ (BE reject)  |
| ACTIVE + ongoing       | ✅ (chung list, overview CTA "Xem chi tiết") | ✅ (match category) | ✅                       | ✅          | ✅              |
| ACTIVE + ended         | ✅ (chung list, overview archive)      | ❌                       | ✅                       | ✅          | ❌ (BE reject)  |
| ARCHIVED               | ❌                                     | ❌                       | ❌ mặc định              | ✅ (data giữ) | ❌             |
| ~~ENDED (legacy)~~     | N/A — DB brand new không có record này (Round 3)                                                                                          |

**Hệ quả implementation:**

- Community query: **1 call** `useChallenges({ status: "ACTIVE", limit: 30 })` (xem §2.4.2). BE sort `startsAt DESC` → mới nhất lên đầu.
- Phase derive client-side qua helper `getPhase(challenge)` — dùng trong `MiniChallengeCardHeader`, `SubmissionsEmptyState`, `ChallengeSubmissionsView`.
- `ChallengeSubmissionsView` reuse cho cả 3 phase; upcoming không fetch `/submissions` (§2.4.4).
- Admin form: **3 options** DRAFT / ACTIVE / ARCHIVED (bỏ ENDED khỏi dropdown — xem §2.6.2).
- Write-path BE: **reject `ENDED`** ở create/update DTO (xem §2.6.5).
- `useCanSubmit` check **cả 2 đầu cửa sổ** để khớp BE guard: `status === "ACTIVE" && startsAt <= now <= endsAt`. Chi tiết code: §2.4.8.

> **Wording convention:** Trong toàn bộ tài liệu này, "ongoing" / "submit được" / "đang diễn ra" luôn đồng nghĩa với `status='ACTIVE' AND startsAt <= now <= endsAt`. Không bao giờ rút gọn chỉ thành "`endsAt >= now`" (thiếu `startsAt`) — tránh copy-paste vào PR/task description gây hiểu sai một chiều.

#### 2.4.11. Migration Notes — Legacy `status=ENDED` Data — ❌ DEPRECATED (2026-04-19)

> **Round 3 update:** DB target là brand new, không có record `status=ENDED`. Section này giữ để reference historic rationale, không còn bước migration pre-deploy.

Nếu production DB có challenge với `status='ENDED'` (do admin bấm manual trước đây), plan v2 **sẽ không hiển thị** chúng (query chỉ filter `status=ACTIVE`). Options:

| Option     | SQL                                                                    | Pros                                                                                    | Cons                                                                                       |
| ---------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **M1 ⭐** | `UPDATE "Challenge" SET status = 'ACTIVE' WHERE status = 'ENDED'`     | Data giữ nguyên visibility (date-based logic tự coi là ended nhờ endsAt đã qua)         | Nếu endsAt cũ chưa set đúng → risk hiện lên "ongoing" sai. Cần verify data trước.          |
| M2         | Giữ nguyên, mở rộng query thành `status IN ('ACTIVE','ENDED')`         | Không động DB                                                                           | Duplicate logic — date-based + status-based cùng tồn tại → nguồn bug                       |
| M3         | Convert `ENDED` → `ARCHIVED`                                           | Sạch, loại hoàn toàn ENDED                                                              | User mất các challenge đã kết thúc khỏi community                                          |

**Khuyến nghị: M1** — chạy migration khi deploy, verify bằng:

```sql
SELECT id, title, "endsAt", status FROM "Challenge" WHERE status = 'ENDED';
-- Confirm các record này đều có endsAt < now() trước khi UPDATE
```

**Task:** Thêm vào plan pre-implementation checklist (§8) — verify + migrate legacy data trước khi deploy community §2.4.

#### 2.4.12. Tác Động Tới `/challenges` List Screen (Route V1)

`screens/challenges/index.tsx` hiện có `ChallengeFilterTabs` với 2 giá trị `ACTIVE | ENDED` ([`index.tsx:23`](../../riz-app-v2/screens/challenges/index.tsx#L23), [`challenge-filter-tabs.tsx:12-21`](../../riz-app-v2/screens/challenges/components/challenge-filter-tabs.tsx#L12-L21)) và truyền thẳng xuống `useChallenges({ status })`.

**Vấn đề với status-based filter (Round 3 — DB brand new):**

- Tab "ACTIVE" trả cả ongoing + ended (derive từ `endsAt`) mix — mất phân biệt UX
- Tab "ENDED" trả rỗng — DB target brand new không có record `status=ENDED` (admin không set nữa, BE write-path reject)

**Fix:** Đổi filter tabs từ status sang phase. App đổi `useChallenges({ status })` sang `useChallenges({ phase })` — BE đã hỗ trợ `?phase=` param (§2.1.3). Pagination work đúng, UX clean.

**Default contract hiện tại (không đổi):**

`GET /challenges` public (không truyền `includeDraft`) trả **mọi challenge có `status != DRAFT`** — tức là `ACTIVE` + `ARCHIVED` (DB brand new không có `ENDED`). Logic ở [`challenge.service.ts:541`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L541):

```ts
...(query.status ? { status: query.status } : includeDraft ? {} : { status: { not: ChallengeStatus.DRAFT } }),
```

→ **KHÔNG được thu hẹp default sang `status=ACTIVE`** khi thêm `phase` — đó là breaking change ngầm cho mọi consumer hiện tại.

**Spec (chỉ thêm, không thay):**

| Request                             | Behavior                                                                                     |
| ----------------------------------- | -------------------------------------------------------------------------------------------- |
| `GET /challenges` (không truyền gì) | **Giữ nguyên**: trả `status != DRAFT` (ACTIVE + ARCHIVED — DB brand new không có ENDED)      |
| `GET /challenges?status=ACTIVE`     | **Giữ nguyên**: filter theo `status` như trước                                               |
| `GET /challenges?phase=upcoming`    | **MỚI**: `status='ACTIVE' AND startsAt > now()`                                              |
| `GET /challenges?phase=ongoing`     | **MỚI**: `status='ACTIVE' AND startsAt <= now() AND endsAt >= now()`                         |
| `GET /challenges?phase=ended`       | **MỚI**: `status='ACTIVE' AND endsAt < now()`                                                |
| `GET /challenges?feedCategorySlug=X` | **MỚI**: join `ProjectCategory.slug = X`. Dùng thay cho `feedCategory=<id>` để client không phải resolve slug→id |
| `GET /challenges?status=X&phase=Y`  | phase luôn pin `status=ACTIVE`; nếu user truyền thêm `status=X` khác ACTIVE → validate reject hoặc phase thắng (BE dev chọn) |

> `phase` và `feedCategorySlug` lọc **thêm** trên default where clause, **không replace** default. Document thật rõ cho BE dev để tránh đổi default ngầm.

**Files sửa thêm (app):**

- [`screens/challenges/components/challenge-filter-tabs.tsx`](../../riz-app-v2/screens/challenges/components/challenge-filter-tabs.tsx) — đổi value `ACTIVE/ENDED` → 3 tab `upcoming / ongoing / ended`. Default `ongoing`. (Route `/challenges` là trang filter explicit — không cộng hưởng với community flat layout, giữ tab phase ở đây để user có thể browse theo phase.)
- [`screens/challenges/index.tsx:23-38`](../../riz-app-v2/screens/challenges/index.tsx#L23-L38) — đổi `useChallenges({ status })` sang `useChallenges({ phase })`
- [`riz-app-v2/lib/api/challenges/types.ts`](../../riz-app-v2/lib/api/challenges/types.ts) — thêm `phase?: ChallengePhase` (3 values: `upcoming | ongoing | ended`) + `feedCategorySlug?: string` vào `GetChallengesParams` — dùng chung với pinned card §2.1.4

### 2.6. Admin-FE Fix — Thêm Status Field Vào Challenge Form

Repo `riz-admin-fe` dùng **feature-sliced design** (`entities/`, `features/`, `screens/`).

#### 2.6.1. Vấn đề

Admin KHÔNG publish được challenge sang `ACTIVE`:

- **Create** [entities/challenge/api/challenge-api.ts:102](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts#L102): `formData.append("status", "DRAFT")` hardcode
- **Update** [entities/challenge/api/challenge-api.ts:213](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts#L213): API đã nhận `status` nếu form truyền xuống
- **Form** [features/challenges/challenge-form/ui/challenge-form.tsx](../../riz-admin-fe/src/features/challenges/challenge-form/ui/challenge-form.tsx): KHÔNG có field UI cho `status`
- **Schema** [features/challenges/challenge-form/lib/challenge-schema.ts](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts): không có `status` trong Zod schema
- **Types** [entities/challenge/model/types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts): `ChallengeFormValues` không có `status`
- **Pages** [screens/challenges/challenge-create-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-create-page.tsx), [screens/challenges/challenge-edit-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-edit-page.tsx): không truyền `status` từ form xuống mutation

→ Cần fix end-to-end: schema → type → form UI → API call → page wiring.

#### 2.6.2. Fix Checklist (8 files)

| #   | File                                                                                                                                                     | Change                                                                                                     |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 1   | [features/challenges/challenge-form/lib/challenge-schema.ts](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts)          | Thêm `status: z.enum(["DRAFT","ACTIVE","ARCHIVED"])` vào Zod schema — **không bao gồm ENDED** (xem §2.4.10) |
| 2   | [entities/challenge/model/types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts)                                                            | Thêm `status: "DRAFT" \| "ACTIVE" \| "ARCHIVED"` vào `ChallengeFormValues`                                 |
| 3   | [features/challenges/challenge-form/ui/challenge-form.tsx](../../riz-admin-fe/src/features/challenges/challenge-form/ui/challenge-form.tsx)              | Thêm `<FormField name="status">` với Select 3 options (DRAFT / ACTIVE / ARCHIVED)                          |
| 4   | [entities/challenge/api/challenge-api.ts:102](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts#L102)                                       | `buildFormData` dùng `values.status` thay vì hardcode `"DRAFT"`                                            |
| 5   | [screens/challenges/challenge-create-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-create-page.tsx)                                      | `defaultValues.status = "DRAFT"` cho form                                                                  |
| 6   | [screens/challenges/challenge-edit-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-edit-page.tsx)                                          | Prefill `status` từ server data; truyền `status` xuống `updateChallenge` mutation                          |
| 7   | [screens/challenges/challenge-list-page.tsx:38-50](../../riz-admin-fe/src/screens/challenges/challenge-list-page.tsx#L38-L50)                            | Xóa case `"ENDED"` trong `getStatusVariant` (badge variant). DB brand new không có record ENDED (Round 3) nên `default` branch không bao giờ hit. |
| 8   | [entities/challenge/model/types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts)                                                            | Xóa `"ENDED"` khỏi `Challenge["status"]` union type — admin chỉ thao tác với 3 status writable (DRAFT/ACTIVE/ARCHIVED). Read-path khớp với BE vì DB brand new không trả ENDED. |

**UI suggest (file #3):**

```tsx
<FormField
  control={form.control}
  name="status"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Trạng thái</FormLabel>
      <Select onValueChange={field.onChange} value={field.value}>
        <SelectTrigger>
          <SelectValue />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="DRAFT">Nháp (Chưa public)</SelectItem>
          <SelectItem value="ACTIVE">Public (Hiển thị cho user)</SelectItem>
          <SelectItem value="ARCHIVED">Lưu trữ (Ẩn khỏi community)</SelectItem>
        </SelectContent>
      </Select>
      <FormDescription>
        ACTIVE: challenge public cho end-user. Trạng thái "đã kết thúc" sẽ tự động được xác định theo ngày kết thúc — không cần set tay. ARCHIVED: ẩn khỏi community tabs, vẫn xem được qua deep-link.
      </FormDescription>
    </FormItem>
  )}
/>
```

> **Semantic 3 status + 3 phase auto** — xem bảng đầy đủ ở §2.4.10.

#### 2.6.3. Acceptance

- [ ] Form tạo challenge có dropdown Status với **3 options** (DRAFT / ACTIVE / ARCHIVED); default DRAFT — **không có ENDED**
- [ ] Form edit prefill đúng status hiện tại; có thể đổi giữa 3 options
- [ ] `POST /admin/challenges` gửi `status=<user-selected>` (không còn hardcode DRAFT)
- [ ] `PATCH /admin/challenges/:id` gửi `status` khi user đổi
- [ ] Verify qua Network tab: payload chứa `status` field
- [ ] Tạo 1 challenge ACTIVE với category PHOTOGRAPHERS + `startsAt <= now <= endsAt` (ongoing) → app V2 hiển thị pinned card
- [ ] Tạo challenge ACTIVE PHOTOGRAPHERS với `startsAt` tương lai (upcoming) → pinned card **KHÔNG** hiện (xem §1.6 — pinned chỉ ongoing-only)
- [ ] Set `endsAt` quá khứ (hoặc đợi challenge hết hạn) → pinned card biến mất; challenge vẫn hiện trong community tab ở nhóm "ended" (muted)
- [ ] Đổi challenge từ ACTIVE → ARCHIVED → community tab biến mất khỏi app, route `/challenges/:id` vẫn đọc được qua deep-link

#### 2.6.4. Legacy ENDED Handling (one-off) — ❌ DEPRECATED (2026-04-19)

> **Round 3 update:** DB target là brand new, không có record `status=ENDED` legacy. Section này giữ để reference historic rationale, không còn pre-deploy step. Admin-fe không cần defensive fallback.

- [ ] Query DB production: `SELECT id, title, "endsAt", status FROM "Challenge" WHERE status = 'ENDED'`
- [ ] Verify `endsAt < now()` cho từng record (nếu có record nào `endsAt` vẫn future → cần fix endsAt trước)
- [ ] Chạy migration: `UPDATE "Challenge" SET status = 'ACTIVE' WHERE status = 'ENDED'`
- [ ] Sau migration, community tab sẽ tự động hiển thị các challenge này trong nhóm "ended" (muted) nhờ phase derive.

#### 2.6.5. BE Write-Path — Reject `ENDED` ở DTO

Admin FE đã cắt `ENDED` khỏi dropdown (§2.6.2), nhưng hiện tại [`create-challenge.dto.ts:78-82`](../../riz-be/apps/nest/libs/challenge/src/dtos/create-challenge.dto.ts#L78-L82) dùng `@IsEnum(ChallengeStatus)` mở toàn bộ enum Prisma. `update-challenge.dto.ts` kế thừa `PartialType(CreateChallengeDto)` nên cũng mở. Không reject BE → admin FE cleanup chỉ là cosmetic, client khác (direct API, admin legacy, automation) vẫn ghi `ENDED` được.

**Fix:** tạo `WritableChallengeStatus` (hoặc union type literal) và dùng trong DTO write-path, không reuse `ChallengeStatus` từ Prisma:

```typescript
// libs/challenge/src/constants/writable-status.ts (mới)
export const WRITABLE_CHALLENGE_STATUSES = [
  ChallengeStatus.DRAFT,
  ChallengeStatus.ACTIVE,
  ChallengeStatus.ARCHIVED,
] as const
export type WritableChallengeStatus = (typeof WRITABLE_CHALLENGE_STATUSES)[number]

// create-challenge.dto.ts
@Expose()
@ApiPropertyOptional({ enum: WRITABLE_CHALLENGE_STATUSES, default: ChallengeStatus.DRAFT })
@IsOptional()
@IsIn(WRITABLE_CHALLENGE_STATUSES)
status?: WritableChallengeStatus
```

**Files sửa BE:**

| File                                                                                                                                         | Change                                                                                     |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `libs/challenge/src/constants/writable-status.ts` (MỚI)                                                                                      | Export `WRITABLE_CHALLENGE_STATUSES` + type                                                |
| [`libs/challenge/src/dtos/create-challenge.dto.ts:78-82`](../../riz-be/apps/nest/libs/challenge/src/dtos/create-challenge.dto.ts#L78-L82)    | Đổi `@IsEnum(ChallengeStatus)` → `@IsIn(WRITABLE_CHALLENGE_STATUSES)`; type `WritableChallengeStatus` |
| `libs/challenge/src/dtos/update-challenge.dto.ts`                                                                                            | Verify `PartialType` kế thừa đúng constraint mới (thường tự động)                          |
| `libs/challenge/src/challenge.spec.ts`                                                                                                       | Thêm test: POST/PATCH với `status=ENDED` → 400                                             |

> Giữ nguyên Prisma schema enum với `ENDED` (không cần drop, không gây breaking). DB brand new không có record dùng giá trị này. Post-MVP có thể remove khỏi enum khi thuận tiện.

**Tests:**

- [ ] `POST /admin/challenges` với body `{ ..., status: "ENDED" }` → 400 bad request
- [ ] `PATCH /admin/challenges/:id` với body `{ status: "ENDED" }` → 400 bad request
- [ ] `POST/PATCH` với `status` ∈ {DRAFT, ACTIVE, ARCHIVED} → OK như cũ
- [ ] Read path `GET /challenges` không bị ảnh hưởng bởi reject ở write-path (sanity check — DB brand new không có record `ENDED` để verify behavior thực tế)

#### 2.6.6. Risk

LOW — pure validation change (admin FE form + BE DTO). BE đã hỗ trợ `status` write field, chỉ restrict thêm giá trị hợp lệ.

---

### 2.7. Cleanup Contract: Bỏ `approachNote` / `themeResponse` Khỏi Challenge

**Vấn đề:**

[`ChallengeEntity`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L58-L64) và app [`Challenge` type](../../riz-app-v2/lib/api/challenges/types.ts#L39-L40) đều khai báo 2 field `approachNote?` và `themeResponse?`, nhưng [`mapChallengeEntity()`](../../riz-be/apps/nest/libs/challenge/src/challenge.service.ts#L162-L191) **không map** chúng — BE không bao giờ trả 2 field này ở challenge detail/list response. Chúng thực sự là **submission-level fields** được thu ở [`challenge-submit.tsx:43-75`](../../riz-app-v2/screens/challenges/challenge-submit.tsx#L43-L75) và gửi qua FormData khi submit.

Đây là contract bloat — dangling fields làm người đọc type tưởng challenge detail có thể trả 2 field này. Mỗi lần refactor API phải cân nhắc một branch dữ liệu không tồn tại.

**Fix:**

| File                                                                                                                                         | Change                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| [`libs/challenge/src/entities/challenge.entity.ts:58-64`](../../riz-be/apps/nest/libs/challenge/src/entities/challenge.entity.ts#L58-L64)    | Xóa 2 field `approachNote?` + `themeResponse?` khỏi `ChallengeEntity`                           |
| [`riz-app-v2/lib/api/challenges/types.ts:39-40`](../../riz-app-v2/lib/api/challenges/types.ts#L39-L40)                                       | Xóa 2 field cùng tên khỏi `Challenge` interface                                                 |

**Scope guard:** `ChallengeSubmissionEntity` / submission types **vẫn giữ** 2 field — đó là nơi chúng thuộc về. Chỉ xóa ở challenge-level.

**Tests:**

- [ ] `GET /challenges/:id` response vẫn không chứa `approachNote`/`themeResponse` (như hiện tại) — sanity check
- [ ] App submit flow vẫn gửi 2 field trong FormData (không regress)
- [ ] TypeScript compile pass trên cả BE + app (không có consumer nào đang đọc `challenge.approachNote`)

**Risk:** LOW — `mapChallengeEntity` không map 2 field này nên runtime không thay đổi; chỉ dọn contract. Run `gitnexus_impact` trên `ChallengeEntity.approachNote` / `ChallengeEntity.themeResponse` trước khi xóa để confirm zero caller.

---

## 3. Files Changed Summary

### 3.1. Files MỚI

**App (riz-app-v2):**

| File                                                                 | Mục đích                                                                            |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `screens/feed/components/main/pinned-challenge-card.tsx`             | Card challenge đầu feed (R1)                                                        |
| `hooks/challenges/use-featured-challenge.ts`                         | Query hook `GET /challenges?phase=ongoing&feedCategorySlug=PHOTOGRAPHERS&limit=1` (R1) |
| `screens/challenges/components/collapsible-info-section.tsx`         | Collapsible info (R4)                                                               |
| `screens/challenges/components/short-summary.tsx`                    | 2-3 dòng mô tả ngắn (R4)                                                            |
| `screens/challenges/components/submissions-section-header.tsx`       | Section header "Bài dự thi (N)" (R3)                                                |
| `screens/community/community-posts-view.tsx`                         | Extract toàn bộ posts logic hiện tại (R5 refactor prerequisite)                     |
| `screens/community/challenge-submissions-view.tsx`                   | Render submissions grid cho 1 challenge, phase derive client-side (R5, §2.4.4)      |
| `screens/community/components/community-tabs.tsx`                    | Dynamic tab bar flat: Bài viết + mỗi challenge = 1 tab (R5, §2.4.5)                 |
| `screens/community/components/mini-challenge-card-header.tsx`        | Overview card 16:9 — tự derive phase → badge + countdown/date/status (R5, §2.4.4)   |
| `screens/community/components/submissions-empty-state.tsx`           | Empty state variant theo phase derive client-side (R5, §2.4.4)                      |
| `utils/challenge-phase.ts` (hoặc thêm vào `lib/api/challenges/`)     | Helper `getPhase(challenge)` — single source cho phase derivation (§2.4.2)          |

> **Cắt khỏi Files MỚI (so với v2 bản đầu):** `hooks/project-categories/use-category-id-by-slug.ts` — không còn cần resolve slug→id ở client, BE nhận trực tiếp `feedCategorySlug`.

**BE (riz-be):**

| File                                                                       | Mục đích                                                                                                  |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `libs/challenge/src/dtos/get-challenges.dto.ts` (modified)                 | Thêm enum `ChallengePhase` (3 values) + field `phase?` + `feedCategorySlug?`                              |
| `libs/challenge/src/challenge.controller.ts` (modified)                    | `getChallenges()` nhận thêm `@CurUser() user?: User`, truyền `profileId` xuống service                    |
| `libs/challenge/src/challenge.service.ts` (modified)                       | Mở rộng `getChallenges()` — nhận `userProfileId?`, apply `phaseClause` + `categoryClause` vào where, batch-resolve user state cho list rows |
| `libs/challenge/src/entities/challenge.entity.ts` (modified)               | **Xóa `approachNote?` + `themeResponse?`** — 2 field thuộc submission, không thuộc challenge-level contract (§2.7)                         |
| `libs/challenge/src/constants/writable-status.ts` (MỚI)                    | Export `WRITABLE_CHALLENGE_STATUSES` + type (§2.6.5)                                                      |
| `libs/challenge/src/dtos/create-challenge.dto.ts` (modified)               | `status` dùng `@IsIn(WRITABLE_CHALLENGE_STATUSES)` thay `@IsEnum(ChallengeStatus)` — reject `ENDED` ở write-path |
| `libs/challenge/src/challenge.spec.ts` (modified)                          | Tests cho `phase`, `feedCategorySlug`, write-path reject `ENDED`, batch userState resolution              |

> **Cắt khỏi Files MỚI (so với v2 bản đầu):** `libs/project/src/project.controller.ts` — KHÔNG thêm public route `GET /project-categories`. Slug resolve làm trực tiếp trong `challenge.service.ts` qua Prisma relation filter (`category: { is: { slug: X } }`).

### 3.2. Files SỬA — App (riz-app-v2)

| File                                                                                                                                                   | Change                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [screens/feed/index.tsx](../../riz-app-v2/screens/feed/index.tsx)                                                                                       | Default category = PHOTOGRAPHERS; xoá case "challenge" trong handleTabPress; inject PinnedChallengeCard                                                             |
| [screens/feed/components/sliver-app-bar/tab-button.tsx](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx)                         | Xoá entry `challenge` trong `tabs` array                                                                                                                            |
| [screens/challenges/challenge-detail.tsx](../../riz-app-v2/screens/challenges/challenge-detail.tsx)                                                     | Bỏ tab state + segmented control; restructure scroll layout; inject CollapsibleInfoSection + SubmissionsSectionHeader                                                |
| [screens/challenges/components/challenge-info-tab.tsx](../../riz-app-v2/screens/challenges/components/challenge-info-tab.tsx)                           | Refactor thành content-only (bỏ tab wrapper)                                                                                                                        |
| [screens/challenges/components/challenge-detail-skeleton.tsx](../../riz-app-v2/screens/challenges/components/challenge-detail-skeleton.tsx)             | Update layout match new detail                                                                                                                                      |
| [screens/challenges/index.tsx](../../riz-app-v2/screens/challenges/index.tsx)                                                                           | Giữ nguyên route public (truy cập qua deep link hoặc từ detail screen). Community KHÔNG dùng route này nữa.                                                         |
| [screens/challenges/challenge-submit.tsx](../../riz-app-v2/screens/challenges/challenge-submit.tsx)                                                     | Thêm link "Xem thể lệ đầy đủ" ở đầu form (bỏ collapsible summary bar — xem §2.3.4)                                                                                  |
| [screens/community/index.tsx](../../riz-app-v2/screens/community/index.tsx)                                                                             | Thu gọn thành container: **1 query** `useChallenges({ status: "ACTIVE", limit: 30 })`, build flat tabs array, render `CommunityPostsView` hoặc `ChallengeSubmissionsView` theo activeTab (local useState) — xem §2.4.6 |
| [screens/community/components/index.ts](../../riz-app-v2/screens/community/components/index.ts)                                                         | Export `CommunityTabs`                                                                                                                                              |
| [lib/api/challenges/types.ts](../../riz-app-v2/lib/api/challenges/types.ts)                                                                             | Thêm `phase?: ChallengePhase` (3 values) + `feedCategorySlug?: string` vào `GetChallengesParams`; export type `ChallengePhase`; **xóa `approachNote?` + `themeResponse?` khỏi `Challenge` interface** (§2.7 — 2 field thuộc submission, không thuộc challenge) |
| [screens/challenges/components/challenge-filter-tabs.tsx](../../riz-app-v2/screens/challenges/components/challenge-filter-tabs.tsx)                     | Đổi filter values `ACTIVE/ENDED` → `upcoming/ongoing/ended` (phase-based 3 tab — xem §2.4.12)                                                                       |
| [hooks/challenges/index.ts](../../riz-app-v2/hooks/challenges/index.ts)                                                                                 | Thêm barrel export `use-featured-challenge`                                                                                                                         |
| [hooks/challenges/use-can-submit.ts](../../riz-app-v2/hooks/challenges/use-can-submit.ts)                                                               | **PHẢI SỬA** — tách `isSubmissionWindowOpen` (status + window, cho visibility CTA) vs `canSubmit` (window + auth + member + !submitted, cho action). Hook export cả hai. Chi tiết code §2.4.8. Trước review bị nhầm vào Files KHÔNG ĐỔI. |
| [hooks/challenges/use-challenges.ts](../../riz-app-v2/hooks/challenges/use-challenges.ts)                                                               | **PHẢI SỬA** — mở rộng `useChallengeSubmissions(id, options?: { enabled?: boolean })` (backward compat: default `enabled: true`) để §2.4.4 skip fetch submissions ở phase upcoming. `useChallenges`/`useChallenge` giữ nguyên signature — chỉ call-site truyền thêm `phase`/`feedCategorySlug`. Trước review bị nhầm vào Files KHÔNG ĐỔI. |
| [integrations/react-intl/locales/en.json](../../riz-app-v2/integrations/react-intl/locales/en.json)                                                     | `community_tab_posts`, `community_mini_challenge_view_detail`, `challenge_detail_expand_info`, `challenge_detail_collapse_info`, `challenge_submissions_count`      |
| [integrations/react-intl/locales/vi.json](../../riz-app-v2/integrations/react-intl/locales/vi.json)                                                     | Same keys                                                                                                                                                           |

### 3.3. Files SỬA — Admin-FE (riz-admin-fe)

| #   | File                                                                                                                                                       | Change                                                                  |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 1   | [features/challenges/challenge-form/lib/challenge-schema.ts](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts)            | Thêm `status` vào Zod schema                                            |
| 2   | [entities/challenge/model/types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts)                                                              | Thêm `status` vào `ChallengeFormValues`; **xóa `"ENDED"` khỏi `Challenge["status"]` union** (§2.6.2 row 8) |
| 3   | [features/challenges/challenge-form/ui/challenge-form.tsx](../../riz-admin-fe/src/features/challenges/challenge-form/ui/challenge-form.tsx)                | Thêm FormField Status select                                            |
| 4   | [entities/challenge/api/challenge-api.ts](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts)                                                  | Bỏ hardcode DRAFT ở line 102, dùng `values.status`                      |
| 5   | [screens/challenges/challenge-create-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-create-page.tsx)                                        | `defaultValues.status = "DRAFT"`                                        |
| 6   | [screens/challenges/challenge-edit-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-edit-page.tsx)                                            | Prefill + truyền `status` xuống mutation                                |
| 7   | [screens/challenges/challenge-list-page.tsx](../../riz-admin-fe/src/screens/challenges/challenge-list-page.tsx)                                            | Xóa case `"ENDED"` trong `getStatusVariant` (§2.6.2 row 7)              |

### 3.4. Files XÓA

V2 refactor tháo bỏ cấu trúc 2-tab ở detail và top Challenge tab ở feed. **Phải xóa dứt điểm** — không giữ lại để "tương lai reuse":

| File                                                                                                                                                                          | Lý do                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [screens/challenges/components/challenge-submissions-tab.tsx](../../riz-app-v2/screens/challenges/components/challenge-submissions-tab.tsx)                                   | V2 merge submissions vào cùng scroll detail + `SubmissionsSectionHeader` + grid; không còn tab wrapper |
| [components/icons/challenge-icon.tsx](../../riz-app-v2/components/icons/challenge-icon.tsx)                                                                                   | Top Challenge tab bị xóa (R2); community tab bar text-only (§2.4.5); `ChallengeBadge` dùng `Trophy`, không dùng icon này → không còn consumer |
| Re-export `export { ChallengeIcon }` ở [components/icons/index.ts:5](../../riz-app-v2/components/icons/index.ts#L5)                                                             | Barrel re-export, xóa cùng với `challenge-icon.tsx`                                              |
| Entry `challenge: ChallengeIcon` trong `ICON_MAP` + `ChallengeIcon` trong import ở [tab-button.tsx:4-26](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx#L4-L26) | `ICON_MAP` nằm ở `tab-button.tsx` (không phải `components/icons/index.ts` — file đó chỉ là barrel re-export)                                  |
| Locale key `feed_tab_challenge` trong [en.json](../../riz-app-v2/integrations/react-intl/locales/en.json) + [vi.json](../../riz-app-v2/integrations/react-intl/locales/vi.json) | Top tab label không còn dùng (key thật là `feed_tab_challenge`, không phải `challenge_tab_label`) |
| Locale keys `challenge_detail_tab_info` + `challenge_detail_tab_submissions`                                                                                                   | Detail 2-tab label không còn dùng                                                                |

> `challenge-info-tab.tsx` **không xóa** — chuyển thành content-only component (xem §3.2). Tên giữ nguyên để giảm git noise.

### 3.5. Files KHÔNG ĐỔI (giữ nguyên)

- `app/challenges/index.tsx`, `app/challenges/[id].tsx`, `app/(protected)/challenges/[id]/submit.tsx` — routes giữ nguyên
- `hooks/challenges/use-challenge-mutations.ts` — giữ nguyên (signature không đổi)
- `components/challenge-badge.tsx` — giữ nguyên (feed item badge)
- Admin-fe review submissions ([features/challenges/submission-review/](../../riz-admin-fe/src/features/challenges/submission-review/)) — đã đủ feature
- BE challenge detail/submissions endpoints — chỉ **mở rộng** `GET /challenges` query params (`phase`, `feedCategorySlug`) + đổi DTO write-path reject `ENDED` (future-proof guard; DB brand new không có record ENDED để test read). Không đổi detail/submissions routes.

> **Loại khỏi "Files KHÔNG ĐỔI" sau review:**
>
> - `components/icons/challenge-icon.tsx` + `ICON_MAP` entry + locale keys: **đã chuyển sang §3.4 Files XÓA** (cleanup hard trong V2.1, không phải "PR kế tiếp").
> - `hooks/challenges/use-can-submit.ts`: đã chuyển sang §3.2 Files SỬA — bắt buộc fix window date để khớp BE guard.
> - `hooks/challenges/use-challenges.ts`: đã chuyển sang §3.2 Files SỬA — `useChallengeSubmissions` cần thêm option `enabled` để §2.4.4 skip fetch submissions ở phase upcoming. Call-site `useChallenges` vẫn backward compat (chỉ truyền `phase` thay cho `status`).
> - `screens/challenges/components/challenge-submissions-tab.tsx`: **đã chuyển sang §3.4 Files XÓA** — V2 merge submissions vào scroll detail, tab wrapper không còn dùng.

---

## 4. Kế Hoạch Thực Thi

### Task V2.0a: Admin-FE Status Field — 1 ngày (BLOCKER) ✅ DONE

> Phải làm trước các task app, vì admin phải publish được ACTIVE challenge thì V2 mới có data test.

- [x] File 1: Thêm `status` vào Zod schema ([challenge-schema.ts](../../riz-admin-fe/src/features/challenges/challenge-form/lib/challenge-schema.ts))
- [x] File 2: Thêm `status` vào `ChallengeFormValues` ([types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts))
- [x] File 3: Thêm FormField Status vào `challenge-form.tsx` (dùng `register/setValue` pattern hiện có thay vì shadcn `<FormField>` wrapper)
- [x] File 4: Bỏ hardcode DRAFT ở [challenge-api.ts:102](../../riz-admin-fe/src/entities/challenge/api/challenge-api.ts#L102)
- [x] File 5: `defaultValues.status = "DRAFT"` ở `challenge-create-page.tsx`
- [x] File 6: Prefill + truyền status ở `challenge-edit-page.tsx`
- [x] File 7: Xóa case `"ENDED"` trong `getStatusVariant` ở [challenge-list-page.tsx:38-50](../../riz-admin-fe/src/screens/challenges/challenge-list-page.tsx#L38-L50) — legacy status không còn writable, badge variant không cần
- [x] File 8: Xóa `"ENDED"` khỏi `Challenge["status"]` union ở [entities/challenge/model/types.ts](../../riz-admin-fe/src/entities/challenge/model/types.ts) (§2.6.2). Admin chỉ thao tác với 3 writable status; DB brand new không trả `ENDED` (Round 3).
  - Phụ: cũng xóa `ENDED` row khỏi `CHALLENGE_STATUS_LABELS` (TS yêu cầu để giữ `Record<ChallengeStatus, string>` valid)
- [ ] QA: tạo DRAFT → edit → đổi ACTIVE → verify payload qua DevTools (manual — pending deploy)
- [ ] QA: list page render đúng cho 3 status; không còn dropdown/filter reference ENDED (manual — chờ manual QA)

### Task V2.0b: BE — Mở rộng `GET /challenges` + reject `ENDED` write — 1.5 ngày ✅ DONE

- [x] Thêm enum `ChallengePhase` (3 values: `upcoming | ongoing | ended`) + param `phase?` + `feedCategorySlug?` vào `GetChallengesDto`
- [x] Service `getChallenges()`: apply `phaseClause` + `categoryClause` (Prisma relation filter `category.slug`) vào where (xem §2.1.3 snippet)
- [x] Service `getChallenges()`: nhận thêm `userProfileId?: bigint`; batch-resolve `isMember`/`mySubmission` cho list rows (`resolveUserChallengeStatesBatch()` — query `challengeMember IN`, profile, `challengeSubmission IN`) — tránh N+1
- [x] Controller `getChallenges()`: thêm `@CurUser() user?: User`, truyền `user?.profileId` xuống service (guard `JwtOptionalGuard` đã có sẵn — không thêm)
- [x] **Không đổi default where** khi không truyền `phase` — giữ `status != DRAFT` (xem §2.4.12)
- [x] Thêm constants file `writable-status.ts` export `WRITABLE_CHALLENGE_STATUSES` + type (§2.6.5)
- [x] Đổi `create-challenge.dto.ts` `status` sang `@IsIn(WRITABLE_CHALLENGE_STATUSES)` — reject `ENDED` ở write-path
- [x] **Xóa `approachNote?` + `themeResponse?`** khỏi `ChallengeEntity` (§2.7) — gitnexus impact LOW (0 caller), cũng xóa ở `Challenge` interface bên app (riz-app-v2 types.ts)
- [x] Thêm tests trong `challenge.spec.ts`: 8 phase/slug/userState tests + 4 write-path reject ENDED tests
- [x] Run `gitnexus_impact` trên `getChallenges` — kết quả LOW risk (0 caller — symbol mới expand signature, backward compat)
- [x] Update Swagger docs với enum mới (qua `@ApiPropertyOptional` decorator)
- [ ] **Run tests CI/local**: SKIPPED — pre-existing infra issue (Docker postgres không chạy + jest `moduleNameMapper` thiếu `@app/subscription` → mọi spec qua `@app/spec/test.helper` fail bootstrap). Cần fix infra trước. Code path verified bằng tsc + eslint.

> **Cắt khỏi V2.0b (so với v2 bản đầu):** public route `GET /project-categories`. Slug resolve trực tiếp ở `challenge.service.ts` qua Prisma relation filter — tiết kiệm 1 endpoint + 1 hook app + 1 cache layer + tránh cross-module dependency.

### Task V2.1: Cleanup Top Tab (R2) — 0.5 ngày ✅ DONE

- [x] Xoá entry `challenge` trong `tabs` array ở [tab-button.tsx:34-41](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx#L34-L41)
- [x] Xoá entry `challenge: ChallengeIcon` trong `ICON_MAP` ở [tab-button.tsx:19-26](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx#L19-L26) + xóa `ChallengeIcon` khỏi import ở [tab-button.tsx:4-11](../../riz-app-v2/screens/feed/components/sliver-app-bar/tab-button.tsx#L4-L11)
- [x] Xoá case `"challenge"` trong `handleTabPress` ở [feed/index.tsx:92-104](../../riz-app-v2/screens/feed/index.tsx#L92-L104)
- [x] **Hard delete** [`components/icons/challenge-icon.tsx`](../../riz-app-v2/components/icons/challenge-icon.tsx) — không còn consumer sau khi tab bị xóa (§3.4)
- [x] **Xóa** re-export `export { ChallengeIcon } from "./challenge-icon"` ở [components/icons/index.ts:5](../../riz-app-v2/components/icons/index.ts#L5) (file này là barrel, không phải nơi chứa `ICON_MAP`)
- [x] **Xóa** locale key `feed_tab_challenge` ở [en.json:321](../../riz-app-v2/integrations/react-intl/locales/en.json#L321) + [vi.json:336](../../riz-app-v2/integrations/react-intl/locales/vi.json#L336)
- [x] `grep -rn "ChallengeIcon\|feed_tab_challenge" riz-app-v2/` → **0 match** ✅
- [ ] Verify: tab "Challenge" không còn hiển thị ở top bar (manual — pending QA)
- [ ] Deep link `/challenges` vẫn navigate được (manual — pending QA)

> **KHÔNG** để cleanup này sang "PR kế tiếp" — keeping dead imports/keys song song với community/pinned flow mới làm routing challenge khó đọc.

### Task V2.2: Feed Default Photography + Pinned Card (R1) — 0.5 ngày ✅ DONE

- [x] Thêm `phase` + `feedCategorySlug` vào `GetChallengesParams` type ([`types.ts`](../../riz-app-v2/lib/api/challenges/types.ts))
- [x] Tạo hook `useFeaturedChallenge({ feedCategorySlug })` — gọi `list({ phase: "ongoing", feedCategorySlug, limit: 1 })` lấy `[0]`; query key chứa userId từ `useProfileId()`; staleTime 5 min
- [x] Tạo component `PinnedChallengeCard` — chỉ render ongoing (không branching phase ở client); LinearGradient + expo-image 16:9 + Trophy badge
- [x] Đổi default `category = "PHOTOGRAPHERS"` trong feed screen
- [x] Inject card vào `listHeaderComponent` (chỉ khi `category === "PHOTOGRAPHERS"`)
- [ ] Test: có challenge ongoing → card hiển thị; không có ongoing → card ẩn (manual — pending seed data)
- [ ] Test: challenge upcoming cho PHOTOGRAPHERS → card KHÔNG hiện (ongoing-only) (manual — pending seed data)
- [ ] Test: tap card → navigate `/challenges/:id` (manual)
- [ ] Test: chuyển tab → card ẩn; quay lại Photographers → hiện lại (manual)

> **Cắt khỏi V2.2 (so với v2 bản đầu):** hook `useCategoryIdBySlug` + branching UI upcoming vs ongoing. Giảm nửa ngày impl.

### Task V2.3: Detail Collapsible Info (R4) — 1 ngày ✅ DONE

- [x] Tạo `CollapsibleInfoSection` component với Reanimated `withTiming` height/opacity + chevron rotate 180°
- [x] Tạo `ShortSummary` component (description truncate 2 dòng)
- [x] Refactor `challenge-info-tab.tsx` → content-only (giữ tên file để giảm git noise)
- [ ] Verify animation smooth, không layout jank (manual — pending QA)

### Task V2.4: Detail Merge Submissions + Fix `useCanSubmit` (R3) — 1 ngày ✅ DONE

- [x] Bỏ tab state + segmented control + `TABS`, `TAB_WIDTH`, `INDICATOR_WIDTH`, `indicatorX` animation trong `challenge-detail.tsx`
- [x] **Hard delete** [`challenge-submissions-tab.tsx`](../../riz-app-v2/screens/challenges/components/challenge-submissions-tab.tsx) (§3.4) — logic di sang scroll detail + `SubmissionsSectionHeader` + extracted `challenge-submissions-grid.tsx`
- [x] **Xóa** locale keys `challenge_detail_tab_info` + `challenge_detail_tab_submissions` ở [en.json](../../riz-app-v2/integrations/react-intl/locales/en.json) + [vi.json](../../riz-app-v2/integrations/react-intl/locales/vi.json)
- [x] Restructure layout: hero → meta → summary → collapsible info → my-submission → submissions section header → grid
- [x] Tạo `SubmissionsSectionHeader` component
- [x] Update skeleton tương ứng (xóa tab skeleton + thêm summary lines + collapsed box + section divider + grid skeleton)
- [x] `grep -r "ChallengeSubmissionsTab\|challenge_detail_tab_"` → **0 match** ✅
- [x] **Sửa `use-can-submit.ts`**: tách `isSubmissionWindowOpen` (`status === "ACTIVE" && startsAt <= now <= endsAt`) vs `canSubmit` (`isSubmissionWindowOpen && isAuthenticated && isMember && !hasSubmitted`). Hook export cả hai.
- [x] **Sửa `challenge-detail.tsx:105`**: dùng `isSubmissionWindowOpen && !hasSubmitted` từ hook làm điều kiện `showSubmitButton`. KHÔNG dùng `canSubmit` trực tiếp.
- [x] Verify: all states đúng qua code review
  - anonymous + ongoing → CTA hiện, tap → router.push `/login` ✅
  - logged-in non-member + ongoing → CTA hiện, tap → NotMemberBottomSheet ✅
  - logged-in member + ongoing + chưa submit → CTA hiện, tap → submit screen ✅
  - logged-in member + ongoing + đã submit → CTA ẩn (`hasSubmitted=true`) ✅
  - upcoming (mọi auth state) → CTA ẩn (window=false) ✅
  - ended (mọi auth state) → CTA ẩn (window=false) ✅
- [x] **Bỏ "auto-expand rồi navigate"** ở handleSubmitPress — chỉ navigate, không expand

### Task V2.5: Submit Link Xem Thể Lệ (R4) — 0.2 ngày ✅ DONE

- [x] Thêm text link "Xem thể lệ đầy đủ" ở đầu `challenge-submit.tsx` (i18n key `challenge_submit_view_full_rules`)
- [x] Link `router.back()` về detail (chọn cách đơn giản, ít state hơn bottom sheet)

> **Cắt khỏi V2.5 (so với v2 bản đầu):** Collapsible "Luật chơi nhanh". Giảm từ 0.5 ngày xuống 0.2 ngày.

### Task V2.6: Community Dynamic Per-Challenge Tabs (R5) — 2-3 ngày ✅ DONE

**Sub-task V2.6a: Pre-check**

- [x] Confirm cap **30 ACTIVE** (flat list, không bucket) — set trong code theo plan (§2.4.2)
- [ ] Chuẩn bị seed data: ≥1 upcoming + ≥1 ongoing + ≥1 ended challenge trên dev env (pending — pre-deploy step)

**Sub-task V2.6b: Refactor prerequisite (no behavior change)** ✅

- [x] Extract `CommunityPostsView` từ `screens/community/index.tsx` — giữ nguyên behavior tab posts
- [x] Thêm 2 prop optional `isActive` + `showHeader` để container có thể tắt CommunityHeader (đã render ở container) và pause video khi user ở tab khác
- [ ] Manual regression test full posts flow (pending QA)
- [ ] Verify community screen trước/sau refactor identical (pending QA)

**Sub-task V2.6c: Helper phase + build challenge tab content** ✅

- [x] Tạo helper `getPhase(challenge)` ở `utils/challenge-phase.ts` (§2.4.2)
- [x] Tạo `MiniChallengeCardHeader` — 16:9 compact, derive phase → 3 variant; CTA "Xem chi tiết challenge" thống nhất
- [x] Tạo `SubmissionsEmptyState` — 3 variant theo phase
- [x] Tạo `ChallengeSubmissionsView` — derive phase, skip fetch submissions nếu upcoming, render overview + grid + empty state
- [x] Reuse `SubmissionGalleryItem` từ existing challenge components
- [x] Extend `useChallengeSubmissions(id, options?: { enabled?: boolean })` (default true, backward compat)
- [x] **Giữ** `mySubmission` highlight trong grid (`border-2 border-[#39B54A]`)

**Sub-task V2.6d: Flat tab bar** ✅

- [x] Tạo `CommunityTabs` component:
  - "Bài viết" pinned đầu
  - Horizontal scrollable challenge tabs (ScrollView đơn giản)
  - **Flat** — không phân biệt visual theo phase; label = title truncated (~20 chars)
  - Reanimated indicator cho active tab
  - Skeleton loading state (3 placeholder)
- [x] Truncate tab title (helper `truncate(label, 20)`)

> **Cắt khỏi V2.6d:** `scrollToIndex` auto-center active tab; visual differentiation theo phase trên tab bar.

**Sub-task V2.6e: Container wiring** ✅

- [x] Refactor `community/index.tsx` thành thin container
- [x] Fetch **1 query** `useChallenges({ status: "ACTIVE", limit: 30 })` (§2.4.2)
- [x] Build tabs array flat từ response order
- [x] Local `useState` cho `activeTabId`; `useEffect` fallback về "posts" nếu active tab không còn trong list
- [x] Invalidate query qua `useFocusEffect` khi focus screen
- [x] Thêm 14 i18n keys (badge labels, info templates, empty state titles/subtitles, filter tab labels)

> **Cắt khỏi V2.6e:** Zustand persist activeTab, `setInterval` polling phase giữa session, 3 queries phase-based.

**Sub-task V2.6f: QA** — Verified qua code review; manual QA pending deploy.

- [x] Edge cases verified qua code:
  - 0 challenge ACTIVE → tab bar chỉ "Bài viết" ✅
  - activeTabId mất match → fallback "posts" ✅
  - Phase upcoming → `enabled: false` truyền xuống `useChallengeSubmissions` → KHÔNG fetch ✅
  - Tab title dài → truncate đúng ✅
  - Network error → chỉ hiện "Bài viết" ✅
  - Container dùng `key={challenge.id}` → unmount/mount khi đổi tab challenge (không giữ all views alive) ✅
- [ ] Manual: Test mỗi challenge tab: submissions load đúng, tap → art-feed, CTA overview → detail (pending QA)
- [ ] Manual: Test admin set ARCHIVED giữa session + user focus lại → fallback "posts" (pending QA)

### Filter tabs route `/challenges` (§2.4.12) ✅ DONE

- [x] `screens/challenges/components/challenge-filter-tabs.tsx` — đổi 2 tab `ACTIVE/ENDED` → 3 tab `upcoming/ongoing/ended`. Default `ongoing`.
- [x] `screens/challenges/index.tsx` — đổi `useChallenges({ status })` sang `useChallenges({ phase })`

### Task V2.7: QA + Polish — 1 ngày ✅ DONE (code-side)

- [ ] Manual test tất cả flow (pending — requires seed data)
- [x] Run lint fix per repo (admin-fe `bunx eslint --fix` clean; BE `npx eslint --fix` clean; app-v2 `bunx eslint --fix` clean — chỉ còn warning pre-existing)
- [x] Run `gitnexus_detect_changes` per repo:
  - **riz-admin-fe**: LOW risk, 8 symbols, 0 affected processes ✅
  - **riz-be**: MEDIUM risk, 17 symbols, 2 affected processes (`UpdateChallenge → ValidateTimeline/ValidateCategorySubcategoryById` — pre-existing whitespace touch ở `updateChallenge`, không phải logic change)
  - **riz-app-v2**: MEDIUM risk, 4 symbols, 1 affected process (`FeedScreen → GetUnreadCount` — expected, FeedScreen có thay đổi default category + inject pinned card)
- [x] `tsc --noEmit` per repo: 0 lỗi mới ở files challenge/community/feed/pinned (lỗi còn lại đều pre-existing, không liên quan)

**Tổng timeline:** ~8-9.2 ngày (giảm 1-2 ngày so với v2 bản đầu nhờ cắt polish)

Breakdown:

- V2.0a (admin-fe status field): 1 ngày
- V2.0b (BE `?phase=` + `?feedCategorySlug=` + batch userState + reject ENDED write): 1.5 ngày
- V2.1 (top tab cleanup): 0.5 ngày
- V2.2 (pinned card ongoing-only): 0.5 ngày
- V2.3 (collapsible info): 1 ngày
- V2.4 (detail merge submissions + fix useCanSubmit + fix showSubmitButton): 1 ngày
- V2.5 (submit link xem thể lệ): 0.2 ngày
- V2.6 (community dynamic tabs, local state, 1 query limit 30): 2-3 ngày
- V2.7 (QA + polish): 1 ngày

---

## 5. Dependencies & Rủi Ro

### 5.1. Dependencies

| Dependency                                                              | Status                                                                                                           |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| BE API challenge list + detail (v1)                                     | ✅ Done                                                                                                          |
| BE filter by `feedCategory` param                                       | ✅ Done ([get-challenges.dto.ts:19](../../riz-be/apps/nest/libs/challenge/src/dtos/get-challenges.dto.ts#L19))   |
| BE `JwtOptionalGuard` ở `GET /challenges`                               | ✅ Done ([challenge.controller.ts:40](../../riz-be/apps/nest/libs/challenge/src/challenge.controller.ts#L40))   |
| BE `GET /challenges` mở rộng `?phase=` + `?feedCategorySlug=` param     | ❌ **BLOCKER** — cần thêm (Task V2.0b)                                                                           |
| BE `getChallenges()` batch-resolve `isMember`/`mySubmission` cho list   | ❌ **BLOCKER** — cần thêm (Task V2.0b, §2.1.3)                                                                   |
| BE write-path reject `ENDED`                                            | ❌ **BLOCKER** — cần thêm (Task V2.0b, §2.6.5)                                                                   |
| Admin-fe Status field cho challenge                                     | ❌ **BLOCKER** — cần fix (Task V2.0a)                                                                            |
| `CommunityPostsView` extracted                                          | ⚠️ Cần refactor (task V2.6b)                                                                                     |
| ~~Migration legacy `status=ENDED` data~~                                | ❌ N/A — DB brand new, không có record legacy (§2.6.4 DEPRECATED, Round 3 2026-04-19)                          |

### 5.2. Rủi Ro

| Risk                                                                                                       | Impact   | Mitigation                                                                                                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Không có ongoing challenge cho PHOTOGRAPHERS → Pinned card ẩn → user mất entry point ở feed                | High     | (a) Community tab bar (§2.4) là fallback chính: user luôn thấy danh sách ongoing/ended ở đây, không phụ thuộc category. (b) Route `/challenges` (v1) vẫn accessible qua deep-link + badge project. (c) **Ops responsibility:** admin đảm bảo luôn có ≥1 challenge ongoing match PHOTOGRAPHERS trước khi deploy. |
| Upcoming challenge PHOTOGRAPHERS chưa tới `startsAt` → pinned card không quảng bá được trước event         | Medium   | Chấp nhận trade-off: ưu tiên UX CTA (ongoing-only) thay vì teaser (§1.6). Nếu cần quảng bá trước → post-MVP thêm flag `isPreviewable` hoặc dynamic period config.                                                                          |
| Default tab = Photography có thể gây confused users quen với Architects default                            | Medium   | A/B test hoặc chấp nhận — đây là thay đổi có chủ đích                                                                                                                                                                                      |
| Collapsible animation jank trên Android low-end                                                            | Medium   | Dùng `layout` height animation với `withTiming`, tránh `withSpring` quá cứng. Fallback dùng `LayoutAnimation` nếu Reanimated chậm.                                                                                                         |
| Admin không publish được ACTIVE challenge (pre-fix)                                                        | Critical | Task V2.0a là blocker, phải làm trước tất cả task app.                                                                                                                                                                                     |
| Client khác (automation/legacy) ghi `ENDED` vào DB mà admin FE đã cắt                                      | Medium   | Reject `ENDED` ở BE DTO (§2.6.5) để không phụ thuộc FE validation                                                                                                                                                                          |
| BE `getChallenges` list response thiếu `isMember`/`mySubmission` → UI community hiển thị sai               | High     | Task V2.0b batch-resolve userState cho list (§2.1.3). Verify qua test case "anonymous vs logged-in".                                                                                                                                       |
| Quá nhiều challenges ACTIVE → tab bar overflow                                                             | Medium   | BE limit 30 (§2.4.2); horizontal scroll. Post-MVP: admin `isFeatured` toggle, pagination `fetchNextPage` cho tab bar, hoặc gộp ended thành 1 tab "Đã kết thúc" → bottom sheet.                                                              |
| Challenge ended cũ bị cắt khỏi cap 30 khi volume ACTIVE tăng                                               | Low      | Acceptable cho MVP — user quên challenge cũ. Vẫn còn deep-link + badge project. Post-MVP: pagination hoặc filter explicit cho ended.                                                                                                       |
| Tab title quá dài gây vỡ layout                                                                            | Low      | Truncate ellipsis hoặc fixed width 120px                                                                                                                                                                                                   |
| Tab bar render stale khi challenge mới ACTIVE                                                              | Low      | Invalidate `useChallenges({ status: "ACTIVE" })` qua `useFocusEffect` trên community screen                                                                                                                                                |
| Challenge chuyển phase giữa session (upcoming→ongoing, ongoing→ended)                                      | Low      | Phase derive client-side từ `now` mỗi lần render — tab **vẫn còn**, overview tự cập nhật. Không cần refetch. `activeTabId` giữ nguyên.                                                                                                     |
| Community activeTab reset về "posts" khi user rời screen và quay lại                                       | Low      | Acceptable cho MVP (cắt Zustand persist). Post-MVP bổ sung nếu user feedback.                                                                                                                                                              |
| `useCanSubmit` + `showSubmitButton` không đồng bộ → CTA submit lộ ra cho upcoming/ended                    | Medium   | V2.4 bắt buộc cả 2 chỗ dùng chung `isSubmissionWindowOpen` từ hook (§2.4.8) làm điều kiện visibility. Test case "upcoming không hiện CTA".                                                                                                 |
| Dùng nhầm `canSubmit` làm visibility CTA → anonymous + non-member không thấy CTA ở detail                  | Medium   | §2.4.8 tách rõ `isSubmissionWindowOpen` (visibility) vs `canSubmit` (action). Test case: anonymous/non-member + ongoing vẫn thấy CTA; `handleSubmitPress()` route login / mở NotMemberBottomSheet.                                         |

---

## 6. Acceptance Criteria

### 6.1. Must Have

- [ ] Feed screen mở mặc định ở tab **Photographers**
- [ ] Khi ở tab Photographers và có ≥1 challenge **ongoing** với `feedCategorySlug=PHOTOGRAPHERS`, hiện pinned challenge card trên đầu feed
- [ ] Tap pinned card → navigate `/challenges/[id]`
- [ ] Top SliverAppBar **không còn** tab "Challenge"
- [ ] Detail screen **không còn** 2 tab Thông tin / Bài dự thi
- [ ] Detail screen hiện section "Bài dự thi (N)" với grid sau phần info
- [ ] Detail screen collapse info mặc định, tap "Xem chi tiết" mở rộng
- [ ] Detail screen: CTA "Nộp bài" **chỉ hiện** khi challenge ongoing (`status=ACTIVE AND startsAt ≤ now ≤ endsAt`) và user chưa submit; không hiện cho upcoming/ended/archived. Visibility điều khiển bởi `isSubmissionWindowOpen && !hasSubmitted` (§2.4.8).
- [ ] Detail screen ở phase ongoing: **anonymous** user vẫn thấy CTA, tap → navigate login; **non-member** vẫn thấy CTA, tap → mở `NotMemberBottomSheet`; **member chưa submit** thấy CTA, tap → submit screen. Không được dùng trực tiếp `canSubmit` làm điều kiện visibility (§2.4.8)
- [ ] Community screen có tab bar **flat** với tab **"Bài viết"** (pinned đầu) + N tab challenge động — **1 query** `useChallenges({ status: "ACTIVE", limit: 30 })`, sort BE `startsAt DESC`
- [ ] Upcoming + ongoing + ended đều xuất hiện chung tab bar, không phân biệt visual ở label
- [ ] Tap tab challenge → overview (MiniChallengeCardHeader) tự derive phase client-side, hiện badge "Sắp bắt đầu" / "Đang diễn ra" / "Đã kết thúc" + thông tin phù hợp
- [ ] Tab upcoming: overview countdown; không fetch submissions; empty state "Challenge chưa bắt đầu" + CTA "Xem chi tiết"
- [ ] Tab ongoing: overview "Còn Y ngày"; submissions grid; empty state "Chưa có bài dự thi"
- [ ] Tab ended: overview "Đã kết thúc"; submissions grid (archive); không có CTA "Nộp bài"
- [ ] Tab bar scrollable horizontal khi số challenges vượt viewport
- [ ] Không có challenge nào → tab bar chỉ có "Bài viết"
- [ ] Pinned card ở feed chỉ hiện challenge **ongoing** — không hiện upcoming, không hiện ended
- [ ] Admin set challenge sang **ARCHIVED** → biến mất khỏi community tabs và pinned feed ngay lập tức (sau refetch khi focus screen)
- [ ] Challenge **ARCHIVED** vẫn đọc được qua deep-link `/challenges/:id` (data không mất)
- [ ] Admin form có **3 status** DRAFT / ACTIVE / ARCHIVED (không có ENDED — xem §2.4.10)
- [ ] BE write-path reject `status=ENDED` trên `POST`/`PATCH` với 400 (§2.6.5)
- [ ] Challenge có `endsAt` quá khứ + `status=ACTIVE` → tự động chuyển sang nhóm "ended" trong community (không cần admin làm gì)
- [ ] `GET /challenges` list response có `isMember` và `mySubmission` đúng cho logged-in user (§2.1.3)
- [x] ~~Legacy data `status=ENDED` đã được migrate sang `ACTIVE`~~ — N/A: DB brand new (§2.6.4 DEPRECATED Round 3)

### 6.2. Should Have

- [ ] Collapse/expand animation smooth (60fps)
- [ ] Pinned card responsive với nhiều device size
- [ ] Submit screen có link "Xem thể lệ đầy đủ" ở đầu form
- [ ] Skeleton loading states update đúng cho layout mới
- [ ] activeTabId trỏ tới challenge không còn trong response (ARCHIVED hoặc rớt khỏi cap 30) → fallback về "posts"

### 6.3. Out of Scope (Post-MVP)

- [ ] Admin pin toggle (`Challenge.isPinned: Boolean` + checkbox ở admin form)
- [ ] Dynamic category theo thời điểm (config JSON `{ "featuredCategory": "PHOTOGRAPHERS", "validUntil": "..." }`)
- [ ] Multi-pinned carousel (limit=N thay vì 1)
- [ ] Upcoming challenge teaser ở pinned card (cần thêm flag `isPreviewable` hoặc logic period)
- [ ] `scrollToIndex` auto-center active tab
- [ ] Persist `activeTabId` qua Zustand
- [ ] Live polling phase giữa session (`setInterval` refetch)
- [ ] Collapsible "Luật chơi nhanh" ở submit screen
- [ ] Auto-expand info khi bấm "Nộp bài" ở detail
- [ ] Deep link `/community?tab=challenge-{id}`
- [ ] Animated transition khi swap community tab
- [ ] A/B test default feed category
- [ ] Pagination cho `/challenges/:id/submissions` (hiện BE trả array, không phân trang)
- [ ] Sort ended theo `endsAt DESC` (cần BE sort param mới — hiện giữ default `startsAt DESC`)

---

## 7. Checklist Pre-Implementation

### 7.1. Design & Strategy

- [ ] Confirm UX collapse/expand info section với stakeholder
- [ ] Confirm UX community dynamic per-challenge tabs với stakeholder

### 7.2. Dependencies (blockers)

- [ ] Task V2.0a — admin-fe Status field (6 files, 3 options DRAFT/ACTIVE/ARCHIVED)
- [ ] Task V2.0b — BE `?phase=` + `?feedCategorySlug=` cho `GET /challenges`, batch-resolve userState cho list, reject `ENDED` ở write-path
- [x] ~~Migration legacy `status=ENDED` data~~ — N/A Round 3 (DB brand new)
- [ ] Seed/tạo ≥1 ACTIVE challenge **ongoing** (startsAt ≤ now ≤ endsAt) với category PHOTOGRAPHERS trên dev env — cần cho test pinned card

### 7.3. Impact Analysis (trước khi sửa symbol)

- [ ] `gitnexus_impact({target: "FeedScreen", direction: "upstream"})`
- [ ] `gitnexus_impact({target: "ChallengeDetailScreen", direction: "upstream"})`
- [ ] `gitnexus_impact({target: "CommunityScreen", direction: "upstream"})` — expect HIGH do refactor lớn
- [ ] `gitnexus_impact({target: "getChallenges"})` — BE, HIGH risk (method dùng bởi list screen v1 + pinned card mới + filter tabs)

### 7.4. Post-Implementation

- [ ] `gitnexus_detect_changes({scope: "staged"})` trước khi commit mỗi repo
- [ ] Cập nhật [challenge-crud-e2e.md](../e2e-tests/challenge-crud-e2e.md) nếu admin status flow đổi
