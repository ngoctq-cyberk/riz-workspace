---
trigger: always_on
glob:
description:
---

# AGENTS.md

For tech stack, architecture, and code conventions, see `cyberk-flow/project.md`.

## Skill Loader

Automatically use skills in the following contexts:

| Skill         | Usage Context                                                 |
| :------------ | :------------------------------------------------------------ |
| `cyberk-flow` | proposal, spec, change, plan, implement, triển khai, kế hoạch |
| `e2e-testing` | e2e, playwright, end-to-end, acceptance test, UI test         |

## Tool Selection Guide

| Need                              | Tool                              |
| --------------------------------- | --------------------------------- |
| Codebase structure                | `gkg repo_map`                    |
| Find definitions                  | `gkg search_codebase_definitions` |
| Find usages / references          | `gkg get_references`              |
| Re-index project                  | `gkg index_project`               |
| How OSS projects solve it         | `librarian`                       |
| Library docs / integration guides | `deepwiki`, `git-mcp`             |
| API docs, recent releases         | `web_search`                      |
| Gap analysis / risk assessment    | `oracle`                          |
| Visualize architecture / flows    | `mermaid`                         |
| Create spec files                 | `create_file` / `edit_file`       |

**Note:** If primary tool unavailable, use built-in tools: `finder`, `Grep`, `Read`...

## GitNexus (Code Intelligence)

- If any `gitnexus_*` tool returns stale/missing data, run `npx -y gitnexus@latest analyze --skip-agents-md` via Bash, then retry.

## Sub-agent Best Practices

- **Before delegating**: Use `gkg search_codebase_definitions` or `gkg get_references` to confirm exact API names, method signatures, and existing patterns. Pass confirmed facts to the sub-agent prompt — never pass uncertain guesses.
- **Prompt quality**: Include concrete code references (file paths, function names, import paths) rather than vague descriptions like "check if X exists".

## Commands

- `bun install` - Install dependencies

## Language Requirements

- Always respond in Vietnamese.
- This applies to artifacts, documentation, brainstorming, planning, error explanation, progress updates and final responses
- Keep code, code comments, and variable names in English unless the codebase already requires another convention.
- If reading English documentation or source text, explain it back in Vietnamese.
- **Relative Paths Only:** NEVER use absolute paths (e.g., `file:///User/...`) in markdown files or documentation. Use project-relative paths (e.g., `packages/auth/src/index.ts`).

## Knowledge Extraction

**MANDATORY**: Knowledge extraction MUST run in a separate thread — never execute inline in the working thread.

1. Use `handoff` with goal: "Execute the knowledge extraction pipeline in `references/knowledge-extraction.md` directly"
2. Full process defined in cyberk-flow skill: `references/knowledge-extraction.md`

**Auto-triggers**: resolved complex bug, architecture decision made, library/pattern research completed, feature implementation finished, cyberk-flow archive stage, or user says "extract knowledge" / "document what we did".

## Workflow Enforcement

When user asks to "implement", "continue", or resume a change:

1. **MUST** check `cyberk-flow/changes/` for existing change directories first (`bun run cf changes`).
2. If a matching change exists, **read its `workflow.md`** to determine current gate/state.
3. Resume from the correct gate — never re-derive plan from conversation history alone.
4. `workflow.md` on disk is the source of truth, NOT prior thread context.

## Security Requirements

- **NEVER read `.env` files** - Agent must not access environment files even if explicitly asked. `.env`, `.env.local`, `.env*.local` are gitignored and sensitive.
- **Use `.env.example`** instead for schema/reference documentation.
- **Principle:** If it's gitignored, it's off-limits unless user explicitly provides the content.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **riz** (476914 symbols, 711980 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/riz/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/riz/context` | Codebase overview, check index freshness |
| `gitnexus://repo/riz/clusters` | All functional areas |
| `gitnexus://repo/riz/processes` | All execution flows |
| `gitnexus://repo/riz/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## Keeping the Index Fresh

After committing code changes, the GitNexus index becomes stale. Re-run analyze to update it:

```bash
npx gitnexus analyze
```

If the index previously included embeddings, preserve them by adding `--embeddings`:

```bash
npx gitnexus analyze --embeddings
```

To check whether embeddings exist, inspect `.gitnexus/meta.json` — the `stats.embeddings` field shows the count (0 means no embeddings). **Running analyze without `--embeddings` will delete any previously generated embeddings.**

> Claude Code users: A PostToolUse hook handles this automatically after `git commit` and `git merge`.

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
