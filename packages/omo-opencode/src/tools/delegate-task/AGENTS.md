# src/tools/delegate-task/ - Task Delegation Engine

**Generated:** 2026-11-18 | c2691818a9

## OVERVIEW

The `task` tool implementation delegates work to subagents via background or sync sessions. It resolves categories, models, skills, and manages both async and synchronous execution flows. Harness-neutral task selection and retry primitives are extracted to [`packages/delegate-core/`](../../../../../packages/delegate-core); this directory keeps OpenCode session/tool execution wiring. Built-in categories are defined alongside user category overrides.

## TWO EXECUTION MODES

| Mode | Flow | Use Case |
|------|------|----------|
| **Background** (`run_in_background=true`) | Launch → BackgroundManager → poll → notify parent | The standard spawn for every target (explore, librarian, categories); the parent keeps working or ends its response and the completion notification wakes it |
| **Sync** (`run_in_background=false`) | Create session → send prompt → poll until idle → return result | A short child whose result gates the parent's very next call |

## KEY FILES

| File | Purpose |
|------|---------|
| `tools.ts` | `createDelegateTask()` factory - main entry point |
| `executor.ts` | Execution-routing barrel: re-exports the `execute*` entry points and resolvers |
| `types.ts` | `DelegateTaskArgs`, `DelegateTaskToolOptions`, `ToolContextWithMetadata` |
| `category-resolver.ts` | Map category name → model + config |
| `subagent-resolver.ts` | Map subagent_type → agent + model |
| `model-selection.ts` | Model availability checking + fallback |
| `skill-resolver.ts` | Resolve `load_skills[]` → skill content for injection |
| `prompt-builder.ts` | Build system/user prompt with skill content, categories |
| `available-models.ts` | `getAvailableModelsForDelegateTask()` - provider model listing |
| `subagent-request-preflight.ts` | Reject direct Sisyphus-Junior, plan→plan, and coordinator targets |
| `timing.ts` | Sync-polling / unstable-monitor timings (+ `__setTimingConfig` test seams) |

## SYNC EXECUTION CHAIN

```
sync-task.ts → sync-session-creator.ts → sync-prompt-sender.ts → sync-session-poller.ts → sync-result-fetcher.ts
```

Each file handles one step. `sync-continuation.ts` handles session continuation (resume with task_id).

## BACKGROUND EXECUTION

```
background-task.ts → BackgroundManager.launch() → (async polling) → background-continuation.ts
```

`background-continuation.ts` handles `task_id` resume for existing background tasks.

## CATEGORY RESOLUTION

1. Check user-defined categories (`pluginConfig.categories`)
2. Fall back to 9 built-in categories (provider-partitioned: `google-categories.ts` 2, `openai-categories.ts` 6, `kimi-categories.ts` 1; collected in `builtin-categories.ts`)
3. Resolve model from category config
4. Check model availability → fallback if unavailable

## MODEL STRING PARSER

Moved out of this directory: `packages/omo-opencode/src/shared/model-string-parser.ts` handles the `"model variant"` format (e.g., `"gpt-5.5 medium"` → model=`gpt-5.5`, variant=`medium`).

## UNSTABLE AGENT TRACKING

`unstable-agent-task.ts` marks tasks from categories/agents known to be unstable (e.g., free models). Enables `unstableAgentBabysitter` hook monitoring.

## ANTI-PATTERNS

- `category` XOR `subagent_type` - supplying both is a contract mistake (category wins in execution).
- `task_id` is a continuation session id (`ses_...`), never a background id; background launch returns `bg_...` for `background_output`.
- Omitted `run_in_background` counts as `false` (sync); only explicit `true` is background.
- Sync session creation and unstable background launch deny the `question` permission.
- `zauc-mocks-subagent-resolver/` is test-only; do not add runtime code there.
