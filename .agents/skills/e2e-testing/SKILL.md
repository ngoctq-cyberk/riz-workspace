---
name: e2e-testing
description: "Playwright-based E2E testing with optional Web3/Phantom module. Use when adding UI acceptance tests, creating test plans, generating specs from plans, or healing failing tests."
---

# E2E Testing

Playwright-based E2E testing skill with two browser automation backends (CLI default, Test Agents MCP optional) and optional Web3/Synpress module (Phantom wallet). Can integrate with spec-driven workflows.

## When to Use

- Adding acceptance tests for user-visible UI flows
- Regression safety for auth, navigation, critical journeys
- Generating E2E tests from specs or test plans
- Healing/fixing broken E2E tests automatically
- Web3 wallet flows (Phantom connect, SIWE sign-in, transactions) — optional module

## Artifacts

| Artifact | Location | Description |
|----------|----------|-------------|
| Test plans | `e2e/plans/*.md` | Human-readable scenario plans |
| Test specs | `e2e/specs/**/*.spec.ts` | Executable Playwright tests |
| Fixtures | `e2e/fixtures/*` | Shared test helpers and setup |
| Wallet setup | `e2e/wallet-setup/*` | Synpress cache config (Web3 only) |

## Core Rules

1. **No orchestrators in webServer** — never use turbo/nx/lerna as Playwright `webServer.command`. Run app dev scripts directly with absolute `cwd`. See [troubleshooting/webserver-orphans.md](references/troubleshooting/webserver-orphans.md).
2. **Test isolation** — each test should be independent and runnable in any order.
3. **Resilient selectors** — prefer `getByRole()` > `getByText()` > `getByLabel()` > `getByTestId()` > CSS.
4. **CI safety** — disable traces/videos in CI to prevent secrets leakage.

## Escalation & Time-Boxing (MANDATORY)

These rules prevent wasting time in debugging loops. **Violations are considered skill failures.**

### Definitions

- **Fix attempt** = any code change intended to address the failure.
- **Retry cycle** = `run single test → analyze artifacts → 1 fix attempt`. Each cycle increments `attempt` counter.
- **Wall time** = total elapsed time on this test, including test runtime + tool calls + analysis.

### Budget Limits

| Scope | Hard Limit | Action on Breach |
|-------|-----------|-----------------|
| **Single test debug** | **2 retry cycles** | STOP retrying, escalate to next tool |
| **Single test wall time** | **180 seconds** total | Abandon current approach, escalate |
| **Single run >60s** | Only **1 retry** allowed | Escalate immediately after 1 failed fix |
| **Full suite** | **5 minutes** total debug time | Stop, summarize failures, ask user |

### Mandatory Budget Log

After EVERY test run during debugging, you MUST print a status line:

```
🔄 Budget: test=<name>, attempt=<N>/2, elapsed=<Xs>/180s, snapshot_taken=<yes/no>, next=<action>
```

Example: `🔄 Budget: test=edit-todo, attempt=1/2, elapsed=45s/180s, snapshot_taken=yes, next=web_search`

This is **not optional**. Skipping the budget log is a skill violation.

### Escalation Ladder (State Machine)

Per failing test, follow this exact sequence — **do NOT skip steps when tool is available**:

1. **Isolate** — Run only the failing test with `--grep` or `test.only`, with `--timeout 15000`.
2. **Snapshot** — Before ANY fix, capture DOM state:
   - Primary: `playwright-cli snapshot` or `browser_snapshot`
   - Fallback (if unavailable): `--trace on-first-retry` → view trace artifact
   - Last resort: `page.screenshot()` in the test itself
3. **Fix attempt 1** — Make one targeted fix based on the snapshot. Re-run isolated. Print budget log.
4. **Web search** (if attempt 1 failed) — Search for the specific error + framework combo (e.g., `"playwright getByRole input value" site:playwright.dev`).
5. **Fix attempt 2** — Apply web search findings. Re-run isolated. Print budget log.
6. **Oracle** (if attempt 2 failed) — Ask oracle with: test code + error message + DOM snapshot/trace.
7. **Final fix** — Apply oracle suggestion. Re-run. Print budget log.
8. **Bail out** (if still failing):
   - **Critical/blocking test** → STOP and ask user for guidance. Do NOT auto-fixme.
   - **Non-critical test** → Mark `test.fixme("Selector: <what>, Attempted: <what>, Snapshot: <path>")` and move on.

### Anti-Patterns (NEVER DO)

- ❌ Retrying the same locator strategy with minor variations (e.g., `hasText` → `hasText` with different string)
- ❌ Waiting for full timeout (60-90s) before investigating — always use `--timeout 15000` when debugging
- ❌ Debugging without a DOM snapshot — snapshot is REQUIRED before every fix
- ❌ Running the entire suite repeatedly to debug a single test — isolate with `test.only` or `--grep`
- ❌ Spending >2 minutes on a single locator — if `getByRole` doesn't work, snapshot and pivot
- ❌ Skipping the budget log line after a test run

