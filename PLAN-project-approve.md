# Project Approval Gate Before App Feed

## Tóm Tắt

Thêm moderation gate cho project user tạo: regular user tạo/clone project sẽ vào `PENDING_APPROVAL`, chỉ hiện trên app feed/public project detail sau khi admin publish. Tái dùng hạ tầng đã có sẵn của `Project.status`, `reviewedBy`, `reviewedAt`, `/admin/projects`, và `PATCH /admin/projects/bulk-save`; không dựng bảng mới.

## Thay Đổi Chính

- Backend `riz-be`
  - Đổi Prisma default của `Project.status` từ `PUBLISHED` sang `PENDING_APPROVAL` bằng migration mới.
  - Trong các flow tạo project user-facing (`createProject`, `createProject3D`, `cloneProject`, debug/create-from-walls nếu còn dùng), set status tường minh:
    - `USER` -> `PENDING_APPROVAL`, `reviewedBy/reviewedAt = null`.
    - `ADMIN/SUPERADMIN` -> `PUBLISHED`, tự set `reviewedBy = user.profileId`, `reviewedAt = now` để không làm hỏng admin-created Panorama template.
  - Siết public visibility:
    - `GET /project` chỉ trả `PUBLISHED`, trừ khi query `authorId` là chính current user thì được thấy project của mình ở mọi status.
    - `GET /project/:id` chỉ cho xem nếu project `PUBLISHED`, hoặc current user là author, hoặc current user là `ADMIN/SUPERADMIN`; còn lại trả 404.
  - Giữ `PATCH /admin/projects/bulk-save` làm moderation API, nhưng validate không cho cùng một project id xuất hiện trong cả `publishIds` và `rejectIds`.

- Admin FE `riz-admin-fe`
  - Sửa màn Projects Management theo pattern rõ như Posts/Challenge: default tab/filter là `Pending`, có `Pending`, `Published`, `Rejected`.
  - Thay toolbar “Save” hiện tại bằng action rõ ràng:
    - Pending: `Publish` và `Reject`.
    - Published: `Reject`.
    - Rejected: `Publish`.
  - Detail dialog giữ preview project và có nút `Publish/Reject` theo status hiện tại.
  - Invalidate `adminProjectKeys.all` sau moderation như hiện tại.

- App `riz-app-v2`
  - Thêm `ProjectStatus` vào type `ProjectResponse`.
  - Sau khi tạo project thành công, toast/message đổi thành ý nghĩa “đã gửi chờ duyệt”, vẫn điều hướng về Creator Studio.
  - Creator Studio hiển thị badge status cho project của chính user: `Pending approval`, `Published`, `Rejected`.
  - Feed không cần đổi query chính vì backend `/trending/feed` đã filter `PUBLISHED`.

## Test Plan

- Backend tests:
  - `POST /project` với regular user tạo project status `PENDING_APPROVAL`.
  - Project pending không xuất hiện ở `GET /project`, `/trending/feed`, và không xem được qua `GET /project/:id` bởi anonymous/other user.
  - Author vẫn thấy project pending trong creator list/detail.
  - Admin publish project qua `/admin/projects/bulk-save`, sau đó project xuất hiện lại ở public list/feed/detail.
  - Admin-created Panorama/project vẫn `PUBLISHED` để không làm hỏng template flow.
  - Reject project thì bị remove khỏi trending list như behavior hiện tại.

- Admin FE checks:
  - Pending tab hiển thị project mới tạo.
  - Bulk publish/reject không gửi cùng id vào cả hai list.
  - Detail dialog publish/reject xong refresh đúng tab/count.

- App checks:
  - Create project success copy đổi sang pending-review.
  - Creator Studio hiện status badge.
  - Feed không hiện project pending.

## Giả Định

- Scope v1 chỉ gate project khi tạo/clone; chỉnh sửa project đã published chưa tự reset về `PENDING_APPROVAL`.
- Không thêm rejection reason UI/API ở v1, vì project moderation API hiện tại chỉ publish/reject theo status.
- Challenge submission approval giữ contract riêng: public challenge submissions vẫn dựa trên `ChallengeSubmissionStatus.APPROVED`; project feed/public project vẫn dựa trên `Project.status`.
- Trước khi implement sẽ tạo change mới trong `cyberk-flow/changes/` vì hiện chưa có change directory khớp feature này.
