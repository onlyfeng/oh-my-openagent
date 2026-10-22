# senpi-desktop-session

**Generated:** 2026-10-22 / f9de7acf36

## OVERVIEW

The session runtime that sits between the wire protocol and a platform backend: it accepts requests, routes read-only work to the backend directly, and funnels every mutating request through ONE choke point (`mutate`) that admits the process-wide transaction lock, applies the fail-closed `gate()`, runs the action under `catch_unwind`, and emits exactly one audit event.

Invariants this crate owns:

- **Read-only work never passes the choke point.** Only `MutatingAction`s take the lock and the gate.
- **Suspension always wins over cancellation.** A stop or a cancellation observed at admission or just before the action sends no input.
- **Every attempted mutation produces exactly one audit event.** There is no silent mutation path.
- **Failure releases holds.** `release_all` runs on both failure and suspension, and the focus/cursor are restored.

## STRUCTURE

| Path | Role |
|------|------|
| `src/mutate.rs`, `src/mutate/` | `SessionSafety` and the single mutation choke point: transaction lock, `gate`, focus/cursor capture, the action under `catch_unwind`, `release_all` on failure or suspension, the restore, then the audit event. |
| `src/session.rs` | `Session`, `Pending`, `THREAD_NAME`: the session loop, request admission, and pending-request bookkeeping. |
| `src/request.rs` | `Op`, `Response`: the internal request/response shape the session dispatches on. |
| `src/selection.rs` | `BackendFactory`, `BackendSelection`, `SelectionError`: picking the platform backend (fake for tests) at session start. |
| `src/grant.rs`, `src/grant/` | `ControlSlot`: what the session may touch for this run. |
| `src/timeouts.rs` | `SessionTimeouts`, `OPERATION_TIMEOUT`, `CLOSE_TIMEOUT`: the bounded waits every operation honours. |
| `src/audit/` | The audit sink: one record per mutation with its outcome. |
| `src/worker*.rs`, `src/worker_*/` | Per-domain workers: capture, input, ax, menus, clipboard, zoom, observe, holds. |
| `src/restore.rs`, `src/restore/` | Focus and cursor restoration after an action. |
| `src/pointer.rs`, `src/selection.rs`, `src/cancel_on_drop.rs` | Pointer bookkeeping, target selection, and the drop guard that cancels in-flight work. |
| `src/test_support.rs`, `src/test_support/` | Fixtures for the integration tests. |
| `tests/` | Session-level behaviour: `session.rs`, `display_targets.rs`, `capture_budget.rs`, `control_grant.rs`, `menus.rs`, `bounded_holds.rs`, `session_options.rs`. |

## SCREENSHOT BUDGET

Capture is budgeted per session (see `tests/capture_budget.rs`): the engine returns a bounded, downscaled frame rather than an unbounded screenshot, so a runaway loop cannot flood the model with pixels. Treat the budget as part of the public behaviour, not as an optimisation.

## CONVENTIONS

- **One choke point.** Never mutate through a worker directly; all input goes through `mutate`, which is what makes gating and auditing unconditional.
- **No sleeps for synchronisation.** Every wait is an explicit timeout or an observable event; tests await state with bounded timeouts.
- **Panics are contained.** The action runs under `catch_unwind`; a backend panic becomes a typed error plus `release_all`, never a dead session with held keys down.
- **Audit before you return.** A code path that mutates without writing its audit event is a bug.
- **Restore on every exit path.** Focus/cursor restore and hold release belong in guards, not in happy-path code.

## COMMANDS

```bash
cargo test -p senpi-desktop-session --locked
cargo clippy -p senpi-desktop-session --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
