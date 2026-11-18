# senpi-desktop-backend-x11

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The X11 backend (Linux-gated): pure-Rust `x11rb` connection, XTEST pointer/keyboard input, X11 capture, and an XI2 global stop-chord listener. Accessibility comes from `senpi-desktop-backend-atspi`, held optionally.

## STRUCTURE

| Path | Role |
|------|------|
| `src/lib.rs` | Exports `X11Backend`, `X11Capture`, `X11Connection`, `X11Input`, `X11InputConnection`, `Xi2Listener`, `DEFAULT_STOP_CHORD`, `BACKEND_NAME`. |
| `src/backend.rs` (+ `backend_tests/`, `backend_live_tests.rs`) | `Backend` contract implementation and its tests. |
| `src/capture/` | X11 screen capture. |
| `src/input/` | Pointer/keyboard input via XTEST. |
| `src/stop_path/` | XI2-based global stop chord listener. |
| `src/ax_owner_live_tests.rs` | `#[ignore]d live owner-correlation probes (CI: `xvfb-run`). |

## CONVENTIONS

- Linux-only crate: dependencies are target-scoped in `Cargo.toml` (`x11rb` with `res`, `xkeysym`, plus `senpi-desktop-backend-atspi` for AX - never duplicate accessibility code here).
- Live tests are exact-named and `#[ignore]d; a normal `cargo test` stays hermetic.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-x11 --locked            # on Linux
cargo clippy -p senpi-desktop-backend-x11 --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
