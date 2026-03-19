# siwe-sign-in.spec.ts

Example SIWE happy-path test using Phantom wallet.

```ts
import { expect, test } from "../fixtures/phantom.fixture";

test("sign in with Phantom via SIWE", async ({ page, phantom }) => {
  await page.goto("/sign-in");

  // Connect wallet via RainbowKit
  const connectWalletButton = page.getByRole("button", { name: "Connect Wallet" });
  await expect(connectWalletButton).toBeVisible();
  await connectWalletButton.click();

  const rainbowKitDialog = page.getByRole("dialog");
  await rainbowKitDialog.waitFor();

  await page.getByRole("button", { name: "Phantom" }).click();
  await phantom.connectToDapp();

  await rainbowKitDialog.waitFor({ state: "hidden" });

  // Sign SIWE message
  const signInButton = page.getByRole("button", { name: "Sign in with Wallet" });
  await expect(signInButton).toBeVisible();
  await signInButton.click();
  await phantom.confirmSignature();

  // Assert redirect to authenticated route
  await page.waitForURL((url) => !url.pathname.startsWith("/sign-in"));
  await expect(page).toHaveURL(/\/(mint)?$/);
});
```
