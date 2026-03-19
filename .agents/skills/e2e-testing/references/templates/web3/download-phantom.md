# download-phantom.ts

Downloads Phantom CRX from Chrome Web Store. Required because Synpress's built-in URL (`crx-backup.phantom.dev`) is broken. Synpress extracts this `.crx` into `.cache-synpress/phantom-chrome-latest/` (unpacked dir) during cache build.

```ts
import fs from "node:fs";
import path from "node:path";

const PHANTOM_EXTENSION_ID = "bfnaelmomeimhlpmgjnjophhpkkoljpa";
const CHROME_UPDATE_URL = `https://clients2.google.com/service/update2/crx?response=redirect&prodversion=120.0.0.0&acceptformat=crx2,crx3&x=id%3D${PHANTOM_EXTENSION_ID}%26uc`;
const CACHE_DIR = path.join(process.cwd(), ".cache-synpress");
const OUTPUT_PATH = path.join(CACHE_DIR, "phantom-chrome-latest.crx");

async function main() {
  if (fs.existsSync(OUTPUT_PATH)) {
    return;
  }

  fs.mkdirSync(CACHE_DIR, { recursive: true });

  const response = await fetch(CHROME_UPDATE_URL, { redirect: "follow" });
  if (!response.ok) {
    throw new Error(`Failed to download Phantom CRX: ${response.status} ${response.statusText}`);
  }

  const buffer = await response.arrayBuffer();
  fs.writeFileSync(OUTPUT_PATH, Buffer.from(buffer));
}

if (import.meta.main) {
  main();
}
```
