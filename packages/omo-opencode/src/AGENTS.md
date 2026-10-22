# src/ - Plugin Source

**Generated:** 2026-10-22 / f9de7acf36

## STOP. THIS IS THE OPENCODE PLUGIN. QA IS MANDATORY. EVERY SINGLE TIME YOU CHANGE ANYTHING HERE.

> **EVERYTHING UNDER THIS `src/` IS WIRED DIRECTLY INTO OPENCODE. IF YOU EDIT A HOOK, A TOOL, AN AGENT, A FEATURE, A CONFIG SCHEMA, AN MCP, A CLI COMMAND, A PLUGIN HANDLER, OR ANYTHING ELSE IN HERE, YOU MUST QA IT AGAINST REAL OPENCODE. ALWAYS. EVERY SINGLE TIME. NO EXCEPTIONS.**

**"It typechecks" is NOT QA. "`bun test` is green" is NOT QA.** YOU MUST DRIVE REAL OPENCODE AND RECORD THE EVIDENCE TO DISK. NO EVIDENCE == NO QA == NO COMMIT == NO PUSH.

**ALWAYS RUN THE `opencode-qa` SKILL** (`.agents/skills/opencode-qa/`) to map the EXPECTED IMPACT and the FULL CHANGE SCOPE of your edit:

1. **MAP THE BLAST RADIUS** with the skill router (CLI / server + SSE hook proof / TUI smoke / DB inspection), BEFORE and AFTER your change.
2. **ISOLATE EVERYTHING.** Any QA that SPAWNS opencode MUST run in an isolated XDG sandbox (`XDG_DATA_HOME` / `XDG_CONFIG_HOME` / `XDG_STATE_HOME` / `XDG_CACHE_HOME` pointed at temp dirs). **NEVER pollute the real `~/.local/share/opencode/opencode.db`.** PROVE it: `SELECT count(*) FROM session` unchanged before vs after.
3. **PROVE THE HOOK / EVENT FIRED.** Changed a lifecycle hook? Prove the matching event hit the wire (`scripts/sse-hook-probe.sh --event <name>`). Changed a tool? Drive it via `opencode run --format json` and assert on the structured events.
4. **USE tmux** for TUI smoke (`scripts/tui-smoke.sh`) and interactive driving; assert REAL behavior via `opencode run` or the server API + SSE, not the TUI pane.

**RECORD THE EVIDENCE UNDER `.omo/evidence/<YYYYMMDD>-<short-slug>/`** (one organized subfolder per change; the directory is gitignored, so the files stay local and the PR body carries the summary): WHY THERE IS NO REGRESSION (before/after + isolation proof + exact commands and output) and PROOF THAT EVERY INTENDED CHANGE LANDED (new behavior observed on real opencode). See the root [`AGENTS.md`](../../../AGENTS.md) "STOP. QA IS MANDATORY" section for the full mandate, which also covers the Codex side.

**ALWAYS. EVERY TIME. NO EXCEPTIONS.**

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
| `plugin-interface.ts` | 12 OpenCode hook handlers wired into `Hooks` (a further 2, `experimental.session.compacting` + `experimental.compaction.autocontinue`, are wired in `src/testing/create-plugin-module.ts`, for 14 total) |
| `create-managers.ts` | TmuxSessionManager, BackgroundManager, SkillMcpManager, ConfigHandler |
| `create-tools.ts` | SkillContext + AvailableCategories + ToolRegistry composition |
| `create-hooks.ts` | 5-tier composition: `createCoreHooks() + createContinuationHooks() + createSkillHooks()` |
| `create-runtime-tmux-config.ts` | `isTmuxIntegrationEnabled()` + `createRuntimeTmuxConfig()` |

## INITIALIZATION (`createPluginModule()`)

```
serverPlugin(input, options)
  installAgentSortShim()           # canonical agent ordering shim over Array.prototype.{toSorted,sort}
  initConfigContext()              # opencode-vs-openagent layout flag
  logLegacyPluginStartupWarning()  # warn on the legacy oh-my-opencode entry
  migrateLegacyWorkspaceDirectory()# copy .sisyphus/ state forward to .omo/
  detectDuplicateOmoPlugin()       # early exit on a duplicate omo/openagent plugin
  detectExternalSkillPlugin()      # warn on skill-plugin conflicts
  injectServerAuthIntoClient()     # auth headers into the shared SDK client
  loadPluginConfig()               # user + walked project JSONC -> Zod safeParse -> migrate
  recordPluginTelemetry()
  ensureTuiPluginEntry()           # always; BTW stays available when the sidebar is off
  initLiveServerRoute() + setLiveParentWakeRoutingDisabled() + warmLiveServerProbe()
  selectRuntimeSecuritySkills() + createRuntimeSkillSourceServer()
  initI18n()                       # locales/ strings
  setAgentSortOrder()              # configured agent_order
  initializeOpenClaw()             # only when openclaw config exists
  checkTeamModeDependencies()      # only when team_mode.enabled
  startTmuxCheck()                 # only when tmux integration is on
  createManagers() -> createTools() -> createHooks() -> createPluginInterface()
  createPluginDispose()            # adds session.compacting + compaction.autocontinue + dispose
```

