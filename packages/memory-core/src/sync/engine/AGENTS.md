# src/sync/engine/ — Device Sync Engine Internals

## OVERVIEW

~66 files, ~8.7k LOC (direct files only; subdirs like `adapters/` have their
own scopes). The cycle-stage implementation of `SyncEngine`: receive, local
capture, apply, push, legacy cycle. Exposed publicly via the
`@oh-my-opencode/memory-core/sync/engine` subpath barrel (`index.ts`).
Parent: [`src/sync/AGENTS.md`](../AGENTS.md) owns the crypto/trust model —
not repeated here.

## WHERE TO LOOK

| File | Role |
|------|------|
| `engine.ts` | `SyncEngine` core; constructed by `open.ts` (`openDeviceSync`) |
| `receive.ts` / `apply.ts` | Verify + apply untrusted remote operations |
| `local-capture.ts` / `push-cycle.ts` | Local change capture and outbound push |
| `legacy-cycle.ts` | Pre-engine journal compatibility |
| `engine-deps.ts` | Injected ports (store, crypto, transport, clock, locks) |
| `batches.ts`, `atomic.ts` | Batch assembly and atomic publication |
| `engine-*.test.ts` | Crash, durability, fuzz, join/adopt, review, trigger suites |

## CONVENTIONS

- Cycle stages are explicit modules; dependencies arrive via injected ports
  (fake store/server/crypto, manual clock, in-process locks, fault injection
  in `engine-harness.test-support.ts`).
- Tests import engine internals directly — that is intentional; only the
  barrel is the package boundary.
- Behavioral invariants are test-enforced: no oversized items sent, no foreign
  peer operations imported, no peer edits lost, unsafe received paths
  rejected, a copied device's journal is not resent.

## ANTI-PATTERNS

- Do not run a sync effect without admitted operation authority.
- Do not add a private-key path around `DeviceKeyring`.
- Do not weaken crash-recovery ordering (receive/apply/capture/push order is
  pinned by `engine-crash` / `engine-journaled-recovery` tests).

## QA

```bash
bun test packages/memory-core/src/sync/
bun run --cwd packages/memory-core typecheck
```
