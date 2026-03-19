# webServer Orphan Processes

## Problem

After Playwright E2E tests finish, dev server ports remain occupied (EADDRINUSE), blocking subsequent runs.

## Root Cause

Using process orchestrators (turbo, nx, lerna) as Playwright `webServer.command` creates independent process groups that escape Playwright's cleanup.

```
Broken:  Playwright → shell → turbo → actual server (orphaned)
Fixed:   Playwright → shell → actual server (killed on cleanup)
```

## Fix

Run each app's own dev script directly with absolute `cwd`:

```ts
webServer: [
  {
    command: "<your-dev-command>",
    port: 3000,
    cwd: path.resolve(__dirname, "<server-app-dir>"),
    reuseExistingServer: !process.env.CI,
  },
]
```

See [setup/webserver-integration.md](../setup/webserver-integration.md) for the full config pattern.

## Deep Dive

For full root cause analysis (process groups, why `gracefulShutdown` is unavailable with Synpress, contributing factors), see `cyberk-flow/knowledge/playwright-webserver-orphan-process.md`.