### Quick Diagnostic Checklist

Before retrying a failing test, answer these questions:

1. **Did I take a snapshot?** If no → snapshot first
2. **Is the element actually in the DOM?** If no → the app has a bug, not the test
3. **Is the element in a different state?** (e.g., inside an input instead of a span) → change locator strategy entirely
4. **Is there stale data?** → add cleanup in `beforeEach`
5. **Am I hitting a framework-specific pattern?** → web_search for that framework + Playwright

## Browser Automation Backends

Two approaches for AI-assisted browser exploration and test authoring:

| | **Playwright CLI** (default) | **Test Agents MCP** (optional) |
|---|---|---|
| **Interface** | Bash commands (`playwright-cli open/click/snapshot`) | MCP tool calls (`mcp__playwright-test__*`) |
| **Token cost** | **~27K** — snapshots/screenshots saved to disk | **~114K** — returned inline in context |
| **Best for** | Explore UI, author tests, quick debugging | Planner→Generator→Healer workflow, deep introspection |
| **Setup** | `npm install -g @playwright/cli` | Auto-loaded via skill `mcp.json` |

**Recommendation**: Use CLI as default. Escalate to Test Agents MCP when you need step-by-step replay, auto-healing loops, or locator generation.

→ CLI details: load `playwright-cli` skill (full command reference, test generation, session management, etc.)

## Workflows

### 1. Manual — Write tests directly

Read setup guide → write `.spec.ts` files → run with `npx playwright test`.

### 2. CLI-assisted — Explore → Snapshot → Write tests (recommended)

Token-efficient workflow using Playwright CLI. Load `playwright-cli` skill for full command reference.

1. **Explore** — `playwright-cli open http://localhost:3001 --headed` → `snapshot` → discover element refs
2. **Author** — map snapshot refs to `getByRole()` locators, write `.spec.ts` (see `playwright-cli` skill's test-generation reference)
3. **Run** — `npx playwright test`
4. **Heal** — `playwright-cli open` failing page → `snapshot` → compare with test selectors → fix

### 3. Test Agents MCP — Plan → Generate → Heal

AI-assisted test authoring using Playwright Test Agents MCP server. Higher token cost but richer introspection.

| Agent | Input | Output |
|-------|-------|--------|
| **Planner** | App URL + seed test | Markdown test plan |
| **Generator** | Markdown plan | `.spec.ts` files |
| **Healer** | Failing test name | Fixed/passing test |

→ [references/workflows/test-agents-mcp/planner.md](references/workflows/test-agents-mcp/planner.md) | [generator.md](references/workflows/test-agents-mcp/generator.md) | [healer.md](references/workflows/test-agents-mcp/healer.md)

## Modules

### Base Playwright (default)

Standard Playwright E2E — parallel-friendly, no special constraints.

→ Setup: [references/setup/base-playwright.md](references/setup/base-playwright.md)
→ Templates: [references/templates/base/playwright-config.md](references/templates/base/playwright-config.md) | [seed-spec.md](references/templates/base/seed-spec.md)

### Web3 / Synpress (optional)

Phantom wallet automation via Synpress. Adds constraints: single worker, version pinning (`@playwright/test@1.48.2`), cache-based setup with post-cache fixup pattern.

→ Setup: [references/setup/web3-synpress.md](references/setup/web3-synpress.md) | Env vars: [references/setup/environment-variables.md](references/setup/environment-variables.md)

## References

| Topic | File |
|-------|------|
| **Setup** | |
| Playwright CLI | Load `playwright-cli` skill |
| Base Playwright setup | [setup/base-playwright.md](references/setup/base-playwright.md) |
| Test Agents MCP setup | [setup/test-agents-mcp.md](references/setup/test-agents-mcp.md) |
| Web3/Synpress setup | [setup/web3-synpress.md](references/setup/web3-synpress.md) |
| Environment variables | [setup/environment-variables.md](references/setup/environment-variables.md) |
| webServer integration | [setup/webserver-integration.md](references/setup/webserver-integration.md) |
| **Workflows — Test Agents MCP** | |
| MCP planner | [workflows/test-agents-mcp/planner.md](references/workflows/test-agents-mcp/planner.md) |
| MCP generator | [workflows/test-agents-mcp/generator.md](references/workflows/test-agents-mcp/generator.md) |
| MCP healer | [workflows/test-agents-mcp/healer.md](references/workflows/test-agents-mcp/healer.md) |
| **Templates** | |
| Base config template | [templates/base/playwright-config.md](references/templates/base/playwright-config.md) |
| Base seed test | [templates/base/seed-spec.md](references/templates/base/seed-spec.md) |
| Web3 templates | [templates/web3/](references/templates/web3/) (phantom-setup, fixture, download, testnet enabler, SIWE spec) |
| **Troubleshooting** | |
| Web3/Phantom issues | [troubleshooting/web3-phantom.md](references/troubleshooting/web3-phantom.md) |
| webServer orphans | [troubleshooting/webserver-orphans.md](references/troubleshooting/webserver-orphans.md) |
