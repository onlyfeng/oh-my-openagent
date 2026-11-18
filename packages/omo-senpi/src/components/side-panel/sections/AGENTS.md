# side-panel/sections/ — Side-Panel Row Builders

**Score ~11** (20 files; pure row-builder domain, composed by parent `rows.ts`).

## OVERVIEW

Pure builder functions that turn side-panel snapshots into `PanelRow[]` for each panel section (agents, context, files, goal, memory, session, tools, usage, location). The parent `rows.ts` composes the main builders; shared formatting primitives live in `layout.ts`. No host I/O — each builder takes a snapshot plus explicit width/time/cap inputs.

## WHERE TO LOOK

| File | Builder |
|------|---------|
| `agents.ts` | `buildAgentRows`, `buildAgentCardRows` |
| `context.ts` | `buildContextRows` |
| `files.ts` | `buildFileRows` |
| `goal.ts` | `buildGoalRows` |
| `memory.ts` | `buildMemoryRows`, `buildMemoryDetailRows` |
| `session.ts` | `buildSessionRows` |
| `tools.ts` | `buildToolRows` |
| `usage.ts` | `buildUsageRows` |
| `location.ts` | Location rows |
| `layout.ts` | Shared formatting primitives |

## CONVENTIONS

- Each builder is pure: snapshot + explicit width/time/cap inputs → readonly `PanelRow[]`. No I/O, no globals.
- Formatting primitives centralized in `layout.ts`; do not re-implement per section.
- Colocated `*.test.ts` per section (Bun `#given/#when/#then`).

## ANTI-PATTERNS

- Keep builders pure — do not add host/config reads inside a section builder; pass values in.

## COMMANDS

- `bun test packages/omo-senpi` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json`
