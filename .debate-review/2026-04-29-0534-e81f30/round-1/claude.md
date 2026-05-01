# Round 1 — Claude

### F-001: `project.service.ts` does NOT expose `hasVideo` or `images[].type`, breaking video playback contract for `/project` endpoints

- **Severity**: blocker
- **Category**: contract
- **File**: riz-be/apps/nest/libs/project/src/project.service.ts:137-170
- **Status**: open
- **Claim**: The `hasVideo` flag and `ProjectImage.type` field were added to `ProjectEntity` / `ProjectImageEntity` (project.entity.ts:98-100, 236-238) and the FE consumes them in `screens/feed/helper/transform.ts:13-17` and `screens/art-feed/components/project-feed-item.tsx:68-71`. However, `project.service.ts` (which serves `GET /project`, `GET /project/:id`, bookmarked templates, etc.) was not updated: its `include` block never adds `_count.attachments` (lines 137-170, 244-274, 358-381, 729-740) and its private `mapToProjectEntity` (lines 999-1096) never sets `hasVideo` and never copies `media.type` into `images[]`. Because `useProjects` (FE `hooks/use-projects.ts:22`) hits `/project`, the entire art-feed video-playback path receives `images[].type === undefined` and `hasVideo === undefined`. As a result, in `project-feed-item.tsx` every media item is mapped to `type: "image"` (line 68) and the carousel never enters video mode. The feature is functionally broken on the only screen that calls `MediaCarousel` with the new tap handler (art-feed).
- **Evidence**: `riz-be/apps/nest/libs/project/src/project.service.ts:1058-1067` builds images without `type`; lines 1028-1093 never include `hasVideo`. `riz-app-v2/hooks/use-projects.ts:22` shows art-feed uses `projectsApi.list` → `/project`, not `/trending/feed`. `riz-app-v2/screens/art-feed/index.tsx:69` confirms art-feed wires through `useProjects`.
- **Suggested fix**: Mirror the admin-trending include block (`_count: { select: { attachments: { where: { type: VIDEO, usage: PROJECT_IMAGE } } } }`) and add `type: media.type` + `hasVideo` mapping in project.service’s `mapToProjectEntity`; ideally route project.service through `ProjectMapperHelper.mapProjectData` so this cannot drift again.

### F-002: `getProject` (single project detail) also missing `_count` and `type` mapping

- **Severity**: high
- **Category**: contract
- **File**: riz-be/apps/nest/libs/project/src/project.service.ts:354-398
- **Status**: open
- **Claim**: `getProject` (used by FE `useProject` / `projectsApi.getById` at `lib/api/projects/index.ts:36`) excludes the `_count` selector and returns images without `type`. Any consumer (e.g. project detail screen, panorama modal fallback at `project-feed-item.tsx:163`) cannot tell which media items are videos. Since the BE entity now declares `type: MediaType | null` and `hasVideo?: boolean` as part of the response shape, this is a silent contract drift between list-trending and detail endpoints.
- **Evidence**: `project.service.ts:362-381` shows `global.attachments` selected without `_count`; `mapToProjectEntity` body at lines 1058-1067 omits `type`. FE relies on these fields in `project-feed-item.tsx:68-71`.
- **Suggested fix**: Add the same `_count.attachments where type=VIDEO,usage=PROJECT_IMAGE` and propagate `type` through the mapper for `getProject` and `getBookmarkedTemplates`.

### F-003: Repeated `_count` Prisma include literally duplicated 5× in admin-trending.service.ts — drift risk

- **Severity**: medium
- **Category**: maintainability
- **File**: riz-be/apps/nest/libs/project/src/admin-trending.service.ts:283-287,523-527,563-567,665-669,896-900
- **Status**: open
- **Claim**: The same nested Prisma include (`_count: { select: { attachments: { where: { type: MediaType.VIDEO, usage: MediaUsage.PROJECT_IMAGE } } } }`) is copied verbatim five times across the file. Any future change (e.g. switch to `MediaUsage.PROJECT_VIDEO`, add audio type, or scope by status) must touch all five sites; missing one will silently produce divergent `hasVideo` results between trending pages and items endpoints. The same comment applies to the duplicated `attachments where usage=PROJECT_IMAGE, orderBy order asc, take 1` block.
- **Evidence**: `admin-trending.service.ts` line numbers above. Prisma supports extracting `Prisma.ProjectGlobalInclude` literals into a constant or builder.
- **Suggested fix**: Extract a shared const `PROJECT_FEED_GLOBAL_INCLUDE` (or helper function) used by all four trending queries plus `listProjects`, so the predicate lives in one place.

