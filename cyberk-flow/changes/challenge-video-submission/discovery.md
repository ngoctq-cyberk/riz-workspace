# Discovery: challenge-video-submission

## Findings

### Backend (no changes needed)
- POST/PATCH `/challenges/:id/submissions` đã nhận `videos[]` multipart, max 5 files, 250 MB/file
- Video được xử lý qua S3 → Vimeo, trả về `playerUrl`, `hlsUrl`, `dashUrl`, `thumbnailUrl` trong `Media` table (type=VIDEO)
- Response có `project.attachments: Media[]` — mỗi video có `{id, type, url, hlsUrl, dashUrl, thumbnailUrl, duration}`

### Mobile (riz-app-v2)
- `challenge-submit.tsx`: chỉ có section ảnh, `isValid` force `images.length > 0`
- Types: `PublicChallengeSubmission` và `MySubmission` không có trường video/media
- Libraries sẵn có: `expo-image-picker` (pick video), `expo-video` (playback)
- API call dùng FormData generic — chỉ cần append thêm field `videos`

### Admin (riz-admin-fe)
- `ChallengeSubmission` type chỉ có `imageUrl: string` — không có `media[]`
- `listSubmissions` gọi thẳng API, không qua transformer — BE có thể đã trả về `project.attachments` nhưng chưa được map
- `submission-detail-dialog.tsx`: render `<img src={submission.imageUrl}>` đơn lẻ
- `challenge-detail-page.tsx`: submissions table hiện `imageUrl` thumbnail

## Gap Analysis

| Gap | Impact |
|-----|--------|
| Mobile: không pick được video | User không upload được video |
| Mobile: `isValid` force ảnh | Submission chỉ video bị block |
| Mobile: types thiếu `media[]` | Không render được video trong gallery |
| Admin: types thiếu `media[]` | Video BE trả về bị bỏ qua |
| Admin: detail dialog chỉ show 1 ảnh | Admin không xem được video khi review |

## Options

**Option A**: Phase 1 only — pick video từ library, Vimeo iframe embed cho admin, không record.
**Option B**: Full — thêm record từ camera, custom HLS player, progress bar.

**Khuyến nghị**: Option A (Phase 1) — đủ value, ít risk, align với PLAN.
