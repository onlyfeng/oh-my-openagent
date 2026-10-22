# crates/ - Rust computer-use engine workspace

**Generated:** 2026-10-22 / f9de7acf36

## OVERVIEW

The Rust side of omo's `computer` tool: a 12-crate Cargo workspace producing the `senpi-desktop-engine` binary that the TypeScript packages `packages/senpi-desktop-{protocol,engine,service,tool,prelude}` drive over a frozen JSON-RPC contract. Crate naming follows the senpi integration convention (like `senpi-task`); the design is tracked in code-yeongyu/oh-my-openagent#8893.

Member list lives in the root `Cargo.toml` `[workspace] members`; `crates/desktop/README.md` is the human-facing crate-role map and is not duplicated here.

## STRUCTURE

| Crate | Role |
|-------|------|
| `senpi-desktop-core` | Types, errors, frames, keys, AX registry, backend traits, and the frozen JSON-RPC contract. Has `schema/`, `docs/`, `fixtures/`. |
| `senpi-desktop-safety` | Fail-closed `Supervisor`, stop-path registry, `gate()`. |
| `senpi-desktop-session` | Session loop, the `mutate()` choke point, screenshot budget, audit. |
| `senpi-desktop-engine` | The binary: wires a session to a backend and serves the RPC contract. |
| `senpi-desktop-backend-fake` | Deterministic in-process backend for tests (`fixtures/`). |
| `senpi-desktop-backend-macos` | macOS backend; `native/` + `build.rs` for the Objective-C/C bridge; has `examples/`. |
| `senpi-desktop-backend-x11` | X11/XTEST backend (CI runs it under `xvfb-run`). |
| `senpi-desktop-backend-atspi` | AT-SPI accessibility backend for Linux (CI runs it under `dbus-run-session` + `xvfb-run`). |
| `senpi-desktop-backend-wayland` | Wayland backend (`testdata/`). |
| `senpi-desktop-backend-win32` | Windows backend (CI QA job `desktop-windows-qa.yml`). |
| `senpi-desktop-xpc` | macOS XPC transport (`TRANSPORT.md`, `scripts/`). |
| `desktop` | Docs-only crate holding the README crate map. |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Crate roles and platform support | `crates/desktop/README.md` (the authoritative map) |
| Frozen RPC contract / frames | `crates/senpi-desktop-core/src/` (+ `schema/`) |
| Approval and stop semantics | `crates/senpi-desktop-safety/src/` |
| Screenshot budget and audit | `crates/senpi-desktop-session/src/` |
| Adding a platform backend | Implement the traits from `senpi-desktop-core`, add a workspace member, register it in `senpi-desktop-engine` |
| TS side that consumes it | `packages/senpi-desktop-{protocol,engine,service,tool}/` |

## CONVENTIONS

- **Every dependency is an exact `=` pin.** `bun script/check-cargo-pinned-deps.mjs` (wired into `bun run test:desktop`) fails on an unpinned or caret/tilde version anywhere in the workspace.
- **The JSON-RPC contract is frozen.** Changing a frame, key, or method means changing `senpi-desktop-core` and every TS consumer together; the TS types in `packages/senpi-desktop-protocol` must match.
- **Fail closed.** Safety decisions live in `senpi-desktop-safety` and the `mutate()` choke point; a backend never bypasses them.
- **Live (display-dependent) tests are `#[ignore]` and exact-named.** CI runs them under `xvfb-run` / `dbus-run-session` on Linux and in the Windows QA job; a normal `cargo test` stays hermetic.
- **Platform backends stay behind the trait boundary.** No platform `cfg` branches leak into core, safety, or session.

## COMMANDS

```bash
# from the repo root
cargo test --workspace --locked                 # hermetic unit/integration suite
cargo clippy --workspace --all-targets --locked -- -D warnings
cargo build --release -p senpi-desktop-engine --locked

# the TS half of the same surface
bun run test:desktop
```

CI: `.github/workflows/desktop-engine.yml` (Linux clippy + test + per-platform release builds, plus the ignored live tests under `xvfb-run` / `dbus-run-session`) and `.github/workflows/desktop-windows-qa.yml` (Windows backend QA).

## ANTI-PATTERNS

- Never introduce an unpinned or workspace-inherited dependency version in a crate.
- Never change a frame or method name on one side of the contract only.
- Never add a stop-path or approval shortcut outside `senpi-desktop-safety`.
- Never make a hermetic test depend on a display, a live accessibility bus, or a real desktop session.
