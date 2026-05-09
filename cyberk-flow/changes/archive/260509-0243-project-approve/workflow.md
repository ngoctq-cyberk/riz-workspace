# Workflow State: project-approve

> **Source of truth:** Workflow stages/gates → this file · Task completion → `tasks.md`
>
> **Checkbox states:** `[ ]` pending · `[/]` in progress · `[x]` done · `[-]` skipped/N/A

## Plan

- [x] 1. Context Review & Complexity Triage
  - [x] Read project context from AGENTS.md and `PLAN-project-approve.md`
  - [x] Run `cf changes`
  - [x] Choose `change-id`, run `cf new project-approve`
  - [x] Classify complexity: **standard**
  - [x] Check escalation flags: `cross-boundary`, `new-api-contract`, `data-migration`
  - [x] Record complexity + escalation flags in this file (Notes section)
- [x] 2. Discovery
  - [x] Inspect backend project create/list/detail/admin moderation paths
  - [x] Inspect admin Projects Management patterns and Posts tab pattern
  - [x] Inspect app project response, upload toast, Creator Studio card
  - [x] Fill `discovery.md`
  - [x] **Gate: user approved direction** — `PLAN-project-approve.md` is the approved direction
- [x] 3. Proposal
  - [x] Fill `proposal.md`
  - [x] UI Impact & E2E decision recorded in `proposal.md`
- [x] 4. Specs (Delta Format)
  - [x] Created specs for project approval gate and UI
  - [x] Each requirement has testable scenarios
- [x] 5. Design & Risk Assessment
  - [x] Create `design.md`
  - [x] Risk map records GitNexus CRITICAL on `getProject`
- [x] 6. Tasks
  - [x] Fill `tasks.md`
- [x] 7. Validation
  - [-] Oracle review skipped: direct implementation from approved plan
  - [x] `cf validate` passes
  - [x] Checklist: scenarios, appetite, questions resolved

## Implement

<!-- RULE: After completing each task, immediately mark it [x] in tasks.md AND log in Revision Log below. -->

- [x] 1. Read all change artifacts (workflow.md, proposal.md, design.md, tasks.md)
- [x] 2. Execute tasks sequentially in dependency order
- [x] 3. Update: mark `- [x]` in tasks.md + log in Revision Log after EACH task
- [/] 4. Verify Gate — run commands from `project.md` § Commands, **MUST execute and observe pass** _(mark `[-]` if N/A)_:
  - [/] Type check
  - [x] Lint
  - [-] Test
  - [-] E2E
- [x] 5. Review (adaptive — skip for trivial or doc/design-only):
  - [x] Code Review: Codex uncommitted-change review
- [x] 6. Findings triage: accepted P1 anonymous public list visibility finding
- [x] 7. Review Fix Loop _(max 3 rounds — fix, re-verify, re-review)_
- [x] 8. Validation
  - [x] **Gate: user approved implementation**
  - [-] Extract knowledge unavailable in this thread

## Archive

- [-] Deploy Gate _(skip if `project.md` § Commands → Deploy is N/A)_:
  - [-] Run deploy command
  - [-] Run smoke test
- [x] Apply deltas: `cf_apply` <!-- auto-ticked by script -->
- [x] Archive change: `cf_archive` <!-- auto-ticked by script -->
- [ ] Commit all changes
- [ ] Refresh gitnexus index: `npx -y gitnexus@latest analyze --skip-agents-md`

## Notes

**Complexity**: standard
**Escalation flags**: `cross-boundary` (riz-be + riz-admin-fe + riz-app-v2), `new-api-contract` (ProjectResponse.status), `data-migration` (Prisma default change)

**Blast radius**:

- `getProject` GitNexus risk CRITICAL: 8 direct callers in `ProjectService`. Mitigation: keep existing arguments optional and only add public-detail visibility check.
- `bulkSaveProjectStatus` GitNexus risk LOW.
- `ProjectsManagementPage` GitNexus risk LOW.
- GitNexus could not resolve app `ProjectItem`; fallback was direct source inspection.

## Revision Log

<!-- Format: YYYY-MM-DDTHH:MM:SSZ (ISO 8601 UTC). Get timestamp: date -u +%Y-%m-%dT%H:%M:%SZ -->

| DateTime (UTC)       | Author | Phase     | What Changed                                                                                                                                                                                                      | Why                                                              |
| -------------------- | ------ | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 2026-05-09T00:00:00Z | Codex  | Plan      | Created `project-approve` change from `PLAN-project-approve.md`; recorded standard scope, cross-boundary/API/migration risks.                                                                                     | Repo workflow requires a change directory before implementation. |
| 2026-05-09T00:00:00Z | Codex  | Implement | Implemented backend approval gate, admin moderation tabs/actions, app pending-review copy/status badge, and delta specs.                                                                                          | Execute approved project approval plan.                          |
| 2026-05-09T00:00:00Z | Codex  | Verify    | BE build, admin build, changed-file lint, `cf validate project-approve`, `git diff --check`, and GitNexus detect changes completed. App full tsc remains blocked by existing baseline errors outside this change. | Record verification status honestly.                             |
| 2026-05-09T02:42:10Z | Codex  | Review    | Accepted and fixed P1 anonymous `GET /project` visibility leak by only allowing own-author bypass when `authorId` is present and matches the current user.                                                        | Prevent pending/rejected projects from leaking in anonymous list. |
| 2026-05-09T02:42:10Z | Codex  | Archive   | Marked deploy gate skipped because `project.md` names deploy systems but provides no concrete deploy or smoke command.                                                                                            | Archive workflow allows deploy skip when deploy is not configured. |
