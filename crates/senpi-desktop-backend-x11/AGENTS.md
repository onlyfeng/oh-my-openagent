# senpi-desktop-backend-x11 - X11 backend (Linux, pure-Rust x11rb)

## OVERVIEW

Linux-only (`#![cfg(target_os = "linux")]`) X11 backend over pure-Rust `x11rb`: RandR display enumeration, EWMH windows, root-window capture, XTEST/`XSendEvent` input, AT-SPI accessibility, XI2 stop path. Earned its file at score 10: distinct platform domain, 41 Rust files / 6.0k LOC.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Crate root | `src/lib.rs` | Re-exports `X11Backend`, `X11Capture`, `X11Connection`, `X11Input`, `X11InputConnection`, `Xi2Listener`, `DEFAULT_STOP_CHORD` |
| Input | `src/input/mod.rs` (261 LOC) | `X11Input` state machine; `keysym_variants`, `Keymap`, `InputServer`, XTEST path |
| Backend wiring | `src/backend.rs` (289) | Largest file; core `Backend` contract implementation |
| Capture | `src/capture/connection.rs` (255) | `XServer`, root-window capture |
| Stop path | `src/stop_path/` | `Xi2Listener`, default chord |
| Tests | `input/tests.rs` (283), `input/live_tests.rs` (275), `backend_tests.rs` (246) | Live tests need X11; shared-transaction restore checks need Xvfb + `xfwm4` |

## CONVENTIONS

- Private implementation modules with a narrow crate-root re-export; `capture`/`input`/`stop_path` are `mod.rs` directory modules.
- Unit and live test modules are `#[cfg(test)]`; live tests are `*_live_tests.rs`.
- Dependencies are declared Linux-only; package metadata inherits the workspace.

## ANTI-PATTERNS

- Never link `libX11` or `libxcb`; the crate is deliberately pure-Rust `x11rb`.
- Background input to a filtering toolkit is refused with `BackgroundUnavailable`, never retried in the foreground.
- An earlier focus snapshot must not overwrite a newer user choice.

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-x11
cargo test -p senpi-desktop-backend-x11   # run from workspace root
```