## CONFIG LOADING (Phase pipeline)

```
loadPluginConfig(directory, ctx)
  1. User: ~/.omo/omo.jsonc
  2. Walked configs: <pwd up to $HOME>/.omo/omo.jsonc
  3. mergeConfigs(user, walked)
     - agents/categories/claude_code: deepMerge (recursive, prototype-pollution safe)
     - disabled_*: Set union
     - mcp_env_allowlist: user-only (security)
     - playwright_mcp_args: user-only (security)
     - others: override replaces
  4. Zod safeParse → defaults for omitted fields
  5. Legacy configuration is migrated once at startup into the unified OMO chain
```

## HOOK COMPOSITION (5-tier)

Counts verified from each composer's return object. Numbers in brackets show counts when `team_mode.enabled`.

```
createHooks()
  ├─→ createCoreHooks()
  │   ├─ createSessionHooks()     # 23: preemptiveCompaction,
  │   │                             sessionNotification, thinkMode, modelFallback,
  │   │                             anthropicContextWindowLimitRecovery, autoUpdateChecker,
  │   │                             astGrepSgProvision,
  │   │                             agentUsageReminder, nonInteractiveEnv, interactiveBashSession,
  │   │                             goal, editErrorRecovery, delegateTaskRetry, ulwExecute,
  │   │                             prometheusMdOnly, sisyphusJuniorNotepad, noSisyphusGpt,
  │   │                             noHephaestusNonGpt, hephaestusAgentsMdInjector,
  │   │                             questionLabelTruncator, taskResumeInfo,
  │   │                             runtimeFallback, legacyPluginToast
  │   ├─ createToolGuardHooks()   # 17 [+1 with team-mode]: commentChecker, toolOutputTruncator,
  │   │                             directoryAgentsInjector, directoryReadmeInjector,
  │   │                             emptyTaskResponseDetector, rulesInjector, tasksTodowriteDisabler,
  │   │                             writeExistingFileGuard, bashFileReadGuard, hashlineReadEnhancer,
  │   │                             jsonErrorRecovery, readImageResizer, todoDescriptionOverride,
  │   │                             webfetchRedirectGuard, fsyncSkipWarning,
  │   │                             notepadWriteGuard, planFormatValidator [+ teamToolGating]
  │   └─ createTransformHooks()   # 4 [+2 team-mode, +1 monitor-gated]: claudeCodeHooks,
  │                                  keywordDetector, contextInjectorMessagesTransform,
  │                                  toolPairValidator [+ teamModeStatusInjector,
  │                                  teamMailboxInjector, monitorStatusInjector (monitor.enabled)]
  ├─→ createContinuationHooks()   # 7: stopContinuationGuard, compactionContextInjector,
  │                                  compactionTodoPreserver, todoContinuationEnforcer (boulder),
  │                                  unstableAgentBabysitter, backgroundNotificationHook, atlasHook
  └─→ createSkillHooks()          # 2: categorySkillReminder, autoSlashCommand

  Direct event handlers (src/plugin/event.ts, when team_mode.enabled): +4
    team-idle-wake-hint, team-lead-orphan-handler,
    team-member-error-handler, team-member-status-handler
```

Total: 54 base, 61 with team-mode, 62 with monitor enabled. Authoritative per-tier breakdown: [`hooks/AGENTS.md`](hooks/AGENTS.md). Each tier produces an object whose values are `(input, output) => void` handlers; the matching OpenCode handler invokes them in registration order via `safeHook()` wrappers.

## SUBSYSTEM INVENTORY

| Subdir | Purpose | Has AGENTS.md |
|--------|---------|---------------|
| `agents/` | 11 agent factories + dynamic prompt builder | yes (+ atlas, hephaestus, prometheus, sisyphus, sisyphus-junior, builtin-agents) |
| `hooks/` | 54-62 lifecycle hooks across 62 dirs | yes (+ atlas, anthropic-context-window-limit-recovery, auto-update-checker, claude-code-hooks, comment-checker, compaction-context-injector, keyword-detector, ralph-loop, rules-injector, runtime-fallback, todo-continuation-enforcer) |
| `tools/` | 14 native tool dirs (+1 shared utilities dir); LSP + AST-grep moved to built-in MCPs | yes (+ background-task, call-omo-agent, delegate-task, hashline-edit, look-at, skill) |
| `features/` | 24 feature modules (including `btw-side`, `opengateway-provider`; some now shimming `team-core`, `tmux-core`, `skills-loader-core`, `mcp-client-core`, and `claude-code-compat-core`) | yes (+ 16 sub-AGENTS.md including team-mode, background-agent, btw-side, claude-code-*) |
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
| `types/` | Ambient `.d.ts` declarations (markdown modules) | no |
| `help/` | CLI help schema definitions (acp, doctor, sandbox, status) | no |
| `locales/` | i18n strings (en, zh): toasts + model-fallback labels | no |

## NOTES

- `plugin-interface.ts` is the **only** layer that talks to OpenCode's `Plugin` API. Every other file goes through it.
- Reach for `shared/` before adding helpers anywhere else; duplicate utilities WILL be flagged in review.
- Path aliases are forbidden. Use relative imports within a module, barrel imports across modules.
