# senpi-desktop-safety

**Generated:** 2026-10-22 / f9de7acf36

## OVERVIEW

The fail-closed kill switch. Side-effecting input may fire only while `Supervisor::input_allowed` holds, and every mutating action passes `gate()` before a backend is touched. Ported from gajae-code (`pi-natives/src/computer/{executor,supervisor}.rs`) and generalized from a single kill switch to a registry of stop paths.

Two rules decide everything here:

1. **No live, fresh stop path means no input.** A stop path is the `Global` listener (OS hotkey) or, only when policy opts in via `computer.allowHostRelayOnlyStop`, the `HostRelay` alone.
2. **A stop latches suspension until a user-only `reset()`.** Reset demands a `UserReset`, so automation cannot clear its own kill switch.

## STRUCTURE

| Path | Role |
|------|------|
| `src/supervisor.rs`, `src/supervisor/` | The supervisor: `input_allowed`, active stop paths, `ActiveStopPath`, `StopPathFailure`, `StopPolicy`, heartbeat freshness, and the latched suspension. |
| `src/gate.rs`, `src/gate/` | `gate()` + `PermissionGate` / `LockState` / `FrameContext` / `FrameId` / `GateError` / `StopPathReason`. Fixed check order, first failure wins: suspension, live+fresh stop path under policy, lock screen, input permission, then (coordinate actions only) the expected frame. Reads the state it is handed and performs no I/O. |
| `src/action.rs`, `src/action/` | `MutatingAction`: the closed set of actions that must pass the gate. |
| `src/stop_path.rs`, `src/stop_path/` | `StopPathListener`, `Chord`, `HostRelay`, `StopPathError`: registration and heartbeat of each stop source. |
| `src/hold_signal.rs`, `src/hold_signal/` | `HoldSignal`: the hold/release signalling used by long presses. |
| `src/reset.rs` | `ResumeToken`, `UserReset`: the only path out of a latched suspension. |
| `src/clock.rs` | `Clock` / `MonotonicClock` / `FakeClock`: freshness is measured on an injectable clock so tests never depend on wall time. |
| `src/lib.rs` | Re-exports: `MutatingAction`, `HoldSignal`, `Clock`, `gate`, `PermissionGate`, `ResumeToken`, `UserReset`, `Chord`, `HostRelay`, `StopPathListener`. |

## CONVENTIONS

- **Fail closed, always.** When in doubt the answer is "do not send input"; a permissive default is a security bug.
- **Check order is part of the contract.** Do not reorder the gate checks or let a later check short-circuit an earlier one; suspension must win over cancellation.
- **The gate performs no I/O.** Everything it needs arrives in its arguments; that is what makes it testable off-device.
- **Time comes from `Clock`.** Never call `Instant::now()` inside policy logic; inject `FakeClock` in tests, never sleep.
- **A stop latch is user-clearable only.** Never add an automated path that calls `reset()` without a `UserReset`.
- **Host-relay-only mode is explicit policy.** It widens the trust boundary, so it must be opt-in and visible in the config surface.

## COMMANDS

```bash
cargo test -p senpi-desktop-safety --locked
cargo clippy -p senpi-desktop-safety --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
