# src/features/tmux-subagent/ — Tmux Pane Management

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

This module provides state-first tmux integration for background agent sessions (31 source + 17 test files, ~9.8k LOC; `manager.ts` 1.4k and `manager.test.ts` 3.2k are the hotspots). It handles split decisions, grid planning, polling, and lifecycle events. Reusable tmux command, layout, and pane primitives are extracted to [`packages/tmux-core/`](../../../../../packages/tmux-core); this directory is the OpenCode session adapter.

## CORE ARCHITECTURE

```
TmuxSessionManager (manager.ts)
  ├─→ DecisionEngine: Should we spawn/close panes?
  ├─→ ActionExecutor: Execute spawn/close/replace actions
  ├─→ PollingManager: Monitor pane health
  └─→ EventHandlers: React to session create/delete
```

All tmux command execution is centralized through the shared `runTmuxCommand` runner (via the local adapter shim `src/shared/tmux/`). Do NOT add direct `Bun.spawn([tmux,...])` calls in this module. They will drift from the retry/timeout/terminal-error discipline.

## KEY FILES

| File | Purpose |
|------|---------|
| `manager.ts` | `TmuxSessionManager`: main class, session tracking, event routing (constructed in `src/create-managers.ts`; also used by team-mode layout and lifecycle tools) |
| `environment-eligibility.ts` | `selectTmuxManagerEnvironmentPredicate`: `inline` isolation uses `isTmuxPaneCompatible`, other modes `isNativeTmux` |
| `decision-engine.ts` | Evaluate window state → produce `SpawnDecision` with actions |
| `action-executor.ts` | Execute `PaneAction[]` (close, spawn, replace) |
| `grid-planning.ts` + `tmux-grid-constants.ts` | Calculate pane layout given window dimensions; grid constants |
| `stale-tmux-resource-sweeper.ts` | `sweepStaleTmuxResources`: removes leftover tmux resources, returns a sweep report |
| `spawn-action-decider.ts` | Decide spawn vs replace vs skip |
| `spawn-target-finder.ts` | Find best pane to split or replace |
| `polling-manager.ts` | Health polling for tracked sessions |
| `types.ts` | `TrackedSession`, `WindowState`, `PaneAction`, `SpawnDecision` |

## PANE LIFECYCLE

```
session.created → spawn-action-decider → grid-planning → action-executor → track session
session.deleted → cleanup tracked session → close pane if empty
```

## LAYOUT CONSTRAINTS

- `MIN_PANE_WIDTH`: 52 chars, `MIN_PANE_HEIGHT`: 11 lines (`types.ts`)
- Grid limits (`tmux-grid-constants.ts`): `MAX_COLS` 2, `MAX_ROWS` 3, `MAX_GRID_SIZE` 4, `DIVIDER_SIZE` 1
- Main pane preserved (never split below minimum)
- Agent panes split from remaining space

## ISOLATION MODES

Supports `inline` (panes in the current window), `window` (dedicated window), and `session` (isolated tmux session). Cmux-compatible/headless environments are detected (`isCmuxCompatEnvironment` from `src/shared/tmux/cmux-detect.ts`) and degrade to deferred-attach queues with readiness TTLs; never-activated panes hit a hard timeout via `polling-manager.ts`.

## EVENT HANDLERS

| File | Event |
|------|-------|
| `session-created-handler.ts` | New background session → spawn pane |
| `session-deleted-handler.ts` | Session ended → close pane |
| `session-created-event.ts` | Event type definition |
