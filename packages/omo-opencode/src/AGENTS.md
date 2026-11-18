# src/ - Plugin Source

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## QA (BINDING)

Everything under `src/` is wired directly into OpenCode: any change here (hook, tool, agent, feature, config schema, MCP, CLI command, plugin handler) MUST run the `opencode-qa` skill (`.agents/skills/opencode-qa/`) in an isolated XDG sandbox (never pollute the real `~/.local/share/opencode/opencode.db`) and record evidence under `.omo/evidence/<YYYYMMDD>-<short-slug>/`. "It typechecks" and "`bun test` is green" are NOT QA: drive real OpenCode and prove both no regression and that every intended change landed. Full mandate: root [`AGENTS.md`](../../../AGENTS.md) QA GATE.

## OVERVIEW

Entry `index.ts` is a thin wrapper: it builds the `PluginModule` via `createPluginModule()` from `testing/create-plugin-module.ts` and default-exports it (plus `opencode-v2/server-entry` `setup`). The staged initialization lives in that factory, across the directories below. Cross-cutting adapter helpers live in `shared/`; barrel `index.ts` files establish module boundaries. Several former implementation directories now act partly as OpenCode-facing shims over extracted Core packages.

## KEY FILES

| File | Purpose |
|------|---------|
| `index.ts` | Plugin entry; default-exports the `PluginModule` with `{ id, server }` plus the v2 `setup` |
| `testing/create-plugin-module.ts` | The real staged `serverPlugin()`: config, telemetry, i18n, managers, tools, hooks, interface, dispose |
| `opencode-v2/server-entry.ts` | OpenCode v2 server entry (`opencodeV2Server.setup`) |
| `plugin-config.ts` | JSONC parse, multi-level merge (user + walked project), Zod v4 validation, migration |
| `plugin-state.ts` | `createModelCacheState()`: model resolution cache shared across handlers |
| `plugin-interface.ts` | 12 OpenCode hook handlers wired into `Hooks`; 2 more (`experimental.session.compacting` + `experimental.compaction.autocontinue`) wire in `create-plugin-module.ts` (14 total) |
| `create-managers.ts` | TmuxSessionManager, BackgroundManager, SkillMcpManager, ConfigHandler |
| `create-tools.ts` | SkillContext + AvailableCategories + ToolRegistry composition |
| `create-hooks.ts` | 5-tier composition: `createCoreHooks() + createContinuationHooks() + createSkillHooks()` |

## INITIALIZATION (`createPluginModule()`)

```
serverPlugin(input, options)
  installAgentSortShim() + initConfigContext() + logLegacyPluginStartupWarning()
  migrateLegacyWorkspaceDirectory()   # copy .sisyphus/ state forward to .omo/
  detectDuplicateOmoPlugin() + detectExternalSkillPlugin()   # dup-plugin early exit; skill-plugin warn
  injectServerAuthIntoClient()       # auth headers into the shared SDK client
  loadPluginConfig() + recordPluginTelemetry()
  ensureTuiPluginEntry()             # always; BTW stays available when the sidebar is off
  initLiveServerRoute() + setLiveParentWakeRoutingDisabled() + warmLiveServerProbe()
  selectRuntimeSecuritySkills() + createRuntimeSkillSourceServer()
  initI18n() + setAgentSortOrder()   # locales/ strings; configured agent_order
  initializeOpenClaw()               # only when openclaw config exists
  checkTeamModeDependencies() + startTmuxCheck()   # only when team_mode.enabled / tmux on
  createManagers() -> createTools() -> createHooks() -> createPluginInterface()
  createPluginDispose()              # session.compacting + compaction.autocontinue + dispose
```

## CONFIG LOADING

`loadPluginConfig(directory, ctx)`: 1. user `~/.omo/omo.jsonc`; 2. walked `<pwd up to $HOME>/.omo/omo.jsonc`; 3. `mergeConfigs` — `agents`/`categories`/`claude_code` deepMerge (recursive, prototype-pollution safe), `disabled_*` Set union, `mcp_env_allowlist` + `playwright_mcp_args` user-only (security), others override-replace; 4. Zod safeParse fills defaults; 5. legacy config migrated once at startup into the unified OMO chain.

## HOOK COMPOSITION (5-tier)

`createHooks()` = `createCoreHooks()` (session 23 / tool-guard 17 [+1 team-mode] / transform 4 [+2 team-mode, +1 monitor-gated]) + `createContinuationHooks()` (7) + `createSkillHooks()` (2), plus 4 direct team event handlers in `plugin/event.ts` (team-idle-wake-hint, team-lead-orphan-handler, team-member-error-handler, team-member-status-handler). Totals: 54 base, 61 with team-mode, 62 with monitor. Authoritative per-tier roster: [`hooks/AGENTS.md`](hooks/AGENTS.md). Each tier produces an object whose values are `(input, output) => void` handlers; the matching OpenCode handler invokes them in registration order via `safeHook()` wrappers.

## SUBSYSTEM INVENTORY

| Subdir | Purpose | Has AGENTS.md |
|--------|---------|---------------|
| `agents/` | 11 agent factories + dynamic prompt builder | yes (+ atlas, hephaestus, prometheus, sisyphus-junior, builtin-agents) |
| `hooks/` | 54-62 lifecycle hooks across 61 dirs | yes |
| `tools/` | 14 native tool dirs (+1 shared utilities dir); LSP + AST-grep moved to built-in MCPs | yes (+ background-task, call-omo-agent, delegate-task, hashline-edit, look-at, skill) |
| `features/` | 24 feature modules (incl. `btw-side`, `opengateway-provider`; some now shimming `team-core`, `tmux-core`, `skills-loader-core`, `mcp-client-core`, `claude-code-compat-core`) | yes (+ 17 sub-AGENTS.md incl. team-mode, background-agent, btw-side, claude-code-*) |
| `shared/` | Cross-cutting adapter utilities plus shims over extracted Core packages, barrel-exported | yes |
| `cli/` | Commander.js CLI: install, run, doctor, mcp-oauth, boulder | yes (+ config-manager, doctor, run) |
| `plugin/` | 12 OpenCode hook handlers + hook composition | yes |
| `config/` | Zod v4 schema files | yes |
| `plugin-handlers/` | 6-phase config loading pipeline | yes |
| `openclaw/` | Bidirectional Discord/Telegram/HTTP integration | yes |
| `__tests__/` | Plugin-level integration tests + perf fixtures | yes |
| `mcp/` | 4 built-in MCPs (3 remote + local stdio lsp) | yes |
| `testing/` | Test utilities + `create-plugin-module.ts` | yes |
| `config-migration/` | Legacy config discovery + transform plans (consumed by senpi config-startup + codex startup) | yes |
| `types/`, `help/`, `locales/` | Ambient `.d.ts` (markdown modules); CLI help schemas (acp, doctor, sandbox, status); i18n strings (en, zh) toasts + model-fallback labels | no |

## NOTES

- `plugin-interface.ts` is the **only** layer that talks to OpenCode's `Plugin` API. Every other file goes through it.
- Reach for `shared/` before adding helpers anywhere else; duplicate utilities WILL be flagged in review.
- Path aliases are forbidden. Use relative imports within a module, barrel imports across modules.
