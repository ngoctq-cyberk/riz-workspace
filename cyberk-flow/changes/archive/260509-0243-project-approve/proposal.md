# Proposal: project-approve

## Why

Regular user projects should not appear in public feed/detail until admin moderation approves them.

## Appetite

M — cross-boundary backend/admin/app change with one Prisma default migration.

## Scope

- Backend: default new projects to `PENDING_APPROVAL`, explicitly publish admin-created projects, enforce public visibility, validate bulk moderation payloads.
- Admin FE: Projects Management defaults to Pending tab and exposes explicit Publish/Reject actions.
- App: Project API type includes status, create success copy says pending review, Creator Studio shows status badge.

## Out of Scope

- Rejection reason API/UI.
- Reset published projects to pending when edited.
- Challenge submission approval semantics.

## UI Impact & E2E

- **User-visible UI behavior affected?** YES
- **E2E required?** REQUIRED
- **Justification**: Admin moderation actions and app pending-review status are user-visible workflow changes.
- **Target user journeys**:
  1. Admin opens Projects Management and lands on Pending.
  2. Admin publishes/rejects selected projects from the correct tab.
  3. User creates a project and sees pending-review copy.
  4. User opens Creator Studio and sees project status badge.

## Risk

MEDIUM overall. `getProject` impact is CRITICAL in GitNexus because many internal project flows call it; implementation keeps the existing parameters optional and only adds a public visibility check.
