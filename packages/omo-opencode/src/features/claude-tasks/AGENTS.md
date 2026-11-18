# src/features/claude-tasks/ — Task Schema + Storage

**Generated:** 2026-11-18 / c2691818a9 (update pass; prior 2026-05-15)

## OVERVIEW

3 non-test files. File-based task persistence with atomic writes, locking, and OpenCode todo API sync.

## TASK SCHEMA

```typescript
const TaskSchema = z.object({
  id: string                    // T-{uuid} auto-generated
  subject: string               // Short title
  description: string           // Detailed description (required)
  status: "pending" | "in_progress" | "completed" | "deleted"
  activeForm?: string           // Current form/template
  blocks: string[]              // Tasks this blocks (required array)
  blockedBy: string[]           // Tasks blocking this (required array)
  owner?: string                // Agent/session
  metadata?: Record<string, unknown>
}).strict()                     // unknown keys rejected
```

## FILES

| File | Purpose |
|------|---------|
| `types.ts` | Task interface + status types |
| `storage.ts` | `readJsonSafe()`, `writeJsonAtomic()`, `acquireLock()` (async, bounded wait), `resolveLockWaitTimeoutMs()`, `generateTaskId()`, `listTaskFiles()`, task dir/path helpers |
| `index.ts` | Barrel exports |

## STORAGE

- Location: `.omo/tasks/` directory
- Format: JSON files, one per task
- Atomic writes: temp file → rename
- Locking: file-based lock for concurrent access. `acquireLock()` is async and waits out ordinary
  contention with jittered retry (default 5s budget, override with `OMO_TASK_LOCK_WAIT_TIMEOUT_MS`);
  it reports `acquired: false` only once that budget is exhausted. A lock older than 30s is reclaimed
  as stale regardless of the wait budget.
- Critical section: only the atomic task-file write is done under the lock; Todo API sync happens
  after release
- Sync: Changes pushed to OpenCode Todo API after each update
