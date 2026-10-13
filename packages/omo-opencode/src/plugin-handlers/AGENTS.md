# src/plugin-handlers/ -- Config Hook Pipeline

**Generated:** 2026-10-13 / dc373d26f

## CRITICAL: AGENT ORDERING

The default agent order is **sisyphus → hephaestus → prometheus → atlas**. User config may override it with `agent_order`; omitted core agents fall back to this default order.

This order is enforced via two cooperating mechanisms:
1. `DEFAULT_AGENT_ORDER` in `src/shared/agent-ordering.ts` supplies the fallback order used when `agent_order` is absent or incomplete.
2. `reorderAgentsByPriority()` in `agent-priority-order.ts` controls object key insertion order in the agent map produced by `applyAgentConfig`.
3. `installAgentSortShim()` in `src/shared/agent-sort-shim.ts` narrows `Array.prototype.toSorted` and `Array.prototype.sort` so that whenever the sorted array contains two or more ranked agent objects, OpenCode's `Agent.list()` (and any other sort site) returns the active configured/default order. The shim is installed once at plugin entry, before any agent registration, and its rank map is updated after plugin config loads.

### Why a Sort Shim

OpenCode 1.4.x sorts agents purely by `agent.name` via Remeda `sortBy`, which uses native string `<` / `>` comparison (NOT `localeCompare`). It currently ignores the agent `order` field. Until that lands (sst/opencode#19127), object-key insertion order alone does not survive `Agent.list()`, and biasing the sort key with invisible characters all failed:
- ZWSP (U+200B): `Bun.stringWidth` returns 0 but terminals (Ghostty, WezTerm, Alacritty, certain Windows Terminal builds) render it as 1-cell wide. Visible gap in the status bar; column truncation in the agent picker (#3259).
- U+2060 WORD JOINER, U+00AD SOFT HYPHEN, ANSI escape: same width-mismatch class.
- Removing the prefix and relying on insertion order alone falls back to alphabetical Atlas → Hephaestus → Prometheus → Sisyphus.

The sort shim resolves this by intercepting only the narrow case it cares about, with strict activation guards to prevent collateral damage from a global prototype patch:
- The activation predicate (`isAgentArray`) requires `arr.length >= 2`, every element is a non-null object with a string `.name`, and at least 2 elements have a `.name` ranked by the active order. This rejects mixed-type arrays (numbers, strings, plain objects without `.name`) so unrelated `.sort()` / `.toSorted()` calls execute native semantics.
- The comparator never throws on mixed input -- it defensively extracts `.name` and falls back to the user-supplied `compareFn`.
- `installAgentSortShim()` is idempotent.

### History

Agent ordering has caused 15+ commits, 8+ PRs, and multiple reverts. Notable milestones:
- #3260 (merged): removed ZWSP injection. Reverted by `0d5b08744` because OpenCode 1.4.x ignores `order`, and removal alone causes alphabetical fallback (Atlas → Hephaestus → Prometheus → Sisyphus).
- #3329 (merged): introduced `CANONICAL_CORE_AGENT_ORDER` and locked the policy. Insertion order alone still does not survive OpenCode's `Agent.list()` sort.
- #3267 (closed): proposed a sort shim. Closed at the time on the assumption that #3329 was sufficient. Revived in this commit with cubic P1 mitigations (defensive comparator, strict activation predicate, idempotent install).

### Forbidden Patterns

DO NOT introduce:
- ZWSP, U+2060, U+00AD, ANSI escape, or any other invisible / control character in agent names, display names, or object keys.
- ASCII spaces or other visible sort prefixes on agent names.
- Alternative ordering constants outside `DEFAULT_AGENT_ORDER` / `CANONICAL_CORE_AGENT_ORDER`, or ordering code that bypasses `validateAgentOrder`.
- Object.entries() iteration-order dependencies.
- Agent name string comparisons that skip `getAgentConfigKey` / `stripInvisibleAgentCharacters` (legacy ZWSP-baked data must keep resolving).

The sort shim in `src/shared/agent-sort-shim.ts` is the ONLY supported runtime ordering mechanism. Remove it once OpenCode honors the agent `order` field (sst/opencode#19127).

PRs attempting any of the forbidden patterns will be rejected.

## OVERVIEW

20 non-test files (+19 tests). `createConfigHandler(deps)` in `config-handler.ts` builds the OpenCode `config` hook; it is created in `create-managers.ts` and wired as `managers.configHandler` by `plugin-interface.ts`.

## PIPELINE (config-handler.ts, in order)

| Step | Handler | Purpose |
|------|---------|---------|
| 0 | `setAdditionalAllowedMcpEnvVars`, `applyOpenGatewayProviderConfig` | `mcp_env_allowlist` + OpenGateway provider injection |
| 1 | `applyProviderConfig` | Model context limits, anthropic-beta headers, trusted vision models |
| 2 | `loadPluginComponents` | Claude Code plugin discovery (error isolation; `retryableLoadFailure` skips the agent cache) |
| 3 | `applyHookConfig` | Push plugin hook configs into claude-code-hooks |
| 4 | `applyAgentConfig` | Agent assembly; result snapshot cached by `{agent, default_agent, model, skills}` key and replayed on hit |
| 5 | `applyToolConfig` + `attachHiddenConfigKeyAliases` | Per-agent permissions; hidden config-key aliases for display-named agents |
| 6 | `applyMcpConfig` | Builtin (`createBuiltinMcps`) + `.mcp.json` + plugin MCPs |
| 7 | `applyCommandConfig` | Commands/skills merge |
| 8 | `applyRuntimeSkillSourceConfig` | Only when a runtime skill source URL exists |

## FILES

| File | Purpose |
|------|---------|
| `config-handler.ts` | Orchestrator above (~200 lines) |
| `agent-config-handler.ts` | `applyAgentConfig`: delegates to source loader, assembly, finalizer |
| `agent-source-loader.ts` / `agent-skill-discovery.ts` | Agent sources + skill discovery |
| `agent-config-assembly.ts` | Source filtering, default-agent selection (`getConfiguredDefaultAgent`), largest file (~245) |
| `agent-config-finalizer.ts` / `agent-override-protection.ts` | Final map ordering + protect builtin overrides |
| `agent-config-types.ts` | Shared agent-config types |
| `hook-config-handler.ts` | `applyHookConfig` |
| `plugin-components-loader.ts` | CC plugin discovery |
| `mcp-config-handler.ts` / `command-config-handler.ts` / `tool-config-handler.ts` / `provider-config-handler.ts` | One phase each |
| `prometheus-agent-config-builder.ts` / `plan-model-inheritance.ts` | Prometheus config + plan demotion (inherits fallback_models) |
| `agent-priority-order.ts` / `agent-key-remapper.ts` | Ordering + key to display name |
| `category-config-resolver.ts` | `resolveCategoryConfig` (re-exported by config-handler) |
| `index.ts` | Barrel |

## TOOL PERMISSIONS

| Agent | Granted | Denied |
|-------|---------|--------|
| Librarian | grep_app_* | - |
| Atlas, Sisyphus, Prometheus | task, task_*, teammate | - |
| Hephaestus | task | - |
| Default (all others) | - | grep_app_*, task_*, teammate, LSP |

## MULTI-LEVEL CONFIG MERGE

Config loading is NOT here: it lives in `../plugin-config/omo-config-chain.ts` (`loadOmoOpenCodeConfigChain`), which delegates to `@oh-my-opencode/omo-config-core`. This directory only consumes the resolved `OhMyOpenCodeConfig`.

- `agents`, `categories`, `claude_code`: deep merged
- `disabled_*` arrays: Set union
