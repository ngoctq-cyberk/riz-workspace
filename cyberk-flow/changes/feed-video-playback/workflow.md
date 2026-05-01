# Workflow State: feed-video-playback

> **Source of truth:** Workflow stages/gates → this file · Task completion → `tasks.md`
>
> **Checkbox states:** `[ ]` pending · `[/]` in progress · `[x]` done · `[-]` skipped/N/A

## Plan

- [x] 1. Context Review & Complexity Triage
  - [x] Read `project.md` for project context
  - [x] Run `cf changes` + `cf specs`
  - [x] Choose `change-id`, run `cf new feed-video-playback`
  - [x] Classify complexity: **small** (additive, ~7 files, tái sử dụng component có sẵn)
  - [x] Check escalation flags: `cross-boundary` (riz-be + riz-app-v2), `new-api-contract` (additive `type` field)
  - [x] Record complexity + escalation flags in this file (Notes section)
- [x] 2. Discovery
  - [x] Select relevant workstreams (Architecture / Patterns / Constraints)
  - [x] Execute workstreams (đọc code FE + BE liên quan, gitnexus_impact analysis)
  - [x] Fill `discovery.md` — findings, gap analysis, options, risks
  - [x] **🚪 Gate: user approved direction** — user chọn Option A (tap-to-play, không auto-play)
- [x] 3. Proposal
  - [x] Fill `proposal.md` — Why, Appetite, Scope, Capabilities, Impact, Risk
  - [x] **MANDATORY** UI Impact & E2E decision recorded in `proposal.md`
- [x] 4. Specs (Delta Format)
  - [x] Create specs per capability at `specs/<capability>/spec.md`
  - [x] Each requirement has ≥1 testable scenario
  - [ ] **🚪 Gate: user approved specs** — present spec list + key requirements
- [x] 5. Design & Risk Assessment
  - [x] Create `design.md` — gap analysis, architecture decisions, Risk Map
  - [x] MEDIUM risk → include interface sketch (Mermaid)
  - [ ] **🚪 Gate: design reviewed**
- [x] 6. Tasks
  - [x] Fill `tasks.md` — execution-ordered, dependency-aware checklist
  - [x] Each task has: Deps, Refs, Done criteria, Test, Files, Approach
- [ ] 7. Validation
  - [-] Oracle review — skipped (small change, well-scoped)
  - [ ] Findings triage: accept/reject each finding with rationale
  - [ ] `cf validate` passes (or errors justified)
  - [ ] Checklist: scenarios ✓, appetite ✓, questions resolved ✓
  - [ ] **🚪 Gate: user approved plan** — present checklist + recommend `/cf-build`

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
- [ ] 5. Review (adaptive — skip for trivial or doc/design-only):
  - [ ] Code Review: `cf-review-master` subagent
- [ ] 6. Findings triage: accept/rebut each finding with rationale
- [ ] 7. Review Fix Loop _(max 3 rounds — fix, re-verify, re-review)_
- [ ] 8. Validation
  - [ ] **🚪 Gate: user approved implementation**
  - [ ] Extract knowledge

## Archive

- [ ] Deploy Gate _(skip if `project.md` § Commands → Deploy is N/A)_:
  - [ ] Run deploy command
  - [ ] Run smoke test
- [ ] Apply deltas: `cf_apply` <!-- auto-ticked by script -->
- [ ] Archive change: `cf_archive` <!-- auto-ticked by script -->
- [ ] Commit all changes
- [ ] Refresh gitnexus index: `npx -y gitnexus@latest analyze --skip-agents-md`

## Notes

**Complexity**: small
**Escalation flags**: `cross-boundary` (riz-be + riz-app-v2), `new-api-contract` (additive `type` field cho `ProjectImage`)

**Key decisions**:
- User chọn tap-to-play (không auto-play) ở art-feed.
- Tái sử dụng pattern `SubmissionMediaModal` + `CommunityFeedInlineVideo` (challenge feature).
- Backend change additive — không break existing callers.
- Admin frontend (`riz-admin-fe`) out of scope phase này.

**Blast radius (gitnexus_impact)**:
- `mapProjectData` HIGH theo gitnexus (6 callers, 3 affected processes: `getTrendingFeed`, `listProjects`, `getListItems`) → MITIGATED do additive only.
- FE `transformProjectToMasonryItem` LOW (1 caller).
- FE `FeedItem` interface MEDIUM (7 importers) — thêm field optional, không break.

## Revision Log

<!-- Format: YYYY-MM-DDTHH:MM:SSZ (ISO 8601 UTC). Get timestamp: date -u +%Y-%m-%dT%H:%M:%SZ -->

| DateTime (UTC) | Author | Phase | What Changed | Why |
| -------------- | ------ | ----- | ------------ | --- |
| 2026-04-29T04:24:30Z | quyngoc + Claude | Plan | Created change skeleton via `cf new feed-video-playback`. Filled discovery, proposal, specs (2 capabilities), design, tasks. Marked Plan stages 1-6 done. | Migrate ad-hoc plan.md sang cyberk-flow workflow theo yêu cầu user. |
| 2026-04-29T09:30:00Z | quyngoc + Claude | Implement | T1-T9 complete: BE entity + mapper expose `type`, FE types + `hasVideo`, transform cover fallback, Play overlay FeedItem, tap-to-play ProjectFeedItem. Lint + BE build pass. | Implement feed-video-playback theo tasks.md. |
| 2026-04-29T10:20:00Z | Codex | Review Fix Loop | Synced docs with current code: challenge videos use `PROJECT_IMAGE`, project/trending responses expose `_count`/`hasVideo`, `MediaCarousel` uses `activeVideoIndex` + `onActiveIndexChange`, and call sites were updated. | Keep change docs aligned after A-001..A-005 and P1 review fixes. |
| 2026-04-29T06:39:58Z | Codex | Documentation | Updated Verify Gate docs: changed-file lint passes; full `riz-app-v2` typecheck remains blocked by existing baseline errors outside this change, while `MediaCarouselProps/isVideoActive` errors are resolved. | Avoid docs claiming a full typecheck pass that current codebase does not have. |
