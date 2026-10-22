# senpi-desktop-core

**Generated:** 2026-10-22 / f9de7acf36

## OVERVIEW

The shared contract layer of the desktop (computer-use) engine: wire types, errors, capture frames, key names, the accessibility-node registry, and the `Backend` / `AxBackend` traits every platform backend implements. Every other crate in the workspace depends on this one; it depends on none of them.

The JSON-RPC contract is FROZEN. `schema/engine.schema.json` is the machine-readable schema, `src/protocol_schema.rs` embeds it, and `protocol_conformance_test.rs` / `protocol_test.rs` / `protocol_holds.rs` pin it. Changing a type, method, or field here is a cross-package change: the TypeScript types in `packages/senpi-desktop-protocol` must move in the same commit.

## STRUCTURE

| Path | Role |
|------|------|
| `src/protocol.rs`, `src/methods.rs`, `src/protocol_params.rs`, `src/protocol_results.rs` | The wire protocol: envelope, method names, per-method params and results. |
| `src/protocol_schema.rs`, `schema/engine.schema.json`, `src/protocol_validator.rs` | Schema generation, the checked-in schema, and runtime validation. |
| `src/protocol_holds.rs` | Invariants that must hold across every method (the conformance harness). |
| `src/backend.rs` | The `Backend` / `AxBackend` traits plus `DeliveryMode`; the platform boundary. |
| `src/types/` | Target kinds, coordinate frames, tiers, approvals, capability descriptors. |
| `src/frame/`, `src/ax/`, `src/keys/`, `src/menus/`, `src/holds/` | Capture frames, the AX node registry, key/chord parsing, menu trees, hold bookkeeping. |
| `src/error.rs` | `DesktopError` / `ErrorCode` / `TccPermission`: the single error vocabulary all backends return. |
| `src/clipboard.rs` | Clipboard access behind the `system-clipboard` feature (off by default so hermetic tests never touch the real clipboard). |
| `docs/engine-protocol.md` | The prose description of the protocol. |

## CONVENTIONS

- **Contract first.** A new capability starts as a type + method here, then reaches the backends, then the engine, then the TypeScript client - never the other way around.
- **Errors are enumerated.** Return `DesktopError` with an existing `ErrorCode`; never invent an untyped string or panic on a user-facing failure path.
- **No I/O in core.** This crate describes and validates; backends perform. Anything that touches the OS belongs in a backend crate.
- **Feature-gate host access.** Real clipboard access sits behind `system-clipboard`; default builds stay hermetic.
- **Schema is checked in.** Regenerate `schema/engine.schema.json` and keep the conformance tests green in the same change as the Rust edit.

## COMMANDS

```bash
cargo test -p senpi-desktop-core --locked
cargo test -p senpi-desktop-core --features system-clipboard --locked --lib -- --ignored
cargo clippy -p senpi-desktop-core --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
