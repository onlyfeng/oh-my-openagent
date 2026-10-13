# src/features/claude-tasks/ — Task Schema + Storage

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

2 source files + 2 tests, no barrel (import `./types` / `./storage` directly). File-based task persistence with atomic writes and locking. Consumers: `src/tools/task/` (`task-create`, `task-get`, `task-list`, `task-update`, `todo-sync`).

## TASK SCHEMA

```typescript
// TaskSchema: z.object({...}).strict(); unknown keys are rejected
interface Task {
  id: string              // T-{uuid} from generateTaskId()
  subject: string
  description: string     // required
  status: "pending" | "in_progress" | "completed" | "deleted"
  activeForm?: string
  blocks: string[]        // required
  blockedBy: string[]     // required
  owner?: string
  metadata?: Record<string, unknown>
}
```

## FILES

| File | Purpose |
|------|---------|
| `types.ts` | Zod `TaskSchema` / `TaskStatusSchema` + inferred types |
| `storage.ts` | `getTaskDir()`, `resolveTaskListId()`, `readJsonSafe()`, `writeJsonAtomic()`, `listTaskFiles()`, `acquireLock()` (async, bounded wait), `generateTaskId()` |

## STORAGE

- Location: `sisyphus.tasks.storage_path` if set (relative to cwd), else `<opencode config dir>/tasks/<task_list_id>`
- Format: JSON files, one per task
- Atomic writes: temp file → rename
- Locking: file-based lock for concurrent access. `acquireLock()` is async and waits out ordinary
  contention with jittered retry (default 5s budget, override with `OMO_TASK_LOCK_WAIT_TIMEOUT_MS`);
  it reports `acquired: false` only once that budget is exhausted. A lock older than 30s is reclaimed
  as stale regardless of the wait budget.
- Critical section: only the atomic task-file write is done under the lock; Todo API sync happens
  after release
- Sync: OpenCode Todo API sync lives in `src/tools/task/todo-sync.ts`, not here
