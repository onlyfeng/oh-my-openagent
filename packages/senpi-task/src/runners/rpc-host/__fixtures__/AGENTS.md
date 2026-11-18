# src/runners/rpc-host/__fixtures__/ - Shared Fake-Host Fixture Seam

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 15 (14 files; distinct domain - the fixture seam imported by rpc-host suites and by task tests outside this directory).

## OVERVIEW

Fake host/transport/wire/session behavior plus controlled recovery clocks: the single source of expectation shared by independent RPC-host suites.

## WHERE TO LOOK

| File | Role |
|------|------|
| `fake-host.ts` | `FakeHost` - the hub fake |
| `fake-host-transport.ts` / `fake-host-wire.ts` / `fake-host-sessions.ts` | Transport, wire, and session fakes layered under `FakeHost` |
| `fake-host-probe.ts` | `fakeProtocolInfo` / `probeFakeHost` |
| `fake-input-admissions.ts` | `FakeInputAdmissions` - input admission fake |
| `host-world.ts` / `host-world-ports.ts` / `host-world-recovery.ts` | Controlled host world + ports + recovery behavior |
| `manual-recovery-clock.ts` | Controlled clock for recovery timing |
| `cancel-lane.ts` | `startCancelLane` - cancellation lane fixture |
| `daemon-launch.ts` | Shared daemon launch inputs - the single source keeping independent suites consistent |
| `register-stores.ts` / `ask-host-default-root.ts` | Store registration, default ask-host root |

## CONVENTIONS

- Fixtures favor isolated real local processes/sockets and controlled clocks; no timing luck.
- Fixtures are seams, not assertions: suites import them directly (no barrel) - keep exports stable when refactoring.

## ANTI-PATTERNS

- Do not fork a private copy of a fixture into a suite - a divergent copy defeats the shared-expectation purpose; extend the shared fixture instead.
- Do not let a fixture leak state across tests; each suite builds its own world via these helpers.

## COMMANDS

```bash
bun test packages/senpi-task/src/runners/rpc-host/     # consumers of these fixtures
bun run --cwd packages/senpi-task typecheck
```
