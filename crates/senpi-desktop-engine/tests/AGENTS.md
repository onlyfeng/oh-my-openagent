# tests - engine integration harness

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Integration tests that drive the BUILT `senpi-desktop-engine` binary over stdio (41 files, ~7.1k LOC; 24 test crates). Earned its file via the shared `common/` harness - 14+ test crates spawn through it.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `common/mod.rs` | The harness: `Engine` spawns `--stdio` (`CARGO_BIN_EXE_senpi-desktop-engine`) with EXACTLY the given engine variables; `fake_backend(scenario)` builds `SENPI_DESKTOP_BACKEND=fake:<scenario>`; line reader on a thread. |
| `common/scenario.rs` | `Scenario` builders: `headless`, `capture`, `snapshot_ref`, `make_stop_path_live`. |
| `fixtures/` | Scenario JSONs: `v1-consumers/` (10 scripted stdio sessions + `scenarios/`), `delay-capture-5000ms.json` (`DELAYED_CAPTURE`), and `two-displays-one-window.json` borrowed from `senpi-desktop-backend-fake/fixtures` (`TWO_DISPLAYS`). |
| `stdio.rs`, `stdio_headless.rs` (+ `headless/`), `serve*.rs` | Framing behavior, headless request/error paths, serve modes (`serve.rs` 401 LOC, `serve_oneshot`, `serve_pipe`). |
| `v1_consumers.rs` (699 LOC) | The largest suite: replayed v1 consumer sessions incl. stale-ref, dangerous chord, user-press-mid-type. |
| `conformance_replay.rs`, `method_allowlist.rs` | Protocol conformance fixtures; exposed-method allowlist. |
| remaining crates | `control_grant`, `bounded_holds`, `cursor_overlay`, `release_helper`, `helper_containment`, `late_reply_quarantine`, `launch_context_process`, `launch_gate`, `observe`, `region_zoom`, `resource_caps`, `stop_path`, `trace`, `xpc`, `secure_audit_engine_process.rs`, `selftest_lock_path.rs`. |

## CONVENTIONS

- Every wait is bounded by `HANG_GUARD` (30 s) - a hang guard only; no assertion depends on latency.
- `ENGINE_ENV` is the whitelist of variables a test may set (`SENPI_DESKTOP_BACKEND`, `SENPI_DESKTOP_FAKE_CLOCK`, `SENPI_DESKTOP_OPERATION_TIMEOUT_MS`, `SENPI_DESKTOP_CLOSE_TIMEOUT_MS`); `Engine::spawn` starts the child with exactly those and nothing else.
- Fixtures are data: a new client-visible behavior gets a scripted JSON, not hardcoded lines.

## ANTI-PATTERNS

- Never assert on timing - assert on the ordered stream of JSON lines the engine emits.
- Never read the child's stderr as protocol output; only stdout lines are protocol.

## COMMANDS

```bash
cargo test -p senpi-desktop-engine --locked                       # all integration targets
cargo test -p senpi-desktop-engine --locked --test v1_consumers   # one target (targets = file stems)
cargo clippy -p senpi-desktop-engine --all-targets --locked -- -D warnings
```

Parent: [`crates/senpi-desktop-engine/AGENTS.md`](../AGENTS.md).
