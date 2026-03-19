# Web3 / Synpress Setup (Phantom Wallet)

Phantom wallet automation via Synpress v4. Optional module — only needed for Web3 wallet flows.

> **Why Phantom?** MetaMask v13 (MV3) has critical Synpress issues: service worker idle before persistence, regex limits, hash sensitivity, LavaMoat scuttling.

## Prerequisites

- Base Playwright setup completed (see [base-playwright.md](base-playwright.md))
- Dedicated test wallet with no real funds, testnet only

## Constraints

| Constraint | Reason |
|------------|--------|
| `workers: 1` | Wallet extension shares browser state |
| `fullyParallel: false` | Single browser context required |
| `@playwright/test@1.48.2` | Pinned due to `@synthetixio/synpress-cache` peer dependency |

**Always pin exact versions** — no caret (`^`). Upgrade Playwright + Synpress together and rebuild cache.

## Step 1: Install

```bash
bun add -D @synthetixio/synpress@4.1.2 dotenv-cli
```

## Step 2: Scripts

Pipeline: download CRX → build cache → enable testnet → run tests.

```json
{
  "e2e:download-phantom": "bun run e2e/scripts/download-phantom.ts",
  "e2e:enable-testnet": "dotenv-cli -e .env.e2e.local -- bun run e2e/scripts/enable-phantom-testnet.ts",
  "e2e:cache": "bun run e2e:download-phantom && dotenv-cli -e .env.e2e.local -- synpress e2e/wallet-setup --headless --phantom && bun run e2e:enable-testnet",
  "e2e:cache:force": "bun run e2e:download-phantom && dotenv-cli -e .env.e2e.local -- synpress e2e/wallet-setup --headless --phantom --force && bun run e2e:enable-testnet",
  "e2e:test:siwe": "dotenv-cli -e .env.e2e.local -- playwright test e2e/specs/siwe-sign-in.spec.ts"
}
```

## Step 3: Environment Variables

→ See [environment-variables.md](environment-variables.md) for full reference and `.env.e2e.example` template.

## Step 4: Create Files

See [templates/web3/](../templates/web3/) for all template files:

```
e2e/
├── fixtures/
│   ├── phantom.fixture.ts       # → phantom-fixture.md
│   └── required-env.ts          # → required-env.md
├── scripts/
│   ├── download-phantom.ts      # → download-phantom.md
│   └── enable-phantom-testnet.ts  # → enable-phantom-testnet.md
├── wallet-setup/
│   └── phantom.setup.ts         # → phantom-setup.md
└── specs/
    └── siwe-sign-in.spec.ts     # → siwe-sign-in-spec.md
```

## Step 5: Build Cache and Run

```bash
bun run dev          # start dev servers first
bun run e2e:cache    # build wallet cache (one-time)
bun run e2e:test:siwe
```

## Architecture: Post-Cache Fixup

Two-phase approach:

1. **Cache build** — `defineWalletSetup` imports wallet only (minimal → stable cache hash)
2. **Post-cache fixup** — enables testnet mode via direct `chrome.storage.local` manipulation (no UI automation)

The fixup script uses Playwright's `serviceWorker.evaluate()` to write directly to Phantom's internal MMKV storage (`mmkv:items:localStorage` → `accounts:developerMode`). This avoids brittle UI selectors that break on Phantom updates.

This also avoids Synpress's `extractWalletSetupFunction` regex limits (3 levels of brace nesting) and keeps the cache hash stable across code changes.

## Wallet UI Adapting

`connectToDapp()` only **approves** the permission popup — you must click the wallet button in your app's UI first.

| Library | Connect Flow |
|---------|-------------|
| RainbowKit | `button "Connect Wallet"` → dialog → `button "Phantom"` |
| ConnectKit | `button "Connect"` → modal → `button "Phantom"` |
| Web3Modal | `w3m-connect-button` → `w3m-wallet-button[name="Phantom"]` |

## Cache Lifecycle

Hash-based: changing `phantom.setup.ts` generates a new hash. Rebuild with `e2e:cache:force` or `rm -rf .cache-synpress`.
