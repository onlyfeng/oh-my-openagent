# src/sync/ — Memory Sync Engine Root (direct files)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 16 (22 direct files, 7 subdirs, barrel, security-critical exports)

## OVERVIEW

Security boundary of memory synchronization: device keys, RFC 8785 canonical JSON, RFC 9180 HPKE sealing, secret redaction, operation authority, and the remote mirror. Parent covers memory-core anatomy; subdirectories `account/`, `crypto/`, `item/`, `space/` have their own files (or earn them this run). `engine/`, `genesis/`, `recovery/` are covered here only by pointer.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel re-exports ONLY `keyring-port`, `mirror`, `redact` |
| `keyring-port.ts` | `DeviceKeyring` — the sync engine's ONLY path to this device's private keys. Ed25519 signing pair + X25519 sealing pair, generated on-device; epoch keys arrive HPKE-wrapped |
| `canonical-json.ts` + `canonical-json-serialize.ts` + `canonical-json-types.ts` | RFC 8785 JCS for signed preimages: strict parse (duplicate keys refused at every depth), re-serialize, `requireCanonicalJson` proves the bytes signed are canonical |
| `hpke-sealed-box.ts` | RFC 9180 HPKE sealed-box open (LabeledExtract/LabeledExpand per spec); 48-byte minimum sealed box |
| `redact.ts` | The ONE secret scanner: matches on a normalised shadow (format chars + zero-width stripped, NBSP folded), spans map back to rewrite original bytes |
| `operation-authority.ts` | `SyncOperationAuthority` — captured caller authority; `admit` throws if the owner retired; optional `boundary` holds the credential-writer lock through a generation check |
| `publication.ts` | `publishSyncRename` — credential-writer lock spans admission and the synchronous rename, on every raw retry |
| `contract-version.ts` | `SYNC_CONTRACT_VERSION` (mirrors omo-service `contract/version.ts`), `SYNC_CAPABILITIES` (`phrase`), `syncHeaders()` |
| `pin-dir.ts` | `syncPinDir(memoryRoot)` → `<root>/sync-device` — trust pins live BESIDE identity repositories, never inside |
| `device-retire.ts` | The ONE seam retiring a sync trust pin (#12280); called exactly when the principal's private keys are discarded; imports nothing from `crypto/` |
| `mirror.ts` (+ `mirror.test-support.ts`, `mirror-hook-pipes.test.ts`) | Remote mirror upload; output MUST pass `redact.ts` first |

## CONVENTIONS

- Every server/relay answer is untrusted input: verify through the client chain (`crypto/`) before use.
- Security-sensitive files stay small and single-purpose; `keyring-port.ts`, `canonical-json*.ts`, `redact.ts`, `hpke-sealed-box.ts` are the review hotspots.
- Subdir barrels (`account/`, `crypto/`) expose narrow type-plus-factory surfaces; consumers also import narrow submodules directly.

## ANTI-PATTERNS

- NEVER add a second private-key path around `DeviceKeyring`.
- NEVER log or return matched secret text from `redact.ts` — only pattern class and span.
- NEVER copy secret-bearing raw logs/config into the mirror without the redaction layer.
- NEVER let a sync effect run without admitted operation authority (retired owner = refusal).

## COMMANDS

```bash
bun test packages/memory-core/src/sync/          # focused
bun run --cwd packages/memory-core test          # package gate (bun test src/)
bun run --cwd packages/memory-core typecheck     # tsgo --noEmit
```
