# senpi-desktop-backend-win32 - Windows backend + portable input logic

## OVERVIEW

Win32 backend with host-portable delivery/input/stop-path logic split out for testing on any OS; Windows-only backend, UI Automation AX, capture, and integrity modules are cfg-gated. Earned its file at score 12: distinct platform domain with the richest portable-logic surface.

## STRUCTURE

```
src/
├── lib.rs       # exports delivery, input, stop_path, DEFAULT_STOP_CHORD (+ Win32Backend, HotkeyListener on Windows)
├── delivery/    # toolkit/event compatibility matrix, class recognition, text units
├── input/       # portable held/keys/messages + win32 dispatch, barrier, recovery, cursor_placement, typing_progress
├── capture/     # per-monitor-v2 geometry and compositing
├── ax/          # UI Automation
└── stop_path/   # chord vocabulary/parsing; HotkeyListener (Windows-only)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Route selection | `src/input/dispatch.rs` | Input routing and held-state recovery |
| Background pointer | `src/input/background_pointer.rs` | Hit-testing without foreground |
| Foreground sync | `src/input/barrier.rs` | Delivery barrier, focus guard |
| Delivery matrix | `src/delivery.rs` | `EventKind`, `would_be_silently_dropped`, `posts_double_click` |
| Capture geometry | `src/capture/frame.rs` | Logical vs physical pixel conversion |
| UIA integration | `src/ax/mod.rs` | UI Automation backend |
| Live probes | `live_tests.rs` | Ignored by default; print `key=value` facts |

## CONVENTIONS

- Pure portable logic is separated from Win32 and tested on every host; runtime code uses `#[cfg(target_os = "windows")]`, some native logic is test-visible via `#[cfg(any(test, target_os = "windows"))]`.
- Display/window geometry is global logical coordinates; captured pixels and Win32 input use physical desktop pixels; monitor DPI is metadata, not an atlas transform.
- Capture uses per-monitor-v2 awareness.
- Live probes require an interactive Windows desktop and run with `-- --ignored --nocapture`.

## ANTI-PATTERNS

- UI Automation must never spawn PowerShell.
- Background delivery must not silently fall back to foreground input; unsupported toolkit/event combinations return `BackgroundUnavailable` and never touch the foreground or cursor.
- Window-targeted input refuses elevated windows; foreground input uses a focus guard and delivery barrier.
- Never cache `IUIAutomation` across threads: it is not `Send`; COM initialization and UIA client use stay on the session thread.

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-win32
cargo test -p senpi-desktop-backend-win32                          # portable + host tests
cargo test -p senpi-desktop-backend-win32 -- --ignored --nocapture # live probes (interactive Windows)
```
