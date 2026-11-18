# omo-computer-session

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v3 session runtime: the mutate funnel (pre-input marker, audit, settle), the refs registry, bounded observation/diff/batch, the interaction lock, and budget permits - the twin of `senpi-desktop-session` carrying the v3 error/report vocabulary.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `mutate/` | The side-effect funnel (`Session::mutate`): binding `(issuer, scope, callId, runGeneration)`, the pre-input marker persisted BEFORE the first OS call, the backend call, settle, read-back, `ActionReport`, and one audit line per side effect. Submodules: `funnel`, `binding`, `call`, `record`, `recovery`, `poison`, `report`, `diagnostics`, `memory_markers`. |
| `refs/` | `RefRegistry`: evidence-based re-identification, `Generation` / `Identity`, eight generations per window, session-wide non-reused element numbers. `mint.rs` / `resolve.rs` are the identity ladder. |
| `audit/` | Closed typed record model; text/keys recorded as length + digest, never raw; hash-chained JSONL segments (`writer.rs`, `segment.rs`), rotation and bounded retention. `gc/` deletes only prefixed artifacts and trace session dirs. |
| `observe/`, `diff/`, `batch/` | `observe` / `drill` / `find`, `diff::decide`, `batch::run` - the bounded observation pipeline over core's ax limits. |
| `lock/`, `marker/`, `budget/` | Machine-wide interaction lock; marker stores; permits and budget. |
| `session.rs` (private) | `Session`, `legacy_capabilities`, and the `NOT_OPEN` / `NO_GLOBAL_LISTENER` / `POINT_REQUIRED` refusals. |

## CONVENTIONS

- No retry anywhere in the funnel (crate doc). A marker that cannot be persisted means the input is not sent; `recovery` handles the aftermath, never a resend.
- Audit locking: appends take the hand's `audit.lock`, NEVER a byte-range lock on the data file - a byte-range lock would fail every concurrent reader (`audit/writer.rs`). On every OS two processes appending at an offset stay serialized by the lock file.
- Unreadable evidence or failed cleanup is never silently treated as success.
- The `pub use` list in `src/lib.rs` is the whole public surface (`Binding`, `Mutation`, `Refused`, `SecureInputProbe`, `SecureProbe`, `RefRegistry`, `Unresolved`, `Session`, ...); `session` and `test_process` stay private.

## ANTI-PATTERNS

- Never mutate outside `mutate` - it is the only path that orders marker, input, and audit.
- Never redact by omission: an audit record that cannot be written fails the mutation path it belongs to.
- Never reuse an element number or mint a ref without identity evidence.

## COMMANDS

```bash
cargo test -p omo-computer-session --locked
cargo clippy -p omo-computer-session --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
