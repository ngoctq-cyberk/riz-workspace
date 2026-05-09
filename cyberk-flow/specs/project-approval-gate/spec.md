# project-approval-gate Specification

## Purpose
TBD
## Requirements

### Requirement: New regular-user projects require approval

Projects created or cloned by `USER` SHALL be saved with `PENDING_APPROVAL`. `reviewedBy` and `reviewedAt` SHALL remain null for pending user-created projects. Projects created by `ADMIN` or `SUPERADMIN` SHALL be saved with `PUBLISHED`, `reviewedBy`, and `reviewedAt`.

#### Scenario: Regular user creates a project

- **GIVEN** authenticated user role is `USER`
- **WHEN** the user creates a project through a user-facing project endpoint
- **THEN** project status is `PENDING_APPROVAL`

#### Scenario: Admin creates a template project

- **GIVEN** authenticated user role is `ADMIN` or `SUPERADMIN`
- **WHEN** the admin creates a project through a project create endpoint
- **THEN** project status is `PUBLISHED` and review metadata is set

### Requirement: Public project visibility is gated by status

The public project list SHALL return `PUBLISHED` projects only, except author-owned list queries may include the author's own non-published projects. Public project detail SHALL return 404 for non-published projects unless the viewer is the author or an admin.

#### Scenario: Other user opens pending project detail

- **GIVEN** project status is `PENDING_APPROVAL`
- **WHEN** an anonymous viewer or a different regular user requests `GET /project/:id`
- **THEN** response is 404

#### Scenario: Author opens pending project detail

- **GIVEN** project status is `PENDING_APPROVAL`
- **WHEN** the project author requests `GET /project/:id`
- **THEN** response contains the project

### Requirement: Admin moderation payload is unambiguous

`/admin/projects/bulk-save` SHALL reject payloads where the same project id appears in both `publishIds` and `rejectIds`.

#### Scenario: Duplicate moderation id

- **GIVEN** `publishIds = ["1"]` and `rejectIds = ["1"]`
- **WHEN** admin calls `/admin/projects/bulk-save`
- **THEN** response is 400

