# senpi-task - Senpi Task State Machine + Tool Surface

**Generated:** 2026-11-18 / c2691818a9 (update pass; subsystem detail lives in the per-dir AGENTS.md files)

## OVERVIEW

The Senpi-coupled engine behind the `omo-senpi` task component: a durable task state machine, a persistent record store, three child runners (in-process, RPC process, and the in-session engine host), a residency/TTL/reconcile lifecycle, an exactly-once completion notifier, a steering engine, a named-team runtime, and the 4 task + 6 lead-only team `ToolDefinition`s (the `dag` tool is registered by `omo-senpi`). Package: `@oh-my-opencode/senpi-task` (private, `sideEffects: false`). `@code-yeongyu/senpi` and `typebox` are optional peers (`package.json:62`) so pure state/store/schema code stays runnable without a live Senpi import; runner and tool code that needs the Senpi surface is isolated. Do not import `packages/omo-opencode` from here.

## ANATOMY

| Area | Path | Purpose |
|------|------|---------|
| State + store | `src/state/`, `src/store/` | `TaskStatus` (7) + `ResidencyState` (5), `TaskRecord`, `transitionTaskRecord` (late/invalid-transition audits); `createTaskRecordStore` JSONL store (mtime+size-validated read cache, capped-16 LRU append-fd pool, `resolveStateDir`, redaction). Own AGENTS.md. |
| Runners | `src/runners/` | `InProcessRunner` (shares parent tool closures; `ChildSpec.completion` defaults `final-text`, `turn` for tool-only children) + `RpcProcessRunner` (child Senpi process, JSON-RPC steer/abort/prompt). Own AGENTS.md (+ `rpc/`, `rpc-host/`). |
| Manager + lifecycle | `src/manager/`, `src/lifecycle/` | `createTaskManager` (runners, concurrency, name registry, depth policy, execution mode); `createTaskLifecycle` (residency admission, shutdown suspension, crash reconcile + scoped revival under the fenced admission lease, two-phase TTL tombstone sweep). Own AGENTS.md; see SESSION SUSPEND AND RESUME REVIVAL. |
| Isolation | `src/isolation/` | Checkout isolation over `@oh-my-opencode/isolation-core` as an injectable port (`runtime`/`prepare`/`settle`/`salvage`); a wiring supplying neither isolation port refuses isolated spawns as `isolation_unavailable`. |
| Completion + steering | `src/completion/`, `src/steering/` | Exactly-once completion notifier + `routeCompletion` routing table; `createSteeringEngine` send/interrupt/cancel over live or resident children (own AGENTS.md). |
| Team + tools | `src/team/`, `src/tools/` | Named-team registry, durable mailboxes with injection-driven delivery, lead poller, member extension, tasklist, shutdown handshake; `task/` (single or `tasks:[...]` batch), `control/`, `output/`, `team/` tools. Own AGENTS.md files. |
| DAG | `src/dag/` | `compileDag` -> `createDagScheduler` dependency-frontier admission (waves informational only), WAL journal + checkpoints, fingerprint-keyed run reuse/amend, retry/send, crash recovery, skill materialization. Own AGENTS.md. |
| Agents + category | `src/agents/`, `src/category/` | `loadAgents`/`mapOmoConfigAgents`, builtin curated + plan-gated agents, `resolveAgent` persona/model resolution; `resolveCategory` + per-provider builtin tables with the `requiresModel` and chain-viability gates and `<Category_Context>` append hooks. Own AGENTS.md. |
| Others | `src/workpool/`, `src/kernel-tools/`, `src/eval-handles/`, `src/__adversarial__/`, `src/lazy/`+`src/senpi/`+`src/config/` | `workpool` tool semantics; eval-kernel tool contract; eval-handle host for live child sessions; seeded 200-iteration chaos bench (4 W1 invariants); lazy dynamic-import barrels + settings resolver. |

## PUBLIC API (`src/index.ts` barrel)

| Tool | Factory | File |
|------|---------|------|
| `task` | `createTaskTool` | `tools/task/tool.ts:10` |
| `task_send` | `createTaskSendTool` | `tools/control/send.ts` |
| `task_cancel` | `createTaskCancelTool` | `tools/control/cancel.ts:60` |
| `task_output` | `createTaskOutputTool` | `tools/output/output.ts` |

