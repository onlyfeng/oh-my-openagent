# data/ — Side-Panel Data Adapters

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 18 (10 files, ~1.6k LOC; the data seam of `components/side-panel/`, read directly from task-engine records).

## OVERVIEW

Reads task engine records and host facts into side-panel rows: children, facts, goal, memory, child output.

## WHERE TO LOOK

| File | Role |
|------|------|
| `task-records.ts` | `panelChildFromRecord` / `panelChildrenFromRecords`, `PanelTaskRecord` / `PanelTaskRunStats` — maps recovery state to child status |
| `facts.ts` | `panelFactsFrom` / `PanelHostFacts` — guarded host facts |
| `goal.ts` | `createPanelGoalReader` — goal reads |
| `memory.ts` | `createPanelMemoryReader` — durable memory + nudge reads |
| `child-output.ts` | `childOutputRows`, `NO_OUTPUT_NOTICE` — child output rows |

## CONVENTIONS

- Children are scoped by parent session — the panel never sees other sessions' children.
- Every module pairs with a colocated `*.test.ts`.

## ANTI-PATTERNS

- Host fields are guarded and stay ABSENT rather than throwing on unknown shapes.
- Missing cost is NOT zero — omit the field instead of defaulting it.
- A missing session id returns no children, never all sessions' children.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/side-panel/data
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
