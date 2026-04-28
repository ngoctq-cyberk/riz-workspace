# Design: challenge-video-submission

## Architecture Decisions

### AD1: Không tạo bottom sheet riêng cho video
Thay vì tạo `video-picker-bottom-sheet.tsx` mới, dùng trực tiếp `expo-image-picker` trong submit screen để đơn giản hơn. Bottom sheet chỉ cần thiết khi cần "Record" vs "Library" — Phase 2.

### AD2: Không generate local thumbnail bằng expo-video-thumbnails
Phase 1: dùng `expo-image-picker` trả về `thumbnailUri` sẵn trong response khi pick video (thuộc field `assets[i].uri` + có thể lấy thumbnail từ poster frame). Nếu không có, show placeholder icon video. Phase 2 mới dùng `expo-video-thumbnails`.

### AD3: Admin dùng Vimeo iframe embed
Đơn giản nhất, không cần thêm library. Phase 2 mới upgrade lên react-player với HLS.

### AD4: API transformer tại entity layer (admin)
Thêm `transformSubmission()` function trong `challenge-api.ts` tương tự `transformChallenge()` đã có. Map `project.attachments` → `media[]`.

## Risk Map

| Risk | Level | Mitigation |
|------|-------|------------|
| `expo-image-picker` không trả về `duration` | LOW | Đọc `assets[i].duration` (milliseconds) — field có sẵn trong ImagePickerAsset |
| BE chưa populate `project.attachments` trong list endpoint | MEDIUM | Kiểm tra response thực tế; nếu không có, chỉ update detail endpoint |
| Vimeo iframe không load trên iOS WKWebView | LOW | allowsInlineMediaPlayback + mediaPlaybackRequiresUserAction |
| `imageUrl` null trong admin list gây crash | LOW | Optional chaining + fallback |

## File Map

### Mobile
- `riz-app-v2/lib/api/challenges/types.ts` — thêm `media[]` vào `PublicChallengeSubmission` + `MySubmission`
- `riz-app-v2/screens/challenges/challenge-submit.tsx` — thêm video section
- `riz-app-v2/screens/challenges/components/submission-gallery-item.tsx` — play overlay
- `riz-app-v2/screens/challenges/components/my-submission-card.tsx` — video thumbnail
- `riz-app-v2/integrations/react-intl/locales/en.json` — thêm keys
- `riz-app-v2/integrations/react-intl/locales/vi.json` — thêm keys

### Admin
- `riz-admin-fe/src/entities/challenge/model/types.ts` — thêm `SubmissionMedia` + `media[]`
- `riz-admin-fe/src/entities/challenge/api/challenge-api.ts` — transformer
- `riz-admin-fe/src/features/challenges/submission-review/ui/submission-detail-dialog.tsx` — media gallery
- `riz-admin-fe/src/features/challenges/submission-review/ui/submission-media-gallery.tsx` — NEW
- `riz-admin-fe/src/screens/challenges/challenge-detail-page.tsx` — thumbnail fallback