`task` is spawn-only: one `prompt` or a non-empty `tasks:[...]` batch (sync batches aggregate results; background batches return ids + queue positions; module map in [`src/tools/task/AGENTS.md`](src/tools/task/AGENTS.md)). Steer, revival, team messaging, shutdown approval go through `task_send`; output/status/transcript peeks through `task_output`. Team tools (6, lead-only, canonical order): `team_create`, `team_delete`, `task_create`, `task_get`, `task_list`, `task_update` - members never receive the lead family, only team-scoped `task_send` (`packages/omo-opencode` still uses its prior task/team names; cross-edition parity is a follow-up). Engine primitives: `createTaskManager`, `createTaskLifecycle`, `createCompletionNotifier`/`routeCompletion`/`shouldNotifyStatus`, `createSteeringEngine`, `InProcessRunner`, `RpcProcessRunner`, `createTaskRecordStore`/`resolveStateDir`, `transitionTaskRecord`, `resolveCategory`, `loadAgents`/`mapOmoConfigAgents`, `resolveAgent`/`BUILTIN_AGENTS`, the team runtime + typed errors. DAG subpath `./dag` plus deep subpath exports (`./agents-builtin`, `./category-builtins`, `./category-resolver`, `./rpc-spawn`, `./rpc-model-admission`, `./task-renderers`, `./renderer-text`, `./notice-box`).

### Category gating (summary)

