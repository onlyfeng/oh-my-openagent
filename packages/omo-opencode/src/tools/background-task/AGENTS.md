# src/tools/background-task/ — Background Task Tool Wrappers

**Generated:** 2026-10-13

## OVERVIEW

28 files (~2.7k LOC incl. tests). Tool-layer wrappers for `background_output` and `background_cancel`, registered by `createBackgroundTools()` in `../index.ts`. Does NOT implement the background execution engine; that lives in `src/features/background-agent/`. This directory provides the LLM-facing tool interface.

## THREE TOOLS

| Tool | Factory | Purpose |
|------|---------|---------|
| `background_output` | `createBackgroundOutput` | Get results from a running/completed background task |
| `background_cancel` | `createBackgroundCancel` | Cancel running task(s) |
| `createBackgroundTask` | exported, not registered by `createBackgroundTools()` | Launch-tool factory (`description`, `prompt`, `agent` args) kept for callers that need it |

## KEY FILES

| File | Purpose |
|------|---------|
| `index.ts` + `tools.ts` | Barrel + wiring: `index.ts` re-exports the factories from `tools.ts` |
| `create-background-output.ts` | `background_output` tool: fetch task results by task_id |
| `create-background-cancel.ts` | `background_cancel` tool: cancel by taskId or all=true |
| `create-background-task.ts` | `createBackgroundTask` launch-tool factory (exported only) |
| `clients.ts` | Client interfaces for background output and cancel |
| `session-messages.ts` | Fetch session messages from OpenCode |
| `full-session-format.ts` | Format full session output (messages, thinking blocks) |
| `task-result-format.ts` | Format task result for LLM consumption |
| `task-status-format.ts` | Format task status (running/completed/error) |
| `message-dir.ts` | Temp directory for message exchange |
| `truncate-text.ts` | Truncate large output to fit context |
| `time-format.ts` | Human-readable duration formatting |
| `delay.ts` | Polling delay utility |
| `with-sdk-call-timeout.ts` | SDK call timeout wrapper (pinned by `sdk-call-timeout.test.ts`) |
| `types.ts` | `BackgroundTaskOptions`, result/status types |
| `constants.ts` | Timeout defaults, polling intervals |

## BACKGROUND OUTPUT MODES

```
background_output(task_id, block=false)  -> check current status/result
background_output(task_id, block=true)   -> wait until complete (`timeout` ms: default 60000, max 600000; timeout appends a "still running" note)
background_output(task_id, full_session=true) -> full transcript (+ include_tool_results, since_message_id, from_end)
background_output(task_id, message_limit=N) -> last N messages only (capped at 200)
background_output(task_id, include_thinking=true) -> include thinking blocks (thinking_max_chars default 2000)
```

## RELATIONSHIP TO BACKGROUND ENGINE

```
tools/background-task/  ← LLM tool interface
features/background-agent/  ← execution engine (BackgroundManager)
```

`createBackgroundOutput` queries `BackgroundManager.getTask(task_id)` — it does not manage task state.
