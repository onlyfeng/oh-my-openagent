# src/tools/output/ - The `task_output` Tool

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 22 (27 files incl. `transcript/` + `__fixtures__/`; barrel re-exported from the package barrel). Tool-name semantics and the package tool envelope pattern are owned by [`../../tools/AGENTS.md`](../AGENTS.md).

## OVERVIEW

Output/status/transcript peeks: params schema, polling guard, bounded transcript rendering, run-stats renderers, snapshots.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: `createTaskOutputTool`, `runTaskOutput`, `TaskOutputParams`, `renderTranscript`, `TRANSCRIPT_MAX_CHARS`, `buildTaskSnapshot`, `awaitingInputDetails` |
| `output.ts` | Tool body - params parsing, record lookup, snapshot build |
| `render.ts` | `renderTranscript` - transcript rendering bounded by `TRANSCRIPT_MAX_CHARS` |
| `renderers.ts` | Run-stats/status rendering rows (own + run-stats suites) |
| `snapshot.ts` | `buildTaskSnapshot` - status snapshot incl. awaiting-input + suspension states |
| `transcript/` | Transcript source assembly |
| `types.ts` | Shared types |
| `polling-guard.test.ts` | Polling behavior guard |
| `__fixtures__/` | Fixtures shared by the suites |

## ANTI-PATTERNS

- Transcript output stays bounded by `TRANSCRIPT_MAX_CHARS` - never widen the bound without its test.
- Snapshots must reflect awaiting-input and suspended states accurately (`snapshot-awaiting-input.test.ts`, `snapshot-suspension.test.ts`) - a stale snapshot misleads `task_send` decisions upstream.

## COMMANDS

```bash
bun test packages/senpi-task/src/tools/output
bun run --cwd packages/senpi-task typecheck
```
