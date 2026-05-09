# Discovery: project-approve

## Findings

- `Project.status` existed with `PUBLISHED` default in Prisma.
- User create paths include `createProject`, `createProject3D`, `createProject3DFromWalls`, and `cloneProject`.
- `GET /project` already forces `PUBLISHED` unless `authorId` matches current user.
- `GET /project/:id` did not enforce status visibility before this change.
- Admin moderation already exists through `/admin/projects/bulk-save`.
- Admin Projects Management used an `ALL/PUBLISHED/REJECTED` filter and a confusing Save action.
- App `ProjectResponse` did not model project status, and create success copy implied immediate success/publication.

## Risks

- `getProject` has many internal callers; add only optional role handling and keep author/internal calls working.
- Admin-created Panorama templates must remain published; create helpers must branch on `ADMIN/SUPERADMIN`.
- Bulk moderation must send disjoint publish/reject ids.
