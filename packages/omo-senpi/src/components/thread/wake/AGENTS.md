# components/thread/wake/ — Cross-Session Wake

## OVERVIEW

24 files, ~4,780 LOC. Wakes a sleeping/idle session in response to a carrier
event (message, notification, follow-up): engine-version gating, target
resolution, carrier transport, consumer startup, pending sweeps, and legacy
opening. Entry `runtime.ts` (composition) with `wake-runtime.ts` as the
compatibility adapter (`startWakeAtSession`, `createSameDeviceWake`,
`intakeAdmission`). **Score 18** (24 files, transport + lifecycle density).
Parent: [`src/AGENTS.md`](../../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `runtime.ts` | Runtime composition: carrier, connection, consumer, minimum-version, host-ensure |
| `carrier.ts` | `createCarrier` — wake leases, opening, event handling, evaluation, retries, sweeps (hotspot, 531 LOC) |
| `carrier-connection.ts`, `engine-minimum.ts` | Host connection; `MIN_WAKE_ENGINE_VERSION`, `engineDropsRetainedRecord` |
| `session-target.ts`, `session-waker.ts` | `resolveWakeTarget`, `WakeOutcome`, `SessionWaker` |
| `consumer-start.ts`, `pending-sweep.ts` | Delivery consumer startup; pending-record sweeps |
| `engine-state.ts`, `host-ensure.ts` | Engine state; host startup |
| `legacy-open.ts` | `createLegacyWakeOpen` compatibility |
| `testing/` | Fake hosts and fixtures adjacent to implementation |

## CONVENTIONS

- Small single-purpose modules; lifecycle seams (leases, epochs, retries)
  are injected for deterministic tests.
- Engine-version filter runs before any retained-record decision: an engine
  below minimum drops retained records rather than misinterpreting them.

## ANTI-PATTERNS

- Do not bypass the minimum-version gate when adding new record kinds.
- Do not let a wake retry loop spin without the carrier's evaluation/backoff.

## QA

```bash
bun test packages/omo-senpi/src/components/thread/wake
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
