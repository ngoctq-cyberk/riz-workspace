# PLAN — Tích hợp tính năng nộp bài thi với video cho Challenge

> **Status:** Draft
> **Date:** 2026-04-28
> **Scope:** `riz-app-v2` (mobile) + `riz-admin-fe` (web admin)
> **Backend status:** ✅ API đã sẵn sàng — không cần thay đổi BE

---

## 1. Tổng quan

Hiện tại tính năng nộp bài thi (challenge submission) chỉ hỗ trợ **ảnh**. Backend đã được mở rộng để nhận thêm field `videos[]` (multipart) trên cả 2 endpoint create/update submission, với giới hạn **5 videos/submission**. Video sẽ được upload qua S3 → Vimeo (pull mode), trả về `playerUrl`, `hlsUrl`, `dashUrl`, `thumbnailUrl` thông qua `Media` table (type=VIDEO).

Plan này mô tả các thay đổi cần làm ở **mobile app** (cho phép user pick + upload video) và **admin FE** (cho phép admin xem video khi review submission).

---

## 2. Backend — Reference (KHÔNG cần thay đổi)

### Endpoints sẵn sàng

| Method | Endpoint | Field | Max |
|--------|----------|-------|-----|
| POST | `/challenges/:id/submissions` | `images[]`, `videos[]` | 20 ảnh / 5 video |
| PATCH | `/challenges/:id/submissions/:submissionId` | `images[]`, `videos[]` | 20 ảnh / 5 video |

