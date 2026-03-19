# Playwright Web3 Config Template

Config with Synpress constraints applied. Use as a separate config or merge into your main config.

```ts
import path from "node:path";
import { fileURLToPath } from "node:url";
import { defineConfig } from "@playwright/test";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  testDir: "./e2e/specs",
  timeout: 90_000,
  fullyParallel: false,       // Required: wallet extension shares state
  workers: 1,                 // Required: single browser context
  retries: 0,
  reporter: [["html", { outputFolder: "playwright-report", open: "never" }], ["list"]],
  use: {
    baseURL: process.env.E2E_BASE_URL ?? "http://localhost:3001",
    trace: process.env.CI ? "off" : "retain-on-failure",
    screenshot: "only-on-failure",
    video: process.env.CI ? "off" : "retain-on-failure",
  },
  webServer: [
    {
      command: "<your-dev-command>",
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

## Key Differences from Base Config

| Setting | Base | Web3 |
|---------|------|------|
| `timeout` | 30s | 90s |
| `fullyParallel` | `true` | `false` |
| `workers` | auto | `1` |
| `retries` | CI: 2 | `0` |