### F-004: `MediaCarousel` `onItemPress` callback closure churn defeats memoization

- **Severity**: low
- **Category**: performance
- **File**: riz-app-v2/screens/art-feed/components/project-feed-item.tsx:66-85
- **Status**: open
- **Claim**: `mediaItems` is rebuilt on every render of `ProjectFeedItem` (line 66 — `const mediaItems = (project.images ?? []).map(...)`). The `handleItemPress` callback at line 78-85 lists `mediaItems` in its dependency array, so it produces a new function reference on every parent re-render. Because `MediaCarousel` is `React.memo` (media-carousel.tsx:20), prop-equality is shallow; passing a fresh `onItemPress` re-renders the whole carousel + every video/image item on every parent state update (e.g. likeCount tick, bookmark spinner, comment thread open). Memoizing the array (via `useMemo` keyed on `project.images`) and using a stable callback would preserve memo benefits.
- **Evidence**: `project-feed-item.tsx:66-76` allocates fresh array each render; `media-carousel.tsx:20-27,51-62` shows `MediaCarousel` is memoized but `renderItem` depends on `onItemPress`.
- **Suggested fix**: Wrap `mediaItems` in `useMemo([project.images])` and use a ref-based or index-only `handleItemPress` whose dependency is `mediaItems` (which will then be stable).

### F-005: `isVideoActive` is one-shot global per item; second video auto-plays on swipe without user tap

- **Severity**: low
- **Category**: correctness
- **File**: riz-app-v2/screens/art-feed/components/project-feed-item.tsx:78-85,189-195
- **Status**: open
- **Claim**: `handleItemPress` only sets `setIsVideoActive(true)` and never resets it. When a project contains multiple videos, tapping video #0 enables `isVideoActive`; subsequently swiping to video #1 makes the carousel evaluate `isActive = isVideoActive && index === activeIndex.value` (media-carousel.tsx:57) and auto-plays video #1 without an explicit tap. Combined with the sole reset on `project.id` change (line 61-63), the state model conflates "user pressed play on this carousel" with "every video in this carousel should auto-play on focus", which the bug summary in the design doc treats as a per-item interaction.
- **Evidence**: `project-feed-item.tsx:59-63,78-85`; `media-carousel.tsx:57` uses the same `isVideoActive` boolean for all indices.
- **Suggested fix**: Track active video by index (e.g. `activeVideoIndex: number | null`), and gate `isActive` per item; reset on swipe-away if the product wants pause-on-leave behaviour.

### F-006: Video URL fallback discards `mediumUrl` when present and falls through to raw `url`

- **Severity**: low
- **Category**: correctness
- **File**: riz-app-v2/screens/art-feed/components/project-feed-item.tsx:69-71
- **Status**: open
- **Claim**: The intent comment says "mediumUrl is the HLS URL (preferred), fall back to url". The mapping `url: img.type === "VIDEO" ? (img.mediumUrl ?? img.url) : img.url` is correct for the URL field, but the next line sets `mediumUrl: img.type === "VIDEO" ? undefined : img.mediumUrl`. This drops the HLS URL from `mediumUrl`, which means downstream `MediaCarouselItem` (media-carousel-item.tsx:48 `imageUrl = item.mediumUrl ?? item.url`) loses the ability to differentiate between MP4 and HLS sources. More importantly, if `img.mediumUrl` is the HLS URL and `img.url` is the original MP4, the FE never preserves the original MP4 anywhere in `MediaItem`; if HLS playback fails the player has no fallback. Previous (image) behavior preserved both URLs.
- **Evidence**: `project-feed-item.tsx:69-71`; `media-carousel-item.tsx:48,52,71` consumes `item.url` for both the inline video uri and the thumb path.
- **Suggested fix**: Preserve both URLs explicitly — e.g. set `url: img.url` and `mediumUrl: img.mediumUrl` for video as well, and add a dedicated field (e.g. `hlsUrl`) on `MediaItem` so the player can pick HLS first and fall back to MP4.

### F-007: `images.find((i) => i.type !== "VIDEO")` matches images with `type === undefined`, contradicting `i.type === "VIDEO"` filter symmetry

