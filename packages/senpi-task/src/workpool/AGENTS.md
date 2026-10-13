# src/workpool - Keyed Workpool Engine

Engine behind the `workpool` tool (`src/tools/workpool.ts` + `workpool-schema.ts`): a named pool of agent workers drains keyed items, workers return results through `workpool yield`, and the parent gets one aggregate on close. 18 implementation files (about 1.15k LOC), 9 tests, 6 `__fixtures__/` scenario files; no `index.ts` barrel, consumers import modules directly (`createWorkpoolEngine` is wired in `manager/manager.ts`). Earned this file: distinct domain with durable state, recovery rules, and a parent-capability boundary. Manager-side admission/launch lives in [`../manager/AGENTS.md`](../manager/AGENTS.md).

## MODULE MAP

| Path | Purpose |
|------|---------|
| `engine.ts` | `createWorkpoolEngine`: create / push / close / inspect / cancel / yield / attach / dispose; composes store, dispatcher, worker capability, kernel-tool grants, aggregate delivery. |
| `dispatcher.ts` | `createWorkpoolDispatcher`, `orderIdleWorkers`: scheduling, admission, assignment binding, completion observation, cancellation, reconciliation. Wakes are events after one handoff, never polling. |
| `store.ts` / `record-schema.ts` | `createWorkpoolStore(stateDir)`: durable pool records under `<stateDir>/workpools`, zod strict schema, directory lock serializing parent/name creation across hosts. |
| `types.ts` / `schema.ts` / `default-mode.ts` | `PoolId` (`wp_` + 32 hex), `ItemId` (`wi_`), mode `fresh` or `keep_alive` (default `keep_alive`), `WorkpoolError` + `WORKPOOL_ERROR_CODES`; `WorkpoolCommandSchema` discriminated on `op`; `canonicalJson`. |
| `worker-turn.ts` / `yield.ts` / `yield-event.ts` | Assignment turns keyed by `(pool_id, generation, task_id, run_epoch)`; yield receipts `accepted` / `duplicate` / `refused`; yield events read back from the task record log. |
| `reconcile.ts` | Crash recovery of in-flight turns; an unprovable item becomes uncertain instead of silently retried. |
| `aggregate.ts` | `poolWorkIsFinished`, `aggregateResults`, `deliverAggregate` (one aggregate per closed pool). |
| `worker-capability.ts` / `worker-kernel-tools.ts` / `worker-tool-identity.ts` | Yield capability bound to the worker; parent kernel-tool resolution for pool workers; yield-tool marker. |
| `process-worker.ts` | `registerProcessWorkpoolWorker`: reuses the member-extension bundle for process workers but never starts a team poller or task engine. |
| `ports.ts` / `measurement.ts` | Admission request contracts; fresh vs keep_alive token/cost measurement runs with budget ceilings. |

## CONVENTIONS

- Lock order is pool then task everywhere; no await while either record is locked (`worker-turn.ts`).
- A reused (same-name) pool keeps its creation tool grant; a same-name create must request exactly that grant (`store.ts`).
- Worker identity comes from the dedicated launch, never from tool arguments (`process-worker.ts`).
- Tests are scenario-driven over `__fixtures__/` (admission, adversarial, delivery, recovery, restart).

## ANTI-PATTERNS

- NEVER persist executable kernel capability: only normalized tool NAMES are stored; the live capability stays in the runtime binding map and every spawn re-resolves (`engine.ts`).
- Parent-defined worker tools without a live JS-kernel capability fail closed (`kernel_tool_missing` / `kernel_tool_stale`).
- An assigned worker may only yield its current assignment; parent identity checks reject worker self-calls.
- A cancelled pool never spawns another worker; its pool binding is released at once while in-flight children keep their own bindings until destruction.
- Never hash item identity alone when reconciling legacy records; the initial prompt lives in the launch spec (`reconcile.ts`).

## QA

```sh
bun test packages/senpi-task/src/workpool packages/senpi-task/src/tools/workpool.test.ts
```

Parent: [`../AGENTS.md`](../AGENTS.md); package contracts in [`../../AGENTS.md`](../../AGENTS.md).
