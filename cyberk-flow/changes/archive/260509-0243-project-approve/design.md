# Design: project-approve

## Decisions

- Use existing `Project.status`, `reviewedBy`, and `reviewedAt`; do not add a new moderation table.
- Prisma default changes to `PENDING_APPROVAL`, but create services still set status explicitly to avoid role ambiguity.
- Admin-created projects are immediately `PUBLISHED` and self-reviewed by the admin profile.
- Public detail checks status after fetching project data and returns 404 unless the current viewer is author or admin.
- Admin FE uses tabs instead of an all-status dropdown, following the Posts Management mental model.

## Risk Map

| Risk                                       | Level  | Mitigation                                                              |
| ------------------------------------------ | ------ | ----------------------------------------------------------------------- |
| `getProject` has 8 direct callers          | HIGH   | Add optional role parameter only; preserve existing profileId behavior. |
| Admin templates accidentally pending       | MEDIUM | Explicitly publish ADMIN/SUPERADMIN create paths.                       |
| Bulk payload publishes and rejects same id | LOW    | Reject duplicate id in controller before service call.                  |
| Existing app clients missing `status`      | LOW    | Keep app type field optional.                                           |
