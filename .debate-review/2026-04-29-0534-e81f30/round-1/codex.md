# Round 1 — Codex

### F-001: Challenge videos are excluded from `hasVideo` and feed media
- **Severity**: high
- **Category**: contract
- **File**: riz-be/apps/nest/libs/project/src/admin-trending.service.ts:285
- **Status**: open
- **Claim**: `hasVideo` is counted only from `PROJECT_IMAGE` attachments, but challenge videos are persisted as `PROJECT_ATTACHMENT`. For video challenge submissions, this makes `hasVideo` false and also excludes the video from the `attachments` list used to build `images`.
- **Evidence**: `admin-trending.service.ts` filters both cover attachments and `_count` with `usage: MediaUsage.PROJECT_IMAGE`; challenge videos are created as `type: MediaType.VIDEO, usage: MediaUsage.PROJECT_ATTACHMENT` in `riz-be/apps/nest/libs/challenge/src/challenge.service.ts:490`.
- **Suggested fix**: Align the query/storage contract by including challenge video attachments in the project media response and count, or store playable challenge videos under the same usage the project feed queries.

### F-002: Project list/detail endpoints still omit video fields
- **Severity**: high
- **Category**: contract
- **File**: riz-be/apps/nest/libs/project/src/project.service.ts:1058
- **Status**: open
- **Claim**: `/project` and `/project/:id`, which power the art-feed screen, still map `images` without `type` and never set `hasVideo`. `ProjectFeedItem` depends on `img.type === "VIDEO"` to render/play videos, so videos opened from the feed/detail path will be treated as images or not exposed at all.
- **Evidence**: `images: project.global.attachments.map((media) => ({ id, url, mediumUrl, thumbnailUrl, order, width, height, aspectRatio }))` has no `type`, and the same mapper never assigns `hasVideo`.
- **Suggested fix**: Reuse the updated project mapper or add `type` plus the same video count/include logic to all `ProjectEntity` response paths, not only trending/admin feed paths.

### F-003: Active video state is not cleared when carousel page changes
- **Severity**: medium
- **Category**: correctness
- **File**: riz-app-v2/screens/art-feed/components/project-feed-item.tsx:61
- **Status**: open
- **Claim**: After a video is activated, `isVideoActive` remains true until `project.id` changes. Swiping the carousel to another item does not reset this state, so the previously active video can keep playing offscreen or stay active until the row is recycled for another project.
- **Evidence**: The only reset is `useEffect(() => { setIsVideoActive(false); }, [project.id])`, while `MediaCarousel` derives active playback from `isVideoActive && index === activeIndex.value`.
- **Suggested fix**: Track the active video index and clear/update it from carousel page changes, or expose an `onActiveIndexChange` path that pauses the prior video when the user swipes.
