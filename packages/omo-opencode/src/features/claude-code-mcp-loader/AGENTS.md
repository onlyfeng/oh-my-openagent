# src/features/claude-code-mcp-loader/ — Tier 2 MCP Loader (.mcp.json)

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

This module loads `.mcp.json` files from project/user scopes and expands `${VAR}` env vars. It feeds Tier 2 of the 3-tier MCP system into `mcp-config-handler.ts` during Phase 5 of config loading. Shared parsing, env expansion, and transform primitives are extracted to [`packages/claude-code-compat-core/`](../../../../../packages/claude-code-compat-core); this directory keeps the OpenCode adapter path stable.

## WHY IT EXISTS

Claude Code ecosystem ships MCPs via `.mcp.json` files with `${VAR}` env var placeholders. OmO consumes these unchanged so existing Claude Code MCP configs work.

## LOAD PIPELINE

```
loadMcpConfigs(ctx)
  → scope-filter.ts: discover .mcp.json at project + user scopes
  → loader.ts: parse JSON
  → env-expander.ts: replace ${VAR} with process.env[VAR]
  → transformer.ts: map Claude Code format → OpenCode McpLocal / McpRemote shape
  → return LoadedMcpServer[]
```

## MCP FORMAT

```jsonc
// .mcp.json
{
  "mcpServers": {
    "my-stdio": {
      "type": "stdio",
      "command": "node",
      "args": ["server.js"],
      "env": {
        "API_KEY": "${MY_API_KEY}"
      }
    },
    "my-http": {
      "type": "http",       // "sse" legacy → mapped to http
      "url": "https://example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${MY_TOKEN}"
      }
    }
  }
}
```

## KEY FILES

| File | Purpose |
|------|---------|
| `index.ts` | Barrel: re-exports the compat-core loader |
| `loader.ts` | Shim: `loadMcpConfigs()` main entry (compat-core) |
| `types.ts` | Shim: `ClaudeCodeMcpServer`, `LoadedMcpServer`, `McpScope` |
| `env-expander.ts` | LOCAL: recursive `${VAR}` substitution via `@oh-my-opencode/utils` `expandEnvReferences*`; defaults to `trusted: false`, checks `isAllowedMcpEnvVar`, logs blocked names (test: `env-expander.test.ts`) |
| `configure-allowed-env-vars.ts` | Shim: allowlist configuration from `mcp_env_allowlist` |
| `transformer.ts` | Shim: Claude Code format to OpenCode `Mcp` shape |
| `scope-filter.ts` | Shim: project vs user scope precedence |

## THREE-TIER MCP CONTEXT

| Tier | Loader | Scope |
|------|--------|-------|
| 1. Built-in | `src/mcp/` `createBuiltinMcps()` | Global, 3 remote HTTP MCPs + 1 local stdio MCP (`lsp`) |
| 2. **Claude Code** | **This module** | **From `.mcp.json`, project + user** |
| 3. Skill-embedded | `src/features/skill-mcp-manager/` | Per-session, from SKILL.md YAML |

## SECURITY

- **Env var allowlist**: `mcp_env_allowlist` config restricts which env vars can be expanded
- **No shell execution**: `${VAR}` is string replacement only, not shell `$()`
- **Secrets redaction**: `env-cleaner.ts` (in skill-mcp-manager) filters known secret patterns from logs

## RELATED

- Phase 5 integration: `src/plugin-handlers/mcp-config-handler.ts`
- Skill-embedded MCPs (Tier 3): `src/features/skill-mcp-manager/`
- Built-in MCPs (Tier 1): `src/mcp/`
