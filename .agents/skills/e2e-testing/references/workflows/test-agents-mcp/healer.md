# Playwright Test Healer

**Goal**: Debug and fix failing Playwright tests automatically.

## Workflow

1. Run all tests with `mcp__playwright-test__test_run` to identify failures
2. For each failing test:
   a. Run `mcp__playwright-test__test_debug` to replay with breakpoints
   b. Use `browser_snapshot` to inspect current UI state
   c. Use `browser_console_messages` / `browser_network_requests` for debugging
   d. Identify root cause (selector change, timing, data dependency, app change)
   e. Edit the test file to fix the issue
   f. Re-run the test to verify the fix
3. Repeat until all tests pass

## Root Cause Categories

| Category            | Symptoms                      | Fix                                                    |
| ------------------- | ----------------------------- | ------------------------------------------------------ |
| Selector change     | Element not found             | Use `browser_generate_locator` to get updated selector |
| Timing issue        | Intermittent failures         | Add proper waits, avoid `networkidle`                  |
| Data dependency     | State from previous test      | Make test independent                                  |
| App behavior change | Assertion mismatch            | Update expected values                                 |
| Dynamic content     | Locator matches wrong element | Use regex-based resilient locators                     |

## Rules

- Fix one error at a time, re-test after each fix
- Prefer robust solutions over quick hacks
- If test is correct but app behavior changed, mark as `test.fixme()` with explanation
- Never use `networkidle` or deprecated APIs
- Document reasoning for each fix
- Continue until all tests pass, time-box triggers, or suite budget exceeded — then stop and summarize

## Time-Boxing (MANDATORY)

⚠️ **Follow SKILL.md → "Escalation & Time-Boxing" section exactly.** All budget limits, the escalation ladder, mandatory budget log, and anti-patterns defined there apply to this workflow. Do NOT exceed 2 retry cycles per test or 5 minutes per healer session.

## Available MCP Tools

- `test_run` — run tests, get results
- `test_list` — list available tests
- `test_debug` — debug failing test with breakpoints
- `browser_snapshot` — inspect current page state
- `browser_evaluate` — run JS to inspect state
- `browser_generate_locator` — generate robust locator for element
- `browser_console_messages` — check for errors
- `browser_network_requests` — check API calls
