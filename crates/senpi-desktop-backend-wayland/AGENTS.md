# senpi-desktop-backend-wayland - Wayland backend (Linux)

## OVERVIEW

Linux-only (`#![cfg(target_os = "linux")]`) Wayland backend: XDG-portal + optionally PipeWire capture, lazy libei input, pure-Rust XKB keymaps, and a GlobalShortcuts-portal stop path. Earned its file at score 12: distinct platform domain, 53 Rust files / 7.3k LOC, largest symbol surface of the backend crates.

## STRUCTURE

```
src/
├── backend.rs / backend/      # WaylandBackend: core Backend contract
├── capture/ + capture/pipewire/  # ScreenCast portal, runtime-loaded PipeWire, Screenshot fallback
├── input/ + input/xkb/        # libei input; pure-Rust XKB layout/symbol resolution
├── portal/                    # RemoteDesktop runtime, tokens, cleanup
├── stop_path/                 # GlobalShortcuts portal listener and re-arm
├── test_support/ + fake_eis/  # portal iface fakes, libei protocol fake
└── testdata/                  # .xkb keymap fixtures (data, not complexity)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Crate root | `src/lib.rs` | `WaylandBackend`, `GlobalShortcutsListener`, `STOP_SHORTCUT_ID`, `STOP_PATH_UNAVAILABLE` |
| Backend dispatch | `src/backend.rs` (263 LOC) | Coordinates AT-SPI, lazy libei, portal capture, geometry validation, clipboard |
| Capture orchestration | `capture/mod.rs` (271) | `PortalCapture::capture`: ScreenCast/PipeWire then Screenshot fallback |
| PipeWire loading | `capture/pipewire/` | `libloading` on `libpipewire-0.3.so.0`; `crop_window`, `derive_displays` |
| Stop path | `stop_path/global_shortcuts.rs` (251) | Portal lifecycle and re-arm |

## CONVENTIONS

- Advertises only `"background"` delivery; inability to target/raise an unfocused window returns an explicit compositor-constraint error.
- Capabilities are reported as proven or unavailable, never assumed; unknown geometry yields pixels-only frames and coordinate input is refused.
- PipeWire is optional and dynamically loaded; the Screenshot portal is the fallback.
- XKB resolution stays within the compositor's active group; input connects lazily and caches a failed connection.
- Live/environment tests are `#[cfg(test)]`-gated and run with `-- --ignored`.

## ANTI-PATTERNS

- No link-time `libxkbcommon` or PipeWire: runtime loading is the intended optional path; do not add static linkage.
- Never guess display scale/layout for portal images or unknown-size streams.
- Never open an input session from capture just to learn layout; never silently substitute an unavailable requested display.
- Never persist portal restore tokens or libei socket sessions (`portal/token_cleanup.rs`).
- Never silently resolve a key from another XKB group.
- Never claim foreground activation or background-window input support; compositor policy forbids it.

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-wayland
cargo test -p senpi-desktop-backend-wayland                # unit tests
cargo test -p senpi-desktop-backend-wayland -- --ignored   # live environment tests
```
