# senpi-desktop-backend-macos - macOS native backend

## OVERVIEW

macOS-only (`#![cfg(target_os = "macos")]`) `rlib` implementing `Backend`/`AxBackend`: `screencapture` capture, SkyLight background input, CGEventTap stop path, focus restoration, and non-prompting permission probes. Earned its file at score 10: distinct platform domain with the strictest event-routing invariants in the workspace.

## STRUCTURE

```
src/
├── backend.rs     # wires capture/input/ax/skylight/stop_path + focus/cursor/permissions
├── capture/       # screencapture + display-only CoreGraphics fallback
├── input/         # background (SkyLight), global (CGEventPost), canary, keys
├── skylight/      # private SkyLight FFI: SLEventPostToPid
├── stop_path/     # CGEventTap kill switch, re-arm lifecycle
├── ax/            # accessibility implementation behind MacAx
└── examples/tcc-responsible.rs
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Public facade | `src/lib.rs` | `MacosBackend`, `MacAx`, `is_trusted`, `CgEventTapListener`, `DEFAULT_STOP_CHORD` (ctrl+opt+cmd+escape) |
| Stop-path tap | `stop_path/tap.rs` (290 LOC) | Tied largest file with `focus.rs`; tap lifecycle and re-arm |
| Canary runs | `input/canary.rs` (288) | `CanaryMode`/`CanaryResult` |
| Focus restore | `focus.rs` (290), `front_app.rs`, `launch_services.rs` | |
| Live desktop tests | `live_tests.rs` modules | Separate from colocated unit tests |

## CONVENTIONS

- Narrow public facade: explicit `pub use` at the crate root; implementation modules stay private.
- Permission capability probes never prompt; actionable errors are produced on denied actions.
- Native APIs go through framework crates or `#[link] unsafe extern "C"` with local SAFETY comments.
- Tests colocate (`tests.rs`, inline modules, `#[path]` includes); live tests are named `live_tests.rs`.

## ANTI-PATTERNS

- Background delivery must never route through `CGEventPost(kCGHIDEventTap)`; it posts via SkyLight `SLEventPostToPid` only, with the documented dual-post exception.
- `input/global.rs` is the only path allowed to call `CGEventPost(kCGHIDEventTap)`.
- Capture never prompts for permission and never uses ScreenCaptureKit.
- Permission probes never open privacy settings; that is reserved for denied actions and cached once per permission.
- `responsible.rs` Info.plist reads are bounded: reject non-regular/oversized input, never re-read or substitute.

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-macos   # macOS only (cfg-gated)
cargo test -p senpi-desktop-backend-macos
```
