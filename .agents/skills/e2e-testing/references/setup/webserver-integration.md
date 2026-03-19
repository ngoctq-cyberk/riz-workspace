# webServer Integration

How to configure Playwright's `webServer` to auto-start dev servers during tests.

## The Rule

**Never use process orchestrators (turbo, nx, lerna) as Playwright `webServer.command`.**

They create independent process groups that escape Playwright's cleanup, leaving orphaned processes bound to ports (EADDRINUSE).

## Correct Pattern

Run each app's own dev script directly with absolute `cwd`:

```ts
import path from "node:path";
import { fileURLToPath } from "node:url";
import { defineConfig } from "@playwright/test";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  webServer: [
    {
      command: "<your-dev-command>",         // e.g., "npm run dev", "bun run dev"
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
});
```

## Why This Matters

| Config | Process tree | Cleanup |
|--------|-------------|---------|
| **Broken**: `turbo dev` | Playwright → shell → turbo → server | Turbo's children orphaned |
| **Fixed**: direct command | Playwright → shell → server | All killed on cleanup |

## Best Practices

| Practice | Rationale |
|----------|-----------|
| Absolute `cwd` via `path.resolve(__dirname, ...)` | Works when Playwright runs from any directory |
| `reuseExistingServer: !process.env.CI` | CI fails fast on leaked processes; local reuses |
| App's own `dev` script | Avoids `npx`/`bunx` resolution issues |

## Deep Dive

For full root cause analysis, see `cyberk-flow/knowledge/playwright-webserver-orphan-process.md`.
