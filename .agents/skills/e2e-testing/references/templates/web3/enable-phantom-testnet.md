# enable-phantom-testnet.ts

Post-cache fixup: launches cached Synpress profile and enables Phantom's "Testnet Mode" by directly modifying `chrome.storage.local` via the extension's service worker. See [architecture rationale](../../setup/web3-synpress.md#architecture-post-cache-fixup).

## How it works

Instead of automating the UI (Settings → Developer Settings → Testnet Mode toggle), this script uses Playwright's `serviceWorker.evaluate()` to call `chrome.storage.local.get/set` directly in Phantom's extension context.

**Storage location:**
- Key: `mmkv:items:localStorage` (Phantom's MMKV storage blob)
- Sub-key: `accounts:developerMode`
- Value: `{ value: { isDeveloperMode: true, version: 1 }, version: 0 }`

**Advantages over UI automation:**
- No password unlock needed
- No CSS/data-testid selectors that break on Phantom updates
- No onboarding dismissal, no menu navigation
- Read-after-write verification ensures correctness

## Important: Internal schema dependency

This relies on Phantom's internal MMKV storage schema. If Phantom changes the key structure, this script will fail with a diagnostic error that dumps available storage keys.

**To rediscover the key if it breaks:**
1. Launch browser with Phantom extension via `chromium.launchPersistentContext`
2. Get the service worker via `context.serviceWorkers()`
3. Run `sw.evaluate(() => chrome.storage.local.get(null, console.log))`
4. Search for `developerMode`, `testnet`, or `testMode` in keys/values

## Template

```ts
import fs from "node:fs";
import path from "node:path";
import { type BrowserContext, chromium, type Worker } from "@playwright/test";

// chrome.storage API types (available inside extension service worker evaluate context)
declare namespace chrome {
  namespace storage {
    namespace local {
      function get(key: string | null, callback: (items: Record<string, unknown>) => void): void;
      function set(items: Record<string, unknown>, callback: () => void): void;
    }
  }
  namespace runtime {
    const lastError: { message: string } | undefined;
  }
}

const STORAGE_KEY = "mmkv:items:localStorage";
const DEV_MODE_SUBKEY = "accounts:developerMode";

async function getPhantomServiceWorker(context: BrowserContext): Promise<Worker> {
  const extensionWorkers = context
    .serviceWorkers()
    .filter((sw) => sw.url().includes("chrome-extension://"));

  if (extensionWorkers.length === 1) return extensionWorkers[0] as Worker;

  if (extensionWorkers.length > 1) {
    const urls = extensionWorkers.map((sw) => sw.url()).join(", ");
    throw new Error(`Expected 1 extension service worker, found ${extensionWorkers.length}: ${urls}`);
  }

  const sw = await context.waitForEvent("serviceworker", {
    timeout: 10_000,
    predicate: (w: Worker) => w.url().includes("chrome-extension://"),
  });
  return sw as Worker;
}

async function main() {
  const cacheDir = path.join(process.cwd(), ".cache-synpress");
  const dirs = fs.readdirSync(cacheDir).filter((d) => {
    const full = path.join(cacheDir, d);
    return fs.statSync(full).isDirectory() && d !== "phantom-chrome-latest";
  });

  for (const dir of dirs) {
    const userDataDir = path.join(cacheDir, dir);
    console.log(`Enabling testnet for cache: ${dir}`);

    const context = await chromium.launchPersistentContext(userDataDir, {
      headless: false,
      args: [
        `--disable-extensions-except=${path.join(cacheDir, "phantom-chrome-latest")}`,
        `--load-extension=${path.join(cacheDir, "phantom-chrome-latest")}`,
      ],
    });

    try {
      const sw = await getPhantomServiceWorker(context);

      const result = await sw.evaluate(
        async ({ storageKey, devModeSubkey }) => {
          const getLocal = (key: string) =>
            new Promise<unknown>((resolve, reject) => {
              chrome.storage.local.get(key, (items) => {
                if (chrome.runtime.lastError)
                  return reject(new Error(chrome.runtime.lastError.message));
                resolve((items as Record<string, unknown>)[key]);
              });
            });

          const setLocal = (obj: Record<string, unknown>) =>
            new Promise<void>((resolve, reject) => {
              chrome.storage.local.set(obj, () => {
                if (chrome.runtime.lastError)
                  return reject(new Error(chrome.runtime.lastError.message));
                resolve();
              });
            });

          const getAllKeys = () =>
            new Promise<string[]>((resolve, reject) => {
              chrome.storage.local.get(null, (items) => {
                if (chrome.runtime.lastError)
                  return reject(new Error(chrome.runtime.lastError.message));
                resolve(Object.keys(items));
              });
            });

          // Wait for MMKV blob to be initialized by Phantom
          const deadline = Date.now() + 10_000;
          let blob: unknown;
          while (Date.now() < deadline) {
            blob = await getLocal(storageKey);
            if (blob === undefined || blob === null) {
              await new Promise((r) => setTimeout(r, 200));
              continue;
            }
            if (typeof blob === "object" && !Array.isArray(blob)) break;
            throw new Error(
              `MMKV blob "${storageKey}" has unexpected type: ${typeof blob}`,
            );
          }
          if (!blob || typeof blob !== "object") {
            const allKeys = await getAllKeys();
            throw new Error(
              `MMKV blob "${storageKey}" not found after 10s. Available keys: ${allKeys.join(", ")}`,
            );
          }

          const record = blob as Record<string, unknown>;
          const existing = record[devModeSubkey] as
            | { value?: { isDeveloperMode?: boolean } }
            | undefined;
          const before = existing?.value?.isDeveloperMode ?? false;

          record[devModeSubkey] = { value: { isDeveloperMode: true, version: 1 }, version: 0 };
          await setLocal({ [storageKey]: record });

          // Read-after-write verification
          const verified = await getLocal(storageKey);
          const verifiedBlob =
            verified && typeof verified === "object" ? (verified as Record<string, unknown>) : undefined;
          const after =
            (verifiedBlob?.[devModeSubkey] as { value?: { isDeveloperMode?: boolean } } | undefined)?.value
              ?.isDeveloperMode === true;
          if (!after) {
            throw new Error("Write verification failed: accounts:developerMode not enabled after set");
          }

          return { before, after };
        },
        { storageKey: STORAGE_KEY, devModeSubkey: DEV_MODE_SUBKEY },
      );

      console.log(`  Developer mode: ${result.before} → ${result.after}`);
      await new Promise((r) => setTimeout(r, 2000));
    } finally {
      await context.close();
    }
    console.log("  Done.");
  }
}

if (import.meta.main) {
  main().catch((err: unknown) => {
    console.error(err);
    process.exitCode = 1;
  });
}
```
