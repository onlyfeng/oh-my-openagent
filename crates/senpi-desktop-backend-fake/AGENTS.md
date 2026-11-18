# senpi-desktop-backend-fake

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v2 deterministic in-process backend for tests: `FakeBackend` implements the core `Backend` / `AxBackend` traits from one scenario JSON, records every OS-level side effect into a `RecordingSink`, and runs each scenario under every persona. It makes no OS calls.

## STRUCTURE

| Path | Role |
|------|------|
| `src/` | `FakeBackend`, `FakeClock`, `Fault`, personas, scenario parsing, scripting, timelines, stop-path seam. |
| `fixtures/` | 24 scenario JSONs: the v1 behaviors (`menus`, `delay-5000ms`, `error-codes-amend-*`, `user-intervened`, ...) and the `v2-*` family (`v2-ax-web`, `v2-ax-dense`, `v2-epoch-*`, `v2-frame-identity*`, `v2-scripting`, `v2-trace-secure-field`, `v2-user-active-in-target`, ...). |
| `tests/` | Scenario-driven integration tests (menus, scripting, faults). |

## CONVENTIONS

- Scenario JSON mixes camelCase desktop geometry (`pixelWidth`, `isPrimary`) with snake_case accessibility fields (`native_role`) - it mirrors the wire shapes it fakes. Load via `FakeScenario::load` with `CARGO_MANIFEST_DIR`.
- `v2-ax-web.json` is the dense-web oracle: a 592-node AX tree, depth 19 through anonymous `group` wrappers. Skeleton output must fit the 2,000-token budget while keeping navigation links at logical depth 3, and drill-down cuts must reach every interactive node.
- New test behavior goes into a scenario file, not a special case in `FakeBackend`.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-fake --locked
cargo clippy -p senpi-desktop-backend-fake --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
