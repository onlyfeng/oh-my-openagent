# src/sync/crypto/ - Verified Sync Chain Cryptography

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 16 (96 files, ~8.4k LOC, 8 subdirs, barrel, verify-before-use core)

## OVERVIEW

The client-side verification core of memory sync: trust anchors, the append-only keyring/entry chain, batch build/sign, HPKE wraps, recovery phrases, and the hostile-server test benches. Everything here treats served data as untrusted; a row is usable only after the chain verified it.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Anchors | `anchor.ts`, `trust-pin.ts` | Branded `TrustAnchor`/`PinCandidate`/`BootstrapAnchor`; the device's pinned account root (sync-design §7, #10298). Pin store: one dir per account under `<stateDir>/trust-anchor/`, published by atomic rename via the device-peer pattern - a pin is never partial and never overwritten; missing/corrupt/mismatching pin = NO anchor, refuse and tell the user to re-pair |
| Chain | `chain.ts`, `entry.ts`, `envelope.ts`, `links.ts`, `fold.ts` | `appendEntries`/`ChainState`/`ChainStep` over served rows; epoch authority fold (vetoed epochs, cancelled admissions); envelope refusals |
| Build | `build.ts`, `signatures.ts` | `buildBatch` + `signRecord` for outgoing batches (`SyncPayloadRecord`) |
| Read side | `open.ts`, `unwrap.ts`, `receiver.ts`, `receiver-parity.test.ts` | Batch open, wrap unwrap, receiver wire parity |
| Parse | `parse.ts`, `rules.ts`, `rejection.ts` | Strict object/principal parsing, rejection counters |
| Snapshots | `snapshot.ts`, `frozen.ts` | Verified-head snapshot; `ClosedMap` - a read-only map that is NOT a Map (`instanceof Map` false), test with `isReadonlyMap` |
| Recovery | `recovery-kind.ts`, `parse.ts` (`isPhraseKey`, `PHRASE_RECOVERY_RP_ID`), `phrase-chain*.test.ts`, `parse-phrase.test.ts` | Recovery-phrase key parsing lives in `parse.ts` (no standalone `parse-phrase.ts`); phrase chain fixtures |
| Strict Ed25519 | `strict-ed25519/` | `strict-ed25519.ts` + RFC 8032 corpus (`strict-ed25519-corpus.json`, `PIN.json`) and oracle test support |
| Vendored | `vendor/` | Pinned `sync-format` set + `manifest.json` + fixtures; `vendor-integrity.test.ts` pins it |
| Hostile benches | `malicious-server.test.ts`, `server-attacks.test.ts`, `weak-key.test.ts` | Adversarial server/key scenarios |
| Test support | `testing/` | `fork`, `genesis`, `hpke-seal`, `pinned`, `strict-ed25519-oracle`, `sync-v1` supports |

## CONVENTIONS

- Brand types (`declare const` brands) separate verified from unverified values; never cast across the brand boundary.
- The barrel is explicit named exports (no `export *`); keep it curated.
- `account/` consumes this chain through `AccountChain` - verification logic stays here, lifecycle stays there.

## ANTI-PATTERNS

- NEVER trust a fresh bootstrap walk over the pinned anchor to re-establish trust.
- NEVER widen `vendor/` by hand; it is pinned and integrity-tested.
- Don't bypass `strict-ed25519/` prechecks when verifying signatures.

## COMMANDS

```bash
bun test packages/memory-core/src/sync/crypto/
```
