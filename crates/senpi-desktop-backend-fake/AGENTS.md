# senpi-desktop-backend-fake - Deterministic desktop test double

## OVERVIEW

In-memory `Backend`/`AxBackend` implementation driven by JSON scenarios, with fault injection, per-method delays, a controllable clock, and pointer/operation recording; it never touches an OS API. Earned its file at score 9: distinct domain (the workspace-wide deterministic test double that also replays `senpi-desktop-core` conformance fixtures).

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Crate facade | `src/lib.rs` | Re-exports `FakeBackend`, `FakeScenario`, `FakeClock`, `Faults`, `RecordingSink`, and friends |
| Scenario format | `src/scenario.rs` (281 LOC) | JSON parsing, capability overlays, window-reference validation |
| Backend behavior | `src/desktop_impl.rs` (271 LOC) | Fault/delay application, input, capture, recording |
| AX traversal | `src/` ax modules | `AxTree` traversal and mutation over `FakeAxNode` |
| Integration tests | `tests/{ax,capture,input,scenario}.rs` | Shared helpers in `tests/common/mod.rs` |
| Fixtures | `fixtures/*.json` | Scenario inputs |

## CONVENTIONS

- Scenario JSON uses snake_case top-level keys but preserves camelCase inside embedded core wire types; selected structs deny unknown fields.
- Partial capability overlays merge onto `fake_capabilities`; unknown capability keys are rejected.
- Delays/failures are explicit per method: `DelayMs::for_method`, `fail_next`; methods documented as non-delayed/non-failable must stay that way.
- Unit tests live in the library; integration tests live under `tests/` with `tests/common` helpers.

## ANTI-PATTERNS

- Never call an OS API from this crate; every side effect goes through `RecordingSink` (`SinkOp`) for inspection.
- Never delay or fail a method marked non-delayed/non-failable.
- Never accept unknown scenario fields, capability keys, or window references; reject them with `ScenarioError`.

## COMMANDS

```bash
cargo check -p senpi-desktop-backend-fake
cargo test -p senpi-desktop-backend-fake
```
