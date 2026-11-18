# tmux — OpenCode Tmux Surface (Shim)

**Score:** 10 (40 files incl. `tmux-utils/`; a re-export surface over `@oh-my-opencode/tmux-core`; consumed by `src/create-managers.ts`, `features/tmux-subagent/`, `tools/interactive-bash/`).

## OVERVIEW

Thin, deliberately stable import surface. Real logic lives in `packages/tmux-core` (harness-neutral) and in `tmux-utils/` (primitives, own AGENTS.md). New behavior belongs upstream first, then a re-export here.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: `export *` from types / constants / cmux-detect / runner / tmux-utils |
| `runner.ts` | Re-exports `runTmuxCommand` / `TmuxCommandResult` from tmux-core |
| `cmux-detect.ts` | Re-exports `isCmuxCompatEnvironment` from tmux-core |
| `types.ts` | `SpawnPaneResult`, `TmuxConfig`, `TmuxIsolation`, `TmuxLayout` |
| `constants.ts` | Session/poll timing constants |
| `tmux-utils.ts` | Curated re-exports from selected `tmux-utils/` modules (environment, server-health, pane-dimensions) |
| `tmux-utils/` | Primitives with their own contracts — [`tmux-utils/AGENTS.md`](tmux-utils/AGENTS.md) |

## CONVENTIONS

- Cross-package consumers (`team-core`, `openclaw-core`) import `@oh-my-opencode/tmux-core` directly; this shim exists for omo-opencode-internal import paths. Keep it re-export-only.

## ANTI-PATTERNS

- Never implement tmux logic at this level — re-export only; drift between shim and tmux-core is a defect.
- `tmux-utils/index.ts` exports ONLY `killTmuxSessionIfExists`; everything else is consumed via direct file imports. Do not widen that barrel.

## COMMANDS

```bash
bun test packages/omo-opencode/src/shared/tmux/
```
