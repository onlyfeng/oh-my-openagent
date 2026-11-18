# device-session — Device Account Resolution for Memory Sync

**Generated:** 2026-11-18 / c2691818a9

**Score:** 16 (18 TS files, ~3.3k LOC; distinct domain: which signed-in device account memory sync binds to). The parent [`memory/AGENTS.md`](../AGENTS.md) carries the one-row sync context.

## OVERVIEW

Resolves the account a memory-sync-bound session acts as: the CLI credential store (`omo` service-auth) or the desktop sync account, modeled as a discriminated union with an explicit `unavailable` kind. The heavy runtime (credential store + session refresh) loads lazily from the `#omo-device-account-runtime` sidecar (`extensions/omo-device-account.js` when bundled); the source selection itself is built synchronously.

## WHERE TO LOOK

| Task | File |
|------|------|
| Account model + CLI source | `account-source.ts` — `DeviceAccount = CliDeviceAccount \| DesktopSyncAccount \| DesktopSyncUnavailable`, `DeviceAccountSource`, `createCliAccountSource`, `DEFAULT_SERVICE_API`, `KeyringFactory`, `HeldCredentialRelay` |
| Selection policy | `selected-account-source.ts` — `accountSourceFromSelection`, `selectedDeviceAccountSource` |
| Resolver | `resolver.ts` — `createDeviceSessionResolver`, `DeviceSessionRuntime` |
| Lazy sidecar load | `account-runtime.ts` + `account-runtime-loader.ts` (source built once from the loaded sidecar; a failed load is not cached) |
| CLI operations | `cli-account-operation.ts` |
| Behavior pins | `relay-readopt.test.ts`, `relay-readopt-late-service.test.ts`, `revoke-session.test.ts`, `first-session-repo.test.ts`, `keychain-pending.test.ts`, `desktop-account.test.ts`, `sibling-writer.fixture.ts` |

## CONVENTIONS

- Structural ports only; no concrete host types leak in.
- Credential generation + currentness checks surround async operations; private keys stay in credential storage except keyring material.
- A desktop-unavailable selection PRESERVES its reason and must NOT fall through to CLI credential access.

## ANTI-PATTERNS

- Never accept a missing credential generation as a wildcard.
- Never persist replies for a different generation.
- Do not read the CLI credential store from the desktop path.

Parent: [`../AGENTS.md`](../AGENTS.md).
