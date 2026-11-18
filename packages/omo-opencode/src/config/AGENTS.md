# src/config/ - Zod v4 Schema System

**Generated:** 2026-07-17 (7d664b96b)

## OVERVIEW

35 non-test schema files composing `OhMyOpenCodeConfigSchema` (plus `schema/internal/permission.ts` for shared internal helpers). Zod v4 validation with `safeParse()`. All fields optional; omitted fields use defaults from the schema. Auto-emitted via `bun run build:schema` to `dist/oh-my-opencode.schema.json` (root `package.json` export `./schema.json`). Two schema files are shims over core packages; `schema/core-schema-parity.test.ts` is the drift guard keeping the adapter schema in field parity with `@oh-my-opencode/omo-config-core` for the keys core owns (`permission`, `prompt_append`, full category parity).

## SCHEMA TREE

```
config/schema/
├── oh-my-opencode-config.ts    # ROOT: composes all sub-schemas
├── agent-names.ts              # BuiltinAgentNameSchema enum (11 names: sisyphus, hephaestus, prometheus, oracle, librarian, explore, multimodal-looker, metis, momus, atlas, sisyphus-junior)
├── agent-overrides.ts          # AgentOverrideConfigSchema (21 fields per agent)
├── agent-definitions.ts        # custom agent definition schema
├── categories.ts               # 8 built-in + custom categories
├── hooks.ts                    # HookNameSchema (58 enum values, incl. momus-review-persistence + native-edition-nudge; `team-tool-gating` is the only team-* one in schema; others are wired by direct config gates)
├── skills.ts                   # SkillsConfigSchema (sources, paths, recursive)
├── commands.ts                 # BuiltinCommandNameSchema (goal, refactor, ulw-execute, stop-continuation, remove-ai-slops, hyperplan)
├── experimental.ts             # Feature flags incl plugin_load_timeout_ms (min 1000), task_system, max_tools
├── sisyphus.ts + sisyphus-agent.ts  # SisyphusConfigSchema (task system) + SisyphusAgentConfigSchema
├── goal.ts + ralph-loop.ts     # GoalConfigSchema (enabled, auto_start, default_max_iterations) replaces ralph_loop; RalphLoopConfigSchema DEPRECATED (parsed loose at root, migrated to goal in validate.ts)
├── tmux.ts                     # TmuxConfigSchema + TmuxLayoutSchema
├── websearch.ts                # provider: "exa" | "tavily"
├── claude-code.ts              # CC compatibility settings (plugins, plugins_override)
├── comment-checker.ts          # AI comment detection config
├── notification.ts             # OS notification settings
├── git-master.ts               # commit_footer: boolean | string
├── git-env-prefix.ts           # SHIM: re-exports skills-loader-core/config/git-env-prefix
├── browser-automation.ts       # playwright/dev-browser/playwright-cli enum; rejects retired providers with migration guidance; playwright_mcp_args (user-config only)
├── background-task.ts          # Concurrency limits per model/provider, syncPollTimeoutMs
├── fallback-models.ts + runtime-fallback.ts  # FallbackModelsConfigSchema + RuntimeFallbackConfigSchema (reactive provider fallback)
├── babysitting.ts              # Unstable agent monitoring
├── dynamic-context-pruning.ts  # Context pruning settings
├── ulw-execute.ts               # UlwExecuteConfigSchema (auto_commit)
├── openclaw.ts                 # OpenClaw integration settings
├── model-capabilities.ts       # Model capabilities config
├── keyword-detector.ts         # disabled_keywords (ultrawork|search|analyze|team)
├── default-mode.ts             # DefaultModeConfigSchema (auto-inject ultrawork / auto-create goal on session start)
├── i18n.ts                     # I18nConfigSchema (locale override; falls back to LANG env var)
├── monitor.ts                  # MonitorConfigSchema (live_mode_enabled, allowed_commands, batch/ring limits, flush_interval_ms)
├── tui.ts                      # TuiConfigSchema + TuiSidebarConfigSchema (sidebar.enabled)
└── team-mode.ts                # SHIM: re-exports team-core/config (TeamModeConfigSchema lives there)
```

## ROOT SCHEMA FIELDS

