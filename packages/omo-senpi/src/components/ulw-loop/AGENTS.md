# ulw-loop component

Senpi side of the ulw-loop goal loop: reads the session's ulw-loop plan in-process, keeps the footer spinner honest, steers live input with a reminder, and injects at most 8 consecutive continuations. Plan storage and operations belong to the Codex-side toolkit (`packages/omo-codex/plugin/components/ulw-loop/`); this directory only adapts it. Own file because it is a distinct domain (23 files, `index.ts` boundary, nested `sdk/`, score 9).

## WHERE TO LOOK

| Path | Purpose |
|------|---------|
| `index.ts` | `createUlwLoopComponent`: `session_start` / `input` / `agent_end` / `agent_settled` / `tool_result` handlers, `CONTINUATION_LIMIT` (8), `STEERING_REMINDER`, `CONTINUATION_PROMPT`, the removed-tool hint for `omo_agent_toolkit`. |
| `status-source.ts` | `readUlwLoopStatusInProcess`: `createAgentToolkit({ cwd, sessionId, surface: "omo-senpi" }).status()` through the `#omo-agent-toolkit-sdk` alias, wrapped in the old `{ code, stdout }` envelope. |
| `session-scope.ts` | Resolves the host's own session id and the scoped `<cwd>/.omo/ulw-loop/<sessionId>/goals.json`; mirrors the toolkit's `paths.ts` normalization (parity-tested, the adapter may not import that package). |
| `footer-status.ts` | Animated footer frames (`ULW_LOOP_FOOTER_FRAMES`), injectable timers, disposed on switch/shutdown. |
| `sdk/session-toolkit.ts` | `agentToolkit`: the per-call session-bound singleton the eval SDK exports (`extension/agent-toolkit-sdk.ts` -> `plugin/runtime/agent-toolkit-sdk/sdk.js`). |
| `sdk/session-binding.ts` / `sdk/session-file-header.ts` | `toolkitContextFromEnv`: session cwd from `PI_SESSION_CWD`, else the session JSONL header (`PI_SESSION_FILE`), else `process.cwd()`. |
| `sdk/driver-goal.ts` / `sdk/driver-relation.ts` | Read-only relation between the plan and the host goal store; unreadable stores are advisory warnings, never writes. |

## FLOW

- `agent_end` records the run payload plus a status snapshot and refreshes the footer for every ended run; it never decides.
- `agent_settled` decides: skip on a terminal outcome (`readAgentEndOutcome` from `ulw-execute-continuation`), an unscoped session, an inactive plan, or a status identical to the last continuation (`stale-status`); otherwise enqueue `omo-senpi:ulw-continuation` on the idle coordinator and `scheduleFlush()`, or fall back to a `followUp` `sendMessage` when no coordinator is wired.
- A continuable boulder for the same session wins: `findContinuableBoulderWork` skips ulw-loop continuation (`boulder-continuation-active`).
- User-sourced `input` resets the counter; while streaming, an active run appends the reminder to the input text.

## CONVENTIONS

- Every skip logs a `reason` (`continuation-cap-reached`, `terminal-outcome`, `session-id-unavailable`, `inactive`, `stale-status`); `ULW_LOOP_PLAN_MISSING` is debug, other toolkit failures warn.
- Tests inject `readStatus`, `planExists`, and footer timers; `ulw-loop.test-support.ts` holds the fakes.

## ANTI-PATTERNS

- Never fall back to the repo-global `.omo/ulw-loop/goals.json`: without a session id the probe fails closed (`sessionScoped: false`).
- Never decide continuation at `agent_end`; the host still mutates that payload (late Esc, auto-compaction retry) until `agent_settled`.
- Never tell the model to spawn the toolkit CLI or call `omo_agent_toolkit`; status comes from an eval cell via `OMO_AGENT_TOOLKIT_SDK_ROOT`.
- A refused coordinator enqueue is logged, not retried: the continuation is derived state and the next turn re-derives it.
