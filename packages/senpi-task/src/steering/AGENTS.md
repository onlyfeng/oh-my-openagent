# steering - Send / Interrupt / Cancel for Live or Resident Children

**Generated:** 2026-11-18 / c2691818a9

**Score:** 21 (25 files, ~4.4k LOC, barrel; the parent [`../../AGENTS.md`](../../AGENTS.md) gives `src/steering/` one line - this file is its detail).

## OVERVIEW

`createSteeringEngine` composes the steering surface over a live or resident child: send (with delivery modes), interrupt, cancel, redispatch after revival, and the durable prelaunch queue. Pure policy over injected ports (`SteeringPort`, `DestructionPort`) with typed outcomes; Senpi-coupled - it consumes the runner/session contracts the task manager owns.

## WHERE TO LOOK

| Task | File |
|------|------|
| Engine composition | `engine.ts` (+ `engine-policy.ts`) - state/lifecycle/runner contracts + policy |
| Send + delivery | `types.ts` - `SendInput` / `SendOutcome` / `SendDelivery`, `DEFAULT_SEND_DELIVERY`; `controls.ts` |
| Durable pending queue | `pending-steering.ts` - pending sends live in the TASK RECORD (single source of truth), drained in persisted order when the child starts |
| Revival | `revive.ts`, `revive-running.ts`, `revival-rollback.ts` |
| Stale / parked runs | `stale-run.ts`, `host-session-cancel.ts` |
| Public surface | `index.ts` - `createSteeringEngine`, `DEFAULT_SEND_DELIVERY`, and the option/outcome/port types (`CancelOptions`, `CancelOutcome`, `DestructionCause`, `DestructionPort`, `InterruptOutcome`, `RedispatchOutcome`, `SteeringEngine`, `SteeringPort`) |

## CONVENTIONS

- Typed outcomes everywhere; dependencies injected, never imported from the manager.
- No in-memory shadow of pending sends - the record is the truth, so crash replay cannot diverge.
- Tests are behavior-named (`cancel-running-abort-budget`, `granted-revival`, `residency-regression`, `revival-failures`, `revival-uncertainty`, `prior-runs-cost`, `engine-run-epoch`, `host-session-parked`) - run the nearest suite on touch.

## ANTI-PATTERNS

- Do not queue steering in memory beside the record - the two copies will diverge.
- Do not treat a revival failure as a lost child without rolling the claim back (`revival-rollback.ts`).

## COMMANDS

```bash
bun test packages/senpi-task/src/steering
```
