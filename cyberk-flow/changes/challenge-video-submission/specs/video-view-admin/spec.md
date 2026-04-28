# Spec: video-view-admin

## Requirements

### R1: Type update
- `ChallengeSubmission` có thêm field `media: SubmissionMedia[]`
- `SubmissionMedia`: `{id, type: 'IMAGE'|'VIDEO', url, hlsUrl?, dashUrl?, thumbnailUrl?, duration?}`
- `imageUrl` giữ lại làm deprecated fallback

### R2: API transformer
- `listSubmissions` map `project.attachments` → `media[]`
- Nếu BE chưa trả về `project.attachments`, `media` = []

**Scenario R2-1**: BE trả về media
- Given: submission có `project.attachments = [{type:'VIDEO', url: 'vimeo-url', thumbnailUrl: '...'}]`
- Then: `media[0]` = `{type:'VIDEO', url: 'vimeo-url', thumbnailUrl: '...'}`

### R3: Media gallery trong submission detail dialog
- Thay `<img>` đơn lẻ bằng `<SubmissionMediaGallery>`
- Gallery hiển thị tất cả images + videos theo thứ tự
- VIDEO: render Vimeo iframe `<iframe src={media.url} allow="autoplay; fullscreen" allowFullScreen>`
- IMAGE: render `<img>` như hiện tại

**Scenario R3-1**: Submission có cả ảnh và video
- Given: `media = [{type:'IMAGE', url:'img-url'}, {type:'VIDEO', url:'vimeo-url'}]`
- Then: gallery hiển thị 1 ảnh + 1 video iframe theo grid

**Scenario R3-2**: Submission chỉ có video, không ảnh
- Given: `media = [{type:'VIDEO', url:'vimeo-url', thumbnailUrl:'thumb-url'}]`
- Then: gallery hiển thị 1 video iframe; không crash do `imageUrl` null/empty

### R4: List thumbnail fallback
- Nếu submission chỉ có video, `challenge-detail-page.tsx` show `media[0].thumbnailUrl`
- Overlay icon ▶ trên thumbnail nếu là video

**Scenario R4-1**: Submission list — video only
- Given: submission có `imageUrl = null`, `media[0].type = 'VIDEO'`
- Then: list row show `thumbnailUrl` với ▶ overlay, không crash
