# Spec: video-upload-mobile

## Requirements

### R1: Video picker trong submit screen
- User thấy section "Videos" bên dưới section "Photos"
- Tap (+) → pick video từ thư viện (expo-image-picker, mediaTypes: videos)
- Max 5 videos; nút (+) ẩn khi đã đủ 5

**Scenario R1-1**: Pick video thành công
- Given: submit screen mở, videos < 5
- When: tap (+) → chọn 1 video mp4 từ thư viện
- Then: thumbnail hiển thị trong horizontal scroll, videos.length tăng 1

**Scenario R1-2**: Giới hạn 5 video
- Given: đã có 5 video
- Then: nút (+) không hiển thị

### R2: Validate client-side
- Max 5 videos
- Max 250 MB/file → alert nếu vượt
- Max 120s/video → alert nếu vượt
- Định dạng: mp4, mov (filter trong picker)

**Scenario R2-1**: File quá lớn
- Given: user chọn video > 250 MB
- Then: alert "Video quá lớn", video không được thêm vào list

**Scenario R2-2**: Video quá dài
- Given: user chọn video > 120s
- Then: alert "Video quá dài", video không được thêm vào list

### R3: FormData append video
- Videos được append vào FormData với field name `videos`
- Submission với chỉ video (không ảnh) hợp lệ

**Scenario R3-1**: Submit chỉ video
- Given: 0 ảnh, 1 video, title + description đã điền
- Then: isValid = true, submit button enable

### R4: Hiển thị video trong submission gallery
- `SubmissionGalleryItem`: show play overlay icon nếu submission có video
- `MySubmissionCard`: show thumbnail từ Vimeo (media[0].thumbnailUrl) với play badge

**Scenario R4-1**: Gallery item có video
- Given: submission có `media[0].type === 'VIDEO'`
- Then: SubmissionGalleryItem show thumbnail + play overlay icon (▶)
