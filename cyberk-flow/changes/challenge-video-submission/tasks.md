<!-- Tasks are executed sequentially in dependency order (topological sort). -->
<!-- Tasks with no Deps run first; tasks whose Deps are all complete run next. -->

## 1. Types (parallel)

- [ ] T1 Thêm SubmissionMedia type vào mobile
  - **Refs**: specs/video-upload-mobile/spec.md#R4; design.md#File-Map
  - **Done**: Types compile, PublicChallengeSubmission có `media?: SubmissionMedia[]`
  - **Test**: N/A — type-only change
  - **Files**: `riz-app-v2/lib/api/challenges/types.ts`
  - **Approach**: Thêm interface `SubmissionMedia { id, type, url, thumbnailUrl?, duration? }`. Thêm `media?: SubmissionMedia[]` vào `PublicChallengeSubmission` và `MySubmission` (optional để backward compat).

- [ ] T2 Thêm SubmissionMedia type vào admin
  - **Refs**: specs/video-view-admin/spec.md#R1; design.md#File-Map
  - **Done**: Types compile, ChallengeSubmission có `media: SubmissionMedia[]`
  - **Test**: N/A — type-only change
  - **Files**: `riz-admin-fe/src/entities/challenge/model/types.ts`
  - **Approach**: Thêm export `SubmissionMedia` interface với `{id, type, url, hlsUrl?, dashUrl?, thumbnailUrl?, duration?}`. Thêm `media: SubmissionMedia[]` vào `ChallengeSubmission`. Giữ `imageUrl` làm deprecated fallback.

## 2. Admin features (sau T2)

- [ ] T3 Cập nhật API transformer cho submissions
  - **Deps**: T2
  - **Refs**: specs/video-view-admin/spec.md#R2; design.md#AD4
  - **Done**: `media[]` được populate từ `project.attachments`
  - **Test**: N/A — runtime mapping, test thủ công
  - **Files**: `riz-admin-fe/src/entities/challenge/api/challenge-api.ts`
  - **Approach**: Tạo `transformSubmission(api)` function tương tự `transformChallenge`. Map `api.project?.attachments ?? []` → `media[]`. Cập nhật `listSubmissions` và `reviewSubmission` để dùng transformer.

- [ ] T4 Tạo SubmissionMediaGallery component
  - **Deps**: T2
  - **Refs**: specs/video-view-admin/spec.md#R3; design.md#AD3
  - **Done**: Component render đúng IMAGE (<img>) và VIDEO (Vimeo iframe)
  - **Test**: N/A — visual, test thủ công
  - **Files**: `riz-admin-fe/src/features/challenges/submission-review/ui/submission-media-gallery.tsx` (NEW)
  - **Approach**: Component nhận `media: SubmissionMedia[]`. Flex-wrap grid. IMAGE: `<img className="h-48 w-full object-cover rounded-lg">`. VIDEO: `<iframe src={m.url} allow="autoplay; fullscreen" allowFullScreen className="h-48 w-full rounded-lg">`. Nếu `media` empty → fallback sang `imageUrl` prop (string|undefined).

- [ ] T5 Cập nhật SubmissionDetailDialog dùng MediaGallery
  - **Deps**: T4
  - **Refs**: specs/video-view-admin/spec.md#R3
  - **Done**: Dialog hiển thị gallery, approve/reject button vẫn hoạt động
  - **Test**: N/A — visual, test thủ công
  - **Files**: `riz-admin-fe/src/features/challenges/submission-review/ui/submission-detail-dialog.tsx`
  - **Approach**: Import `SubmissionMediaGallery`. Thay `<img src={submission.imageUrl}>` bằng `<SubmissionMediaGallery media={submission.media ?? []} imageUrl={submission.imageUrl} />`.

- [ ] T6 Cập nhật list thumbnail fallback trong challenge-detail-page
  - **Deps**: T2
  - **Refs**: specs/video-view-admin/spec.md#R4
  - **Done**: Row không crash khi `imageUrl` null, video row hiển thị ▶ icon
  - **Test**: N/A — visual, test thủ công
  - **Files**: `riz-admin-fe/src/screens/challenges/challenge-detail-page.tsx`
  - **Approach**: Tìm cell render thumbnail trong submissions table. Thay `submission.imageUrl` bằng `submission.media?.[0]?.thumbnailUrl ?? submission.imageUrl`. Nếu `media[0].type === 'VIDEO'`, add relative div với ▶ overlay icon.

## 3. Mobile features (sau T1)

- [ ] T7 Thêm video section vào challenge-submit
  - **Deps**: T1
  - **Refs**: specs/video-upload-mobile/spec.md#R1,R2,R3; design.md#AD1,AD2
  - **Done**: Pick video → thumbnail preview → submit gửi videos field đến API
  - **Test**: N/A — manual UI test
  - **Files**: `riz-app-v2/screens/challenges/challenge-submit.tsx`
  - **Approach**: Thêm state `videos: ImagePicker.ImagePickerAsset[]`. Hàm `handlePickVideo` dùng `ImagePicker.launchImageLibraryAsync({mediaTypes: ['videos'], allowsEditing: false})`. Validate `fileSize <= 250*1024*1024` và `duration/1000 <= 120`. Section UI tương tự images: horizontal ScrollView với nút (+) + thumbnail card + nút (×). Cập nhật `isValid`: `images.length + videos.length > 0`. Trong `handleSubmit`, append videos vào FormData.

- [ ] T8 Cập nhật submission-gallery-item + my-submission-card
  - **Deps**: T1
  - **Refs**: specs/video-upload-mobile/spec.md#R4
  - **Done**: Gallery item và card hiển thị play overlay khi có video
  - **Test**: N/A — manual UI test
  - **Files**: `riz-app-v2/screens/challenges/components/submission-gallery-item.tsx`, `riz-app-v2/screens/challenges/components/my-submission-card.tsx`
  - **Approach**: `SubmissionGalleryItem`: lấy thumbnail từ `submission.media?.[0]?.thumbnailUrl ?? submission.imageUrl`. Nếu `media[0].type === 'VIDEO'`, overlay View với Play icon (lucide `Play`). `MySubmissionCard`: tương tự, dùng `mySubmission.media?.[0]?.thumbnailUrl ?? mySubmission.imageUrl`.

- [ ] T9 Thêm i18n keys (en + vi)
  - **Deps**: T7
  - **Refs**: PLAN §3.8
  - **Done**: Không có missing key warning khi chạy app
  - **Test**: N/A — config
  - **Files**: `riz-app-v2/integrations/react-intl/locales/en.json`, `vi.json`
  - **Approach**: Thêm keys: `challenge_submit_videos_label`, `challenge_submit_videos_hint`, `challenge_submit_video_too_large`, `challenge_submit_video_too_long`, `challenge_submit_uploading_video`.
