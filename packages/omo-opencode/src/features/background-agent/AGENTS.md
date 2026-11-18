# src/features/background-agent/ — Core Orchestration Engine

**Generated:** 2026-05-20 · **Updated:** 2026-11-18

## OVERVIEW

53 prod modules (49 top-level + 4 in `spawner/`); 125 `.ts` total incl. 72 test files. Manages async task lifecycle: launch → queue → run → poll → complete/error. Concurrency limited per model/provider (default 5). Central to multi-agent orchestration. `manager.ts` (3.6k LOC) and `manager.test.ts` (9.0k LOC) dominate the prod/test surfaces.

## TASK LIFECYCLE

```
LaunchInput → pending → [ConcurrencyManager queue] → running → polling → completed/error/cancelled/interrupt
```

## KEY FILES

| File | Purpose |
|------|---------|
| `manager.ts` | `BackgroundManager` — main class (3.6k LOC, the production hotspot): launch, cancel, resume, getTask, listTasks, event handling, stale cleanup, archive |
| `spawner.ts` | Task spawning: `createTask`/`startTask`/`resumeTask` over `SpawnerContext`; prompt/record/fallback/tmux pieces live in `spawner/` |
| `concurrency.ts` | `ConcurrencyManager` — FIFO queue per concurrency key, slot acquisition/release |
| `task-poller.ts` | 3s interval polling, completion via idle events + stability detection (10s unchanged) |
| `types.ts` | `BackgroundTask`, `LaunchInput`, `ResumeInput`, `BackgroundTaskStatus` |
| `parent-wake-notifier.ts` | Coordinates parent-session wake notification; queue/dedupe/dispatch/history/activity/recovery helpers are split into sibling `parent-wake-*.ts` modules |
| `loop-detector.ts` | Detects polling/event loops that would otherwise burn budget. |
| `error-classifier.ts` | Maps raw provider errors → `BackgroundTaskError` categories. |
| `fallback-retry-handler.ts` | Coordinates retries with the runtime-fallback system. |
| `process-cleanup.ts` | Best-effort cleanup on parent exit. `OMO_DISABLE_PROCESS_CLEANUP=1` opts out entirely. |
| `subagent-spawn-limits.ts` | Enforces per-parent subagent spawn caps. |
| `session-status-classifier.ts` | Normalizes OpenCode session status across versions. |
| `compaction-aware-message-resolver.ts` | Resolves task result content even across mid-task compaction. |
| `attempt-lifecycle.ts` | Tracks retry attempts on a single task. |
| `task-history.ts` | Append-only history for completed tasks. |
| `session-idle-event-handler.ts` | Bridges OpenCode `session.idle` → task-poller completion signal. |
| `session-existence.ts` | Cheap existence check used by recovery code. |
| `abort-with-timeout.ts` | Force-abort tasks past `syncPollTimeoutMs`. |
| `remove-task-toast-tracking.ts` | Strips lingering toast tracker entries on task end. |
| `background-task-notification-template.ts` | Template for parent-session result injection. |
| `index.ts` | Feature barrel: `types.ts` re-exports + `BackgroundManager` + `waitForTaskSessionID`. |
| `task-registry.ts` / `task-snapshot.ts` | Process-global active/completed task registry (100 completed cap) on `globalThis`; frozen UI snapshots. |
| `wait-for-task-session.ts` | Bounded poll of `getTask` for the child session id (exported from the barrel). |
| `session-stopped-on-error.ts` | `MIN_ERRORED_IDLE_POLLS` (10 ≈ 30s): an errored session only counts as stopped when nothing re-prompts it. |

## SPAWNER SUBDIRECTORY

| File | Purpose |
|------|---------|
| `fallback-agent.ts` | `FALLBACK_AGENT` + `buildFallbackBody` + `isAgentNotFoundError` for the runtime-fallback path |
| `task-prompt-body.ts` | Builds the task prompt body |
| `task-record.ts` | `buildTaskRecord` — the `bg_<uuid8>` task record |
| `tmux-callback-invoker.ts` | `invokeTmuxSessionCreatedCallback`: fire-and-forget TmuxSessionManager notify on session creation, gated on `tmuxEnabled && !suppress`; fires immediately — `TMUX_CALLBACK_DELAY_MS = 200` sits in `constants.ts` but is no longer applied here |

## COMPLETION DETECTION

Two signals combined:
1. **Session idle event** — OpenCode reports session became idle
2. **Stability detection** — message count unchanged for 10s (3+ stable polls at 3s interval)

Both must agree before marking a task complete. Prevents premature completion on brief pauses. On completion, `result-handler` → `parent-session-notifier` injects a system message into the parent session.

## CONCURRENCY MODEL

- Key format: `{providerID}/{modelID}` (e.g., `anthropic/claude-opus-4-7`), or the agent name when no model is set
- Default limit: 5 concurrent per key (`BackgroundTaskConfig`; per-model `modelConcurrency` override, `0` = unlimited, provider-level fallback)
- FIFO queue: tasks wait in order when slots full
- Slot released on: completion, error, cancellation

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-opencode/src/features/background-agent   # focused suite
```

`constants.ts`: poll 3s, stability 10s, min idle 5s, task TTL 30m, stale timeout 45m, max tool calls 4000, circuit breaker default-on at 20 consecutive failures.
