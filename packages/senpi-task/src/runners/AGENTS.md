# src/runners/ — Child-Process Runner Trinity

**Score:** 15 (316 files incl. subdirs, ~43k LOC; barrel `index.ts` (48 exports) is the public surface, consumed via `manager/runner.ts` and the package barrel).

## OVERVIEW

Three runner implementations plus shared input/parent-control/type surfaces: `InProcessRunner` (`in-process.ts`), `RpcHostRunner` (`rpc-host.ts` — child as a session inside the parent's engine host), `RpcProcessRunner` (`rpc-process.ts` — spawned child process). Execution-mode choice, daemon ensure/warm, shard sockets, and mode semantics are owned by [`../../AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `types.ts` | The runner contract (`ChildSpec`, `ChildHandle`, `ChildExitFacts/Outcome`, `RunnerErrorFacts`, `RpcChildHandle`, `TerminateOptions`) — widest blast radius |
| `in-process.ts` / `rpc-host.ts` / `rpc-process.ts` | One runner per file; class-based with `*RunnerOptions` constructor injection |
| `input-effects.ts` | `createInputEffects` — input admission shared by BOTH RPC runners |
| `task-parent-control.ts` | `createTaskParentControl` — parent steering wire (`TaskParentWire`, `TaskParentCommand`) |
| `parent-link.ts`, `child-ancestry.ts`, `engine-spawned-by.ts`, `input-admission-read.ts`, `input-delivery.ts` | Small support modules |
| `designated-chain-error.ts`, `designated-fallback-profile.ts`, `retry-fallback-profile.ts`, `pinned-model-equivalence.ts` | Designated-model chain handling |
| `rpc-host/`, `rpc/`, `in-process/`, `child-dialogs/`, `__fixtures__/` | Subdir internals (RPC protocol client, host lifecycle, shared-tool filter, dialog forwarding) |

## INVARIANTS (each mirrored by a test named after it)

- Spawns a senpi RPC child (never `shell:true`).
- Typed session-create-failed — never a silent inMemory/default-dir fallback; a refused client must never start a second host beside the daemon.
- A recorded session never falls back to another endpoint; fallback only via the engine's `set_retry_fallback` chain, held in memory, never persisted.
- A child never runs outside its designated models; an engine that cannot hold the chain is never started (#12006); the host's settings never move the child's designated chain.
- Input identity is immutable: an earlier run's input is never sent into a later run; a retry never changes the semantic input or its identity.
- `undefined` = unreadable, never read as idle (senpi#3266). A start that never produced a session leaves NO binding behind.
- Resumed handles come back IDLE with never a WIDER tool surface than they had (tool floor never inherited, never loosened); `DEFAULT_MAX_CHILD_DEPTH` is enforced, not loosened per spawn.

## CONVENTIONS

- Typed errors (`RunnerError`, `InputUncertainError`, `InputConflictError`, `TaskParentControlError`), never untyped throws; uncertain delivery tells the user not to resend the same text.
- Invariant comments often cite the issue id (`#12006`, `senpi#3266`); keep comment and test in step when editing.
- `rpc-host.test-support.ts` builds per-test temp agent dirs so a case never leaks a connection into the next and two checkouts can run the suite at once.

## COMMANDS

```bash
tsgo --noEmit -p packages/senpi-task/tsconfig.json
bun test packages/senpi-task/src/runners/
```
