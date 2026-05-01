# Discovery: feed-video-playback

## Findings

### Backend (riz-be) — partial gap
- `Media` model có `type: MediaType (IMAGE | DOCUMENT | VIDEO)` (`prisma/schema.prisma:589`).
- Submission video upload qua Vimeo: `Media.url = Vimeo playerUrl`, `Media.mediumUrl = HLS URL`, `Media.thumbnailUrl = Vimeo thumb` (`challenge.service.ts:336-348`).
- **Gap**: `ProjectMapperHelper.mapProjectData` map `images[]` từ `project.global.attachments` nhưng **drop field `type`** (`project-mapper.helper.ts:111-120`). Output schema của `images` chỉ có `{id, url, mediumUrl, thumbnailUrl, order, width, height, aspectRatio}` → FE không phân biệt được image vs video.
- **Gap**: challenge Vimeo videos từng persist bằng `PROJECT_ATTACHMENT` trong khi project readers filter `PROJECT_IMAGE`, làm video không xuất hiện trong `images[]`/`hasVideo`.
- **Gap**: `project.service.ts` list/detail paths chưa include `_count.attachments` video và mapper private chưa emit `images[].type`/`hasVideo`.
- **Gap**: `admin-trending.service.ts` có duplicated include cho `attachments` + `_count.attachments` video ở nhiều query.

### Mobile — riz-app-v2 (FE consumer)
- `ProjectImage` (`lib/api/projects/types.ts:93`) thiếu `type` (đồng bộ với backend gap).
- `MediaCarousel` + `MediaCarouselItem` đã hỗ trợ video render: khi `item.type === "video"` và `isActive` → render `CommunityFeedInlineVideo` (expo-video, native controls). Khi `!isActive` → thumbnail + Play overlay.
- `CommunityFeedInlineVideo` đã có sẵn — dùng `useVideoPlayer` + `VideoView` của expo-video.
- **Pattern tap-to-play hiện có**: `SubmissionMediaModal` (`screens/challenges/components/submission-media-modal.tsx`) — controlled active video index, set index khi user tap thumbnail, clear khi carousel đổi slide/đóng modal.
- **Art-feed gap**: `ProjectFeedItem` từng hardcode `type: "image"` cho mọi media và không truyền state active video thay đổi được → video không play được.
- **Feed gap**: `MasonryItemData` không có flag `hasVideo`. `FeedItem` chỉ render `Image`, không có Play overlay khi item là video.
- **Navigation đã có sẵn**: `masonry-grid-feed.tsx:214-225` đã navigate `/art-feed?profileId&projectId` khi tap → không cần thay đổi.
- Thumbnail cover trong feed: `transformProjectToMasonryItem` lấy `firstImage.mediumUrl || firstImage.url`. Nếu `images[0]` là video → URL là Vimeo player URL → `<Image>` render lỗi.

## Gap Analysis

| Gap | Impact |
|-----|--------|
| Backend mapper drop `type` field | FE không phân biệt được media type → không thể play video |
| Challenge video writer dùng `PROJECT_ATTACHMENT` nhưng project readers filter `PROJECT_IMAGE` | Challenge video bị drop khỏi project response |
| Project service list/detail thiếu `_count` + mapper thiếu `hasVideo` | `/project` và `/project/:id` không expose đủ video contract |
| Trending include duplicated | Dễ divergence khi đổi video filter |
| FE `ProjectImage` không có `type` | Type-level đồng bộ với backend |
| Art-feed hardcode `type: "image"` + không có active video state | Video không play ở art-feed |
| Feed không có Play overlay | UX thiếu signal cho video item |
| Feed cover ưu tiên `images[0]` không lọc IMAGE | Project full video → cover broken |

## Options

**Option A** (chosen): Additive backend change — thêm `type` vào `ProjectImage` response, FE consume + render. Tap-to-play ở art-feed. Feed chỉ Play overlay, navigate sang art-feed.

**Option B**: Auto-play khi viewport visible (Reels-style) ở art-feed. Tốn battery/bandwidth, user explicit reject.

**Option C**: Tạo route mới `/video-viewer` riêng cho video. Thêm complexity, không cần thiết khi art-feed đã có infra.

**Khuyến nghị**: Option A — minimal change, tái sử dụng `MediaCarousel` + `CommunityFeedInlineVideo` đã có.

## Risks

- BE `mapProjectData` HIGH risk theo gitnexus (6 callers, 3 processes) → mitigation: thay đổi additive only, không sửa signature.
- Feed cover thumbnail nếu project full video → fallback `firstVideo.thumbnailUrl`.
- Carousel có nhiều video: tap 1 video active rồi swipe sang slide khác phải clear active video index để tránh playback offscreen/auto-play slide khác.
