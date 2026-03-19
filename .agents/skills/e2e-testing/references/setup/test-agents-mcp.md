# Test Agents MCP Setup

The `playwright-test` MCP server is **auto-loaded by Amp** when the `e2e-testing` skill is invoked (via `mcp.json` next to `SKILL.md`). No manual `.mcp.json` configuration needed.

## Prerequisites

- `@playwright/test` installed in the project (see [base-playwright.md](base-playwright.md))
- Browsers installed (`npx playwright install chromium`)
- `npx playwright` must be available in the project context

## What gets loaded

When the skill is invoked, Amp starts `npx playwright run-test-mcp-server` and exposes tools prefixed `mcp__playwright-test__`. See `mcp.json` for the full allowlist.

## CLI options

The `run-test-mcp-server` command accepts: `--headless`, `-c/--config <file>`, `--host <host>`, `--port <port>`.

To customize, edit `mcp.json` args, e.g.:

```json
{
  "playwright-test": {
    "command": "npx",
    "args": ["playwright", "run-test-mcp-server", "--config", "e2e/playwright.config.ts"]
  }
}
```

## Workflows

1. **Planner** → [workflows/test-agents-mcp/planner.md](../workflows/test-agents-mcp/planner.md)
2. **Generator** → [workflows/test-agents-mcp/generator.md](../workflows/test-agents-mcp/generator.md)
3. **Healer** → [workflows/test-agents-mcp/healer.md](../workflows/test-agents-mcp/healer.md)
