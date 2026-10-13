# manager - Task Manager (spawn, concurrency, outcomes, respawn)

`createTaskManager` (`manager.ts`, 1,535 LOC, `allow: SIZE_OK`) is one closure-backed implementation that keeps the concurrency, queue, live-handle and waiter invariants together. 40 implementation files, 69 tests, 9 fixtures (~17.8k LOC incl. tests), one of the largest directories in the package. Earned its file by size and because every tool, the DAG scheduler, the workpool and the lifecycle reach children through it.

## WHERE TO LOOK

| Concern | Files |
|---------|-------|
| Public surface | `index.ts` barrel, `types.ts` (`TaskManager`, `TaskManagerOptions`, `ManagerStartSpec`, `ManagedRunner`, `TrustedRespawnLaunch`) |
| Runner adapters | `runner.ts` (`createInProcessManagedRunner` / `createRpcManagedRunner`), `child-handle.ts` (`adaptInProcessHandle` / `adaptRpcHandle`, release-on-dispose), `parent-registry-context.ts` (the parent's real senpi `ModelRegistry`, not a structural port) |
| Admission | `concurrency.ts` (`TaskConcurrency`: synchronous leases, model/provider lanes plus an independent global cap), `resident-start.ts` (normal spawns and pool grants share the admission lease), `spill-admission.ts`, `residency-signal.ts` (`residencyChanged`, #8396), `depth-policy.ts`, `names.ts` (`NameRegistry`, lowest free `-N` suffix) |
| Mode + plan | `execution-mode.ts` (`resolveExecutionMode`, `auto` via `AUTO_HOST_CAPABILITIES`), `plan-warm.ts` (warms the lazy barrel before an explicit pin resolves, #9722), `manager-helpers.ts` (`nextRungManagedSpec`) |
| Start failures | `start-failure.ts`, `start-failure-cause.ts` (caught messages are untrusted), `credential-failure.ts` (provider answers no other model on the SAME provider can fix: rejected credential, lapsed subscription), `runtime-fallback-event.ts` |
| Outcomes + ownership | `manager-outcome.ts` (verdicts from freshest handles and on-disk record), `launch-fence.ts` (fail only while the launch's run still holds the record), `pending-stops.ts` (omo#9403), `provisional-exit.ts` (2s confirmation), `fail-stranded-handoff.ts`, `once-only.ts` |
| Revival paths | `manager-respawn.ts` (dispatch on `execution_mode`; daemon children resume the session path their record carries), `manager-reattach.ts`, `interrupted-turn.ts`, `self-resumed-turn.ts` (omo#9069), `host-turn-resumed.ts` (`HOST_TURN_RESUMED_EVENT`, synthesized, never a host event) |
| Workpool | `workpool-admission.ts`, `workpool-start.ts`, `workpool-process-launch.ts` (never loads the parent scheduler in a worker), `workpool-respawn.ts` |
| Observability | `transcript-log.ts` (`TRANSCRIPT_*_EVENT` written into the task JSONL; read by `tools/output`), `observed-model.ts` (effective model from assistant messages, #9722), `child-facts.ts`, `task-sequence.ts` (per-parent spawn ordinals), `isolation-wiring.ts`, `continue-result.ts` |

## CONVENTIONS

- The manager signals processes only through the lifecycle's `defaultSignaller`; terminal teardown goes through the lifecycle destroy port, record writes through `store` locks.
- Every fallback rung builds a fresh spec from that rung's OWN model in every field the runner and the post-start pin check read (`nextRungManagedSpec`, #9722 H1).
- Child subscriptions have two releasers (subscriber and handle disposal); wrap them with `onceOnly`.
- Fixtures under `__fixtures__/` build fake runners, stores and registries; tests use given/when/then titles.

## ANTI-PATTERNS

- NEVER fail a task from a launch that no longer owns its run; cancel, interrupt or another owner may have moved the record (`launch-fence.ts`).
- NEVER publish a terminal state for an unexpected exit before the confirmation window; tree termination can reach the child before its owner (`provisional-exit.ts`).
- While a cancel waits on a lost connection, only that cancel may end the run (`pending-stops.ts`).
- A persisted start-failure reason outside the closed enum marks the failure untrusted; never surface it as a trusted cause (`start-failure.ts`).
- NEVER hand a pool parent kernel tools when the engine supplied no runtime kernel-tool map (`workpool-admission.ts`).

## COMMANDS

```sh
bun test packages/senpi-task/src/manager
bun test packages/senpi-task/src/manager/manager.test.ts   # core invariants
```
