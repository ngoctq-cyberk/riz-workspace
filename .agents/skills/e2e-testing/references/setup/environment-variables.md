# Environment Variables

Stored in `.env.e2e.local` (gitignored). Create `.env.e2e.example` (committed) as schema reference.

| Variable | Required | Description |
|----------|----------|-------------|
| `E2E_WALLET_PASSWORD` | Web3 | Password to encrypt/unlock the wallet vault |
| `E2E_WALLET_SEED_PHRASE` | Web3 | 12-word recovery phrase for the test wallet |

## `.env.e2e.example` Template

```bash
E2E_WALLET_PASSWORD="[REDACTED:password]"
E2E_WALLET_SEED_PHRASE="test test test test test test test test test test test junk"
```

## Loading

Uses `dotenv-cli` (installed as devDependency) — all e2e scripts prefix with `dotenv-cli -e .env.e2e.local --`. Use `dotenv-cli` (not bare `dotenv`) to avoid conflicts with system-level `dotenv` utilities. See [web3-synpress.md](web3-synpress.md#step-2-scripts) for full script definitions.

Validation: use `requireEnv()` helper (see [required-env.md](../templates/web3/required-env.md)) to fail fast on missing vars.

## Gitignore

```
.cache-synpress
.env.e2e.local
```
