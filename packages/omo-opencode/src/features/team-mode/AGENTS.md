# team-mode — Parallel Multi-Agent Coordination

**Generated:** 2026-11-18 (trimmed from 168 lines; runtime detail in [`team-runtime/AGENTS.md`](team-runtime/AGENTS.md))

## OVERVIEW

Spawns coordinated agent teams with shared mailbox, task list, optional tmux layout, and graceful lifecycle. Modeled after Claude Code Agent Teams. **OFF by default.** Enable via `team_mode.enabled` in `.omo/omo.jsonc`; restart OpenCode after enabling. Harness-neutral registry/mailbox/tasklist/state/worktree/tmux-layout primitives are extracted to [`packages/team-core/`](../../../../../packages/team-core); this directory remains the OpenCode adapter for session spawning, hooks, tools, and config integration.

User docs: [`docs/guide/team-mode.md`](../../../../../docs/guide/team-mode.md).

## CONFIG

Schema: [`src/config/schema/team-mode.ts`](../../config/schema/team-mode.ts). Defaults: `max_parallel_members` 4 (1..8), `max_members` 8 (1..8), `max_messages_per_run` 10000, `max_wall_clock_minutes` 120, `max_member_turns` 500, `message_payload_max_bytes` 32768, `recipient_unread_max_bytes` 262144, `mailbox_poll_interval_ms` 3000, optional `base_dir` override, separate `tmux_visualization` toggle.

## 12 TEAM_* TOOLS

Registered via [`src/plugin/tool-registry.ts`](../../plugin/tool-registry.ts) `teamModeToolsRecord` ONLY when enabled. Lifecycle (`tools/lifecycle.ts`): `team_create`, `team_delete`, `team_shutdown_request`, `team_approve_shutdown`, `team_reject_shutdown`. Messaging (`tools/messaging.ts`): `team_send_message` (member name or `*` broadcast). Tasks (`tools/tasks.ts`): `team_task_create`, `team_task_list`, `team_task_update` (claim/complete/delete, atomic lock), `team_task_get`. Query (`tools/query.ts`): `team_status`, `team_list`.

## ELIGIBLE AGENTS + MEMBER KINDS

[`AGENT_ELIGIBILITY_REGISTRY`](types.ts) in `types.ts`, three tiers: `eligible` = sisyphus, atlas, sisyphus-junior (the only three); `conditional` = hephaestus (needs the D-36 `teammate: "allow"` patch, else use `subagent_type: "sisyphus"`); `hard-reject` = read-only/plan-mode agents (oracle, librarian, explore, multimodal-looker, metis, momus, prometheus) — they throw at TeamSpec parse with a message pointing at `task` (delegate-task). Members: `{ kind: "subagent_type", name, subagent_type, prompt? }` or `{ kind: "category", name, category, prompt }` (prompt REQUIRED; routed through sisyphus-junior with the category model).

## MODULE LAYOUT

`index.ts` (barrel) · `types.ts` (OpenCode-facing wrappers + eligibility registry) · `deps.ts` (git/tmux availability) · `member-parser.ts` / `member-guidance.ts` / `member-session-resolution.ts` / `member-session-routing.ts` · `resolve-caller-team-lead.ts` · `team-session-registry.ts` · team-core adapter shims: `team-registry/` (`loader.ts`/`paths.ts`/`validator.ts`), `team-state-store/` (durable state + atomic locks), `team-runtime/` (lifecycle; own AGENTS.md), `team-mailbox/`, `team-tasklist/`, `team-worktree/`, `team-layout-tmux/` · `tools/` (12 tool implementations + tests) · `test-support/`.

## STORAGE

`~/.omo/teams/{name}/` (user) or `<project>/.omo/teams/{name}/` (project wins on collision): `config.json` (TeamSpec), `state.json` (members, sessionIDs, lifecycle), `mailbox/` (one .jsonl per recipient), `tasklist.jsonl`, `worktrees/{member-name}/`.

## LIFECYCLE

`team_create` → load TeamSpec → validate eligibility → spawn members → init mailbox/tasklist/worktrees → optional tmux. Lead delegates via `team_send_message` + `team_task_create`; members claim (`team_task_update status="claimed"`) → execute → report. `team_shutdown_request` → `team_approve_shutdown`/`team_reject_shutdown`; `team_delete` → full cleanup.

## KEY INVARIANTS

1. **Spawn-race-safe resolution:** every team spawn calls `registerTeamSession(sessionId, entry)` synchronously when sessionID is known; every hook resolving sessionID calls `lookupTeamSession` BEFORE `loadRuntimeState` to avoid the spawn-race window.
2. **Deferred ack:** messages are fire-and-forget; recipient acks via separate call.
3. **Locked tasks:** task claiming uses atomic file locks; concurrent claims resolve safely.
4. **Atomic writes:** state changes write to temp file then rename.
5. **Eligible agents only:** rejection at parse, never at runtime.
6. **No nested teams:** members CANNOT call `team_create`.

## INTEGRATION + WHERE TO LOOK

Entry (`src/index.ts`): `checkTeamModeDependencies()` + `ensureBaseDirs()` when enabled. Tools: `src/plugin/tool-registry.ts`. Hooks: `create-transform-hooks.ts` (`teamModeStatusInjector`, `teamMailboxInjector`), `create-tool-guard-hooks.ts` (`teamToolGating`). Events: `src/plugin/event.ts` (4 handlers from `src/hooks/team-session-events/`). Doctor: `src/cli/doctor/checks/team-mode.ts`. Skill: builtin `team-mode` (skills-loader-core, gated on `team_mode.enabled`). Edits: eligibility → `types.ts`; storage format → `types.ts` Zod schemas; worktree → `team-worktree/`; tmux → `team-layout-tmux/layout.ts`; tasks → `team-tasklist/`; mailbox → `team-mailbox/`; orphan recovery → `team-state-store/resume.ts`.

## ANTI-PATTERNS

- Never bypass `team-session-registry` — direct `loadRuntimeState` lookups will hit the spawn-race window.
- Never write team state files without the atomic lock from `team-state-store/locks.ts`.
- Never substitute `task` (delegate-task) for `team_*` tools when the user explicitly asks for team-mode work — they are not equivalent.
- Never allow members to call `team_create` (nested teams are forbidden by `team-tool-gating` hook).
