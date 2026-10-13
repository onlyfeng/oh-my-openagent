# src/features/ — 24 Feature Modules

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Standalone feature modules wired into `plugin/` layer. Each is self-contained with own types, implementation, and co-located tests. Several directories now provide OpenCode adapter shims over extracted Core packages; keep the shim paths that production code imports stable unless the plugin wiring is moved at the same time. A shim whose only importers are tests is deleted, not kept alive by an identity test; tests import the Core package directly.

## MODULE MAP

| Module | Complexity | Has sub-AGENTS.md | Purpose |
|--------|------------|-------------------|---------|
| **team-mode** | HIGH | yes | Parallel multi-agent coordination — 12 `team_*` tools; domain primitives live in `packages/team-core/` with OpenCode runtime/session wiring here |
| **background-agent** | HIGH | yes | Task lifecycle, concurrency (5/key), 3s polling, spawner pattern, circuit breaker; `manager.ts` (3.4k LOC) is the hotspot, parent-wake logic split across `parent-wake-*.ts` |
| **btw-side** | HIGH | yes | Codex-aligned ephemeral side conversations: TUI commands and controls, bounded parent-context injection, session metadata, lifecycle cleanup, and isolation guards |
| **tmux-subagent** | HIGH | yes | Tmux pane management, grid planning, session orchestration via `runTmuxCommand`; reusable tmux primitives live in `packages/tmux-core/` |
| **opencode-skill-loader** | MEDIUM | yes | Re-export shims over `packages/skills-loader-core/`; only `opencode-shell-skill-content.ts` is local |
| **skill-mcp-manager** | MEDIUM | yes | Re-export shims over `packages/mcp-client-core/` tier-3 MCP client lifecycle (every `.ts` is a shim) |
| **builtin-commands** | LOW | yes | 7 command templates: goal, refactor, ulw-execute, stop-continuation, handoff, remove-ai-slops, hyperplan |
| **mcp-oauth** | LOW | yes | Re-export shims over `mcp-client-core/mcp-oauth` (PKCE + DCR + step-up) |
| **claude-code-agent-loader** | LOW | yes | OpenCode adapter for agents from `.opencode/agents/` and Claude Code plugins; shared loader lives in `packages/claude-code-compat-core/` |
| **claude-code-mcp-loader** | MEDIUM | yes | OpenCode adapter for tier-2 MCP loader; `.mcp.json` parse + `${VAR}` env expansion live in `packages/claude-code-compat-core/` |
| **tool-metadata-store** | LOW–MED | no | Tool execution metadata cache; publish/recover lifecycle + task metadata contract |
| **boulder-state** | LOW | yes | Re-export shims over `packages/boulder-state/` (active work plan tracking) |
| **context-injector** | LOW | no | AGENTS.md/README.md injection into session context |
| **hook-message-injector** | LOW | no | System message injection helper used by hooks |
| **run-continuation-state** | LOW | no | Persistent state for `oh-my-opencode run` continuation across invocations |
| **opencode-runtime-skills** | LOW–MED | no | Runtime security-skill source — `selectRuntimeSecuritySkills()` + `createRuntimeSkillSourceServer()` serve security skills to sessions at runtime |
| **claude-code-command-loader** | LOW | no | Load `/commands` from `.opencode/commands/` and Claude Code plugins |
| **claude-tasks** | LOW | yes | Strict Zod task schema + atomic file storage/locking consumed by `tools/task/` |
| **task-toast-manager** | MEDIUM | no | Task progress notifications |
| **claude-code-session-state** | LOW | no | Subagent session state tracking |
| **monitor** | MEDIUM | yes | `monitor_*` tools backend: managed watcher processes, line filtering, ring buffer, batched output injection (gated on `monitor.enabled`) |
| **tui-sidebar** | MEDIUM | yes | TUI sidebar snapshot builder + mirror manager (roster/state derivers rendered into the OpenCode TUI; gated on `tui.sidebar.enabled`) |
| **native-edition-nudge** | LOW | no | TUI dialog nudging toward OmO Native (install / guide / later / never); registered from `src/tui.ts`, state store in `hooks/native-edition-nudge/` |
| **opengateway-provider** | LOW | no | Injects the `opengateway` provider into opencode's live config when a credential exists (`OPENGATEWAY_API_KEY` env or `opengateway` auth.json entry); fills only missing keys, ships bundled `opengateway-models.json` catalog |

## KEY MODULES

### background-agent

Core orchestration engine. `BackgroundManager` manages task lifecycle:
- States: `pending → running → completed | error | cancelled | interrupt`
- Concurrency: per-key (`${providerID}/${modelID}`) limits via `ConcurrencyManager` (FIFO queue)
- Polling: 3s interval, completion detected via idle event AND stability detection (10s unchanged)
- Circuit breaker: automatic failure detection and recovery in `manager-circuit-breaker.test.ts`
- `spawner/`: focused files composing via `SpawnerContext` interface
- Parent-wake coordinator `parent-wake-notifier.ts` (193 LOC, dependency-injected client + enqueue callback) over sibling `parent-wake-*.ts` helpers

### team-mode (~9.7k LOC incl. tests)

Parallel multi-agent coordination, OFF by default. Harness-neutral domain primitives are extracted to `packages/team-core/`; this directory owns OpenCode-specific spawning, tool registration, and hook integration. Subdirs:
- `team-registry/` — load/validate `~/.omo/teams/{name}/config.json`
- `team-state-store/` — durable runtime state with atomic locks
- `team-runtime/` — `team_create`, status, shutdown lifecycle
- `team-mailbox/` — async messaging (send/poll/ack)
- `team-tasklist/` — shared tasks with atomic claiming
- `team-worktree/` — git worktree per member
- `team-layout-tmux/` — optional tmux pane visualization
- `tools/` — 12 `team_*` tool implementations

Eligible members: sisyphus, atlas, sisyphus-junior; hephaestus is conditional. See [`team-mode/AGENTS.md`](team-mode/AGENTS.md).

### opencode-skill-loader (~2k LOC, shims)

Numeric `SCOPE_PRIORITY` (in skills-loader-core): opencode-project 6 > project 5 > opencode 4 > user 3 > config 2 > builtin = shared 1.
- YAML frontmatter parsing, merger dedup, and template resolution all live in Core
- Adapter keeps only OpenCode shell selection (`opencode-shell-skill-content.ts`)
- Provider gating for model-specific skills

### tmux-subagent (~9.8k LOC incl. tests; `manager.ts` 1.4k)

State-first tmux integration. Shared tmux primitives are extracted to `packages/tmux-core/`, while this feature owns OpenCode session tracking and pane decisions. Centralized tmux command execution goes through the shared runner. Direct `Bun.spawn(["tmux", ...])` is FORBIDDEN — would drift from retry/timeout discipline.

### builtin-skills (extracted)

The built-in skill catalog now lives entirely in `packages/skills-loader-core/src/features/builtin-skills/` — see [its AGENTS.md](../../../skills-loader-core/src/features/builtin-skills/AGENTS.md) for the skill table and browser-variant selection rules. Import through `@oh-my-opencode/skills-loader-core/builtin-skills`.

`browser_automation_engine` selects one of `playwright`, `dev-browser`, or `playwright-cli`; the provider enum lives in `config/schema/browser-automation.ts`. Removed providers fail schema validation and the `browser-provider` doctor check reports migration guidance. Browser QA guidance in the shipped skills comes from omowright (the `browser` skill's owned and attached engines); the OpenCode provider enum is independent of it.
