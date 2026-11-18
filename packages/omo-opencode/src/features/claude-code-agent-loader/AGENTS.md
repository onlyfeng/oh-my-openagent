# src/features/claude-code-agent-loader/ -- Claude Code Agent Compatibility Layer

**Generated:** 2026-05-18 | **Updated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Sibling to `claude-code-mcp-loader`. Loads Claude Code agent definitions from `.opencode/agents/`, `~/.claude/agents/`, and inline `opencode.json` config, then translates them to OpenCode `AgentConfig`. Every file here is a one-line `export *` shim re-exporting [`packages/claude-code-compat-core/claude-code-agent-loader`](../../../../claude-code-compat-core/AGENTS.md) — all implementation lives in Core; edit there, not here.

## LOAD PIPELINE (implemented in Core)

```
loadUserAgents() / loadProjectAgents() / loadOpencodeGlobalAgents() / loadOpencodeProjectAgents()
  -> Core loader.ts: discover .md files in agents/ directories
  -> Core agent-definitions-loader.ts: parse YAML frontmatter + body, load from explicit paths
  -> Core json-agent-loader.ts: parse .json / .jsonc agent definitions
  -> claude-code-compat-core opencode-config-agents-reader.ts: read inline agents from opencode.json
  -> Core claude-model-mapper.ts: translate "sonnet" / "opus" / "haiku" -> OpenCode provider/model IDs
  -> return Record<string, ClaudeCodeAgentConfig>
```

## KEY FILES (all `export *` shims over Core)

| File | Re-exports from `@oh-my-opencode/claude-code-compat-core/claude-code-agent-loader/` |
|------|------------------------------------------------------------------------------------|
| `index.ts` | barrel (`...` root) |
| `loader.ts` | `/loader` — `loadUserAgents()`, `loadProjectAgents()`, `loadOpencode*Agents()` |
| `agent-definitions-loader.ts` | `/agent-definitions-loader` — `parseMarkdownAgentFile()`, `loadAgentDefinitions()` |
| `json-agent-loader.ts` | `/json-agent-loader` — `parseJsonAgentFile()` (JSON/JSONC) |
| `claude-model-mapper.ts` | `/claude-model-mapper` — Claude aliases -> OpenCode `providerID/modelID` |
| `types.ts` | `/types` — `ClaudeCodeAgentConfig`, `AgentScope`, `LoadedAgent` |

## INTEGRATION

Phase 3 of config loading: `src/plugin-handlers/agent-source-loader.ts` (invoked by `agent-config-handler.ts`) imports the four scope loaders plus `loadAgentDefinitions` and `readOpencodeConfigAgents` from this barrel, migrates each definition (`migrateAgentConfig`), and populates the agent registry before the plugin interface is built. Runtime consumer: `src/tools/delegate-task/subagent-discovery.ts` re-runs `loadUserAgents()` / `loadProjectAgents()` for subagent discovery.

## COMPANION LOADERS

- **`packages/claude-code-compat-core/src/features/claude-code-plugin-loader/`**: full plugins with commands, skills, hooks, MCPs
- **`claude-code-mcp-loader`**: Tier 2 MCPs from `.mcp.json`
