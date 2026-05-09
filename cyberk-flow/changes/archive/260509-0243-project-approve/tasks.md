<!-- Tasks are executed sequentially in dependency order (topological sort). -->

## 1. Backend

- [x] 1_1 Default new projects to pending approval
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: Prisma schema and migration default `Project.status` to `PENDING_APPROVAL`.
  - **Test**: Prisma/client build or backend typecheck.
  - **Files**: `riz-be/apps/nest/prisma/schema.prisma`, `riz-be/apps/nest/prisma/migrations/**`
  - **Approach**: Change default only; do not rewrite existing data.

- [x] 1_2 Set explicit review fields on create/clone paths
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: USER creates/clones become `PENDING_APPROVAL`; ADMIN/SUPERADMIN creates become `PUBLISHED` with review metadata.
  - **Test**: Backend build/typecheck; targeted project tests if available.
  - **Files**: `riz-be/apps/nest/libs/project/src/project.service.ts`, `riz-be/apps/nest/libs/project/src/project3d.service.ts`
  - **Approach**: Add helper for initial review fields and spread into create data.

- [x] 1_3 Enforce public visibility
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: Public list remains published-only except own author query; detail returns 404 for non-published projects unless author/admin.
  - **Test**: Backend build/typecheck.
  - **Files**: `riz-be/apps/nest/libs/project/src/project.controller.ts`, `riz-be/apps/nest/libs/project/src/project.service.ts`
  - **Approach**: Keep `getProject` signature backward compatible by adding optional role argument.

- [x] 1_4 Validate bulk moderation payload
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: Same project id cannot appear in both `publishIds` and `rejectIds`.
  - **Test**: Backend build/typecheck.
  - **Files**: `riz-be/apps/nest/libs/project/src/admin-project.controller.ts`
  - **Approach**: Validate after ID parsing before service call.

## 2. Admin FE

- [x] 2_1 Convert Projects Management to moderation tabs
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: Default tab is Pending; tabs exist for Pending, Published, Rejected.
  - **Test**: Admin FE typecheck/lint.
  - **Files**: `riz-admin-fe/src/screens/projects/projects-management/ui/projects-management-page.tsx`
  - **Approach**: Follow Posts Management tab pattern.

- [x] 2_2 Replace Save with explicit moderation actions
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: Pending supports Publish/Reject; Published supports Reject; Rejected supports Publish.
  - **Test**: Admin FE typecheck/lint.
  - **Files**: `riz-admin-fe/src/screens/projects/projects-management/ui/projects-management-page.tsx`
  - **Approach**: Send disjoint `publishIds`/`rejectIds` payloads.

## 3. App

- [x] 3_1 Expose and render project status
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: `ProjectResponse.status` exists and Creator Studio shows status badge.
  - **Test**: App typecheck/lint.
  - **Files**: `riz-app-v2/lib/api/projects/types.ts`, `riz-app-v2/screens/creator-studio/components/project-item.tsx`, locale files.
  - **Approach**: Keep status optional for backward compatibility.

- [x] 3_2 Update create success copy
  - **Refs**: `PLAN-project-approve.md`
  - **Done**: 2D and 3D create success messages say the project is pending review.
  - **Test**: App typecheck/lint.
  - **Files**: `riz-app-v2/screens/upload-project/index.tsx`, `riz-app-v2/lib/widgets/scene-customization/saving-overview-state/state.tsx`, locale files.
  - **Approach**: Reuse localized toast copy.

## 4. Verification

- [/] 4_1 Run checks
  - **Deps**: 1_1, 1_2, 1_3, 1_4, 2_1, 2_2, 3_1, 3_2
  - **Done**: Backend/admin/app checks run or blockers recorded. BE build and admin build pass; app full tsc remains blocked by baseline errors outside this change.
  - **Test**: lint/typecheck/build commands per repo.
  - **Approach**: Start with changed-file lint, then build/typecheck where feasible.

- [ ] 4_2 Manual E2E acceptance
  - **Deps**: 4_1
  - **Done**: Admin moderation and app pending-review journeys are manually verified.
  - **Test**: E2E manual.
  - **Approach**: Verify Pending tab default, publish/reject actions, create-project pending copy, and Creator Studio status badge.