- File: [riz-be/apps/nest/libs/challenge/src/challenge.controller.ts:64-156](riz-be/apps/nest/libs/challenge/src/challenge.controller.ts#L64-L156)
- DTO: [create-challenge-submission.dto.ts](riz-be/apps/nest/libs/challenge/src/dtos/create-challenge-submission.dto.ts)
- Service: `ChallengeService.submitChallenge()` đã gọi `uploadVideosToVimeo()` và `createProjectVideosFromVimeo()` để lưu vào `Media` table.

### Định dạng & dung lượng

- Định dạng hỗ trợ: `mp4`, `webm`, `mov`, `avi`, `mkv` (theo `storage.service`).
- Max size mỗi file: **250 MB** (`MESSAGE_ATTACHMENT_MAX_SIZE`).
- Field name FormData: **`videos`** (lặp lại cho mỗi file).

### Response shape (submission entity)

`ChallengeSubmissionEntity` reference tới `project.attachments` (Media[]). Mỗi media VIDEO có:
```ts
{
  id, type: 'VIDEO',
  url,                // Vimeo player URL (iframe-ready)
  hlsUrl, dashUrl,    // streaming URLs
  thumbnailUrl,
  duration            // seconds
}
```

---

## 3. Mobile App (`riz-app-v2`)

### 3.1 Files cần thay đổi

| File | Thay đổi |
|------|----------|
| [screens/challenges/challenge-submit.tsx](riz-app-v2/screens/challenges/challenge-submit.tsx) | Thêm video picker + preview + append vào FormData |
| [hooks/challenges/use-challenge-mutations.ts](riz-app-v2/hooks/challenges/use-challenge-mutations.ts) | Cập nhật type signature nếu cần (FormData đã generic) |
| [lib/api/challenges/index.ts](riz-app-v2/lib/api/challenges/index.ts) | Cập nhật type `ChallengeSubmission` để có `videos[]` |
| `screens/challenges/components/video-picker-bottom-sheet.tsx` | **NEW** — bottom sheet pick video từ library |
| `screens/challenges/components/video-thumbnail-card.tsx` | **NEW** — card preview video kèm play button + remove |

### 3.2 Libraries có sẵn (không cần install thêm)

- `expo-image-picker` — đổi `mediaTypes: ['videos']` để pick video
- `expo-video` — `VideoView` + `useVideoPlayer` cho fullscreen playback
- `expo-video-thumbnails` — `getThumbnailAsync()` để generate preview frame

### 3.3 UI flow

```
[Submission screen]
  ├─ Section "Images" (đã có)
  └─ Section "Videos" (NEW)
       ├─ Horizontal scroll: [+ Add] [video1 thumb] [video2 thumb] ...
       ├─ Tap (+) → VideoPickerBottomSheet
       │     ├─ "Record video" (optional, có thể skip phase 1)
       │     └─ "Choose from library" → expo-image-picker
       ├─ Tap thumb → fullscreen play modal
       └─ Tap (×) → remove khỏi list
```

### 3.4 Validation client-side

- Max **5 videos/submission**.
- Max **250 MB/file** (warn user, kiểm tra `result.assets[i].fileSize`).
- Max duration **120 giây/video** (lý do: chi phí storage + UX feed; có thể discuss).
- Định dạng cho phép: `mp4`, `mov` (mobile chủ yếu xuất 2 loại này).
- Cập nhật `isValid`: chấp nhận submission có **chỉ ảnh, chỉ video, hoặc cả hai** (hiện tại đang force `images.length > 0`).

### 3.5 FormData append pattern

```ts
videos.forEach((vid, i) => {
  formData.append("videos", {
    uri: vid.uri,
    type: vid.mimeType ?? "video/mp4",
    name: `video_${i}.${vid.uri.split('.').pop()}`,
  } as unknown as Blob);
});
```

### 3.6 Upload UX (quan trọng)

Video upload chậm hơn ảnh (10–60s tuỳ size). Cần:
- **Progress bar** ở submit button (Axios `onUploadProgress` đã có sẵn trong axios client → expose qua mutation).
- **Disable back gesture** khi đang upload để tránh user mất bài.
- Increase **request timeout** lên `5 * 60 * 1000` (5 phút) cho endpoint này.
- Hiển thị thông báo "Đang tải video lên, vui lòng không thoát ứng dụng".

### 3.7 Hiển thị video trong submission view

| File | Thay đổi |
|------|----------|
| [screens/community/challenge-submissions-view.tsx](riz-app-v2/screens/community/challenge-submissions-view.tsx) | Render `VideoView` cho media type=VIDEO trong submission gallery |
| [screens/challenges/components/submission-gallery-item.tsx](riz-app-v2/screens/challenges/components/submission-gallery-item.tsx) | Thêm play overlay icon nếu là video |
| [screens/challenges/components/my-submission-card.tsx](riz-app-v2/screens/challenges/components/my-submission-card.tsx) | Show thumbnail từ Vimeo với play badge |

### 3.8 i18n keys cần thêm

- `challenge_submit_videos_label`
- `challenge_submit_videos_hint` ("Tối đa 5 video, dưới 120s, dưới 250MB")
- `challenge_submit_video_picker_record`
- `challenge_submit_video_picker_choose`
- `challenge_submit_video_too_large`
- `challenge_submit_video_too_long`
- `challenge_submit_uploading_video` ("Đang tải video, vui lòng chờ...")

---

## 4. Admin FE (`riz-admin-fe`)

### 4.1 Files cần thay đổi

| File | Thay đổi |
|------|----------|
| [src/entities/challenge/model/types.ts:38-56](riz-admin-fe/src/entities/challenge/model/types.ts#L38-L56) | Thêm `media: SubmissionMedia[]` vào `ChallengeSubmission` |
| [src/entities/challenge/api/challenge-api.ts](riz-admin-fe/src/entities/challenge/api/challenge-api.ts) | Cập nhật transformer để map `project.attachments` → `media[]` (cả VIDEO + IMAGE) |
| [src/features/challenges/submission-review/ui/submission-detail-dialog.tsx](riz-admin-fe/src/features/challenges/submission-review/ui/submission-detail-dialog.tsx) | Replace `<img>` đơn lẻ bằng `<MediaGallery>` (hỗ trợ video) |
| `src/features/challenges/submission-review/ui/submission-media-gallery.tsx` | **NEW** — Carousel/grid hiển thị images + videos |
| `src/features/challenges/submission-review/ui/video-player.tsx` | **NEW** — Wrapper HTML5 `<video>` với poster=thumbnail, controls |

### 4.2 Type updates

```ts
// types.ts
export interface SubmissionMedia {
  id: string;
  type: "IMAGE" | "VIDEO";
  url: string;          // image URL | Vimeo iframe URL
  hlsUrl?: string;
  dashUrl?: string;
  thumbnailUrl?: string;
  duration?: number;
}

export interface ChallengeSubmission {
  // ... existing fields
  media: SubmissionMedia[];  // NEW
  imageUrl?: string;          // DEPRECATED — giữ tạm cho list/thumb, dùng media[0] nếu không có
}
```

### 4.3 Video player choice

Phase 1: Dùng **Vimeo embed iframe** (đơn giản nhất, không cần licensing thêm).
```tsx
<iframe src={video.url} allow="autoplay; fullscreen" allowFullScreen />
```
Phase 2 (nếu cần custom controls): Dùng `react-player` hoặc `video.js` với `hlsUrl`.

### 4.4 Submission list view

Hiện tại admin list đang show `imageUrl` thumbnail. Nếu submission **chỉ có video**, cần:
- Show `media[0].thumbnailUrl` (Vimeo trả về thumbnail).
- Overlay icon ▶ để báo hiệu là video.

File: [riz-admin-fe/src/screens/challenges/challenge-detail-page.tsx](riz-admin-fe/src/screens/challenges/challenge-detail-page.tsx) (submissions table).

### 4.5 Filter (nice-to-have)

Thêm filter "Has video" trong submission list để admin xem các bài có video. Tuỳ chọn cho phase 2.

---

## 5. Phase rollout

### Phase 1 — Core (1 sprint)
- [ ] Mobile: Pick + upload video (chỉ từ library, không record).
- [ ] Mobile: Validate size/duration/count client-side.
- [ ] Mobile: Hiển thị video trong submission cards.
- [ ] Admin: Update type, hiển thị video trong submission detail dialog (Vimeo iframe).
- [ ] Admin: Update list thumbnail fallback.

### Phase 2 — Polish (½ sprint)
- [ ] Mobile: Record video từ camera (`expo-camera` hoặc `expo-image-picker` `launchCameraAsync`).
- [ ] Mobile: Upload progress bar + abort controller.
- [ ] Mobile: Generate local thumbnail trước khi upload (better UX).
- [ ] Admin: Custom video player với HLS (nếu Vimeo embed không đủ).
- [ ] Admin: Filter "Has video".

### Phase 3 — Optional
- [ ] Mobile: Trim video trước khi upload (giảm size).
- [ ] BE: Webhook từ Vimeo báo khi video xử lý xong (hiện tại assume sync).

---

## 6. Edge cases & risks

| Risk | Mitigation |
|------|-----------|
| Upload timeout với video lớn | Tăng axios timeout lên 5 phút riêng cho endpoint này |
| User thoát app khi upload | Chặn back gesture, show warning, dùng `expo-task-manager` cho background upload (phase 2) |
| Vimeo processing delay | Show "Video đang xử lý" placeholder, poll lại sau 30s nếu `playerUrl` chưa sẵn sàng |
| Mạng yếu | Hiển thị progress, cho phép retry, không clear FormData khi fail |
| Format không hỗ trợ | Filter trong picker chỉ cho phép mp4/mov |
| Submission chỉ có video, không ảnh | Bỏ rule `images.length > 0`, validate `images.length + videos.length > 0` |
| Backward compat (admin) | `imageUrl` giữ làm fallback nếu `media[]` chưa map từ BE |

---

## 7. Self-check trước khi gộp

- [ ] Không thay đổi BE (chỉ FE consumer).
- [ ] Mobile build chạy được trên iOS + Android, test pick mp4 + mov.
- [ ] Admin: review submission hiển thị đúng video, approve/reject vẫn hoạt động.
- [ ] Lint + typecheck pass cho cả 2 repo.
- [ ] Test e2e: tạo submission có video → admin thấy → approve → mobile thấy badge approved.
- [ ] Run `gitnexus_detect_changes()` trên cả 2 repo trước khi commit.

---

## 8. Estimate

| Repo | Complexity | Effort |
|------|------------|--------|
| `riz-app-v2` | Medium (UI + upload UX) | **3–4 ngày** |
| `riz-admin-fe` | Low (chỉ thêm video player + type) | **1–2 ngày** |
| QA cross-repo | | **1 ngày** |
| **Total Phase 1** | | **~5–7 ngày** |
