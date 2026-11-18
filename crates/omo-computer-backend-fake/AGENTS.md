# omo-computer-backend-fake

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v3 scripted backend for tests: it implements the `Backend` / `AxBackend` traits of `omo-computer-core` from ONE scenario file. Every OS-level side effect is one line in the `RecordingSink`; time is the `FakeClock` alone; each scenario runs under every `Persona`. It makes no OS calls.

## STRUCTURE

| Path | Role |
|------|------|
| `src/backend.rs` | `FakeBackend` + `StopPathScript`. |
| `src/scenario/`, `src/scripting/`, `src/timeline/` | The scenario model (`FakeScenario`, `ScriptedFailure`, `IdentityChange`, `WindowIdentityScript`, ...), scripting results, timelines (`UserInput`, `LoadingState`). |
| `src/seams.rs` | `SCENARIO_SEAM` / `STOP_PATH_SEAM` / `CLOCK_SEAM`: seams arrive as VALUES (paths handed in by the test), never as ad-hoc reads. |
| `src/sink.rs`, `src/state.rs`, `src/clock.rs` | `RecordingSink`, backend state, `FakeClock` + `ClockSeamInvalid`. |
| `src/ax.rs`, `src/ax_reader.rs`, `src/ax_tree.rs` | AX behavior; tree constants `NODES_PER_MS`, `PAD_ROLE`. |
| `src/frame.rs`, `src/desktop.rs`, `src/handles.rs` | Frames, compositing cap `MAX_COMPOSITE_PIXELS`, hold handles. |
| `src/persona.rs`, `src/faults.rs`, `src/method.rs` | The `Persona` matrix, `Fault` injection, `FakeMethod`. |
| `src/time_guard.rs` | The source-scanning guard below. |
| `fixtures/` | Scenario JSONs (ported from / kept in sync with the v2 fake's fixtures). |

## ANTI-PATTERNS (THIS CRATE)

- `time_guard.rs` FORBIDS in every `src/` file: `Instant`, `SystemTime`, `UNIX_EPOCH`, `sleep(`, `tokio::time`, `chrono::`, `std::env`, `env::` (including grouped `use std::{.. env ..}`). Needles are split in the guard so it does not match its own source.
- The guarded invariants are `fake_clock_is_the_only_time_source` and `the_fake_reads_no_environment`; seams are passed in as values, so there is no legitimate reason to violate either.

## COMMANDS

```bash
cargo test -p omo-computer-backend-fake --locked
cargo clippy -p omo-computer-backend-fake --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../../AGENTS.md).
