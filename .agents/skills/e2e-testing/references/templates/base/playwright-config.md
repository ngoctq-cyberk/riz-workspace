# Base Playwright Config Template

```ts
import path from "node:path";
import { fileURLToPath } from "node:url";
import { defineConfig } from "@playwright/test";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  testDir: "./e2e/specs",
  timeout: 30_000,
  fullyParallel: true,
  workers: undefined, // auto-detect
  retries: process.env.CI ? 2 : 0,
  reporter: [["html", { outputFolder: "playwright-report", open: "never" }], ["list"]],
  use: {
    baseURL: process.env.E2E_BASE_URL ?? "http://localhost:3001",
    trace: process.env.CI ? "off" : "retain-on-failure",
    screenshot: "only-on-failure",
    video: process.env.CI ? "off" : "retain-on-failure",
  },
  webServer: [
    {
      command: "<your-dev-command>",   // e.g., "npm run dev", "bun run dev"
      port: 3000,
      cwd: path.resolve(__dirname, "<server-app-dir>"),
      reuseExistingServer: !process.env.CI,
    },
    {
      command: "<your-dev-command>",
      port: 3001,
      cwd: path.resolve(__dirname, "<web-app-dir>"),
      reuseExistingServer: !process.env.CI,
    },
  ],
  projects: [{ name: "chromium" }],
});
```

## Notes

- Replace `<your-dev-command>` and `<*-app-dir>` with your project values
- See [webserver-integration.md](../../setup/webserver-integration.md) for webServer rules
- For Web3 tests, see [web3 config template](../web3/playwright-web3-config.md)
