# src/plugin/ -- 12 OpenCode Hook Handlers + Hook Composition

**Generated:** 2026-07-17 / 7d664b96b | **Updated:** 2026-11-18 / c2691818a9

## OVERVIEW

Core glue layer. Files assemble the 12 OpenCode hook handlers wired into `PluginInterface` here (an additional 2, `experimental.session.compacting` + `experimental.compaction.autocontinue`, are wired in `src/testing/create-plugin-module.ts`). Each handler file maps to one OpenCode hook type. `changes.md` keeps the dated behavior changelog.

## HANDLER FILES

| File | OpenCode Hook | Purpose |
|------|---------------|---------|
| `tool-registry.ts` | `tool` | 12-38 tools assembled with config gates (team-mode +12, monitor +4, task system +4, hashline +1, interactive_bash +1, look_at +1, goal +3); split across `tool-registry-{core-tools,factories,team-tools,gated-tools,trimming}.ts` |
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
| `session-compacting.ts` | `experimental.session.compacting` | Context + todo preservation across compaction (registered via `create-plugin-module.ts`) |
| `skill-context.ts` / `build-team-idle-wake-hint-client.ts` | (helpers) | Skill/browser/category context for tool creation; team idle-wake-hint client for event handlers |

## HOOK COMPOSITION (hooks/ subdir)

| File | Tier | Count |
|------|------|-------|
| `create-session-hooks.ts` | Session | 24 |
| `create-tool-guard-hooks.ts` | Tool Guard | 19 (incl. `team-tool-gating`, null unless team_mode; `momusReviewPersistence`) |
| `create-transform-hooks.ts` | Transform | 8 slots (2 team-gated, 1 monitor-gated; incl. `btwSideContextInjector` and `contextInjectorMessagesTransform`) |
| `create-skill-hooks.ts` | Skill | 2 (`categorySkillReminder`, `autoSlashCommand`) |
| `create-core-hooks.ts` | Aggregator | Session + Guard + Transform = 51 slots |

`createContinuationHooks()` (7: stop-continuation guard, 2 compaction hooks, todo-continuation enforcer, unstable-agent babysitter, background-notification, atlas) lives in `hooks/create-continuation-hooks.ts`; `hooks/model-fallback-title-updater.ts` re-titles a session when its fallback model changes (200-entry state cap). `src/create-hooks.ts` composes core + continuation + skill hooks and owns `disposeCreatedHooks()`.

## SUPPORT FILES

| File | Purpose |
|------|---------|
| `available-categories.ts` / `session-agent-resolver.ts` | `AvailableCategory[]` for agent prompt injection; which agent owns a session |
| `session-status-normalizer.ts` / `recent-synthetic-idles.ts` / `unstable-agent-babysitter.ts` | Session status normalization across OpenCode versions; synthetic-idle dedup; unstable-agent tracking |
| `stop-continuation.ts` | `stopContinuation()` clears every continuation mechanism (keyword detector, stop guard, todo enforcer, goal) + boulder state |
| `types.ts` | `PluginContext`, `PluginInterface`, `ToolsRecord`, `TmuxConfig` |
| `event-hook-dispatcher.ts` | Event dispatch + safe hook runner for `event.ts` |
| `event-model-fallback.ts` / `event-model-fallback-state.ts` | Model-fallback event handling; `FallbackContinuationContext`, `applyUserConfiguredFallbackChain`, `createModelFallbackContinuationController` |
| `event-session-lifecycle.ts` / `event-team-handlers.ts` / `event-error-utils.ts` / `event-types.ts` | Session lifecycle helpers, team-session handlers, error extraction, event types |
| `ultrawork-model-override.ts` / `ultrawork-db-model-override.ts` / `ultrawork-variant-availability.ts` | Ultrawork model override logic, DB-level override, variant availability |
| `runtime-skill-resolver.ts` | `createRuntimeSkillsResolver` / `readRuntimeHostSkills` for host skill discovery |
| `skill-security-notice.ts` / `session-tool-permissions.ts` / `normalize-tool-arg-schemas.ts` | Skill security notice injection; per-session tool permissions; tool arg schema coercion |
| `native-skills.ts` | Native-skill loader (`createNativeSkills` / `getPluginInputNativeSkills`) feeding lazy `getLoadedSkills` discovery in skill/delegate tools |
| `chat-message/` | Helpers for the `chat.message` handler: `loop-commands.ts` (goal/auto-slash/ulw command parsing), `session-model.ts`, `model-cache-warning.ts`, `prompt-text.ts`, `ulw-execute-message.ts`, `types.ts` |

## TOOL REGISTRATION GATES

```typescript
// src/plugin/tool-registry.ts - createToolRegistry()
const factories: ToolRegistryFactories = { ...defaultToolRegistryFactories, ...toolFactories } // DI for tests
const allTools = {
  ...createCoreTools({ ctx, pluginConfig, managers, skillContext, availableCategories, factories }),
  //    grep/glob/session-manager tools, 2 background_*, call_omo_agent, task, skill, skill_mcp;
  //    look_at only when isMultimodalLookerEnabled; +3 goal tools when pluginConfig.goal?.enabled
  ...(interactiveBashEnabled ? { interactive_bash: factories.interactive_bash } : {}), // +1
  ...createTeamModeToolsRecord({ pluginConfig, ctx, managers, factories }),  // +12 when team_mode.enabled
  ...createMonitorToolsRecord({ pluginConfig, ctx, managers, factories }),   // +4 when monitor.enabled + monitorManager
  ...createTaskToolsRecord({ taskSystemEnabled, pluginConfig, ctx, factories }), // +4 when task system enabled
  ...createHashlineToolsRecord({ pluginConfig, ctx, factories }),            // +1 when hashline_edit
}
// then per tool: normalizeToolArgSchemas -> filterDisabledTools(disabled_tools) -> trimToolsToCap(experimental.max_tools)
// lsp_* tools are supplied by the built-in MCP server "lsp"
```

## KEY PATTERNS

- Each handler exports a function receiving `(hookRecord, ctx, pluginConfig, managers)` → returns the OpenCode hook function.
- Handlers iterate over hook records, calling each hook with `(input, output)` in registration order.
- `safeHook()` wrapper isolates hook errors so one broken hook does not crash the chain.
