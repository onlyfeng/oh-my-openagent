# src/hooks/atlas/ — Master Boulder Orchestrator

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

61 files (34 production `.ts`, 26 tests, own `tsconfig.json`; ~11.4k LOC). The `atlasHook`: Continuation Tier hook that monitors session.idle events and forces continuation when boulder sessions (task-spawned agents, plan work) have incomplete work. Also enforces write/edit policies for subagent sessions, tracks background launches, and gates final-wave approval. (The 90s first-prompt watchdog lives in `../runtime-fallback/`, not here.) Typecheck alone: `tsgo --noEmit -p packages/omo-opencode/src/hooks/atlas/tsconfig.json`.

## WHAT ATLAS DOES

Atlas is the "keeper of sessions" — it tracks every session and decides:
1. Should this session be forced to continue? (if boulder session with incomplete todos)
2. Should write/edit be blocked? (policy enforcement for certain session types)
3. Should a verification reminder be injected? (after tool execution)

## DECISION GATE (session.idle)

```
session.idle event
  → Is this a boulder/ralph/atlas session? (session-last-agent.ts)
  → Is there an abort signal? (is-abort-error.ts)
  → Failure count < max? (state.promptFailureCount)
  → No running background tasks?
  → Agent matches expected? (recent-model-resolver.ts)
  → Plan complete? (todo status)
  → Cooldown passed? (5s between injections)
  → Inject continuation prompt (boulder-continuation-injector.ts)
```

## KEY FILES

| File | Purpose |
|------|---------|
| `atlas-hook.ts` | `createAtlasHook()` — composes event + tool handlers, maintains session state |
| `event-handler.ts` | `createAtlasEventHandler()`: routes events; idle path in `idle-event.ts` + `idle-continuation.ts` + `idle-session-eligibility.ts` |
| `atlas-lifecycle-store.ts` | Per-session state + pending tool-call tracking |
| `idle-constants.ts` | 5s cooldown, 5 min failure backoff, 10 max failures, retry at cooldown + 1s |
| `final-wave-approval-gate.ts` / `final-wave-plan-state.ts` | Final-wave checkbox requires explicit user approval |
| `tool-execute-after-{subagent-completion,plan-tasks,direct-work}.ts` | Post-tool split (subagent completion is the largest, ~246 lines) |
| `boulder-continuation-injector.ts` | Build + inject continuation prompt into session |
| `system-reminder-templates.ts` | Templates for continuation reminder messages |
| `tool-execute-before.ts` | Block write/edit based on session policy |
| `tool-execute-after.ts` | Inject verification reminders post-tool |
| `write-edit-tool-policy.ts` | Policy: which sessions can write/edit? |
| `verification-reminders.ts` | Reminder content for verifying work |
| `session-last-agent.ts` | Determine which agent owns the session |
| `recent-model-resolver.ts` | Resolve model used in recent messages |
| `subagent-session-id.ts` | Detect if session is a subagent session |
| `omo-path.ts` | Resolve `.omo/` directory path |
| `is-abort-error.ts` | Detect abort signals in session output |
| `types.ts` | `SessionState`, `AtlasHookOptions` (`directory`, `backgroundManager`, `isContinuationStopped`, `agentOverrides`, `idleSettleMs`, `autoCommit`) |

## STATE PER SESSION

`SessionState` (types.ts) is mutable per session: `promptFailureCount` + `lastFailureAt` + `pendingRetryTimer` (retry/backoff), `lastContinuationInjectedAt` / `isInjectingContinuation` (cooldown), final-wave approval counters, tool-progress stall tracking (`iterationsSinceLastToolProgress`, `activeContinuationPlanPath`), and `verifiedTaskKeys` (suppress repeat 4-phase reminders).

Max consecutive failures before backoff pause: 10 (`MAX_CONSECUTIVE_PROMPT_FAILURES` in `idle-constants.ts`), then `FAILURE_BACKOFF_MS` = 5 min.

## RELATIONSHIP TO OTHER HOOKS

- **atlasHook** (Continuation Tier): Master orchestrator, handles boulder sessions
- **todoContinuationEnforcer** (Continuation Tier): "Boulder" mechanism for main Sisyphus sessions
- Both inject into session.idle but serve different session types
