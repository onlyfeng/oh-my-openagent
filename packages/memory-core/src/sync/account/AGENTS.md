# src/sync/account/ — Account Lifecycle + Pairing Ceremonies

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 14 (51 files, ~6.6k LOC, barrel, distinct domain: account lifecycle over the crypto chain)

## OVERVIEW

The signed-in account's lifecycle on this device: enable, join/pair, approve, revoke, freeze — every step verified through the pinned trust anchor and the client chain from `../crypto/`. HTTP is a thin untrusted transport; correctness lives in the ceremony code.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | Exposes `approveDevice`, `deviceKeyringFromMaterial`, `enableSync`, `joinAccount`, `createAccountHttp`, `createPairingHttp`, `SyncOperationAuthority`, genesis-bridge types |
| Verified chain view | `account-chain.ts` | This device's verified account sequence: starts at the pinned anchor, walks served rows, holds rows after a refusal or unreadable row as pending; epoch keys open only from chain-minted wraps |
| Enable | `enable.ts`, `enable-phrase.ts` (+ gates/floors/switches tests) | Enable ceremony; phrase gates and service-floor checks; `TrustPinError` handling |
| Join / pairing | `join.ts`, `pairing.ts`, `pairing-http.ts` | Commit-then-reveal ceremony (sync-design §7/§8.1): commitment fixes keys+nonce before seeing the approver's; both screens derive one 6-digit code (relay substitution matches with p=10^-6); approval must be signed over the ceremony transcript |
| Approve | `approve.ts` | Approver half of the ceremony (`ApproveOutcome`) |
| HTTP | `http.ts`, `pairing-http.ts` | omo-service chain routes (`/v1/sync/entries`, `/keys`, `/recovery-wraps`, `recovery/join`); every response untrusted |
| Revocation / freeze | `freeze-settler.ts`, `freeze-eligibility.ts`, `revocation-view.ts`, `revoke-rotation-*` | Freeze settlement and rotation races; revocation projection |
| Ports | `operation-ports.ts`, `account-ports-runtime.ts` | Injectable operation/account ports |
| Test support | `testing/` | `chain-wire-server`, `credential-writer`, `freeze-model`, `genesis-enable`, `phrase-*`, `push-batch`, `revoke-world` supports + phrase-crash fixture |

## CONVENTIONS

- `AccountChain` is the shared verified-state spine: freeze-settler, sealer, verifier, and account operations all consume it — do not keep a second verified copy.
- Pairing ceremonies never trust a relay: transcript signatures + 6-digit code are the acceptance conditions.
- Operations route through `admitSyncOperation` / authority boundaries (see `../operation-authority.ts`).

## ANTI-PATTERNS

- NEVER treat an HTTP answer as verified: chain-verify before use, count (never log) refusals.
- NEVER pin a trust anchor outside a fully verified bootstrap walk (`../crypto/trust-pin.ts` owns the store).
- Don't grow the barrel casually; consumers import narrow submodules directly.

## COMMANDS

```bash
bun test packages/memory-core/src/sync/account/
```
