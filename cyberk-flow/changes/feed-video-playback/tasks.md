<!-- Tasks are executed sequentially in dependency order (topological sort). -->
<!-- Tasks with no Deps run first; tasks whose Deps are all complete run next. -->

## 1. Backend (parallel — additive)

- [x] T1 Thêm `type` vào ProjectImageEntity
  - **Refs**: specs/project-media-type-api/spec.md (Requirement: ProjectImageEntity documents `type` in Swagger); design.md#File-Map
  - **Done**: Entity có field `type: MediaType | null` với decorator Swagger, build/typecheck pass.
  - **Test**: N/A — type-only change, covered by typecheck in T9.
  - **Files**: `riz-be/apps/nest/libs/project/src/entities/project.entity.ts`
  - **Approach**: Thêm `@ApiPropertyOptional({ enum: MediaType })` `@Expose()` `type: MediaType | null` vào `ProjectImageEntity`. Import `MediaType` từ `@prisma/client`.

- [x] T2 Cập nhật ProjectMapperHelper.mapProjectData expose `type`
  - **Refs**: specs/project-media-type-api/spec.md (Requirement: Project image response exposes media type); design.md#AD1
  - **Done**: Type def + mapper output bao gồm `type: media.type` và `hasVideo`. Existing callers (`mapToProjectEntity`, `mapToTrendingProject*`, `getTrendingFeed`, `listProjects`, `getListItems`) không break.
  - **Test**: integration — `apps/nest/libs/project/**/*.spec.ts` (existing project specs cover mapper); manual response inspection via curl/swagger.
  - **Files**: `riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts`
  - **Approach**: Trong type definition thêm `type: MediaType | null` và optional `_count`. Trong mapper output thêm `type: media.type` và `hasVideo: (project.global._count?.attachments ?? 0) > 0`. Import `MediaType` từ `@prisma/client`.

## 2. FE Types (sau T2 — sync với backend)

- [x] T3 Thêm `type` vào ProjectImage (FE types)
  - **Deps**: T2
  - **Refs**: specs/project-media-type-api/spec.md (Requirement: Project image response exposes media type); design.md#AD1
  - **Done**: Types compile, FE consumer có thể đọc `image.type`.
  - **Test**: N/A — type-only.
  - **Files**: `riz-app-v2/lib/api/projects/types.ts`
  - **Approach**: Thêm `type?: "IMAGE" | "VIDEO"` vào interface `ProjectImage`. Optional để backward compat với response cũ.

## 3. Feed (parallel sau T3)

- [x] T4 Thêm `hasVideo` vào MasonryItemData
  - **Deps**: T3
  - **Refs**: specs/art-feed-video-playback/spec.md (Requirement: Feed item shows Play overlay for video projects); design.md#AD3
  - **Done**: Type compile.
  - **Test**: N/A — type-only.
  - **Files**: `riz-app-v2/screens/feed/types/types.ts`
  - **Approach**: Thêm `hasVideo: boolean` vào interface `MasonryItemData`.

- [x] T5 Cập nhật transform: hasVideo + cover fallback
  - **Deps**: T4
  - **Refs**: specs/art-feed-video-playback/spec.md (Requirements: Play overlay flag, Cover thumbnail fallback); design.md#AD4
  - **Done**: `hasVideo` được tính đúng. Cover thumbnail logic ưu tiên IMAGE đầu tiên, fallback `firstVideo.thumbnailUrl`.
  - **Test**: integration — verify trong manual smoke (T11 scenarios 4 & 6).
  - **Files**: `riz-app-v2/screens/feed/helper/transform.ts`
  - **Approach**: Trong `transformProjectToMasonryItem`:
    - `const hasVideo = project.hasVideo ?? images.some(i => i.type === "VIDEO")`.
    - `const firstNonVideo = images.find(i => i.type !== "VIDEO") ?? null`.
    - `const firstVideo = images.find(i => i.type === "VIDEO") ?? null`.
    - Nếu không có non-video image, dùng `firstVideo.thumbnailUrl` làm cover.

- [x] T6 Render Play overlay trong FeedItem
  - **Deps**: T4
  - **Refs**: specs/art-feed-video-playback/spec.md (Requirement: Feed item shows Play overlay); design.md#File-Map
  - **Done**: Play overlay hiển thị khi `item.hasVideo === true`; navigation vẫn hoạt động.
  - **Test**: E2E manual (T11 scenarios 1 & 4).
  - **Files**: `riz-app-v2/screens/feed/components/main/feed-item.tsx`
  - **Approach**: Khi `item.hasVideo`, thêm `<View>` overlay với Play icon (lucide `Play`), style theo `submission-gallery-item.tsx:33-39`. Đặt absolute center; không che bookmark/badge.

