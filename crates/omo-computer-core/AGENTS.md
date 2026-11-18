# omo-computer-core

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v3 contract crate every other `omo-computer-*` crate depends on (rlib; deps `schemars`, `serde`, `serde_json`, `sha2`, unix `libc`; dev-dep `syn =2.0.118` for reader policing). The `src/` module map lives in [`src/AGENTS.md`](src/AGENTS.md); this file covers the generated artifacts, conformance fixtures, and the test modules that police them.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/AGENTS.md` | The contract module map (methods, params, requests, error, report, ax, settings, backend traits). |
| `docs/engine-protocol.md` | GENERATED prose protocol doc, built from `src/methods/table.rs`. Header says "do not edit by hand". |
| `schema/` | The generated `engine.schema.json` (see `src/AGENTS.md`'s schema row). |
| `fixtures/conformance/` | 35 kebab-case scenario JSONs replayed by conformance tests: `error-codes-amend-*`, `stop-path-ladder`, `stop-path-policy`, `user-intervened`, `v3-marker-crash-redrive-uncertain`, `v3-ax-diff-*`, `capture-budget`, `stale-ref`, `bounded-holds-closed`, ... Each is named for the protocol behavior it pins. |
| `src/protocol_test/` | Freshness + allowlist tests (details below). |
| `src/results/` | Method replies and notification params (details below). |
| `parity/` | The v2→v3 port manifest; guarded by `bun script/computer-v3-guards.ts --parity` (see the crates root file). |

## protocol_test/ (freshness and allowlists)

- `assert_fresh` (in `mod.rs`) compares each generated file with its committed copy; setting `OMO_COMPUTER_UPDATE_SCHEMA=1` REWRITES the committed file instead of failing.
- `read_sites.rs` parses `src/requests/` with `syn::visit` to discover schema-field read sites; `readers.rs` fails on a field a reader binds but never uses, and on a schema member no reader binds (`READER_SOURCES`).
- `seams.rs` / `settings.rs` reject unknown seams, unknown settings, and settings the contract has removed.
- Other modules: `backend`, `corpus`, `hello`, `listing`, `requests`.

## results/ (reply types)

- `diagnosis.rs`: `Diagnosis`, `IdentityReport`/`IdentityMatch`, `PermissionReport`/`PermissionState`, `ConsentAuthority`, `ChainLink`/`ChainRole`, `SpawnPath`, `AttachVia`.
- `reads.rs`: `CaptureReply`, `ObserveReply`, `WindowsReply`, `WaitResult`/`WaitOutcome`, `MenuItems`, `ClipboardContent`, `ConcealedReason`, `DisplayInfo`, `PointerPosition`.
- `notifications.rs`: `AuditLine`, `EpochChanged`, `HandConsent`, `HandStatus`, `ForegroundConsent`.
- `v1.rs`: legacy wire types (`AxNode`, `CapabilitiesReply`, `ControlState`, `LegacyFrame`, `LegacyObservation`, `SessionOpened`, `StopPathStatus`, ...). `exec.rs`: `ExecReply` (transparent, preserves the null-mode wire shape).

## CONVENTIONS

- One env var regenerates every generated artifact: `OMO_COMPUTER_UPDATE_SCHEMA=1 cargo test -p omo-computer-core` (schema + protocol doc). Editing `methods/table.rs` without regenerating fails freshness.
- New wire behavior = a new conformance fixture JSON, never a branch in a reader.
- Result structs serde `rename_all = "camelCase"` with `deny_unknown_fields`; wire unions are enums, not optional blobs.
- Knip: `knip.workspace.jsonc`'s single entry is `src/error/tests/check-design-citations.mjs`; root `knip.ts` composes it.

## COMMANDS

```bash
cargo test -p omo-computer-core --locked
cargo clippy -p omo-computer-core --all-targets --locked -- -D warnings
OMO_COMPUTER_UPDATE_SCHEMA=1 cargo test -p omo-computer-core   # regenerate schema + docs
```

Parent: [`crates/AGENTS.md`](../AGENTS.md) - module map: [`src/AGENTS.md`](src/AGENTS.md).
