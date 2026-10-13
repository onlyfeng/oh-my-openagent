# src/tools/call-omo-agent/ — Direct Agent Invocation Tool

**Generated:** 2026-10-13

## OVERVIEW

26 files (~3.6k LOC incl. tests; 13 test files). The `call_omo_agent` tool: direct invocation of named agents (explore, librarian only). Distinct from `delegate-task`: no category system, no skill loading, no category model routing. Fixed agent set, same execution modes (background/sync).

## DISTINCTION FROM delegate-task

| Aspect | `call_omo_agent` | `delegate-task` (`task`) |
|--------|-----------------|--------------------------|
| Agent selection | Named agent (explore/librarian) | Category or subagent_type |
| Skill loading | None | `load_skills[]` supported |
| Model selection | Agent fallback chain (`AGENT_MODEL_REQUIREMENTS`), explicit `model` override builds a session fallback chain | From category config |
| Use case | Quick contextual grep | Full delegation with skills |

## ALLOWED AGENTS

Only `explore` and `librarian` - enforced via `ALLOWED_AGENTS` in `constants.ts` and resolved by `agent-resolver.ts` (case-insensitive, invisible characters stripped; other built-ins, custom agents, and categories must use `task()`). The tool description also pins: prompts MUST be in English, nested subagent depth is tracked and blocked past the configured limit (description-level contract, not enforced in this dir).

## EXECUTION MODES

Same two modes as delegate-task; `tools.ts` validates, then hands sync to `sync-executor.ts` and background to `background-executor.ts`. Background mode with `session_id` returns an error (resume requires sync):

| Mode | File | Description |
|------|------|-------------|
| **Background** | `background-executor.ts` | Async via `BackgroundManager` |
| **Sync** | `sync-executor.ts` | Create session → wait for idle → return result |

## KEY FILES

| File | Purpose |
|------|---------|
| `tools.ts` | `createCallOmoAgent()` factory — validates agent, routes to executor |
| `background-executor.ts` | Launch background work via `BackgroundManager.launch()` |
| `sync-executor.ts` | `executeSync`: create/reuse session -> `dispatchInternalPrompt` (shared prompt gate, never raw SDK prompt) -> poll -> fetch result; created sessions cleaned up in `finally` |
| `agent-resolver.ts` | `resolveCallableAgents` / `clearCallableAgentsCache`: normalize + validate the requested agent |
| `session-creator.ts` | `createOrGetSession` for sync execution |
| `subagent-session-creator.ts` | `resolveOrCreateSessionId`: create session with agent-specific config |
| `completion-poller.ts` | `waitForCompletion`: poll until session idle |
| `message-processor.ts` | `processMessages`: extract the last assistant output as result |
| `message-dir.ts` | Re-exports `getMessageDir` from `../../shared/opencode-message-dir` |
| `types.ts` + `tool-context-with-metadata.ts` | `CallOmoAgentArgs`, `AllowedAgentType`, `CallOmoAgentSyncResult`, `ToolContextWithMetadata` |

## SESSION CONTINUATION

Pass `session_id` to resume an existing session rather than create a new one; sync mode only (`sync-executor.ts` + `session-creator.ts`).

## SYNC LIFECYCLE GUARDS

Pinned by co-located regressions: `sync-executor-reawaken-guard.test.ts` (a finished child is not re-woken), `reused-sync-session-delete-cleanup.test.ts` (resumed sync sessions are deleted after handback), `sync-executor-leak.test.ts`.
