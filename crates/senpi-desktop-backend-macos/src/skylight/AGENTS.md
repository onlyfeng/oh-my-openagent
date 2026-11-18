# skylight - WindowServer event posting

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Background event delivery through WindowServer's private SkyLight SPI: per-window event stamping, dual posting, focus-without-raise records, and the foreground transaction (6 files, ~0.9k LOC). Highest-centrality helper tree in the macOS crate - consumed from `backend`, `input`, `capture`, and focus paths.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `mod.rs` | Posting surface: `stamp_event` (phase, click state, button, target pid/window, click group, window-local coords), the foreground transaction, `require_spi()`. |
| `spi.rs` | Dynamic symbol resolution of the SkyLight SPI; exact function-pointer signatures; `capture_spi` / `front_pid` / `is_available`. |
| `psn.rs` | `ProcessSerialNumber` plumbing, `front_process`, focus records. |
| `focus_records.rs` | `post_key_records`: ordered focus-priming records - no defocus record in the sequence. |
| `partial.rs` | `mark_sent` / `reporting_partial`: the partial-delivery bookkeeping behind the anti-patterns below. |
| `authentication.rs` | Per-process authentication stamping. |

## ANTI-PATTERNS (the posting rules)

- The Background path must NEVER route through `CGEventPost(kCGHIDEventTap)`; posting is `SLEventPostToPid` (plus the public per-pid supplement for plain AppKit) only.
- `post_dual`'s public post is a SUPPLEMENT, never a fallback - do not use it to recover a failed SkyLight post.
- Keyboard events are SkyLight-only: posting keys through both routes would deliver each key twice.
- Once a background call has sent input, never translate a later `BackgroundUnavailable` into a foreground rerun - report `InputFailed` instead, because the input may have landed.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-macos --locked          # on macOS
cargo clippy -p senpi-desktop-backend-macos --all-targets --locked -- -D warnings
```

Parent: [`crates/senpi-desktop-backend-macos/AGENTS.md`](../../AGENTS.md).
