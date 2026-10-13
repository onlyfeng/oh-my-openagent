# tmux-utils — Pane/Session/Layout Primitives

**Generated:** 2026-10-13

**Score:** 16 (32 files in this dir; the whole `shared/tmux/` tree incl. parent `runner.ts`/`cmux-detect.ts`/`constants.ts`/`types.ts`/`tmux-utils.ts` totals 40 files, ~2.6k LOC; own module boundary)

## OVERVIEW

Tmux pane/session/window/layout utilities under `shared/tmux/`. Pure decision logic is separated from process execution: `*-runner.ts` modules perform the actual tmux invocation behind injectable spawn deps (`adapter-deps.ts`, `spawn-process.ts`), so the logic modules stay unit-testable.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Panes | `pane-spawn.ts` (+`pane-spawn-runner.ts`), `pane-close.ts` (+runner), `pane-replace.ts`, `pane-activate.ts`, `pane-command.ts`, `pane-dimensions.ts` |
| Sessions / windows | `session-spawn.ts`, `session-kill.ts` (+`session-kill-runner.ts`; barrel exports `killTmuxSessionIfExists`), `window-spawn.ts` |
| Layout | `layout.ts` + `layout-runner.ts` |
| Health / environment | `server-health.ts`, `environment.ts` |
| Spawn injection | `adapter-deps.ts` (injectable deps), `spawn-process.ts` |
| Stale-resource sweeps | `stale-session-sweep.ts` (+`stale-session-sweep-runtime.test.ts`), `stale-attach-pane-sweep.ts` |

## CONVENTIONS

- Barrel `index.ts` exports ONLY `killTmuxSessionIfExists` (~40 references in `src/`); everything else is consumed via direct file imports (notably `src/create-managers.ts` and `features/tmux-subagent/`). The parent `runner.ts` `runTmuxCommand` (~50 references) adapts `@oh-my-opencode/tmux-core`.
- New tmux invocation goes into a `*-runner.ts` with injected spawn deps — never spawn `tmux` inside logic modules; the separation exists so logic stays unit-testable without a tmux binary.

## ANTI-PATTERNS

- NEVER call `tmux` directly from non-runner files.
- NEVER add a second ad-hoc sweep; extend `stale-*-sweep.ts` instead.
