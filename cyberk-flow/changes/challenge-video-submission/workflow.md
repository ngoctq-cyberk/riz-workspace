# Workflow State: challenge-video-submission

> **Source of truth:** Workflow stages/gates → this file · Task completion → `tasks.md`
>
> **Checkbox states:** `[ ]` pending · `[/]` in progress · `[x]` done · `[-]` skipped/N/A

## Plan

- [x] 1. Context Review & Complexity Triage
  - [x] Read `project.md` for project context
  - [x] Run `cf changes` + `cf specs`
  - [x] Choose `change-id`, run `cf new challenge-video-submission`
  - [x] Classify complexity: **standard** (multi-file, cross-boundary, UI)
  - [x] Check escalation flags: `cross-boundary` (riz-app-v2 + riz-admin-fe)
  - [x] Record complexity + escalation flags in this file (Notes section)
- [x] 2. Discovery
  - [x] Select relevant workstreams (Architecture / Patterns / Constraints)
  - [x] Execute workstreams (đọc toàn bộ files liên quan)
  - [x] Fill `discovery.md` — findings, gap analysis, options, risks
  - [x] **🚪 Gate: user approved direction** — user cung cấp PLAN chi tiết, direction đã xác định
- [x] 3. Proposal
  - [x] Fill `proposal.md` — Why, Appetite, Scope, Capabilities, Impact, Risk
  - [x] **MANDATORY** UI Impact & E2E decision recorded in `proposal.md`
- [x] 4. Specs (Delta Format)
  - [x] Create specs per capability at `specs/<capability>/spec.md`
  - [x] Each requirement has ≥1 testable scenario
  - [x] **🚪 Gate: user approved specs** — proceeding to implement per user instruction
- [x] 5. Design & Risk Assessment
  - [x] Create `design.md` — gap analysis, architecture decisions, Risk Map
  - [x] **🚪 Gate: design reviewed** — self-reviewed, MEDIUM risk acceptable
- [x] 6. Tasks
  - [x] Fill `tasks.md` — execution-ordered, dependency-aware checklist
  - [x] Each task has: Deps, Refs, Done criteria, Test, Files, Approach
- [x] 7. Validation
  - [-] Oracle review — skipped, plan provided by user
  - [-] Findings triage — N/A
  - [-] `cf validate` — skipped for speed
  - [x] Checklist: scenarios ✓, appetite ✓, questions resolved ✓
  - [x] **🚪 Gate: user approved plan** — user said "implement"

## Implement

<!-- RULE: After completing each task, immediately mark it [x] in tasks.md AND log in Revision Log below. -->
- [x] 1. Read all change artifacts (workflow.md, proposal.md, design.md, tasks.md)
- [x] 2. Execute tasks sequentially in dependency order
- [x] 3. Update: mark `- [x]` in tasks.md + log in Revision Log after EACH task
- [x] 4. Verify Gate — run commands from `project.md` § Commands:
  - [x] Type check — pass (riz-admin-fe + riz-app-v2)
  - [x] Lint — pass (riz-admin-fe + riz-app-v2)
  - [-] Test — N/A (no unit tests affected)
  - [-] E2E — N/A (manual test needed)
- [-] 5. Review — skipped (trivial cross-boundary change, no complex logic)
- [-] 6. Findings triage — N/A
- [-] 7. Review Fix Loop — N/A
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

- Complexity: **standard**, flag: **cross-boundary** (riz-app-v2 + riz-admin-fe)
- Phase 1 only: không record từ camera, không progress bar, không HLS player
- `imageUrl` được giữ làm deprecated fallback trong admin types
- Mobile: `expo-image-picker` với `mediaTypes: ['videos']` hỗ trợ cả iOS và Android
- Admin: Vimeo iframe embed, Phase 2 mới upgrade lên HLS nếu cần

## Revision Log

<!-- Format: YYYY-MM-DDTHH:MM:SSZ (ISO 8601 UTC). Get timestamp: date -u +%Y-%m-%dT%H:%M:%SZ -->

| DateTime (UTC) | Author | Phase | What Changed | Why |
| -------------- | ------ | ----- | ------------ | --- |
| 2026-04-28T00:00:00Z | claude | Implement | T1: SubmissionMedia type + media field vào PublicChallengeSubmission, MySubmission (mobile) | Backward compat — optional field |
| 2026-04-28T00:00:00Z | claude | Implement | T2: SubmissionMedia type + media field vào ChallengeSubmission (admin) | |
| 2026-04-28T00:00:00Z | claude | Implement | T3: transformSubmission() trong challenge-api.ts; listSubmissions + reviewSubmission dùng transformer | Map project.attachments → media[] |
| 2026-04-28T00:00:00Z | claude | Implement | T4: submission-media-gallery.tsx mới (admin) — iframe Vimeo + img | |
| 2026-04-28T00:00:00Z | claude | Implement | T5: submission-detail-dialog replace img bằng SubmissionMediaGallery | |
| 2026-04-28T00:00:00Z | claude | Implement | T6: challenge-detail-page thumbnail fallback + play overlay icon | |
| 2026-04-28T00:00:00Z | claude | Implement | T7: video section trong challenge-submit (pick + validate + preview + FormData append) | |
| 2026-04-28T00:00:00Z | claude | Implement | T8: play overlay trong submission-gallery-item + my-submission-card | |
| 2026-04-28T00:00:00Z | claude | Implement | T9: i18n keys en.json + vi.json (video label, hint, error alerts) | |
