# omo-engine-trust-helper

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Native trust FACT helper for the engine CA importer (omo#10762): reads macOS Security.framework or Windows Crypt32 in its own killable process and writes one bounded JSON document of facts to stdout. It makes no trust decision - `packages/omo-native/{macos,windows}-trust-helper.ts` decode the document and `packages/omo-native/engine-trust-policy.ts` decides.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/lib.rs` | Per-OS `run() -> i32`: macOS collects then emits; Windows delegates to `windows::run()` (reads Bun's bundled roots from stdin); Linux returns `EXIT_UNSUPPORTED` - it needs no helper. |
| `src/document.rs` | The stdout contract: `Document { schema: 1, os, complete, sources[], certificates[] }`, `MAX_OUTPUT_BYTES` 8 MiB, exit consts `EXIT_UNSUPPORTED` 64 / `EXIT_INTERNAL` 70 / `EXIT_OUTPUT_CAP` 74 / `EXIT_WATCHDOG` 75. |
| `src/watchdog.rs` | `watchdog::arm` with `HARD_CAP`; the kill path uses `libc::_exit` - never atexit handlers while another thread may be inside Security.framework. |
| `src/macos.rs` | Security.framework chain/cert collection; chain evaluation never fetches from the network. |
| `src/windows.rs` (766 LOC) | Crypt32 certificate-store and CTL collection; the crate's hotspot. `src/windows_document.rs`, `src/stdin.rs` | Windows document shape; stdin root reader. |
| `tests/` | `macos_adapter.rs`, `windows_adapter.rs` (600 LOC), `fixtures/` incl. DER certs. |

## CONVENTIONS

- Collection and policy are deliberately split: Rust gathers facts, TypeScript decides trust. Keep it that way - no policy logic here.
- `complete: false` when an applicable negative source (a trust-settings domain) could not be read completely; incomplete facts are incomplete, never partial trust evidence.
- Output encoding is bounded and never truncates a document; the decoder (`MAC_TRUST_HELPER_MAX_OUTPUT_BYTES` etc.) enforces the same bounds again on the TS side.
- Platform code stays under Cargo cfg sections: `macos` / `windows` modules compile only on their target.

## ANTI-PATTERNS

- Never turn the helper into a decision-maker - verdicts like "trusted" live in `engine-trust-policy.ts`.
- Never fetch from the network during chain evaluation.
- Never write facts anywhere but stdout (one document), and never past `MAX_OUTPUT_BYTES`.

## COMMANDS

```bash
cargo test -p omo-engine-trust-helper --locked
cargo clippy -p omo-engine-trust-helper --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
