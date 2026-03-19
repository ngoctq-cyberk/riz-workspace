# Troubleshooting: Web3 / Phantom

## `dotenv` — "Invalid value for '-e'"

**Error**: `Error: Invalid value for '-e' / '--export': '.env.e2e.local' is not a valid boolean.`

**Cause**: System-level `dotenv` binary (e.g., Python dotenv) conflicts with `dotenv-cli` npm package.

**Fix**: Use `dotenv-cli` (installed as devDependency) in scripts, never bare `dotenv`:
```json
"e2e:test": "dotenv-cli -e .env.e2e.local -- playwright test"
```

## Extension page detection timeout

**Error**: `TimeoutError: Timeout 30000ms exceeded while waiting for event "page"`

**Cause**: Phantom loads as a service worker, not a visible page. `context.waitForEvent("page")` never fires.

**Fix**: Discover extension ID via service workers, then open popup directly:
```ts
async function getExtensionId(context: BrowserContext): Promise<string> {
  for (const sw of context.serviceWorkers()) {
    const match = sw.url().match(/chrome-extension:\/\/([^/]+)/);
    if (match) return match[1];
  }
  for (const p of context.backgroundPages()) {
    const match = p.url().match(/chrome-extension:\/\/([^/]+)/);
    if (match) return match[1];
  }
  const sw = await context.waitForEvent("serviceworker", { timeout: 10_000 });
  const match = sw.url().match(/chrome-extension:\/\/([^/]+)/);
  if (!match) throw new Error("Could not detect Phantom extension ID");
  return match[1];
}

// Open popup directly instead of waiting for page event
const extensionId = await getExtensionId(context);
const page = await context.newPage();
await page.goto(`chrome-extension://${extensionId}/popup.html`);
```

## Phantom testnet enable script fails after Phantom update

**Error**: `MMKV blob "mmkv:items:localStorage" not found after 10s. Available keys: ...`
or `Write verification failed: accounts:developerMode not enabled after set`

**Cause**: Phantom changed its internal MMKV storage schema in a new version.

**Fix**: Rediscover the storage key:
1. Launch browser with Phantom extension via `chromium.launchPersistentContext`
2. Get the service worker: `const sw = context.serviceWorkers().find(w => w.url().includes("chrome-extension://"))`
3. Dump all storage: `await sw.evaluate(() => chrome.storage.local.get(null, console.log))`
4. Search for `developerMode`, `testnet`, or `testMode` in keys/values
5. Update `STORAGE_KEY` and `DEV_MODE_SUBKEY` constants in `enable-phantom-testnet.ts`

## Phantom UI interaction timing failures

**Error**: `TimeoutError: click: Timeout exceeded waiting for locator('[data-testid="settings-menu-open-button"]')`

**Cause**: Script interacts with wallet UI before unlock completes or DOM stabilizes.

**Fix**: Add `waitForTimeout` (500–1000ms) between critical navigation steps:
```ts
await phantomPage.getByRole("button", { name: "Unlock" }).click();
await phantomPage.waitForTimeout(1000);  // wait for unlock to complete

await phantomPage.locator('[data-testid="settings-menu-open-button"]').click();
await phantomPage.waitForTimeout(500);   // wait for menu animation
```
