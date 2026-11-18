# src - Wayland backend implementation

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The live Wayland backend tree (63 files, ~11.5k LOC), pure Rust - no libxkbcommon, no PipeWire C APIs. Deep map of the subtrees the crate file sketches; per-compositor geometry is where the complexity lives.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `backend.rs` (525 LOC) | `WaylandBackend`: capabilities, input/capture orchestration, compositor detection (cached). Honest background-only capabilities. |
| `backend/` (tests only) | Nine per-concern test modules: `gnome_tests`, `kwin_tests`, `hyprland_tests`, `eis_tests`, `eis_safety_tests`, `portal_tests`, `capture_tests`, `layout_tests`, `tests`. |
| `compositor/kwin.rs` (857), `compositor/gnome.rs` (592), `compositor/hyprland.rs` + `hyprland/` | Per-compositor window/geometry readers: KWin via a temporary read-only script, GNOME Shell extension, Hyprland IPC. `parse_windows` / `parse_stack` / `place` / `Detection`. |
| `input/libei.rs` (370), `input/xkb/`, `input/keymap.rs` | libei device lifecycle, group-aware XKB keymap resolution, holds/burst/preflight. Lazy: connected on first input via `LIBEI_SOCKET` or a RemoteDesktop portal session. |
| `capture/` | `screenshot_portal.rs` primary; `pipewire.rs` (ScreenCast when dynamically available), `layout.rs` (scale/geometry evidence), `eis_region.rs`. |
| `portal/`, `stop_path/` | XDG portal flows; GlobalShortcuts stop chord (`STOP_SHORTCUT_ID`, `STOP_PATH_UNAVAILABLE`). |
| `test_support/`, `live_tests.rs` | Hermetic scaffolding + ignored live probes. |

## CONVENTIONS

- Geometry readers are READ-ONLY: Hyprland IPC, a temporary read-only KWin script, a GNOME Shell extension - actual pointer/keyboard injection always stays on libei.
- Capture scale is evidence-derived, never assumed: `capture/layout.rs` accepts an image only when it matches the layout at ONE uniform scale (candidates from each region's reported scale, `UNSET_SCALE_CANDIDATES` `[1.0, 2.0]`, `SCALE_EPSILON` 1e-3); without that evidence frames are pixels-only.
- Compositor detection and portal capability probes are cached after first use.
- Capture prefers ScreenCast/PipeWire where dynamically available and falls back to the Screenshot portal.

## ANTI-PATTERNS

- Never inject input through a compositor shell API (KWin script / GNOME extension) - those channels are geometry-only.
- Never report a capture scale without layout evidence.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-wayland --locked        # on Linux
cargo clippy -p senpi-desktop-backend-wayland --all-targets --locked -- -D warnings
```

Parent: [`crates/senpi-desktop-backend-wayland/AGENTS.md`](../AGENTS.md).