- **Severity**: low
- **Category**: correctness
- **File**: riz-app-v2/screens/feed/helper/transform.ts:13-17
- **Status**: open
- **Claim**: Because `ProjectImage.type` is `"IMAGE" | "VIDEO" | undefined`, the pair `images.some((i) => i.type === "VIDEO")` and `images.find((i) => i.type !== "VIDEO")` treat `undefined` as "non-video". On any endpoint that does not yet emit `type` (see F-001/F-002), `firstNonVideo` will be `images[0]` regardless of its actual type, while `hasVideo` will be `false` (no element matches `=== "VIDEO"`). Combined with the BE contract gap, the cover image picked from a video-only project served by `project.service.ts` will be the video itself rendered as an image — possibly with no thumbnail.
- **Evidence**: transform.ts:13,16; types.ts:99-103 (`type?:` is optional).
- **Suggested fix**: Treat `type === undefined` defensively (e.g. `i.type === "IMAGE"` for the non-video predicate) and document that BE must emit `type` once contract is unified.

### F-008: `_count.attachments` value is unguarded — narrow Prisma typing means undefined access at runtime if include block is missing

- **Severity**: medium
- **Category**: correctness
- **File**: riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts:14-19,125
- **Status**: open
- **Claim**: `ProjectWithRelations` extends Prisma's `GetPayload` with an optional `_count?: { attachments: number }` (line 18). `mapProjectData` then evaluates `(project.global._count?.attachments ?? 0) > 0`. This is safe in JS, but it silently returns `hasVideo: false` whenever the caller forgot to include the `_count` selector — which is exactly the bug surface in F-001/F-002 (project.service.ts callers passing data through this helper would also be silently wrong if they ever migrate). The mapper has no runtime warning, no type-level requirement; it accepts any input and produces a possibly-misleading boolean.
- **Evidence**: project-mapper.helper.ts:14-19,125. The `_count` field is typed optional, so callers never get a TS error for omitting it.
- **Suggested fix**: Make `_count.attachments` required on `ProjectWithRelations`, or have `hasVideo` derive only when `_count` is provided (and otherwise return `undefined` so it is omitted by class-transformer rather than emitting a false `false`).

### F-009: BE `ProjectImageEntity.type` is nullable but FE type is `"IMAGE" | "VIDEO"` (no null) — mismatched optionality

- **Severity**: low
- **Category**: contract
- **File**: riz-be/apps/nest/libs/project/src/entities/project.entity.ts:98-100
- **Status**: open
- **Claim**: BE declares `type: MediaType | null` (`ApiPropertyOptional({ enum: MediaType })`). FE declares `type?: "IMAGE" | "VIDEO"` (riz-app-v2/lib/api/projects/types.ts:102). The two encode the same "may be missing" idea differently: BE will emit `null`; FE typing accepts only `undefined`. JS runtime is forgiving, but the consequence is `img.type === "VIDEO"` works while `img.type !== "VIDEO"` is `true` for `null` — same behaviour as F-007. The contract-level concern is that no prisma-side row should actually have `type === null` if the schema is correctly `MediaType` non-null on `Media.type`; the entity declaration says otherwise.
- **Evidence**: project.entity.ts:98-100 (`type: MediaType | null`), project-mapper.helper.ts:60-62, types.ts:102 in FE. Need to confirm the Media table has `type` non-null in schema.prisma to verify whether `null` is reachable.
- **Suggested fix**: Tighten BE entity to `type: MediaType` (non-null) if Prisma `Media.type` is required, and expand FE union to `"IMAGE" | "VIDEO" | null` until either side is firmed up.

### F-010: Play overlay z-index in masonry `feed-item.tsx` sits behind the inner content View

- **Severity**: nit
- **Category**: style
- **File**: riz-app-v2/screens/feed/components/main/feed-item.tsx:53-72
- **Status**: open
- **Claim**: The play overlay `<View className="absolute inset-0 ...">` (line 54-60) is rendered as the first child of the `<Pressable>`, while the bookmark/title content `<View className="relative z-[1] flex-1 ...">` (line 61) explicitly sets `z-[1]`. RN stacks siblings by source order, but with z-index assigned, the relative child wins. As long as no inner element overlaps the centered play badge (~middle of card), the badge is visually correct. However, the `flex-1 justify-between p-2` View has full-area `flex-1` and may steal touch / hit-testing from the centered area, so taps on the play badge land on the surrounding `Pressable`'s `handlePress`, not on a dedicated play handler. Document whether a single `onPress` is intended.
- **Evidence**: feed-item.tsx:53-72; the design doc says masonry tap should still navigate to detail.
- **Suggested fix**: Either set `pointerEvents="none"` on the overlay (purely decorative) — or move it above the content with explicit `z-[2]`. Confirm intended tap behaviour.
