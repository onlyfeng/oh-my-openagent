# src/tools/skill-mcp/ — Skill-Embedded MCP Tool (OpenCode)

## OVERVIEW

8 files, ~860 LOC. The `skill_mcp` tool: exposes a skill-embedded MCP
server's tools/resources/prompts as callable OpenCode tool operations, backed
by the `SkillMcpManager` (tier-3 client lifecycle; see
`src/features/skill-mcp-manager/`). Parent: [`src/tools/AGENTS.md`](../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel exporting `createSkillMcpTool` + constants/types |
| `tools.ts` | Tool definition + dispatch to manager operations |
| `parse-skill-mcp-arguments.ts` | Argument parsing (server/tool/resource/prompt addressing) |
| `constants.ts` | Tool name and limits |
| `types.ts` | Argument/result types |
| `builtin-mcp-hint.test.ts`, `host-env-grant.test.ts` | Hint and env-grant coverage |

## CONVENTIONS

- All client work delegates to the manager; this directory owns argument
  parsing and result shaping only.
- Environment grants to spawned servers follow the manager's env policy —
  never widen it locally.

## ANTI-PATTERNS

- Do not spawn or connect MCP servers here; that is the manager's job.
- Do not bypass the parse layer with loose `Record<string, unknown>` args.

## QA

```bash
bun test packages/omo-opencode/src/tools/skill-mcp
bun run --cwd packages/omo-opencode typecheck
```

Wired into OpenCode: real-harness QA (`opencode-qa`) for changes.
