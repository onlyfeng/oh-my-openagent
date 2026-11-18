# features/task-toast-manager - Transient Task Toasts

**Generated:** 2026-11-18 | c2691818a9

Task lifecycle toasts (started/progress/completed/failed) for the OpenCode TUI. Distinct domain: the single owner of the toast component lifecycle - the rest of the plugin calls a manager, never renders a toast itself.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `manager.ts` | `TaskToastManager`: create/update/dismiss lifecycle; owns timing and de-dup of identical task events. |
| `types.ts` | `TaskToastStatus`, options/record types. |
| `index.ts` | Barrel: `getTaskToastManager`, `initTaskToastManager`, `resetTaskToastManager` + types. |
| `manager.test.ts` | bun test coverage. |

## CONVENTIONS

- Module exposes a singleton via `getTaskToastManager`/`initTaskToastManager`; tests call `resetTaskToastManager` between cases.
- Toast status is driven by task events, not wall-clock polling.
- The manager de-duplicates identical task events; callers do not need to track what was already shown.

## ANTI-PATTERNS

- Do not render a toast directly from a task hook; always go through the manager so dismissal and de-dup stay centralized.

## COMMANDS

```bash
bun test packages/omo-opencode/src/features/task-toast-manager
```

OpenCode-wired; `opencode-qa` skill + `.omo/evidence/` per the root AGENTS.md mandate.