## 4. Art-feed (parallel sau T3)

- [x] T7 Bật tap-to-play video trong ProjectFeedItem và MediaCarousel
  - **Deps**: T3
  - **Refs**: specs/art-feed-video-playback/spec.md (Requirements: Tap-to-play, Reset on project change, Submission detail regression); design.md#AD2,AD5
  - **Done**: Video item hiển thị thumbnail + Play overlay; tap → video play với expo-video native controls. Swipe sang slide khác clear active video. Submission detail/community carousel không regression.
  - **Test**: E2E manual (T11 scenarios 3 & 5).
  - **Files**: `riz-app-v2/screens/art-feed/components/project-feed-item.tsx`, `riz-app-v2/components/media-carousel/media-carousel.tsx`, `riz-app-v2/components/media-carousel/types.ts`, `riz-app-v2/screens/challenges/components/submission-media-modal.tsx`, `riz-app-v2/screens/community/components/community-feed-item.tsx`, `riz-app-v2/screens/community/components/pending-post-card.tsx`
  - **Approach**:
    - Bỏ hardcode `type: "image"`. Map: `type: img.type === "VIDEO" ? "video" : "image"`.
    - Cho video: `url: img.mediumUrl ?? img.url` (HLS preferred giống `SubmissionMediaModal`).
    - Thêm `const [activeVideoIndex, setActiveVideoIndex] = useState<number | null>(null)`.
    - `useEffect(() => setActiveVideoIndex(null), [project.id])`.
    - Pass: `activeVideoIndex`, `onItemPress={(idx) => { if (mediaItems[idx]?.type === "video") setActiveVideoIndex(idx); }}`, `onActiveIndexChange`.
    - `MediaCarousel` chỉ active video khi `activeVideoIndex` trùng current slide và gọi `onActiveIndexChange` khi snap.

## 5. Verify Gate

- [x] T8 Lint fix các file đã thay đổi
  - **Deps**: T1-T7
  - **Refs**: project commands (CLAUDE.md "Before Completing Coding Tasks")
  - **Done**: ESLint không lỗi trên các file đã thay đổi.
  - **Test**: N/A — tooling step.
  - **Files**: changed files in T1-T7
  - **Approach**:
    - `cd riz-be && npx eslint --fix <changed BE files>`
    - `cd riz-app-v2 && bunx eslint --fix <changed FE files>`

- [/] T9 Type check + unit/integration test
  - **Deps**: T8
  - **Refs**: project commands
  - **Done**: Changed-file FE lint pass. Full `riz-app-v2` typecheck hiện vẫn fail bởi baseline errors ngoài scope; lỗi `MediaCarouselProps/isVideoActive` đã hết.
  - **Test**: integration — `cd riz-be/apps/nest && pnpm test`.
  - **Files**: N/A
  - **Approach**:
    - `cd riz-be && pnpm build`
    - `cd riz-app-v2 && bunx tsc --noEmit`
    - `cd riz-be/apps/nest && pnpm test`

- [ ] T10 gitnexus_detect_changes pre-commit
  - **Deps**: T9
  - **Refs**: CLAUDE.md "Self-Check Before Finishing"
  - **Done**: Affected scope khớp expectations (project mapper + feed/art-feed components).
  - **Test**: N/A — verification step.
  - **Files**: N/A
  - **Approach**: Run `gitnexus_detect_changes({scope: "all"})` trên cả 2 repo. Verify no unexpected scope.

- [ ] T11 E2E manual test
  - **Deps**: T9
  - **Refs**: proposal.md#UI-Impact-E2E
  - **Done**: 7 user journeys dưới đây pass.
  - **Test**: E2E (manual) — RN test infra cho video playback chưa có.
  - **Files**: N/A
  - **Approach**:
    1. Feed: project có challenge video → Play overlay hiển thị đúng vị trí.
    2. Tap project → router navigate `/art-feed?profileId&projectId` đúng project.
    3. Art-feed: thumbnail video + Play overlay → tap → video play với expo-video native controls (play/pause/seek/fullscreen).
    4. Art-feed: sau khi video đang play, swipe sang slide khác → video cũ không còn active và slide video khác không tự play nếu chưa tap.
    5. Project chỉ ảnh: không Play overlay ở feed, carousel ảnh hoạt động bình thường (regression).
    6. Submission detail (challenge feature): video vẫn play như trước (regression).
    7. Project full video: feed cover dùng `firstVideo.thumbnailUrl`, không broken image.
