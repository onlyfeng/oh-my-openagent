# senpi-desktop-backend-atspi

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The shared Linux AT-SPI accessibility backend used by BOTH the X11 and Wayland backends: pure Rust over `atspi` + `zbus`, never libatspi. Backends hold it as `AtSpiAx::new(..).ok()`, so a missing accessibility bus reports `ax: false` instead of failing the whole backend.

## STRUCTURE

| Path | Role |
|------|------|
| `src/lib.rs` | Exports `AtSpiBus`, `AtSpiAx<B>`, `BusResult`, `Extents`, `ScreenPoint`, `LiveBus`, `AxPermission`, `WindowIds`, `BACKEND_NAME`. `AtSpiAx` implements core's `AxBackend`. |
| `src/bus.rs` | `AtSpiBus`: the D-Bus round-trip seam (`windows`, `menu_items`, `menu_select`, `permission`). `fake.rs` is the in-memory bus for hermetic tests. |
| `src/apps.rs`, `src/owner.rs` | Application/window enumeration and owner correlation. |
| `src/actions.rs`, `src/menus.rs`, `src/menu_target.rs`, `src/text.rs`, `src/props.rs` | AX actions, menus (with `MenuTarget`), text ranges, property reads. |
| `src/permission.rs` | AX bus permission state. |
| `src/live.rs`, `src/*_live_tests.rs` | Live D-Bus probes - `#[ignore]d; CI runs them under `dbus-run-session` + `xvfb-run`. |

## CONVENTIONS

- All bus I/O goes through the `AtSpiBus` trait seam; unit tests use the in-memory fake bus and never touch a real session bus.
- Keep this crate holder-agnostic: it must not know whether X11 or Wayland holds it.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-atspi --locked          # on Linux
cargo clippy -p senpi-desktop-backend-atspi --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
