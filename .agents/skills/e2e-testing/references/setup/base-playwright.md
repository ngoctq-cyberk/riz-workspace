# Base Playwright Setup

Standard Playwright E2E testing — no Web3/wallet dependencies.

## Step 1: Install Dependencies

```bash
# Use your package manager (npm, pnpm, yarn, bun)
<pm> add -D @playwright/test
<pm> add -D dotenv-cli          # optional, for env var loading

# Install browser binaries
npx playwright install chromium
```

## Step 2: Add Scripts

Add to `package.json` scripts:

```json
{
  "scripts": {
    "e2e:test": "playwright test",
    "e2e:test:headed": "playwright test --headed"
  }
}
```

## Step 3: Playwright Config

Create `playwright.config.ts`. See [templates/base/playwright-config.md](../templates/base/playwright-config.md) for the full template.

Key points:
- Set `testDir` to your E2E specs directory
- Configure `webServer` to auto-start dev servers (see [webserver-integration.md](webserver-integration.md))
- Disable traces/videos in CI (`process.env.CI ? "off" : "retain-on-failure"`)

## Step 4: Directory Structure

```
e2e/
├── fixtures/
│   └── base.fixture.ts       # Custom fixtures (if needed)
├── plans/
│   └── <feature>.md          # Test plans (from planner or manual)
└── specs/
    ├── seed.spec.ts           # Seed test for MCP agent workflows
    └── <feature>.spec.ts      # Test specs
```

## Step 5: Create Seed Test

The seed test bootstraps the environment for MCP agent workflows:

```ts
import { test, expect } from "@playwright/test";

test("seed", async ({ page }) => {
  await page.goto("/");
  await expect(page.locator("body")).toBeVisible();
});
```

See [templates/base/seed-spec.md](../templates/base/seed-spec.md) for details.

## Step 6: Run Tests

```bash
# Run all E2E tests
<pm> run e2e:test

# Run specific test
npx playwright test e2e/specs/my-feature.spec.ts

# Debug mode
PWDEBUG=1 npx playwright test e2e/specs/my-feature.spec.ts
```

## Debugging

| Method | Command |
|--------|---------|
| Inspector | `PWDEBUG=1 playwright test <spec>` |
| Headed | `playwright test --headed` |
| Trace viewer | `playwright show-trace test-results/<test>/trace.zip` |
| HTML report | `playwright show-report` |
