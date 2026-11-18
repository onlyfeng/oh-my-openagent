# senpi-desktop-backend-win32

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The Windows backend: UI Automation for AX, per-monitor-v2 DPI-aware capture, Win32 input routes, native menus, and the global hotkey stop path. Pure policy modules compile and test on every host; native code is `cfg(target_os = "windows")`.

## STRUCTURE

| Path | Role |
|------|------|
| `src/lib.rs` | Exports `Win32Backend` + `HotkeyListener` (Windows) and `DEFAULT_STOP_CHORD`, `BACKEND_NAME` (all hosts). |
| `src/backend.rs` | The `Backend` / `AxBackend` contract: per-monitor-v2 awareness; global logical display/window coordinates vs physical capture/input pixels. |
| `src/delivery.rs` | PURE compatibility/refusal policy: toolkit-class gating for background `PostMessageW` delivery, `EventKind` / `TargetTraits` / `TextUnit`, UIPI rules. Host-testable. |
| `src/input/` | `dispatch.rs` route selection + held-state lifecycle; `background.rs` background delivery; `held` / `keys` / `messages` pure modules. |
| `src/ax/` | UIA backend - "Never spawns PowerShell." |
| `src/capture/` | Screen capture + DPI handling (`dpi.rs`, `frame.rs`). |
| `src/menus/` | Native menus: attached HMENU preferred, bounded UIA traversal when absent. |
| `src/stop_path/` | `chord.rs` (`Hotkey`, `parse`), `hotkey.rs`, `hook.rs`, `listener.rs`: kill-switch lifecycle. |
| `src/integrity.rs` | Target integrity checks. |

## ANTI-PATTERNS (THIS CRATE)

- Never deliver background input to unsupported toolkit classes or as modifier chords - refuse instead of silently losing it. UIPI-elevated targets are refused (Windows can drop their input unreported).
- No `AttachThreadInput` fallback: a refused foreground request is an error, never a silent send to whatever window is frontmost.
- Menu traversal and UIA menu discovery stay bounded; command selection revalidates the exact target before dispatch.
- UIA names are already decoded - never run them through the native menu-label parser; the UIA command adapter avoids `SelectionItem` and `ExpandCollapse`.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-win32 --locked          # pure policy on any host
cargo clippy -p senpi-desktop-backend-win32 --all-targets --locked -- -D warnings
# ignored interactive probes (machine-readable key=value evidence) - interactive Windows desktop only:
cargo test -p senpi-desktop-backend-win32 --locked -- --ignored --nocapture
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
