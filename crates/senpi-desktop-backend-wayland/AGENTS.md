# senpi-desktop-backend-wayland

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The Wayland backend (Linux-gated): XDG desktop portals via `ashpd` (screenshot / remote desktop), `reis` for input, `zbus` for D-Bus, and dynamic `libloading` for compositor-specific paths. Accessibility comes from `senpi-desktop-backend-atspi`; clipboard goes through core's `system-clipboard` feature (enabled here).

## STRUCTURE

| Path | Role |
|------|------|
| `src/backend.rs`, `src/backend/` | `Backend` contract implementation. |
| `src/portal/` | XDG desktop portal flows (`ashpd`). |
| `src/capture/` | Portal-based capture. |
| `src/input/` | `reis`-based pointer/keyboard input; XKB keymap handling. |
| `src/compositor/` | Compositor-specific behavior behind a dynamic-loading seam. |
| `src/stop_path/` | Wayland stop-chord listener. |
| `src/test_support/`, `src/live_tests.rs` | Test scaffolding + `#[ignore]d live probes. |
| `testdata/` | `fr.xkb`, `us-fr.xkb` (XKB keymap fixtures), `gnome-windows.json` (GNOME window-list fixture) - all embedded with `include_str!`. |

## CONVENTIONS

- Test inputs live in `testdata/` and are compile-time embedded; never read them at runtime.
- Compositor quirks stay inside `compositor/` behind the smallest possible seam.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-wayland --locked        # on Linux
cargo clippy -p senpi-desktop-backend-wayland --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
