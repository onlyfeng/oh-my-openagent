# src/manager/ - TaskManager Engine Internals

**Score:** 16 (170 files, ~29k LOC: ~60 one-concern modules + colocated tests + `__fixtures__/`; barrel `index.ts` is the only public surface - nothing outside the package imports `senpi-task/src/manager` by path).

## OVERVIEW

The concrete `createTaskManager` engine: start/redispatch/continue/forget/list wiring, admission gates, concurrency leases, respawn/pause ownership, outcome tracking, failure classification. Suspend/revival semantics, `spawn_spec` shapes, and execution-mode choice are owned by [`../../AGENTS.md`](../../AGENTS.md); this file maps the internals.

## STRUCTURE

| Cluster | Files |
|---------|-------|
| Core wiring | `manager.ts` (2,175 LOC - the hotspot), `manager-contract.ts` (the `TaskManager` interface), `types.ts` (wiring contract types, ~60 importers), `manager-helpers.ts` |
| Admission | `held-admissions.ts`, `held-send.ts`, `launch-fence.ts`, `launch-task.ts` (`LaunchPorts`), `workpool-admission.ts` |
| Concurrency | `concurrency.ts` (`TaskConcurrency`, `#private` fields) + `concurrency-routing.ts`, `concurrency-waiters.ts`, `parked-leases.ts` |
| Runner/child seam | `runner.ts` (`createInProcessManagedRunner` / `createRpcManagedRunner`), `child-handle.ts` (`ManagedChildHandle` - the ONE handle seam), `manager-reattach.ts` |
| Failure model | `start-failure.ts`, `credential-failure.ts`, `model-refusals.ts`, `manager-outcome.ts`, `runtime-fallback-event.ts`, `interrupted-turn.ts` |
| Resume/pause/respawn | `paused-restore.ts`, `pending-pauses.ts`, `run-pause.ts`, `manager-respawn.ts`, `parent-registry-context.ts`, `execution-mode.ts` |

## CONVENTIONS

- One-concern kebab-case files with a colocated `<subject>.test.ts`; behavior modules define a `*Ports` interface (`PausedRestorePorts`, `HeldSendPorts`, `IsolationWiringPorts`) injected from the concrete manager - `manager-wiring-seams.test.ts` guards the seams.
- Ports-over-concretes: plain readonly objects of injected callbacks; no service locators; the only class is `TaskConcurrency`.
- Dense prose comments above non-obvious rules state the invariant, not the mechanics (e.g. `manager-reattach.ts`: "a rejected reattach never leaves a child nobody owns").
- Tests share `__fixtures__/manager-fakes` (~100 importers - changing the fakes has package-wide blast radius); ordering/race proofs live in `*-races.test.ts` / `*-lifecycle*.test.ts`.
- Status matching on id/enum pairs, never display strings. Fail-closed: unwired deps refuse (`isolation_unavailable`, `spawn_spec_unavailable`) rather than degrade.

## ANTI-PATTERNS

- Never re-dispatch in a loop on a lost lease acquisition - "a release will drain it, never a retry loop" (`concurrency.ts`).
- Resume NEVER silently drifts to senpi's default model; surface a retryable `model_unavailable` (`parent-registry-context.ts`). An unresolved `auto` reads as in-process - no child is routed on a guess (`execution-mode.ts`).
- Never resurrect a settled task; never persist runtime-only objects (executable tools, extensions, member env) onto records; transient kernel-tool grants are process-lifetime only.
- A torn transcript tail is never reinterpreted: an unknown record type ends the walk undecided (`interrupted-turn.ts`).
- No `console.*`, `process.exit`, `any`, or `@ts-ignore` in production files; logging goes through `@oh-my-opencode/utils` `log`, and most modules return typed outcomes instead of logging.

## COMMANDS

```bash
tsgo --noEmit -p packages/senpi-task/tsconfig.json
bun test packages/senpi-task/src/manager/
```