`$schema`, `new_task_system_enabled`, `default_run_agent`, `agent_order`, `agent_definitions`, `disabled_mcps`, `disabled_agents`, `disabled_skills`, `disabled_hooks`, `disabled_commands`, `disabled_tools`, `disabled_providers`, `mcp_env_allowlist`, `hashline_edit`, `telemetry`, `model_fallback`, `agents`, `categories`, `claude_code`, `sisyphus_agent`, `comment_checker`, `experimental`, `auto_update`, `skills`, **`goal`** (new; replaces `ralph_loop`), `ralph_loop` (deprecated; migrated to `goal` in `validate.ts`), `runtime_fallback`, `background_task`, `notification`, `model_capabilities`, `openclaw`, `i18n`, `monitor`, **`team_mode`**, `keyword_detector`, `babysitting`, `git_master` (defaults: `commit_footer: false`, `include_co_authored_by: true`), `browser_automation_engine`, `websearch`, `tmux`, `tui` (default `sidebar.enabled: true`), `sisyphus`, `ulw_execute` (deprecated `start_work` key migrates to it), `default_mode`, `_migrations`.

## VALIDATE PIPELINE (`validate.ts`, 261 LOC)

`validatePluginConfig` composes: `loadOmoOpenCodeConfigChain` ([`../plugin-config/omo-config-chain.ts`](../plugin-config/)) -> `mergeConfigs` ([`../plugin-config/config-merger.ts`](../plugin-config/)) -> unknown-key diagnostics (`findUnknownKeyPaths`) -> `prunePluginView` (`prune-plugin-view.ts` over omo-config-core `pruneInvalidConfigPaths`: drops only the failing values of a view with one warning per dropped dotted path; an unprunable view returns its issues) -> `applyDisabledProviders` ([`../shared/disabled-providers.ts`](../shared/)) -> migrations (`migrateRalphLoopConfig`, `migrateLegacyUlwExecuteKey`: `start_work` -> `ulw_execute`). Pinned by `validate-pipeline.test.ts` and `validate.ulw-execute-shim.test.ts`.

## RALPH_LOOP -> GOAL MIGRATION

`goal` replaces the deprecated `ralph_loop` subsystem. `validate.ts` `migrateRalphLoopConfig()` maps legacy `ralph_loop.{enabled, default_max_iterations}` to `goal.{enabled, auto_start, default_max_iterations}` (`auto_start` defaults `false`, `default_max_iterations` defaults `100`); explicit `goal` config wins over migrated values. The root `ralph_loop` key is parsed loose (`z.record`) only for migration and logs a deprecation warning; `RalphLoopConfigSchema` still ships but is no longer composed into the root. `default_mode.ralph_loop` became `default_mode.goal` (auto-create a goal from the first main-session message). The `ralph-loop` hook and the `ralph-loop` / `ulw-loop` / `cancel-ralph` commands were removed; the `goal` hook and `goal` command replace them.

## TEAM_MODE SCHEMA

The schema ships from [`@oh-my-opencode/team-core/config`](../../../team-core/) — `schema/team-mode.ts` is a re-export shim (11 fields incl. `enabled`, `max_parallel_members` 1..8, `max_members` 1..8, `tmux_visualization`).

When `enabled: true`: 12 `team_*` tools register (`tool-registry.ts` `teamModeToolsRecord`); 3 conditional hooks (`team-mode-status-injector` + `team-mailbox-injector` Transform tier, `team-tool-gating` Tool Guard tier); 4 team-session-event handlers in `src/plugin/event.ts` (`team-idle-wake-hint`, `team-lead-orphan-handler`, `team-member-error-handler`, `team-member-status-handler`); `team-mode` built-in skill loads; doctor check `cli/doctor/checks/team-mode.ts` runs.

## AGENT OVERRIDE FIELDS (per-agent)

`model`, `variant`, `category`, `skills`, `temperature`, `top_p`, `prompt`, `prompt_append`, `tools`, `disable`, `description`, `mode`, `color`, `permission`, `maxTokens`, `thinking`, `reasoningEffort`, `textVerbosity`, `providerOptions`, `fallback_models`, `ultrawork`.

## HOW TO ADD A CONFIG FIELD

1. Create `src/config/schema/{name}.ts` with Zod schema
2. Add field to `oh-my-opencode-config.ts` root schema
3. Reference via `z.infer<typeof YourSchema>` for the TypeScript type
4. Access in handlers via `pluginConfig.{field_name}` (snake_case JSON, snake_case TS field)
5. Run `bun run build:schema` to regenerate `dist/oh-my-opencode.schema.json`
6. If the field is one core owns (`permission`, `prompt_append`, category keys), extend BOTH schemas — `schema/core-schema-parity.test.ts` fails on drift.
