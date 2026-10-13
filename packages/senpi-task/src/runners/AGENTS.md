# runners - Child Execution (in-process, process, host session)

Three runner classes behind one barrel (`index.ts`): `InProcessRunner` (`in-process.ts`), `RpcProcessRunner` (`rpc-process.ts`), `RpcHostRunner` (`rpc-host.ts`). Direct layer: 48 files (~8.5k LOC, mostly cross-runner tests); `in-process/` 26 files, `rpc/` 40 files, `rpc-host/` 85 files. Earned its file by size and as the only place child sessions are constructed; mode selection and the in-process/process contracts are in the package [`AGENTS.md`](../../AGENTS.md) (EXECUTION MODES). Host-session internals: [`rpc-host/AGENTS.md`](rpc-host/AGENTS.md).

## WHERE TO LOOK

| Concern | Location |
|---------|----------|
| Public runner API | `index.ts` (classes, errors, spawn/exit helpers, daemon + shard helpers, child-handle construction) |
| Shared types | `types.ts` (recorded host endpoint a revival must open on), `child-extension-events.ts` (`CHILD_PERMISSION_EVENT`, `TASK_CHILD_EXTENSION_EVENT`; a separate channel, never an `AgentSessionEvent`) |
| Model identity | `pinned-model-equivalence.ts` (priority-alias equivalence), `retry-fallback-profile.ts` (child chain as a session-only settings overlay: `open_session.retryFallback` on a host, `set_retry_fallback` in a process) |
| In-process child | `in-process/child-handle.ts` (live + restored handle, queued input), `child-options.ts`, `child-loader.ts`, `shared-tool-filter.ts`, `kernel-tool-surface.ts`, `curated-readonly-bash.ts`, `runtime-fallback-settings.ts`, `pin-check.ts`, `subagent-prompt.ts`, `host-tools.ts` |
| Process child | `rpc/spawn.ts` (argv, env, executable selection; subpath `./rpc-spawn`), `rpc/senpi-launcher.ts`, `rpc/protocol-client.ts`, `rpc/handle.ts`, `rpc/terminate.ts`, `rpc/exit-mapping.ts`, `rpc/crash-record.ts`, `rpc/heartbeat.ts`, `rpc/model-admission.ts` (subpath `./rpc-model-admission`) + `model-catalog-probe.ts`, `rpc/turn-outcome.ts` / `turn-settlement.ts`, `rpc/ui-auto-answer.ts`, `rpc/parent-extensions.ts` |
| Fixtures | `__fixtures__/caller-settings-world.ts`, `in-process-fallback-session.ts`; per-runner `__fixtures__/` under each subdir; `*.test-support.ts` beside tests |

## CONVENTIONS

- `DEFAULT_MAX_CHILD_DEPTH = 4` (`in-process.ts`); creation failures throw typed `RunnerError`.
- RPC child argv starts with `--no-extensions --no-ask-user`; extensions, model and thinking are appended conditionally. Capabilities are comma-separated and always include `extension_events`.
- Each process child gets its own `SENPI_CODING_AGENT_SESSION_DIR` under the task state dir; member/workpool identity env is stripped before the explicit member env is applied.
- Child stderr is capped to a 4,096-char tail before exit facts are stored (`rpc/exit-mapping.ts`).
- Cross-runner behavior (model pinning, DAG child tool policy, heartbeat deadlines, persistence/resume, runtime fallback, rpc-host busy/cancel/crash) is tested in this direct layer; subdir tests cover their own modules.

## ANTI-PATTERNS

- NEVER replay the original prompt when rebuilding a restored child from its persisted transcript (`in-process/child-handle.ts`).
- NEVER send process signals from `rpc/protocol-client.ts`; termination belongs to `rpc/terminate.ts`.
- MUST strip member identity from both the real child spawn and the model-catalog probe (`rpc/spawn.ts`).
- NEVER put the resolved `rpc-entry` path on argv when the executable itself is selected (`rpc/spawn.test.ts`).
- Curated read-only agents never receive parent kernel tools (`in-process/kernel-tool-surface.ts`).
- `task_send` ALWAYS reaches the addressed child: a steer the child rejects as busy degrades to `followUp` queueing, never a failed delivery (`rpc/handle.ts`).

## COMMANDS

```sh
bun test packages/senpi-task/src/runners
bun test packages/senpi-task/src/runners/rpc/spawn.test.ts   # argv/env contract
```
