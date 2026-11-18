# omo-computer-core/src - the v3 contract modules

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The contract root of computer-use v3: method table, params, requests, results, receipts, frames, refs, traits and tables (19 public modules - the crate every v3 crate depends on). Crate-level facts live in [`omo-computer-core`](../AGENTS.md); this file maps `src/`.

## WHERE TO LOOK

| Module | Role |
|--------|------|
| `backend/` | `Backend` / `AxBackend` traits. Every method has a REFUSING DEFAULT: a backend overrides exactly what it supports and the rest answers a typed `Refusal` with nothing sent. The session calls `post_input` only from `mutate()` - it is the one side-effect sink. |
| `methods/` | `Method` / `METHODS` / `MethodSpec` + the notification set. One-shot prefix `desktop.`; alias `desktop.stop` -> `stopPath.stop`. |
| `params/` | Strict serde params for every method (`deny_unknown_fields`, camelCase): a member the contract does not name cannot ride the wire unread. |
| `requests/` | One reader per params type: takes the decoded params by value and destructures with EVERY field named (no `..`). `protocol_test::readers` parses these files and fails on any field a reader binds but never uses, and on any schema member no reader binds (`READER_SOURCES`). |
| `error/` | The one error contract: the `COMPUTER_*` table keyed by `(engineCode, data.reason)`, total over the closed engine table (33 engine codes, 56 computer codes, 144 reasons, 21 layer failures). APPEND-ONLY (`error/code.rs`): never renumber; engine number = `ENGINE_CODE_BASE` - ordinal. Unknown wire values parse into a typed `UnknownFailure`. |
| `report/` | The result contract: `Disposition` (five delivery classes, SEVEN valid delivery/retry pairs, private fields - built only via constructors or wire parsing), `ActionReport` + `evidence[]`, and the by-verb/platform receipt rule. `tests/shape_cases/` pins valid and invalid wire shapes. |
| `ax/` | The observation model: bounded walk + `Limits` (8 generations/window, 65,536 retained objects/session, 3,000 ms deadline, 250 ms read slice, 2,000 nodes, logical depth 24, skeleton depth 3), ref tokens, the outline grammar, and `DiffFallback` reasons. The registry that MINTS refs lives in `omo-computer-session`; the ninth observe expires the oldest generation. |
| `settings/` | The `computer.*` settings table passed in `session.open` (design row C17): `Settings`, run/fleet limits, table rows/keys/status, typed value enums. |
| `capabilities/` | Capability records: the v1 shape for null-mode sessions and the object-mode shape. |
| `schema.rs` + `../schema/` | `engine_schema()` / `engine_schema_text()` / `SCHEMA_PATH`. `schema/engine.schema.json` is GENERATED - regenerate with `OMO_COMPUTER_UPDATE_SCHEMA=1 cargo test -p omo-computer-core` and never hand-edit. |
| `results/`, `protocol_test/` | Owned by a sibling scope - see the crate root file. |
| `doc.rs`, `env.rs`, `hello.rs`, `listing.rs`, `observation.rs`, `paths.rs`, `session_state.rs`, `untrusted.rs` | Supporting surfaces: doc rendering, env validation, hello, listing, observation types, path policy, session state, untrusted-input guards. `nullable` is private. |

## CONVENTIONS

- Refusing-default traits: a new backend method starts here as a refusing default, then each backend opts in - the contract never breaks existing backends.
- Reader discipline is enforced by source-parsing tests; extend them whenever you add a params field.
- The error vocabulary is closed and append-only: a new failure appends a row, never reuses or renumbers one.
- Bounded AX reads never claim completeness: preserve the walk state when a deadline/budget truncates the read.

Parent: [`crates/AGENTS.md`](../../../AGENTS.md) - crate root: [`omo-computer-core`](../AGENTS.md).
