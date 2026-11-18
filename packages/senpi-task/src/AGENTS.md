# src/ - Root Barrel + Top-Level Helpers (direct files)

**Score:** 15 (23 direct files, 4k LOC, `index.ts` is 713 lines of public API). Parent [`packages/senpi-task/AGENTS.md`](../AGENTS.md) owns the subdirectory anatomy (state, store, runners, dag, ...); this file covers ONLY the directory's own files.

## OVERVIEW

The package's public barrel plus cross-cutting helpers that predate or sit above the subsystem directories: model chains, run statistics, progress, status rows, notice boxes, agent-home resolution.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Public API | `index.ts` - the barrel every consumer imports; engine primitives re-exported from subsystems |
| Model chains | `model-chain.ts` (`buildRuntimeModelChain`), `model-exclusion.ts` (`withoutExcludedModels`) - consumed by `agents/` + `category/` resolvers |
| Run statistics | `run-stats.ts` (`createRunStatsTracker`) - consumed by `progress/` and `manager/`; pinned by 4 suites (`run-stats*.test.ts`) |
| Progress + status | `progress.ts` (`createChildProgress`), `status-line.ts` (status-row formatters), `task-summary.ts` |
| Rendering + misc | `renderer-text.ts` (bounded renderer output), `notice-box.ts`, `agent-home.ts` |
| Chain policy + audits | `luna-deepseek-chain-policy.test.ts`, `senpi-api-tripwire.test.ts` |

## CONVENTIONS

- Injectable clocks back the run-stats/event accounting; tests pass fakes, never sleep.
- Renderer output is bounded by design (`renderer-text.ts`) - never widen a bound without its test.

## ANTI-PATTERNS

- **Static-import guard** (`senpi-static-import-guard.test.ts`): runtime VALUE imports from `@code-yeongyu/senpi` and `@earendil-works/pi-tui` are allowed ONLY behind the memoized dynamic-import barrels in `src/lazy`; type-only static imports are fine. Violating this drags engine/TUI barrels into every fresh RPC-child boot graph. Tests are excluded from the shipped-source guard.
- Do not add new exports without checking the deep-subpath export map in `package.json` - the barrel and subpaths must stay consistent.

## COMMANDS

```bash
bun test --timeout 20000 packages/senpi-task/src           # direct-file suites (subdirs have their own)
bun run --cwd packages/senpi-task typecheck
```
