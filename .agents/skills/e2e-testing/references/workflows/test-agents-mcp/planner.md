# Playwright Test Planner

**Goal**: Explore the app via browser and produce a Markdown test plan.

## Workflow

1. Call `mcp__playwright-test__planner_setup_page` to initialize the browser
2. Use `mcp__playwright-test__browser_snapshot` to explore the current page state
3. Navigate with `browser_navigate`, `browser_click`, `browser_type` to discover all interactive elements
4. Map out user flows, identify critical paths, edge cases, and error scenarios
5. Save the plan with `mcp__playwright-test__planner_save_plan`

## Test Plan Format

```markdown
### 1. Feature Name
**Seed:** `e2e/specs/seed.spec.ts`

#### 1.1 Scenario Name
**Steps:**
1. Navigate to /page
2. Click "Button Name"
3. Fill input with "value"
**Expected:** Success message appears, redirect to /dashboard
```

## Quality Standards

- Each scenario: clear title, step-by-step instructions, expected outcomes
- Assume fresh/blank state for every scenario
- Include happy path, edge cases, error handling, negative tests
- Scenarios must be independent and runnable in any order
- Steps must be specific enough for any tester to follow

## Available MCP Tools

- `planner_setup_page` — initialize browser for exploration
- `planner_save_plan` — save the completed test plan
- `browser_snapshot` — get accessibility tree of current page
- `browser_navigate` / `browser_navigate_back` — page navigation
- `browser_click` / `browser_hover` / `browser_type` — user interactions
- `browser_press_key` / `browser_select_option` — keyboard/form inputs
- `browser_evaluate` / `browser_run_code` — execute JS in page
- `browser_console_messages` / `browser_network_requests` — debug info
- `browser_take_screenshot` — visual capture (use sparingly)
- `browser_drag` / `browser_file_upload` / `browser_handle_dialog` — advanced interactions
