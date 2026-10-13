# src/cli/run/ — Non-Interactive Session Launcher

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

61 TypeScript files (33 impl + 28 colocated tests). Powers the `oh-my-opencode run <message>` command. Connects to OpenCode server, creates/resumes sessions, streams events, and polls for completion. Options: `--agent`, `--model`, `--directory`, `--port` (attaches if in use), `--attach <url>`, `--session-id`, `--on-complete`, `--json`, `--no-timestamp`, `--verbose`.

## EXECUTION FLOW

```
runner.ts
  1. opencode-binary-resolver.ts -> find OpenCode binary
  2. server-connection.ts -> connect to OpenCode server (start if needed)
  3. agent-resolver.ts -> flag -> env -> config -> Sisyphus; runnable-agent-resolver.ts checks the server's agent list (UnknownRunAgentError)
  4. session-resolver.ts -> create new or resume existing session; model-resolver.ts applies --model
  5. events.ts -> stream SSE events from session; prompt-start.ts waits for the prompt to start
  6. event-handlers.ts -> process each event type
  7. poll-for-completion.ts -> wait for todos + background tasks done
  8. on-complete-hook.ts -> execute user-defined completion hook
```

## KEY FILES

| File | Purpose |
|------|---------|
| `runner.ts` | Main orchestration — connects, resolves, runs, completes |
| `server-connection.ts` | Start OpenCode server process, create SDK client |
| `agent-resolver.ts` | Resolve agent: `--agent` flag -> `OPENCODE_DEFAULT_AGENT` env -> `default_run_agent` -> Sisyphus; a disabled pick falls back through core order sisyphus, hephaestus, prometheus, atlas |
| `runnable-agent-resolver.ts` | `resolveRunnableRunAgent()`: validates the resolved name against the server agent list |
| `session-resolver.ts` | Create new session or resume via `--attach` / `--session-id` |
| `events.ts` | SSE event stream subscription |
| `event-handlers.ts` | Barrel over `event-{message,session,tool,toast}-handlers.ts` (message, tool, error, idle) |
| `event-stream-processor.ts` | Dispatches each streamed event to the handler set |
| `stdin-suppression.ts` | `suppressRunInput()`: keeps terminal input from leaking into the run |
| `poll-for-completion.ts` | Poll session until todos complete + no background tasks |
| `completion.ts` | Determine if session is truly done |
| `continuation-state.ts` | Persist state for `run` continuation across invocations |
| `output-renderer.ts` | Format session output for terminal |
| `json-output.ts` | JSON output mode (`--json` flag) |
| `types.ts` | `RunOptions`, `RunResult`, `RunContext`, event payload types |

## AGENT RESOLUTION PRIORITY

```
1. --agent CLI flag
2. OPENCODE_DEFAULT_AGENT environment variable
3. default_run_agent config
4. "sisyphus" (default)
```

## COMPLETION DETECTION

Poll-based with two conditions:
1. All todos marked completed (no pending/in_progress)
2. No running background tasks

`on-complete-hook.ts` executes optional user command on completion (e.g., `--on-complete "notify-send done"`).

Idle detection handles both the deprecated `session.idle` event and `session.status` (`event-session-handlers.ts`). Largest tests: `message-part-delta.test.ts` (657), `completion-continuation.test.ts` (605), `poll-for-completion.test.ts` (589).
