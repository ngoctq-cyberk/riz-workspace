# phantom.setup.ts

Minimal wallet import only. Testnet mode is configured by [post-cache fixup](../../setup/web3-synpress.md#architecture-post-cache-fixup).

```ts
import { defineWalletSetup } from "@synthetixio/synpress";
import { Phantom } from "@synthetixio/synpress/playwright";
import { requireEnv } from "../fixtures/required-env";

const walletPassword = requireEnv("E2E_WALLET_PASSWORD");
const seedPhrase = requireEnv("E2E_WALLET_SEED_PHRASE");

export default defineWalletSetup(walletPassword, async (context, walletPage) => {
  const phantom = new Phantom(context, walletPage, walletPassword);
  await phantom.importWallet(seedPhrase);
});
```
