# src/plugin-handlers/ -- 6-Phase Config Loading Pipeline

**Generated:** 2026-07-17 / 7d664b96b

## CRITICAL: AGENT ORDERING

The default agent order is **sisyphus → hephaestus → prometheus → atlas**. User config may override it with `agent_order`; omitted core agents fall back to this default order.

Enforced by: (1) `DEFAULT_AGENT_ORDER` (`src/shared/agent-ordering.ts`) as the fallback when `agent_order` is absent or incomplete; (2) `reorderAgentsByPriority()` (`agent-priority-order.ts`) for object-key insertion order in the agent map; (3) `installAgentSortShim()` (`src/shared/agent-sort-shim.ts`), installed once at plugin entry before any agent registration, narrowing `Array.prototype.{toSorted,sort}` so `Agent.list()` and every other sort site return the active configured/default order.

### Why a Sort Shim

OpenCode 1.4.x sorts agents by `agent.name` with native string comparison (NOT `localeCompare`) and ignores the `order` field (sst/opencode#19127), so insertion order alone does not survive `Agent.list()`, and invisible-character sort bias all failed the terminal width tests: ZWSP (U+200B) renders 1-cell wide in Ghostty/WezTerm/Alacritty/some Windows Terminal builds despite `Bun.stringWidth` returning 0 (#3259); U+2060 WORD JOINER, U+00AD SOFT HYPHEN and ANSI escapes are the same class. The shim intercepts only the narrow case: `isAgentArray` requires `arr.length >= 2`, all-non-null objects with string `.name`, and >=2 ranked names (mixed-type arrays keep native semantics); the comparator never throws (falls back to the user's `compareFn`); install is idempotent.

### History

15+ commits, 8+ PRs, multiple reverts: #3260 removed ZWSP injection (reverted by `0d5b08744` — OpenCode 1.4.x ignores `order`, removal alone causes alphabetical fallback); #3329 introduced `CANONICAL_CORE_AGENT_ORDER` and locked the policy (insufficient alone); #3267 proposed the sort shim, revived with cubic-P1 mitigations.

### Forbidden Patterns

DO NOT introduce:
- ZWSP, U+2060, U+00AD, ANSI escape, or any other invisible / control character in agent names, display names, or object keys.
- ASCII spaces or other visible sort prefixes on agent names.
- Alternative ordering constants outside `DEFAULT_AGENT_ORDER` / `CANONICAL_CORE_AGENT_ORDER`, or ordering code that bypasses `validateAgentOrder`.
- Object.entries() iteration-order dependencies.
- Agent name string comparisons that skip `getAgentConfigKey` / `stripInvisibleAgentCharacters` (legacy ZWSP-baked data must keep resolving).

The sort shim in `src/shared/agent-sort-shim.ts` is the ONLY supported runtime ordering mechanism. Remove it once OpenCode honors the agent `order` field (sst/opencode#19127).

## OVERVIEW

14 non-test files implementing the `ConfigHandler` -- the `config` hook handler. Executes 6 sequential phases to register agents, tools, MCPs, and commands with OpenCode.

## 6-PHASE PIPELINE

| Phase | Handler | Purpose |
|-------|---------|---------|
| 1 | `applyProviderConfig` | Cache model context limits, detect anthropic-beta headers |
| 2 | `loadPluginComponents` | Discover Claude Code plugins (10s timeout, error isolation) |
| 3 | `applyAgentConfig` | Load agents from 5 sources, skill discovery, plan demotion |
| 4 | `applyToolConfig` | Agent-specific tool permissions |
| 5 | `applyMcpConfig` | Merge builtin + CC + plugin MCPs |
| 6 | `applyCommandConfig` | Merge commands/skills from 9 parallel sources |

## FILES

| File | Purpose |
|------|---------|
| `config-handler.ts` | Main orchestrator, 6-phase sequential |
| `plugin-components-loader.ts` | CC plugin discovery (10s timeout) |
| `agent-config-handler.ts` | Agent loading + skill discovery from 5 sources |
| `mcp-config-handler.ts` | Builtin + CC + plugin MCP merge |
| `command-config-handler.ts` | 9 parallel sources for commands/skills |
| `tool-config-handler.ts` | Agent-specific tool grants/denials |
| `provider-config-handler.ts` | Provider config + model cache |
| `prometheus-agent-config-builder.ts` | Prometheus config + model/fallback_models resolution |
| `plan-model-inheritance.ts` | Plan demotion (inherits model settings incl. fallback_models) |
| `agent-priority-order.ts` / `agent-key-remapper.ts` / `category-config-resolver.ts` | ordering; key→display-name; user-vs-default category |
| `index.ts` | Barrel exports |

## TOOL PERMISSIONS

| Agent | Granted | Denied |
|-------|---------|--------|
| Librarian | grep_app_* | - |
| Atlas, Sisyphus, Prometheus | task, task_*, teammate | - |
| Hephaestus | task | - |
| Default (all others) | - | grep_app_*, task_*, teammate, LSP |

## MULTI-LEVEL CONFIG MERGE

User (`~/.omo/omo.jsonc`) deep-merged with project (`.omo/omo.jsonc`), then Zod defaults. `agents`/`categories`/`claude_code`: deep merged; `disabled_*` arrays: Set union.
