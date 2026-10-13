# senpi-desktop-backend-atspi - Shared Linux AT-SPI accessibility backend

## OVERVIEW

Linux-only (`#![cfg(target_os = "linux")]`) `rlib` implementing core `AxBackend` over the AT-SPI D-Bus registry for both Linux backends (`senpi-desktop-backend-x11`, `senpi-desktop-backend-wayland`). Pure Rust over `atspi` + `zbus`, never libatspi. Earned its file at score 9: distinct domain (the one AX implementation two platform crates share), 21 Rust files / 3.1k LOC.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Crate facade | `src/lib.rs` | `AtSpiAx<B: AtSpiBus = LiveBus>`, `WindowIds`, `AtSpiBus`, `BusResult`, `Extents`, `ScreenPoint`, `LiveBus`, `AxPermission`, `BACKEND_NAME` |
| Bus abstraction | `src/bus.rs`, `src/connection.rs`, `src/live.rs` | `AtSpiBus` trait; `LiveBus` is the real D-Bus connection |
| Window ownership | `src/owner.rs` | `correlate_owner`: joins AT-SPI frames to native X11 XIDs by pid, title, geometry |
| Actions / text / props | `src/actions.rs`, `src/text.rs`, `src/props.rs` | Element actions, text, property reads |
| Menus | `src/menus.rs`, `src/menu_target.rs` | Menu traversal and target resolution |
| Permission probe | `src/permission.rs` | `AxPermission` |
| Test double | `src/fake.rs` (289 LOC, largest file) | In-crate fake bus for unit tests |
| Live fixture | `tests/fixtures/menu_fixture.py` | GTK/AT-SPI menu fixture for ignored live tests |

## CONVENTIONS

- Backends hold it as `AtSpiAx::new(window_ids).ok()`: a missing accessibility bus reports `ax: false` instead of failing the whole backend.
- `WindowIds` is fixed at construction: `AtSpiFrames` (Wayland lists AT-SPI frames by object id) vs native ids AT-SPI cannot name (X11 XIDs, correlated by `owner.rs`).
- `AtSpiAx` is generic over `AtSpiBus`, so unit tests run against `fake.rs` with no D-Bus.
- Tests colocate as `*_tests.rs` modules; live tests (`live_tests.rs`, `menus_live_tests.rs`) are `#[ignore]` and need a real AT-SPI bus.
- Dependencies are Linux-target only (`atspi`, `zbus`, `tokio` with `time`, `senpi-desktop-core`); metadata inherits the workspace.

## ANTI-PATTERNS

- Never link libatspi or add a C dependency; the crate is deliberately pure Rust.
- Never fail backend construction because the accessibility bus is absent; degrade to `ax: false`.
- Never guess an element's owner window; when correlation is ambiguous return `AxOwner::Unknown` (core contract).

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-atspi
cargo test -p senpi-desktop-backend-atspi                 # unit tests (fake bus)
cargo test -p senpi-desktop-backend-atspi -- --ignored    # live AT-SPI tests
```

Parent: repo-root `AGENTS.md` (no `crates/AGENTS.md` exists).