Two independent gates: (1) `requiresModel` - a gated builtin resolves `model_unavailable`, excluded and never routed through its chain, unless a gate model is live in the registry; ANY explicit `omo.json` `categories.<name>` entry bypasses the gate, even description-only; a rung counts only a registry model served by a chain-listed provider (a gateway re-publish opens nothing, #9146). (2) `isCategoryChainViable` - a builtin whose chain has zero registry-resolvable rungs drops out, for ALL builtins. `resolveAvailableCategoryNames(config, registry)` is the public gated list; `listTaskCategories` (built pre-registry) appends ` (requires <model>)` instead; spawn-time `resolveCategory` is the sole authoritative gate. Full detail: [`src/category/AGENTS.md`](src/category/AGENTS.md).

### Builtin curated + plan-gated agents

`agents/builtin/` ships four read-only curated subagents (`explore`, `librarian`, `plan-consultant`, `plan-reviewer`; mirrored fallback chains in `fallback-chains.ts`; a read-only GitHub/HTTPS broker replaces the shell for in-process curated children) plus the three ulw-loop reviewer builtins (`omo-native-{code-reviewer,qa-executor,gate-reviewer}`, `write`-allowlisted for report artifacts, outside `CURATED_READONLY_AGENT_NAMES`); all pinned `executionMode: "in-process"` with a 9-name tool allowlist. `plan-consultant`/`plan-reviewer` are plan-gated (`agents/invocation-guard.ts`): spawnable only after an explicit USER `ulw-plan` request (model-initiated SKILL.md reads never satisfy it), a `.omo/plans/*.md` artifact touched in-session, and no `ulw-execute`; an unwired `resolveSkillInvocations` dep fails CLOSED; the task tool consults the gate pre-spawn (single: `denied`; batch item: `plan_unresolved`).

## TEAM DELIVERY MODEL

Team messaging is injection-driven over durable mailboxes: a send writes a durable unread JSON and returns; delivery steers into the recipient's running turn (never an editable follow-up). The current lead owns one `createLeadPoller` per matching team; the adapter ticks it on `session_start` + every second (suspended during compaction, session switching and shutdown); member inboxes are polled in-child by `member-extension/`. Delivery is reservation-based: unread `<id>.json` -> `.delivering-<id>.json` -> `processed/<id>.json` only after the message is observed in the recipient session (the durable exactly-once ledger); `createSessionMarkerIndex` checks envelope persistence with an incremental byte-offset index (O(1) per tick; rescans on truncation). Every `session_start` runs the recovery chain: flush/drop buffered completions -> reconcile (revive + reattach) -> re-observe liveness -> reclaim stale reservations -> redeliver unnotified completions -> TTL cleanup -> poll leads. Dead process members with a persisted session respawn without prompt replay and rebind via `switch_session`.

Completion routing (`completion/routing.ts`): `shouldNotifyStatus` fires only for externally-caused `completed`/`error`/`lost` (parent-initiated cancel/interrupt return synchronously, never push). `routeCompletion`: `idle` -> `wake`, `streaming` -> `deliver_streaming` (both unconditional; the coordinator batches same-window notifications into ONE injection at the next tool boundary), `compacting`/`session_switching`/`session_shutdown` -> `buffer` until the parent settles.

## EXECUTION MODES

- **in-process (default)**: child runs through the SAME parent tool closures (`filterSharedParentTools` + `mergeChildCustomTools`); parent-only tools drop (`uiOnlyToolNames`), `x_search` remaps to `direct`; child conversations persist under the same `<stateDir>/children/<taskId>/sessions/` layout as process mode, so clean quit and crash recover identically.
- **process**: child Senpi process; steer/abort/prompt cross JSON-RPC (`runners/rpc/protocol-client.ts`); session-start reconciliation can respawn + `switch_session` to the newest persisted JSONL. Team members always use this mode.
- **host** (`task.process_runner: host`): `RpcHostRunner` (`runners/rpc-host/`) opens the child as a SESSION inside the parent's engine host (one host per session tree, `p-<shard>.sock`); `daemon-launch-spec.json` is the ONLY argv source; falls back to `child-process` only on capability/policy mismatch, win32, or bun-less Node.

Mode is chosen by `resolveExecutionMode` (`task.default_execution_mode` + per-agent `execution_mode`); `auto` resolves ONCE per parent session (reachable host advertising the needed capabilities -> `process`/`host`, else in-process; unresolved `auto` reads in-process, never a guess; user-set modes win). Operator: `omo daemon run|adopt|status|stop|handoff|gc|rollback-prepare` (`docs/reference/omo-daemon.md`).

## SESSION SUSPEND AND RESUME REVIVAL

Shutdown suspends instead of disposing: `suspendOnSessionShutdown` (`lifecycle/shutdown.ts`) tears down live handles (no-orphan law) but parks records `persisted_only` (in-process) or `rpc_detached` (process; keeps the last pid for orphan detection), preserving status/epochs/stats + a `suspended` event. The scan covers registry handles AND this session's `pending` records (a queued child has no handle yet); a deliberately-stopped child (`killed: true`, `cancelled`/`lost`) routes through destruction and never revives; registry forget happens FIRST. `task.resume_children: false` keeps the old dispose-all path.

Revival state lives on the record: `spawn_spec` v1 carries plain-data launch facts only (the legacy `{cwd}` shape feeds RPC respawn alone); `notify_on_terminal` persists the background-notify intent; `pending_steering` is the durable prelaunch queue (`task_send` to a suspended child returns `not_continuable` - resume the session instead). `reconcileOnSessionStart(parentSessionId)` revives ONLY that session's children; RECLAMATION (orphaned `resident`: bypasses the capacity gate, per-record expected-owner CAS) and ADMISSION (`persisted_only`/`rpc_detached` needing a slot: batch under the renewable `session-<id>.lock` lease, bounded 5s -> `deferred/lock_contended`) are disjoint; overflow stays suspended `deferred/capacity`, never evicted.

A retryable respawn failure rolls back to the suspended residency and defers; an unrecoverable one disposes a terminal record (result preserved) or marks a non-terminal one `lost`. Suspension never bumps `run_epoch`; a non-terminal reattach bumps it exactly once. A fail-closed resolver matches the persisted `resolved_model` provider+model exactly - `deferred/model_unavailable`, never a substitute. No-rerun: terminal-without-transcript disposed; non-terminal-without-session-file relaunches ONCE from the persisted v1 prompt (then drains `pending_steering`); neither -> `lost`. Only a child interrupted mid-turn gets the continuation nudge (`manager/interrupted-turn.ts` reads the actual last JSONL record).

Completion recovery rides the same session start: `reconcileUnnotifiedNotifications` redelivers terminal children still owing a notification (`notified_epoch < run_epoch` plus legacy `notification_failed_epoch`), dedupe `(task_id, run_epoch)`, epochs patched only via conditional `store.mutate`. Adapter delivery is BATCHED - `enqueue` means queued, `notified_epoch` persists optimistically; `recordDeliveryFailure` is the out-of-band receipt (failed flush / dropped batch window) that rolls the epoch back and re-enters the retry ladder. TTL deletion is two-phase (`lifecycle/ttl.ts`): phase 1 renames to `<taskId>.json.expunging` inside the record lock (invisible to load/list); phase 2 destroys the orphan pid, children dir, spill, log. A tombstoned record is committed to deletion, never resurrected; the retention predicate keeps live handles/claims, non-terminals, undelivered notifications, and `lost` process records until the pid is proven dead.

## QA

`tsgo --noEmit -p packages/senpi-task/tsconfig.json` then `bun test packages/senpi-task`. Colocated `*.test.ts` use given/when/then; the seeded chaos bench (`src/__adversarial__/chaos-bench.test.ts`, 200 iterations, `SEED=<label>` to rerun) asserts (1) exactly-once notification per `(task_id, run_epoch)`, (2) terminal idempotence, (3) no slot leak, (4) no unhandled rejection. Manual QA scripts write disposable fixture trees (`scripts/manual-qa.ts`, `manual-category-qa.ts`, `manual-agents-qa.ts`, `manual-output-qa.ts`); live end-to-end proof runs through the omo-senpi drivers (`task-e2e.mjs`, `team-e2e.mjs` - see [`packages/omo-senpi/AGENTS.md`](../omo-senpi/AGENTS.md)).

Parent: [`packages/AGENTS.md`](../AGENTS.md).
