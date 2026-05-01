## ADDED Requirements

### Requirement: Feed item shows Play overlay for video projects

The masonry feed SHALL display a Play overlay (Play icon in a rounded badge) on items where the underlying project contains at least one video attachment. The flag SHALL be derived from `images[].type === "VIDEO"` and stored on `MasonryItemData.hasVideo: boolean`.

#### Scenario: Project has at least one video

- **WHEN** a project's response contains an image whose `type === "VIDEO"`
- **THEN** the corresponding `FeedItem` renders a Play overlay on top of its thumbnail

#### Scenario: Project has only images

- **WHEN** a project's response contains no items with `type === "VIDEO"`
- **THEN** the corresponding `FeedItem` renders without a Play overlay (existing behavior)

### Requirement: Feed cover thumbnail falls back to video thumbnail

`transformProjectToMasonryItem` SHALL select the cover thumbnail by preferring the first `IMAGE` attachment (`mediumUrl ?? url`). If no image attachment exists, it SHALL fall back to the first video's `thumbnailUrl` (Vimeo-provided thumbnail). If neither is available, the item SHALL be skipped.

#### Scenario: Mixed media — image first

- **WHEN** `images = [VIDEO, IMAGE]`
- **THEN** the cover URL comes from the IMAGE entry, not the video player URL

#### Scenario: Video-only project

- **WHEN** `images = [VIDEO]`
- **THEN** the cover URL is `firstVideo.thumbnailUrl`

#### Scenario: No usable media

- **WHEN** the project has no images and no `primaryImageUrl`
- **THEN** the project is filtered out of the feed list

### Requirement: Feed tap navigates to art-feed for the tapped project

Tapping any feed item (image-only or video) SHALL navigate to `/art-feed?profileId={author.id}&projectId={item.id}` using the existing handler in `masonry-grid-feed.tsx`.

#### Scenario: Tap a video project from feed

- **WHEN** the user taps a `FeedItem` with `hasVideo === true`
- **THEN** the router navigates to `/art-feed` with the correct `profileId` and `projectId` query params

### Requirement: Art-feed plays video on tap (tap-to-play)

`ProjectFeedItem` SHALL map `mediaItems[i].type` from `img.type === "VIDEO" ? "video" : "image"` (no hardcoded type). For video items, the URL SHALL be `img.mediumUrl ?? img.url` (HLS preferred). The component SHALL hold local state `activeVideoIndex` initialized to `null`, set it to the tapped video index via `MediaCarousel.onItemPress`, and reset it to `null` whenever `project.id` changes or `MediaCarousel.onActiveIndexChange` reports a different active slide.

#### Scenario: Tap video thumbnail

- **GIVEN** the art-feed is showing a project with at least one video and `activeVideoIndex === null`
- **WHEN** the user taps the video item in the carousel
- **THEN** `activeVideoIndex` becomes the tapped index and `CommunityFeedInlineVideo` renders with native controls (play, pause, seek, fullscreen)

#### Scenario: Image-only project

- **WHEN** the project has no video items
- **THEN** the carousel behaves exactly as before (image-only carousel, no tap-to-play activation)

#### Scenario: Reset on project change

- **GIVEN** a video is active in `ProjectFeedItem` for project A
- **WHEN** the user scrolls and `ProjectFeedItem` re-renders with project B
- **THEN** `activeVideoIndex` is reset to `null`

#### Scenario: Reset on carousel slide change

- **GIVEN** a video is active in `ProjectFeedItem`
- **WHEN** the user swipes to a different carousel slide
- **THEN** `activeVideoIndex` is reset to `null` and the previous video is no longer active

### Requirement: Submission detail video playback regression-free

The existing `SubmissionMediaModal` (challenge feature) SHALL continue to play video using its own active video index state without behavioral change for the user.

#### Scenario: Submission detail still plays video

- **WHEN** the user opens a submission with a video and taps the thumbnail
- **THEN** the video plays as it did before this change
