# seed.spec.ts

Seed test that bootstraps the environment for MCP agent workflows. The planner/generator agents run this test first to set up fixtures, hooks, and page context.

```ts
import { test, expect } from "@playwright/test";

test("seed", async ({ page }) => {
  await page.goto("/");
  await expect(page.locator("body")).toBeVisible();
});
```

## Notes

- The seed test should import from your project's custom fixture if you have one
- For Web3 projects, create a separate seed that imports from `phantom.fixture.ts`
- Keep the seed test minimal — it's a bootstrap, not a real test
