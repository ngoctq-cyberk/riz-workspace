# Proposal: challenge-video-submission

## Why
Backend đã hỗ trợ video upload. Mobile + admin cần bắt kịp để user có thể nộp bài bằng video và admin review được.

## Appetite
Phase 1 — 1 sprint (~5-7 ngày).

## Scope (Phase 1)

### In scope
- Mobile: pick video từ library, validate (max 5, 250 MB, 120s), append vào FormData
- Mobile: preview thumbnail trong submit screen (horizontal scroll)
- Mobile: show play overlay trong submission gallery + my-submission card
- Mobile: update `isValid` chấp nhận chỉ video hoặc cả hai
- Admin: thêm `SubmissionMedia[]` type, transformer map `project.attachments`
- Admin: media gallery trong submission detail dialog (Vimeo iframe)
- Admin: fallback thumbnail từ `media[0].thumbnailUrl` trong list

### Out of scope (Phase 2)
- Record video từ camera
- Upload progress bar + abort controller
- Custom HLS player
- Filter "Has video" trong admin list

## Capabilities
1. **video-upload-mobile**: User pick và submit video cùng form nộp bài
2. **video-view-admin**: Admin xem video trong submission detail dialog

## Impact
- UI Impact: YES (mobile submit screen + admin dialog)
- E2E: cần test thủ công (pick video → submit → admin xem)

## Risk
- MEDIUM: Upload timeout với file lớn → tăng axios timeout (N/A cho phase 1 vì không thêm progress)
- LOW: Backward compat admin → giữ `imageUrl` fallback

## UI Impact Decision
- Mobile: thêm section "Videos" bên dưới "Photos" trong submit screen
- Admin: replace `<img>` bằng `<MediaGallery>` trong detail dialog — approve/reject vẫn hoạt động

## E2E Decision
Test thủ công: tạo submission có video → admin thấy video → approve/reject hoạt động.
