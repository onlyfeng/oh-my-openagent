# src/features/boulder-state/ — Active Work Plan Tracker

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Adapter shim: 6 re-export files + 2 tests (`storage.test.ts` 1.2k LOC). The implementation lives in [`packages/boulder-state/`](../../../../boulder-state) (`@oh-my-opencode/boulder-state`); `formatDurationHuman` comes from `@oh-my-opencode/utils`. Tracks Sisyphus's "boulder" - the active work plan being rolled across sessions, worktrees, and subagent task delegations. Named after the Sisyphus myth: the boulder must keep rolling until the plan is complete.

Inspected interactively via `bunx oh-my-opencode boulder` (see [`src/cli/boulder/`](../../cli/boulder), registered in `src/cli/runtime-commands.ts`).

## SCHEMA (v2)

```typescript
interface BoulderState {
  schema_version?: 2
  active_work_id?: string
  works?: Record<string, BoulderWorkState>
  active_plan: string                            // absolute path to active .md plan
  started_at: string                             // ISO timestamp
  ended_at?: string
  elapsed_ms?: number
  status?: "active" | "completed" | "paused" | "abandoned"
  session_ids: string[]                          // every session that has rolled the boulder
  session_origins?: Record<string, "direct" | "appended">
  plan_name: string                              // filename of active_plan
  agent?: string                                 // resume agent (atlas | sisyphus | ...)
  worktree_path?: string                         // git worktree root
  task_sessions?: Record<string, TaskSessionState>  // reusable subagent sessions per top-level task
}
```

## FILES

All are re-exports; edit the Core package, not these files.

| File | Re-exports |
|------|------------|
| `types.ts` | `BoulderState`, `BoulderWorkState`, `TaskSessionState`, status types |
| `storage.ts` | Atomic CRUD on `.omo/boulder.json` (`createBoulderState`, `addBoulderWork`, `getWorkForSession`, ...) |
| `constants.ts` | `BOULDER_DIR` (`.omo`), `BOULDER_FILE` (`boulder.json`) |
| `top-level-task.ts` | `readCurrentTopLevelTask` |
| `format-duration.ts` | `formatDurationHuman(ms)` from utils |
| `index.ts` | Barrel |

## LIFECYCLE

```
session.ulwExecute(plan)
  → BoulderState created with active_plan, started_at, plan_name
  → atlas-hook reads BoulderState + `task_sessions` on session.idle (boulder continuation + subagent resume)
session.idle (incomplete plan)
  → todoContinuationEnforcer + atlasHook inspect state
  → Inject CONTINUATION_PROMPT or BOULDER_COMPLETE_PROMPT
session.completed
  → BoulderState status="completed", ended_at, elapsed_ms recorded
```

## INTEGRATION POINTS

| Where | What |
|-------|------|
| [`src/cli/boulder/`](../../cli/boulder) | CLI inspector formats this state |
| [`src/hooks/atlas/`](../../hooks/atlas) | Reads work state + `task_sessions` (subagent resume); drives boulder-complete and parallel-delegation prompts |
| [`src/hooks/ulw-execute/`](../../hooks/ulw-execute) | Creates the BoulderState on `/ulw-execute` invocation |
| [`src/hooks/todo-continuation-enforcer/`](../../hooks/todo-continuation-enforcer) | Session-idle continuation when boulder incomplete |

## STORAGE

```
<worktree-root>/.omo/boulder.json   # gitignored; one file per worktree
```

Atomic writes: temp file, fsync (where supported), rename. File lock prevents concurrent corruption. Schema migrations between versions are handled in the Core `storage.ts`.

## NOTES

- **task_sessions reuse**: same subagent session is reused across iterations for the same top-level task to preserve context.
- **Multi-work**: `works` map allows tracking multiple concurrent plans; `active_work_id` selects the current one.
- **Worktree-scoped**: state lives in the worktree, not user-global; ensures parallel work plans across worktrees stay isolated.
