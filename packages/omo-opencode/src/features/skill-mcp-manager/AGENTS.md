# src/features/skill-mcp-manager/ — Tier-3 Skill-Embedded MCP Shims

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Every non-test `.ts` file here is a ONE-LINE shim: `export * from "@oh-my-opencode/mcp-client-core/skill-mcp-manager/<module>"`. The tier-3 implementation — `SkillMcpManager` client lifecycle, connection factory, transports, OAuth, cleanup — lives in [`packages/mcp-client-core/src/skill-mcp-manager/`](../../../../../packages/mcp-client-core/src/skill-mcp-manager) (guide: [`mcp-client-core/AGENTS.md`](../../../../../packages/mcp-client-core/AGENTS.md)). This directory keeps the OpenCode import paths stable. Do NOT add logic here — extend `mcp-client-core` and keep the shim.

## SHIM MAP

| File | Re-exports from `mcp-client-core/skill-mcp-manager/`
|------|-----------------------------------------------------|
| `index.ts` | barrel (`.../skill-mcp-manager`) |
| `manager.ts` | `SkillMcpManager` main API |
| `types.ts` | manager + client types |
| `connection.ts` / `connection-type.ts` | client factory / transport detection |
| `stdio-client.ts` / `http-client.ts` | transport factories |
| `cleanup.ts` | idle cleanup + signal handlers |
| `oauth-handler.ts` | OAuth token lifecycle |
| `env-cleaner.ts` / `error-redaction.ts` | env filtering / secret masking |

## ROLE (tier 3 of the MCP system)

Per-session, skill-embedded MCP servers declared in SKILL.md YAML frontmatter. Clients are keyed `${sessionID}:${skillName}:${serverName}` (verified in core `manager.ts`): per-session isolation, one skill usable in several sessions concurrently, multiple servers per skill. Tier 1 = built-in MCPs (`src/mcp/`); tier 2 = `.mcp.json` (`../claude-code-mcp-loader/`).

## LOCAL TESTS (the only real code here)

`manager.test.ts`, `connection-race.test.ts`, `connection-env-vars.test.ts`, `disconnect-cleanup.test.ts`, `oauth-handler.test.ts`, `manager-oauth-retry.test.ts`, `http-client.test.ts`, `env-cleaner.test.ts`, `grant-observer.test.ts`, `env-policy-authority.test.ts`, `project-env-policy.test.ts`, `plugin-reload-mcp-survival.test.ts`.

## ANTI-PATTERNS

- NEVER implement behavior in a shim file; its whole body is the single `export * from` line.
- Implementation-detail questions (retry counts, TTLs, OAuth flow) are answered by `mcp-client-core`, not here.
