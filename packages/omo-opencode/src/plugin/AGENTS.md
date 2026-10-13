# src/plugin/ -- 12 OpenCode Hook Handlers + Hook Composition

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Core glue layer. Files assemble the 12 OpenCode hook handlers wired into `PluginInterface` here (an additional 2, `experimental.session.compacting` + `experimental.compaction.autocontinue`, are wired in `src/testing/create-plugin-module.ts`). Each handler file maps to one OpenCode hook type.

## HANDLER FILES

| File | OpenCode Hook | Purpose |
|------|---------------|---------|
| (none here) | `config` | Built by `plugin-handlers/config-handler.ts` `createConfigHandler()`; wired as `managers.configHandler` in `plugin-interface.ts` |
| `tool-registry.ts` | `tool` | 12-38 tools assembled with config gates (team-mode +12, monitor +4, task system +4, hashline +1, interactive_bash +1, look_at +1, goal +3); split across `tool-registry-{core-tools,team-tools,gated-tools}.ts` |
| `tool-definition.ts` | `tool.definition` | Per-tool definition transform (applies todo-description-override) |
| `chat-message.ts` | `chat.message` | First-message variant resolution, session setup, keyword detection, goal command dispatch + default goal auto-start |
| `chat-params.ts` | `chat.params` | Anthropic effort, think mode, runtime fallback model override |
| `chat-headers.ts` | `chat.headers` | Copilot `x-initiator` header injection |
| `command-execute-before.ts` | `command.execute.before` | Pre-command guards (stop-continuation, /goal dispatch, ulw-execute, auto-slash-command) |
| `event.ts` | `event` | Session lifecycle (created/deleted/idle/error/status), openclaw dispatch, runtime fallback, 4 team-session-event handlers (when team_mode.enabled) |
| `tool-execute-before.ts` | `tool.execute.before` | Pre-tool guards (mcp_ strip, bash sleep block, task subagent resolution, skill /goal + /stop-continuation dispatch) |
| `tool-execute-after.ts` | `tool.execute.after` | Post-tool hooks (truncation, comment-checker, hashline read tagging, json-error-recovery) |
| `messages-transform.ts` | `experimental.chat.messages.transform` | Context injection, thinking-block validation, tool-pair validation, keyword detection, category-skill reminder |
| `system-transform.ts` | `experimental.chat.system.transform` | System-message-level transforms |
| `session-compacting.ts` | `experimental.session.compacting` + `experimental.compaction.autocontinue` | `createSessionCompactingHandler` + `createCompactionAutocontinueHandler` (both registered via `create-plugin-module.ts`) |
| `skill-context.ts` | (helper) | Skill/browser/category context shared with tool creation |
| `build-team-idle-wake-hint-client.ts` | (helper) | Build the team idle-wake-hint client wired into event handlers |

## HOOK COMPOSITION (hooks/ subdir)

| File | Tier | Count |
|------|------|-------|
| `create-session-hooks.ts` | Session | 24 |
| `create-tool-guard-hooks.ts` | Tool Guard | 18 (incl. `team-tool-gating`, inert unless team_mode) |
| `create-transform-hooks.ts` | Transform | 8 slots (2 team-gated, 1 monitor-gated; incl. `btwSideContextInjector` and `contextInjectorMessagesTransform`) |
| `create-skill-hooks.ts` | Skill | 2 |
| `create-continuation-hooks.ts` | Continuation | 7 |
| `create-core-hooks.ts` | Aggregator | Session + Guard + Transform = 50 slots |
| `model-fallback-title-updater.ts` | helper | Session title update when `experimental.model_fallback_title` |

`src/create-hooks.ts` calls `createCoreHooks()` + `createContinuationHooks()` + `createSkillHooks()` (all three live in `hooks/` here). `team-tool-gating` is created whenever the hook is enabled and no-ops unless `team_mode.enabled`.

## SUPPORT FILES

| File | Purpose |
|------|---------|
| `available-categories.ts` | Build `AvailableCategory[]` for agent prompt injection |
| `session-agent-resolver.ts` | Resolve which agent owns a session |
| `session-status-normalizer.ts` | Normalize session status across OpenCode versions |
| `recent-synthetic-idles.ts` | Dedup rapid synthetic idle events |
| `unstable-agent-babysitter.ts` | Track unstable agent behavior across sessions |
| `types.ts` | `PluginContext`, `PluginInterface`, `ToolsRecord`, `TmuxConfig` |
| `ultrawork-model-override.ts` | Ultrawork mode model override logic |
| `ultrawork-db-model-override.ts` | DB-level model override for ultrawork |
| `chat-message/` | `chat.message` helpers: goal/loop commands, ulw-execute message, session model record, prompt text, model-cache warning |
| `event-hook-dispatcher.ts` / `event-session-lifecycle.ts` / `event-team-handlers.ts` | `event.ts` split: per-hook dispatch, session lifecycle, team handlers |
| `event-model-fallback.ts` / `event-model-fallback-state.ts` | Event-side fallback state (uses runtime-fallback `getRawFallbackModels`) |
| `stop-continuation.ts` | Shared stop-continuation entrypoints |
| `runtime-skill-resolver.ts` / `ultrawork-variant-availability.ts` | Runtime skill lookup; ultrawork variant availability |
| `normalize-tool-arg-schemas.ts` | Coerce tool arg schemas into a normalized shape |
| `native-skills.ts` | Native-skill loader (`createNativeSkills` / `getPluginInputNativeSkills`) feeding lazy `getLoadedSkills` discovery in skill/delegate tools |

## TOOL REGISTRATION GATES

`tool-registry.ts` spreads core tools (`tool-registry-core-tools.ts`, incl. `createGoalTools` when `goal.enabled`) plus gated records from `tool-registry-team-tools.ts` (`createTeamModeToolsRecord`, +12) and `tool-registry-gated-tools.ts` (`createMonitorToolsRecord` +4, `createTaskToolsRecord` +4, `createHashlineToolsRecord` +1); factories in `tool-registry-factories.ts`. `look_at` and `interactive_bash` are conditional. Then `filterDisabledTools(allTools, disabled_tools)` and the `experimental.max_tools` cap (`tool-registry-trimming.ts`). `lsp_*` tools come from the built-in MCP `lsp`, not the registry.

## KEY PATTERNS

- Each handler exports a function receiving `(hookRecord, ctx, pluginConfig, managers)` → returns the OpenCode hook function.
- Handlers iterate over hook records, calling each hook with `(input, output)` in registration order.
- `safeCreateHook()` (`shared/safe-create-hook.ts`, exposed as `safeHook` inside each composer) isolates hook factory errors so one broken hook does not crash composition.
- `filterDisabledTools(allTools, disabled_tools)` prunes tools listed in `disabled_tools` config.
- `experimental.max_tools` cap trims tool count when set (selects the highest-priority tools).
