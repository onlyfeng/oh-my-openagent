# senpi-desktop-core - Desktop contract crate (traits, frozen wire protocol)

## OVERVIEW

Shared contract for the senpi-desktop stack: `Backend`/`AxBackend` traits, AX registry, frame geometry/encoding, key names, error vocabulary, and the frozen JSON-RPC method table with generated schema/docs. Earned its file at score 17: highest centrality in the workspace - the engine and every backend import it.

## STRUCTURE

```
src/
├── lib.rs         # pub modules: ax, backend, clipboard (feature), error, frame, keys, methods, protocol*, types; engine_schema()
├── methods.rs     # FROZEN wire method table (Effect, Exposure, Method, METHODS, Notification)
├── protocol*.rs   # JSON-RPC request/response/error/notification types, params/results, schemars schema
├── ax/            # registry, handles, operations, snapshot, role normalization
├── types/ frame/ keys/ error.rs backend.rs
docs/engine-protocol.md       # generated protocol doc
schema/engine.schema.json     # generated machine schema
fixtures/conformance/         # JSON scenarios replayed via the fake backend
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Wire method table | `src/methods.rs` | FROZEN; binds wire name, effect, exposure, params, results |
| Schema generation | `src/protocol_schema.rs` + `schema/engine.schema.json` | `engine_schema()` entry |
| Backend traits | `src/backend.rs` | Documented no-op defaults for optional capabilities |
| Error vocabulary | `src/error.rs` | `DesktopError`, `ErrorCode`, `CoreResult`; frozen |
| Conformance corpus | `fixtures/conformance/` | Replayed by `senpi-desktop-backend-fake` |

## CONVENTIONS

- Protocol is JSON-RPC 2.0, one message per NDJSON line; wire structs use serde camelCase/renames and schemars-derived schemas; object schemas reject unspecified fields.
- Clipboard integration is opt-in via the `system-clipboard` feature.
- Optional backend capabilities have no-op trait defaults; where pretending would be unsafe (e.g. clipboard), absence fails explicitly.

## ANTI-PATTERNS

- Never hand-edit `docs/engine-protocol.md` or `schema/engine.schema.json`; regenerate with `SENPI_DESKTOP_UPDATE_SCHEMA=1 cargo test -p senpi-desktop-core`.
- The method table is FROZEN: additions must update schema, protocol docs, and the conformance corpus in the same change.
- The capability field set is FROZEN: populate existing fields instead of adding new ones.
- The error vocabulary is frozen: new codes require a plan amendment (TS protocol and numeric RPC codes derive from them).
- Never guess an accessibility element's owner when the backend returns `AxOwner::Unknown`.

## COMMANDS

```bash
cargo test -p senpi-desktop-core                                   # unit + schema freshness gate
SENPI_DESKTOP_UPDATE_SCHEMA=1 cargo test -p senpi-desktop-core     # regenerate schema + protocol doc
```
