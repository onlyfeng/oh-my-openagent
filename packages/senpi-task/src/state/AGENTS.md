# state/ - Task Record + Transition Core (senpi-task)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 16 (30 files, own barrel, imported by store/lifecycle/manager/tools alike). Parent [`../../AGENTS.md`](../../AGENTS.md) names the enums and lifecycle rules; this file maps the modules. Pure data + functions: no IO, no process access - that purity is what keeps `senpi-task` runnable without a live Senpi import (optional peers).

## STRUCTURE

| File | Role |
|------|------|
| `types.ts` + `record-types.ts` | The vocabulary (`TASK_STATUSES` 7, `RESIDENCY_STATES` 5, `RUNNER_KINDS`, `BACKGROUND_MODES`, `SUSPENSION_REASONS`, `TOKEN_COVERAGE_STATUSES`, ...), `TaskRecord`, the spawn-spec union (`isSpawnSpecV1`; legacy `{cwd}` shape), `PendingSteeringEntry`, effective/resolved-model records, run-stats types |
| `record.ts` | `createTaskRecord` |
| `transitions.ts` + `transition-types.ts` | `transitionTaskRecord` + `markRecordLostForReconciliation`; late/invalid transitions are audited, never silent |
| `id.ts` | Monotonic ids: `createTaskId` / `bumpTaskId` / `parseTaskId` / `syncTaskIdFloor` (`__fixtures__/id-child.ts` pins cross-process monotonicity) |
| `run-fence.ts` | Run epochs + `fenceRun` (`LEGACY_RUN_START_EPOCH`) |
| `start-failure.ts` | Start-failure taxonomy: `TASK_START_FAILURE_KINDS` + reason groups (model / host / session) |
| `messageability.ts`, `transport-loss.ts`, `queued-steering.ts`, `host-liveness.ts`, `resolved-reasoning.ts`, `recovery-presentation.ts` | Single-purpose predicates and projections |

## CONVENTIONS

- Status/residency vocabulary lives ONLY in `types.ts`; every consumer imports it - no string literals.
- The transition table is data (`transitions-table.test.ts` pins it); a new transition is a table row plus an audit expectation, not an `if`.
- Residency-regression, run-pause, and start-failure-reset suites pin the epoch/reset edge cases - extend them with any rule change.

## ANTI-PATTERNS

- Do not add a status/residency value without auditing every switch and the parent lifecycle's epoch rules (non-terminal reattach bumps `run_epoch` exactly once).
- Do not read/write the filesystem or process table here; that belongs to `store/` and `lifecycle/`.
- Do not compare task ids lexically - `id.ts` owns ordering.

## COMMANDS

```bash
tsgo --noEmit -p packages/senpi-task/tsconfig.json
bun test packages/senpi-task/src/state
```
