# Round 2 — Codex rebuts Claude

### R-F-001: `project.service.ts` does NOT expose `hasVideo` or `images[].type`, breaking video playback contract for `/project` endpoints

- **Stance**: agree
- **Reasoning**: Đúng. Các endpoint `/project` và `/project/:id` đi qua `project.service.ts`; include hiện tại chỉ lấy `attachments` mà không lấy `_count`, và mapper riêng của service không copy `media.type` cũng không set `hasVideo`. FE art-feed dùng `useProjects` qua `/project`, nên `ProjectFeedItem` không nhận đủ contract để phân biệt video.
- **Counter-evidence**: `riz-be/apps/nest/libs/project/src/project.service.ts:137`, `riz-be/apps/nest/libs/project/src/project.service.ts:1058`, `riz-app-v2/hooks/use-projects.ts:16`

### R-F-002: `getProject` (single project detail) also missing `_count` and `type` mapping

- **Stance**: agree
- **Reasoning**: Đúng. `getProject` include `global.attachments` nhưng không include `_count`, sau đó gọi cùng mapper `mapToProjectEntity` vốn không trả `images[].type` hoặc `hasVideo`. Điều này ảnh hưởng đường `projectsApi.getById`.
- **Counter-evidence**: `riz-be/apps/nest/libs/project/src/project.service.ts:354`, `riz-be/apps/nest/libs/project/src/project.service.ts:364`, `riz-be/apps/nest/libs/project/src/project.service.ts:398`, `riz-be/apps/nest/libs/project/src/project.service.ts:1058`

### R-F-003: Repeated `_count` Prisma include literally duplicated 5× in admin-trending.service.ts — drift risk

- **Stance**: agree
- **Reasoning**: Đúng. Cùng selector `_count.select.attachments.where` được copy ở nhiều query trong `admin-trending.service.ts`; nếu sau này contract video đổi từ `PROJECT_IMAGE` sang usage khác hoặc thêm điều kiện `deletedAt`, các vị trí này rất dễ lệch nhau.
- **Counter-evidence**: `riz-be/apps/nest/libs/project/src/admin-trending.service.ts:283`, `riz-be/apps/nest/libs/project/src/admin-trending.service.ts:523`, `riz-be/apps/nest/libs/project/src/admin-trending.service.ts:563`, `riz-be/apps/nest/libs/project/src/admin-trending.service.ts:665`, `riz-be/apps/nest/libs/project/src/admin-trending.service.ts:896`

### R-F-004: `MediaCarousel` `onItemPress` callback closure churn defeats memoization

- **Stance**: agree
- **Reasoning**: Đúng. `mediaItems` là array mới trên mỗi render, `handleItemPress` phụ thuộc vào array đó, nên callback cũng đổi theo. Vì `MediaCarousel` là `React.memo` và nhận cả `media` lẫn `onItemPress`, memoization bị vô hiệu hóa khi parent re-render vì state like/bookmark/menu.
- **Counter-evidence**: `riz-app-v2/screens/art-feed/components/project-feed-item.tsx:66`, `riz-app-v2/screens/art-feed/components/project-feed-item.tsx:78`, `riz-app-v2/components/media-carousel/media-carousel.tsx:20`

### R-F-005: `isVideoActive` is one-shot global per item; second video auto-plays on swipe without user tap

- **Stance**: agree
- **Reasoning**: Đúng về rủi ro state model. `isVideoActive` chỉ là boolean cấp carousel và chỉ được set `true`; nó không lưu index video đã được tap và không reset khi carousel chuyển slide. Khi active index chuyển sang media video khác, điều kiện active có thể kích hoạt video đó mà không cần tap riêng.
- **Counter-evidence**: `riz-app-v2/screens/art-feed/components/project-feed-item.tsx:78`, `riz-app-v2/screens/art-feed/components/project-feed-item.tsx:192`, `riz-app-v2/components/media-carousel/media-carousel.tsx:57`

### R-F-006: Video URL fallback discards `mediumUrl` when present and falls through to raw `url`

- **Stance**: refute
- **Reasoning**: Claim bị diễn đạt sai. Code không "falls through to raw url" khi `mediumUrl` có giá trị; toán tử `??` ưu tiên `img.mediumUrl` và chỉ dùng `img.url` khi `mediumUrl` là `null` hoặc `undefined`. Việc player chỉ nhận một `uri` có thể là giới hạn khác, nhưng không chứng minh được fallback bị "discarded when present".
- **Counter-evidence**: `riz-app-v2/screens/art-feed/components/project-feed-item.tsx:70` uses `url: img.type === "VIDEO" ? (img.mediumUrl ?? img.url) : img.url`; `riz-app-v2/components/media-carousel/media-carousel-item.tsx:71` passes only `item.url` to the video player.

### R-F-007: `images.find((i) => i.type !== "VIDEO")` matches images with `type === undefined`

- **Stance**: agree
- **Reasoning**: Đúng. Vì FE type cho `ProjectImage.type` là optional, mọi item không có `type` đều thỏa `type !== "VIDEO"`. Khi backend endpoint cũ không emit `type`, transform sẽ xem phần tử đầu tiên là ảnh thường và fallback `hasVideo` cũng không phát hiện được video.
- **Counter-evidence**: `riz-app-v2/lib/api/projects/types.ts:102`, `riz-app-v2/screens/feed/helper/transform.ts:13`, `riz-app-v2/screens/feed/helper/transform.ts:16`

### R-F-008: `_count.attachments` value unguarded — Prisma typing optional

- **Stance**: agree
- **Reasoning**: Đúng. Helper type cho phép `_count` vắng mặt, và mapper biến trường hợp vắng mặt thành `hasVideo: false`. Điều này làm lỗi include bị che giấu thay vì fail sớm hoặc bị bắt bởi type.
- **Counter-evidence**: `riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts:18`, `riz-be/apps/nest/libs/project/src/helpers/project-mapper.helper.ts:125`

### R-F-009: BE `ProjectImageEntity.type: MediaType | null` vs FE `type?: "IMAGE" | "VIDEO"` mismatched optionality

- **Stance**: refute
- **Reasoning**: Runtime claim "BE may emit null" không đúng với Prisma schema hiện tại: `Media.type` là required, không nullable. Entity annotation đang rộng hơn schema, nhưng dữ liệu từ `media.type` không thể là `null` nếu đi qua Prisma row hợp lệ.
- **Counter-evidence**: `riz-be/apps/nest/prisma/schema.prisma:568` declares `type MediaType` without `?`; `riz-be/apps/nest/prisma/schema.prisma:589` defines `enum MediaType`.

### R-F-010: Play overlay z-index sits behind inner content View

- **Stance**: refute
- **Reasoning**: Phần "may steal hit-testing from play badge area" không được code chứng minh. Overlay và content đều nằm bên trong cùng parent `Pressable`; vùng giữa không có child `Pressable` nào khác, chỉ bookmark ở góc trên phải có handler riêng. Vì vậy tap vào badge trung tâm vẫn đi qua parent `Pressable`.
- **Counter-evidence**: `riz-app-v2/screens/feed/components/main/feed-item.tsx:53` wraps overlay/content in one `Pressable`; only nested press handler is bookmark at `riz-app-v2/screens/feed/components/main/feed-item.tsx:65`.
