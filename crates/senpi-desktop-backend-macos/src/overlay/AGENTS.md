# overlay - cursor-overlay helper

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Optional display-only cursor overlay shared by native input, AX, and capture (16 files, ~1.5k LOC). Earned its file via cross-crate centrality: `Overlay` is consumed from `backend`, `capture`, `ax`, and the input paths.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `mod.rs` | `Overlay` handle + `configure` / `configure_for`: startup policy (see convention below). |
| `helper.rs` | The helper process entry (`pub(crate) run`) - spawned per session, not linked into the engine binary. |
| `client.rs` | `Helper`: spawns the process and owns its transport. |
| `wire.rs`, `panel.rs`, `motion.rs` | The framed serde `Command` protocol; the helper's panel window; pointer motion animation. |
| `coordination.rs` | Per-user runtime lock (`pub(crate) lock_path`) so one overlay helper exists per user. |

## CONVENTIONS

- WindowServer identity is fixed: `WINDOW_TITLE = "senpi-desktop-cursor-overlay"` at `WINDOW_LEVEL = 1000` - together they identify a helper; do not change without migrating detection.
- The helper is started ONLY when per-frame capture exclusion is supported (macOS 14+). Without that API every screenshot taken while the helper is up would fail, so `configure_for(enabled, exclusion_supported)` keeps it down and prints an unavailability note to stderr instead.
- Communication is the framed wire protocol; coordination goes through the per-user runtime lock with bounded waits.

## ANTI-PATTERNS

- Never start the overlay on an OS without per-frame capture exclusion - it breaks capture, it does not degrade.
- The overlay is display-only: never route input, focus, or AX semantics through it.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-macos --locked          # on macOS
cargo clippy -p senpi-desktop-backend-macos --all-targets --locked -- -D warnings
```

Parent: [`crates/senpi-desktop-backend-macos/AGENTS.md`](../../AGENTS.md).
