# omo-computer-safety

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v3 fail-closed safety crate: gate order, the supervisor, stop commit and latch, and the protected-target / dangerous-chord tables. Every refusal is built from the closed `(EngineCode, Reason)` sets of core's error table, before any OS call, with nothing sent.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `gate/` | `gate()` + `Admitted` / `GateRefusal`, `Tables`, `Tier`, `VerbKind`, `DragPoint`: the admission decision every mutation passes. |
| `guard/` | `Guard` and its inputs: `Call`, `Clock` / `MonotonicClock`, `Observed`, `OwnProcess`, `ProcessSource`. |
| `supervisor/` | `Supervisor`, `StopOutcome`, `Exit` / `ProcessExit`, `Holds` / `NoHolds`: stop commit and process supervision. |
| `stop_path/` | `StopPolicy`, `StopPathView` / `StopPathKind` / `StopPathFailure`, `Transport`. |
| `latch.rs`, `rearm.rs` | `SuspendReason` suspension latch; rearming after a stop. |
| `marker/`, `durable/` | `FileMarker` / `MemoryMarker` behind `MarkerStore` / `MarkerState`; durable filesystem storage. |
| `protected/`, `resolve/`, `self_set/` | Protected-target and dangerous-chord tables; `TargetRef` resolution; `HostAdditions` to the self set. |
| `refusal.rs` | The `refusal!` macro (`#[macro_use]`, private): builds refusals from the closed core error sets. |
| `timing.rs`, `session_sink_tests.rs` | Time bounds; sink tests shared with the session crate. |

## CONVENTIONS

- The re-export list in `src/lib.rs` IS the public API (`gate`, `Guard`, `Supervisor`, `FileMarker`, `TargetRef`, ...); modules stay private below it - extend the `pub use` list, don't make a module public.
- Gate ordering is the crate's core asset: guard sources are consulted, tables applied, and the refusal (if any) constructed before anything reaches a backend. New safety checks slot into that order, never after the OS call.
- Fail closed on unreadable evidence: a guard source that cannot be read is a refusal reason, not a pass.

## ANTI-PATTERNS

- Never bypass `gate()` by calling a backend directly - the funnel in `omo-computer-session` is the only caller that admits input.
- Never widen a `GateRefusal` into an untyped string error; the refusal must name a `(EngineCode, Reason)` pair that already exists in core's closed table.

## COMMANDS

```bash
cargo test -p omo-computer-safety --locked
cargo clippy -p omo-computer-safety --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
