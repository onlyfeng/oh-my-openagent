# comment-checker - Senpi Adapter Component (direct directory)

**Score:** 17 (21 files, 2.2k LOC, >20 files, index barrel, injected binary seams). Parent [`packages/omo-senpi/AGENTS.md`](../../../../AGENTS.md): "runs the shared comment-checker flow after write-like tool results when a resolver finds the binary." Core parse/run logic lives in `@oh-my-opencode/comment-checker-core`; the Codex hook edition is a separate package at `packages/omo-codex/plugin/components/comment-checker/`.

## OVERVIEW

Senpi-side component: resolves/downloads the checker binary, runs it after write-like tool results, dedupes per turn, and prepends `COMMENT_CHECKER_FEEDBACK_HEADER` feedback to the next prompt.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Barrel | `index.ts` - `createCommentCheckerComponent`, `COMMENT_CHECKER_FEEDBACK_HEADER`, `downloadSenpiCommentCheckerBinary`, `resolveSenpiCommentCheckerBinary`, `defaultCommentCheckerCacheDir` |
| Registration + event filtering | `component.ts` (156 LOC) - write-tool detection, per-turn dedup, binary init, execution, feedback/failure reporting |
| Binary resolution / download / run | `resolver.ts`, `downloader.ts`, `runner.ts` |
| Input shaping + notices | `hook-input.ts`, `failure-notice.ts`, `types.ts`, `constants.ts`, `utils.ts` |
| Tests | 10 colocated suites incl. `stale-cache`, `missing-package`, `start-failure`, `stdin-pipe`, `turn-reset.characterization` |

## CONVENTIONS

- Binary resolution, download, and execution are injected dependencies - component tests substitute these seams, never the network.
- Runtime is Node: tests explicitly assert `runner.ts`/`resolver.ts` do not depend on Bun globals (the built Senpi runtime runs under Node).
- Shebang permission/start-failure cases are skipped on Windows; the real split-session integration is separately characterized - keep those platform distinctions.

## ANTI-PATTERNS

- Do not import Codex-plugin (`omo-codex`) code here; shared behavior goes through `comment-checker-core`.
- Do not run the binary on non-write tool results; `component.ts` owns the filter.
- Missing binary is a quiet skip, not an error surface.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/comment-checker
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
