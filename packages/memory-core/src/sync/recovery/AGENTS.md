# sync/recovery - Recovery-Key Enrollment + Fresh-Device Recovery

**Score:** 17 (53 files, 9.6k LOC, >20 files, own subtree with bip39/fixtures/testing). Parent: [`packages/memory-core/AGENTS.md`](../../../../AGENTS.md).

## OVERVIEW

Crypto recovery for the sync account: derive recovery keys, enroll them on the server bridge, recover on a fresh device, remove them, and run the phrase-based enrollment/rotation/recovery variant. All platform effects sit behind adapter ports.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Public API surface | `index.ts` - the ONLY intended import for consumers |
| Key derivation | `derive.ts` (`deriveRecoveryKeys`, salts, user handle) |
| Enroll / recover / remove flows | `enroll.ts`, `recover.ts` (`recoverOnFreshDevice`), `remove.ts` |
| Bridge + availability | `bridge.ts` (`createRecoveryBridge`), `availability.ts` (`recoveryAvailability`) |
| Phrase variant | `enroll-phrase.ts` (`enrollPhraseRecoveryKey`, `rotateRecoveryPhrase`, `PhraseAcknowledgeUnverifiedError`), `bridge-enroll-phrase.ts`, `bridge-recover-phrase.ts`, `bridge-recover.ts`, `phrase-hints.ts` |
| Join classification + veto | `join-notice.ts` (`classifyRecoveryJoin`, `VETO_WINDOW_MS`) |
| Crypto primitives + markers | `hpke-seal.ts`, `key-material.ts`, `keyring-append.ts`, `enroll-marker.ts`, `recovery-handles.ts`, `canary` tests |
| Adapter ports | `ports.ts` - `RecoveryBridge`, `RecoveryTransport`, authenticator types |
| Wordlists / test data / fakes | `bip39/`, `fixtures/`, `testing/` (fake world, server, passkey helpers) |

## CONVENTIONS

- Barrel imports use explicit `.ts` extensions.
- Every flow returns readonly discriminated unions: outcomes AND refusals are values, never throws for expected paths.
- Platform effects (network, keystore, passkey) go through `ports.ts`; tests substitute fakes from `testing/`.
- Tests are named by hardening concern (`recover-hostile`, `recover-refusals`, `phrase-redaction`, `bridge-provenance`); keep that taxonomy when adding cases.

## ANTI-PATTERNS

- Do not import internal modules (e.g. `key-material.ts`) from outside - `index.ts` is the surface; `package.json` declares no `./sync-recovery` subpath.
- Do not widen `VETO_WINDOW_MS` or bypass `classifyRecoveryJoin` when admitting a join.
- Phrase hints must stay redacted in logs/UI (`phrase-redaction.test.ts`).

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/sync/recovery
bun run --cwd packages/memory-core typecheck
```
