# crates/ - Rust computer-use engine workspace

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The Rust side of omo's `computer` tool. TWO crate families coexist while the v3 rewrite lands:

- **`senpi-desktop-*` (v2, shipping)** - 11 crates behind the `senpi-desktop-engine` binary that the TypeScript packages `packages/senpi-desktop-{protocol,engine,service,tool,prelude}` drive over a frozen JSON-RPC contract (design: code-yeongyu/oh-my-openagent#8893).
- **`omo-computer-*` (v3, port in progress)** - 10 crates mirroring the same layering (core / safety / session / per-OS backends / engine) behind the `omo-computer-engine` binary. `omo-computer-backend-{atspi,x11,win32}` are still stubs; the port is tracked row-by-row in `omo-computer-core/parity/test-parity-manifest.tsv`.

Plus `senpi-desktop-xpc` (macOS XPC transport), `omo-engine-trust-helper` / `omo-git-job-helper` (sandboxed helpers), and `desktop` (docs-only README crate map). Member list lives in the root `Cargo.toml` `[workspace] members`; `crates/desktop/README.md` is the human-facing v2 crate-role map and is not duplicated here.

## STRUCTURE

| Crate | Role |
|-------|------|
| [`senpi-desktop-core`](senpi-desktop-core/AGENTS.md) | Types, errors, frames, keys, AX registry, backend traits, and the frozen JSON-RPC contract. Has `schema/`, `docs/`, `fixtures/`. |
| [`senpi-desktop-safety`](senpi-desktop-safety/AGENTS.md) | Fail-closed `Supervisor`, stop-path registry, `gate()`. |
| [`senpi-desktop-session`](senpi-desktop-session/AGENTS.md) | Session loop, the `mutate()` choke point, screenshot budget, audit. |
| [`senpi-desktop-engine`](senpi-desktop-engine/AGENTS.md) | The binary: wires a session to a backend and serves the RPC contract. |
| [`senpi-desktop-backend-fake`](senpi-desktop-backend-fake/AGENTS.md) | Deterministic in-process backend for tests (`fixtures/`). |
| [`senpi-desktop-backend-macos`](senpi-desktop-backend-macos/AGENTS.md) | macOS backend; `native/` + `build.rs` for the Objective-C/C bridge; has `examples/`. |
| [`senpi-desktop-backend-x11`](senpi-desktop-backend-x11/AGENTS.md) | X11/XTEST backend (CI runs it under `xvfb-run`). |
| [`senpi-desktop-backend-atspi`](senpi-desktop-backend-atspi/AGENTS.md) | AT-SPI accessibility backend for Linux (CI runs it under `dbus-run-session` + `xvfb-run`). |
| [`senpi-desktop-backend-wayland`](senpi-desktop-backend-wayland/AGENTS.md) | Wayland backend (`testdata/`). |
| [`senpi-desktop-backend-win32`](senpi-desktop-backend-win32/AGENTS.md) | Windows backend (CI QA job `desktop-windows-qa.yml`). |
| `senpi-desktop-xpc` | macOS XPC transport (`TRANSPORT.md`, `scripts/`). |
| `desktop` | Docs-only crate holding the README crate map. |
| `omo-computer-core` / `omo-computer-safety` / `omo-computer-session` / `omo-computer-engine` | v3 twins of core/safety/session/engine; the module map lives in [`omo-computer-core/src/AGENTS.md`](omo-computer-core/src/AGENTS.md). |
| [`omo-computer-backend-fake`](omo-computer-backend-fake/AGENTS.md), `omo-computer-backend-{macos,wayland}` | v3 backend twins (fake = scripted scenarios; macos/wayland ported). |
| `omo-computer-backend-{x11,atspi,win32}` | v3 stubs; the live implementations are still the `senpi-desktop-backend-*` twins. |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Crate roles and platform support | `crates/desktop/README.md` (the authoritative map) |
| Frozen RPC contract / frames | `crates/senpi-desktop-core/src/` (+ `schema/`) |
| Approval and stop semantics | `crates/senpi-desktop-safety/src/` |
| Screenshot budget and audit | `crates/senpi-desktop-session/src/` |
| Adding a platform backend | Implement the traits from `senpi-desktop-core`, add a workspace member, register it in `senpi-desktop-engine` |
| TS side that consumes it | `packages/senpi-desktop-{protocol,engine,service,tool}/` |
| v3 contract module map | `crates/omo-computer-core/src/AGENTS.md` |
| v2→v3 test port state | `crates/omo-computer-core/parity/` (guard: `bun script/computer-v3-guards.ts --parity`) |

## CONVENTIONS

- **The rules below bind BOTH families.** Pinning, contract freeze, fail-closed, hermetic tests, and the trait boundary apply to `senpi-desktop-*` and `omo-computer-*` alike.
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

# v3 parity guard (pending rows allowed; --final rejects them)
bun script/computer-v3-guards.ts --parity
```

CI: `.github/workflows/desktop-engine.yml` (Linux clippy + test + per-platform release builds, plus the ignored live tests under `xvfb-run` / `dbus-run-session`) and `.github/workflows/desktop-windows-qa.yml` (Windows backend QA).

## ANTI-PATTERNS

- Never introduce an unpinned or workspace-inherited dependency version in a crate.
- Never change a frame or method name on one side of the contract only.
- Never add a stop-path or approval shortcut outside `senpi-desktop-safety`.
- Never make a hermetic test depend on a display, a live accessibility bus, or a real desktop session.
