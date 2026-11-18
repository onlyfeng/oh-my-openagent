# senpi-desktop-backend-macos

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The macOS backend - the largest backend crate: AXUIElement accessibility, ScreenCaptureKit capture, CG event input, TCC permissions, responsible-process resolution, and the global stop-path event tap. Objective-C lives in `native/` behind `build.rs`.

## STRUCTURE

| Path | Role |
|------|------|
| `src/backend.rs`, `src/backend/` | `MacosBackend`: capabilities, window raising. |
| `src/ax/` | AXUIElement-based `AxBackend` implementation. |
| `src/capture/` | ScreenCaptureKit route (`sck/client.rs`), the staged signed capture helper, scale/DPI handling; live tests under `live_tests.rs`. |
| `src/input/` | `MacInput` and the input stack: `keys.rs`, `keymap.rs`, `remote.rs` (relays), `user_monitor/`, `canary/`, `live/` (~33 files). |
| `src/stop_path/` | Global stop hotkey event tap: `tap.rs` re-arm ladder, then park until restart. |
| `src/permissions/`, `src/responsible/` | TCC permission state; responsible-process identity - never guessed, rejects unsafe/unbounded plist reads. |
| `src/launch_gate/`, `src/overlay/`, `src/skylight/`, `src/release_helper/`, `src/power.rs`, `src/focus.rs`, `src/front_app.rs`, `src/epoch_sources.rs`, `src/parent_death.rs`, `src/cursor.rs`, `src/screen.rs` | Launch gating, overlay UI, Skylight calls, key-release helper, power assertions, focus/front-app tracking, epoch sources, parent-death watch. |
| `native/senpi-desktop-capture-helper.m` | The Objective-C capture helper. |
| `build.rs` | Compiles + ad-hoc signs the helper for local builds; a staged signed helper is required when the release env var is set. |
| `examples/tcc-responsible.rs` | Runnable TCC / responsible-process probe. |
| `tests/tagged_events/`, `tests/fixtures/` | Swift tagged-event fixtures + the Rust scanner/lexer/driver that consumes them. |

## CONVENTIONS

- Visibility is `pub(crate)` / `pub(super)`; only the backend surface leaves the crate. Only `CanaryMode` / `CanaryResult` re-export publicly from `input`.
- Live desktop checks are grouped under `live/` and stay `#[ignore]`d exact-named.
- The receipt canary is a grant-free probe; its dialog never routes input globally.
- Reading the raw user-input feed is non-consuming - it must not take presses away from the watch's feed.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-macos --locked          # on macOS
cargo clippy -p senpi-desktop-backend-macos --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
