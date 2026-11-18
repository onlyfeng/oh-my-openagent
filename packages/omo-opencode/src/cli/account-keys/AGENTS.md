# src/cli/account-keys — OS Keyring Account Keys

**Generated:** 2026-11-18 | c2691818a9

`omo account-keys` CLI: stores per-account private key material into the OS keyring via the platform's native backend. Distinct domain: the only place private-key bytes cross a process boundary in this package.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `commands.ts` | `configureAccountKeysCommands` — Commander wiring. |
| `keystore.ts` | `createOsKeyStore` — the public keystore factory. Storage is idempotent; re-adding the same account overwrites. |
| `keyring.ts` | Keyring record constants and dependency-injection contracts for the platform backend. |
| `account-keys.test.ts` | bun test coverage. |

## ANTI-PATTERNS

- Secret bytes are passed to CLI backends via **stdin**, never argv. A leak via `ps` would expose the keyring.
- Private key material is never logged or returned from a keystore call.
- Re-adding the same account is idempotent (overwrites); do not introduce a separate "update" path.

## CONVENTIONS

- The keystore is constructed via `createOsKeyStore` with the platform backend injected through `keyring.ts` — never hard-code a backend call inside `commands.ts`.

## COMMANDS

```bash
bun test packages/omo-opencode/src/cli/account-keys
```

Changes here are OpenCode-wired — run the `opencode-qa` skill and record evidence under `.omo/evidence/` per the root AGENTS.md mandate.
