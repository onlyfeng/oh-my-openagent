# src/tools/ — Task / Team / Workpool ToolDefinitions

**Score:** 13 (149 `.ts` files incl. subdirs; the `ToolDefinition` layer registered by omo-senpi's task component).

## OVERVIEW

Tool families over the engine: `task/` (spawn — own [`task/AGENTS.md`](task/AGENTS.md)), `control/` (`task_send` / `task_cancel`), `output/` (`task_output`), `team/` (the 6 lead-only team tools), plus the `workpool` pair and shared stats formatting at this level. Tool names, factories, and batch/spawn semantics are owned by [`../../AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Path | Role |
|------|------|
| `workpool.ts` | `createWorkpoolTool` / `createWorkpoolWorkerTool` / `buildWorkpoolExecute`; `WorkpoolToolDeps` = `TaskToolDeps & { workpools: WorkpoolEngine }` |
| `workpool-schema.ts` | TypeBox `WorkpoolParams` / `WorkpoolYieldParams` / `WorkpoolWorkerYieldParams` |
| `run-stats-format.ts` | `formatRunDuration`, `runStatsSuffix`, `runStatsCardSummary` — shared run-stats rendering |
| `control/`, `output/`, `team/` | Send/cancel, output peek, lead-only team family |

## CONVENTIONS

- Package tool pattern: TypeBox schema in `*-schema.ts`, `parseInput(Schema, value)` at the boundary, and a `result(details)` envelope — `content: [{ type: "text", text: JSON }]` + `details` + optional `isError`.
- Policy failures are error codes (`policy_denied`), never thrown raw errors or silent rewrites; the envelope grants only yield, never caller-supplied identity.
- Senpi imports stay behind the lazy `loadSenpiBarrel`.

## ANTI-PATTERNS

- Spawn policy deny/force is enforced: force with a drifted prompt is `policy_denied`, never a silent rewrite.
- Do not widen the team family to child/member sessions (lead-only, enforced upstream).

## COMMANDS

```bash
bun test packages/senpi-task/src/tools/
```
