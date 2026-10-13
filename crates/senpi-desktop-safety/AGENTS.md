# senpi-desktop-safety - Fail-Closed Desktop Input Gate (Rust)

**Generated:** 2026-10-13 / dc373d26f

Earned its file at score 9 (8-15 band): distinct domain: the safety boundary every desktop computer-use mutation crosses.

## OVERVIEW

Rust `rlib` workspace crate: the fail-closed supervisor, stop-path registry, and ordered permission gate for desktop input. Every side-effecting action passes `gate()` before any backend access. Consumed by all four platform backend crates (macOS, Wayland, Win32, X11), `senpi-desktop-session`, and `senpi-desktop-engine`. The `test-support` Cargo feature bypasses the user-only resume proof and is enabled only by dev-dependencies.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Mutating action taxonomy | `src/action.rs` (`MutatingAction`; read-only requests have NO variant) |
| Ordered input gate | `src/gate.rs` (`gate`, `PermissionGate`, `GateError`, `LockState`); `src/gate/permission_tests.rs` |
| Injected clocks | `src/clock.rs` (`Clock`, `FakeClock`, `MonotonicClock`) |
| Reset proof | `src/reset.rs` (`ResumeToken`, `UserReset`) |
| Stop paths | `src/stop_path/` (`Chord`, `HostRelay`, `StopPathListener`, `StopPathError`) |
| Supervisor + heartbeats | `src/supervisor.rs` (`Supervisor`, `StopPolicy`, `StopSource`, `HEARTBEAT_*_MS`) |

## CONVENTIONS

- Gate checks are ordered and the first failure wins; reorder only with a matching test update.
- Time-dependent logic takes an injected `Clock`; tests use `FakeClock`, never wall-clock sleeps.
- Modules stay private; the public API is explicit `pub use` re-exports in `src/lib.rs`.
- Tests are colocated: each module declares `#[cfg(test)] mod tests;` resolving to `src/<mod>/tests.rs` (plus `gate/permission_tests.rs`); no `#[path]` attributes in this crate.

## ANTI-PATTERNS

- Never route read-only desktop methods through `MutatingAction`; module docs exclude them deliberately.
- Never let a mutation reach a backend without passing the gate first; fail-closed is the invariant.
- Never enable the `test-support` feature from a production dependency.

## QA

```bash
cargo test -p senpi-desktop-safety
```

Parent: repo-root `AGENTS.md` (no `crates/AGENTS.md` exists).
