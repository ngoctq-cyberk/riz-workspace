# Proposal: feed-video-playback

**Author**: quyngoc

## Why
Video bài dự thi đã play được ở submission detail (challenge feature) nhưng ở feed/art-feed thì không. User không thể xem video khi browse feed → mất trải nghiệm và signal engagement (like/comment) trên video bị giới hạn.

## Appetite
S (≤1d) — additive backend, FE tái sử dụng `MediaCarousel` + `CommunityFeedInlineVideo` đã có.

## Scope
- **In**:
  - Backend: `ProjectMapperHelper.mapProjectData` expose field `type` từ `Media.type` cho `images[]`. Update `ProjectImageEntity`.
  - Backend: project list/detail/trending responses include video `_count` để expose `hasVideo`; challenge Vimeo videos persist bằng `PROJECT_IMAGE` để cùng contract với reader.
  - FE types: `ProjectImage.type`, `MasonryItemData.hasVideo`.
  - Feed (masonry): Play overlay khi item có video. Cover thumbnail fallback `firstVideo.thumbnailUrl`.
  - Art-feed: tap-to-play video qua `MediaCarousel` (controlled `activeVideoIndex`).
- **Out**:
  - Auto-play khi viewport visible (user explicit reject).
  - Tách `hlsUrl` thành field riêng (Phase sau).
  - Admin frontend (`riz-admin-fe`) — chưa check, để Phase sau nếu cần.
- **Cut list**: tách `hlsUrl` field; admin support — drop nếu over budget.

## What Changes
- Backend: thêm `type` field trong `ProjectImage` response (additive, **non-breaking**).
- Backend: thêm `hasVideo` trong `ProjectEntity` response từ `_count.attachments` video.
- Mobile: feed Play overlay; art-feed tap-to-play video.
- Submission detail/community carousel: regression-free sau khi đồng bộ `MediaCarousel` prop contract.

## Capabilities
- **New**: `specs/project-media-type-api/spec.md`
- **New**: `specs/art-feed-video-playback/spec.md`

## UI Impact & E2E
- **User-visible UI behavior affected?** YES
- **E2E required?** REQUIRED
- **Justification**: Feed Play overlay là visual change; art-feed tap-to-play là interaction change. E2E thủ công vì RN test infra cho video chưa có.
- **Target user journeys**:
  1. Feed → tap project có video → mở art-feed đúng project.
  2. Art-feed → tap thumbnail video → video play với native controls.
  3. Art-feed → swipe khỏi video đang play → video cũ không active và video khác không auto-play nếu chưa tap.
  4. Project chỉ ảnh → không Play overlay, không tap-to-play (regression).
  5. Submission detail → video vẫn play (regression).

## Risk Level
MEDIUM
- `mapProjectData` HIGH risk theo gitnexus (6 callers, 3 affected processes: `getTrendingFeed`, `listProjects`, `getListItems`) → mitigation: chỉ thêm field, additive only.
- Project full video → cover broken → fallback `firstVideo.thumbnailUrl`.

## Impact
- **Affected specs**: 2 new capabilities.
- **Affected code (key files)**:
  - `riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts`
  - `riz-be/apps/nest/libs/project/src/entities/project.entity.ts`
  - `riz-be/apps/nest/libs/project/src/project.service.ts`
  - `riz-be/apps/nest/libs/project/src/admin-trending.service.ts`
  - `riz-be/apps/nest/libs/challenge/src/challenge.service.ts`
  - `riz-app-v2/lib/api/projects/types.ts`
  - `riz-app-v2/components/media-carousel/media-carousel.tsx`
  - `riz-app-v2/components/media-carousel/types.ts`
  - `riz-app-v2/screens/feed/types/types.ts`
  - `riz-app-v2/screens/feed/helper/transform.ts`
  - `riz-app-v2/screens/feed/components/main/feed-item.tsx`
  - `riz-app-v2/screens/art-feed/components/project-feed-item.tsx`
  - `riz-app-v2/screens/challenges/components/submission-media-modal.tsx`
  - `riz-app-v2/screens/community/components/community-feed-item.tsx`
  - `riz-app-v2/screens/community/components/pending-post-card.tsx`

## Open Questions
- [ ] Admin frontend (`riz-admin-fe`) có cần Play overlay/video player tương tự không? — defer Phase sau.
