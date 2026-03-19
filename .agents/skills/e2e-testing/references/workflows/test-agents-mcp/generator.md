# Playwright Test Generator

**Goal**: Turn a Markdown test plan into executable `.spec.ts` files.

## Workflow

1. Read the test plan markdown file
2. For each scenario:
   a. Call `mcp__playwright-test__generator_setup_page` to initialize browser
   b. Execute each step using `browser_*` MCP tools to verify it works live
   c. Call `mcp__playwright-test__generator_read_log` to get recorded actions
   d. Call `mcp__playwright-test__generator_write_test` with the generated source code

## Generated File Format

```ts
// spec: e2e/plans/<plan-name>.md
// seed: e2e/specs/seed.spec.ts

import { test, expect } from "@playwright/test";

test.describe("Feature Name", () => {
  test("Scenario Name", async ({ page }) => {
    // 1. Navigate to /page
    await page.goto("/page");

    // 2. Click "Button Name"
    await page.getByRole("button", { name: "Button Name" }).click();

    // 3. Verify result
    await expect(page.getByText("Success")).toBeVisible();
  });
});
```

### Web3 Variant

For Web3 tests, import from Synpress fixture instead of vanilla Playwright. Adjust the relative import path based on spec file location:

```ts
// For e2e/specs/web3/<file>.spec.ts → use "../../fixtures/..."
import { expect, test } from "../../fixtures/phantom.fixture";

test.describe("Web3 Feature", () => {
  test("Scenario Name", async ({ page, phantom }) => {
    await page.goto("/page");
    await page.getByRole("button", { name: "Sign" }).click();
    await phantom.confirmSignature();
    await expect(page.getByText("Signed")).toBeVisible();
  });
});
```

## Rules

- One test per file, file name = fs-friendly scenario name
- Test wrapped in `describe` matching the top-level plan item
- Comment before each step (don't duplicate for multi-action steps)
- Use Playwright best practices from the generator log

## Available MCP Tools

- `generator_setup_page` — initialize browser for scenario
- `generator_read_log` — retrieve recorded actions
- `generator_write_test` — save generated test file
- `browser_snapshot` / `browser_click` / `browser_type` — interact and verify
- `browser_verify_element_visible` / `browser_verify_text_visible` / `browser_verify_value` — assertions
- `browser_verify_list_visible` — verify list content
- `browser_wait_for` — wait for conditions
