# src/workpool/ - Durable Parallel Worker Pool

**Score ~20** (38 files, ~4.6k LOC; durable scheduling engine, schema-validated records).

## OVERVIEW

The `workpool` engine behind senpi-task's parallel multi-worker coordination. Owns pool/item lifecycle, durable queue receipt + reconcile, event-driven scheduling, and aggregate delivery. Public API is re-exported through the senpi-task package root (`createWorkpoolTool`, `createWorkpoolWorkerTool`, `buildWorkpoolExecute`, `createWorkpoolStore`, schemas, error codes, and pool/item/caller/event types); `createWorkpoolEngine` itself is internal (not in the package barrel).

## WHERE TO LOOK

| File | Role |
|------|------|
| `engine.ts` | `createWorkpoolEngine` - orchestration: lifecycle, grants, aggregate delivery, subscriptions, attach/dispose (~320 LOC) |
| `dispatcher.ts` | `createWorkpoolDispatcher` - admission, recovery, completion/reconcile, worker reuse (~303 LOC) |
| `store.ts` | Durable JSON record store |
| `schema.ts` / `record-schema.ts` | Strict Zod command/record schemas (discriminated op commands, branded `wp_`/`wi_` ids) |
| `reconcile.ts` | Reconcile durable receipts vs live state |
| `worker-turn.ts` | Worker turn binding (pool generation/task/run epoch) |
| `worker-kernel-tools.ts` / `worker-capability.ts` / `worker-tool-identity.ts` | Kernel tool binding + capability resolution |
| `yield.ts` / `yield-event.ts` | Worker yield semantics |
| `aggregate.ts` | Aggregate result delivery |
| `run-pause.ts` | Run pause/resume |
| `__fixtures__/` | Behavior fixtures (admission, adversarial, delivery, recovery, restart) |

## CONVENTIONS

- Strict Zod object schemas reject extra fields. Readonly persisted records; JSON state under the store.
- Lock order is pool-before-task; never `await` while records are locked.
- Worker turns bind pool generation + task + run epoch; assigned workers may only yield their current assignment (caller identity checked).

## ANTI-PATTERNS

- Recovery must NOT replay the initial prompt (fixture-pinned invariant).
- Persist plain names only for kernel tools; live capability is runtime-bound and re-resolved at each spawn. Do NOT persist live capability/descriptors.

## COMMANDS

- `bun test packages/senpi-task` · `tsgo --noEmit -p packages/senpi-task/tsconfig.json`
