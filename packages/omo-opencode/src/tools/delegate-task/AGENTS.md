# src/tools/delegate-task/ — Task Delegation Engine

**Generated:** 2026-10-13 | ~100 files (~58 implementation, ~44 tests, ~22k LOC incl. tests; `tools.test.ts` alone ~4.9k) + `zauc-mocks-subagent-resolver/`

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
| `tools.ts` | `createDelegateTask()` factory — main entry point |
| `executor.ts` | Route to background or sync execution |
| `types.ts` | `DelegateTaskArgs`, `DelegateTaskToolOptions`, `ToolContextWithMetadata` |
| `category-resolver.ts` | Map category name → model + config |
| `subagent-resolver.ts` | Map subagent_type → agent + model |
| `model-selection.ts` | Model availability checking + fallback |
| `skill-resolver.ts` | Resolve `load_skills[]` → skill content for injection |
| `prompt-builder.ts` | Build system/user prompt with skill content, categories |

## SYNC EXECUTION CHAIN

```
sync-task.ts (spawn reservation, metadata, lifecycle side effects)
  -> sync-task-runner.ts (runSyncTaskLoop; fallback retries via sync-task-fallback.ts, poll-error recovery)
  -> sync-task-deps.ts: sync-session-creator.ts -> sync-prompt-sender.ts -> sync-session-poller.ts -> sync-result-fetcher.ts
  -> sync-completion-message.ts; sync-session-cleanup.ts schedules session deletion
```

Each file handles one step; `sync-task-deps.ts` is the injectable deps record wiring the four session steps. `sync-continuation.ts` (+ `sync-continuation-deps.ts`) handles session continuation (resume with task_id). `sync-result-fetcher.ts` does not accept a complete plan tag merely sketched in model output.

## BACKGROUND EXECUTION

```
background-task.ts → BackgroundManager.launch() → (async polling) → background-continuation.ts
```

`background-continuation.ts` handles `task_id` resume for existing background tasks.

## CATEGORY RESOLUTION

1. Check user-defined categories (`pluginConfig.categories`)
2. Fall back to the built-in 9 categories (`BUILTIN_CATEGORIES` in `builtin-categories.ts`, aggregating `google-`/`openai-`/`kimi-categories.ts`; exposed as `DEFAULT_CATEGORIES` in `constants.ts`, also imported by plugin handlers, config migration, and agent prompt builders)
3. Resolve model from category config (`category-resolver.ts`, `categories.ts` `resolveCategoryConfig`)
4. Check model availability (`builtinCategoryGateModels`) -> fallback if unavailable

## SUBAGENT PREFLIGHT

`subagent-request-preflight.ts` rejects coordinator/meta agents listed in `constants.ts` as subagent targets. Passing both `category` and `subagent_type`: category wins (`tool-description.ts`). Team runtime reuses `resolveCategoryExecution`, `resolveSubagentExecution`, `buildSystemContent`, and `ExecutorContext` from here.

## MODEL STRING PARSER

`parseModelString` lives in `../../shared/model-string-parser.ts` (not in this dir) and handles `"model variant"` format (e.g., `"gpt-5.5 medium"` -> model=`gpt-5.5`, variant=`medium`); consumed by `categories.ts` and `category-resolver.ts`.

## UNSTABLE AGENT TRACKING

`unstable-agent-task.ts` marks tasks from categories/agents known to be unstable (e.g., free models). Enables `unstableAgentBabysitter` hook monitoring.
