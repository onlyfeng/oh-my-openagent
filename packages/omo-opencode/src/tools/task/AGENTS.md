# task - task_* CRUD tools over the claude-tasks store

**Score 8** (14 modules, ~3.2k LOC; distinct domain: task-system CRUD + todo bridge). Registration/gating (`experimental.task_system`) is covered by the parent `tools/AGENTS.md` - not repeated.

## OVERVIEW

Four `ToolDefinition` factories (`createTaskCreateTool`, `createTaskGetTool`, `createTaskList`, `createTaskUpdateTool`) plus the todo bridge, exported through `index.ts`. Storage is NOT here: every read/write delegates to `../../features/claude-tasks/storage` (`getTaskDir`, `writeJsonAtomic`, `acquireLock`, `generateTaskId`).

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Tool factories | `task-create.ts` (auto `T-{uuid}` id, threadID from context, `blockedBy`/`blocks` dependency-planning prompt), `task-get.ts`, `task-list.ts`, `task-update.ts` |
| Todo bridge | `todo-sync.ts` - `syncTaskToTodo` / `syncAllTasksToTodos` / `syncTaskTodoUpdate`, `TodoInfo`; `mapTaskStatusToTodoStatus` drops `deleted` tasks (null) from the todo list, priority is extracted from `metadata` |
| Schema / model | `types.ts` - `TaskObjectSchema` (`.strict()`), `TaskStatusSchema` (`pending` / `in_progress` / `completed` / `deleted`), Claude Code-style aliases, Zod input schemas |

## CONVENTIONS

- Factories take `(config, ctx?)` and return plugin-sdk tools; tool arg schemas are declared inline with `tool.schema.*`, record models live in `types.ts` (Zod) and are shared with the claude-tasks feature.
- All writes go through the claude-tasks store lock + atomic write; lock behavior is pinned by `task-lock-concurrency.test.ts`.
- Heaviest suites: `todo-sync.test.ts` (569 LOC), `types.test.ts` (522), `task-update.test.ts` (427), `task-list.test.ts` (335).

## NOTES

- Distinct from `packages/senpi-task/src/tools/task/` (the Senpi engine's own task tool surface): this is the OpenCode adapter's record model and prompt contract, not a re-export.
- Todo writes go through an injected `TodoWriter` (`{ sessionID, todos }`), so the bridge stays testable without a live session.

## COMMANDS

- Focused: `bun test packages/omo-opencode/src/tools/task`; package typecheck: `bun run --cwd packages/omo-opencode typecheck` (tsgo).
