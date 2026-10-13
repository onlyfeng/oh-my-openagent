# src/tools/skill/ -- Skill and Command Loader Tool

**Generated:** 2026-10-13 | 19 files + `zauc-mocks-skill-tools/`; `scope-priority.ts` and `skill-matcher.ts` are re-export shims over `@oh-my-opencode/skills-loader-core/skill/*`

## OVERVIEW

The `skill` tool. Dual purpose: (1) load a skill by name to inject its SKILL.md content into context, (2) invoke a slash command by name (omit leading slash). Skills may spin up embedded MCP servers on demand. Commands route through the autoSlashCommand hook.

## FILE CATALOG

| File | Purpose |
|------|---------|
| `tools.ts` | `createSkillTool` factory -- resolves name, loads body, returns formatted output |
| `skill-body.ts` | Extracts `<skill-instruction>` block or full SKILL.md template |
| `skill-matcher.ts` | Exact match, short-name fallback, partial-match suggestions (re-export from skills-loader-core) |
| `scope-priority.ts` | Re-export: project (4) > user (3) > opencode / opencode-project (2) > shared / plugin / config / builtin (1) |
| `native-skills.ts` | Merges `PluginInput.skills` entries into discovered skill list |
| `description-formatter.ts` | Builds LLM-visible `<available_items>` listing with scope tags (dedup/order/formatting tests share `description-formatter.test-support.ts`) |
| `mcp-capability-formatter.ts` | Lists skill-embedded MCP tools/resources/prompts for `skill_mcp` calls |
| `session-skill-cache.ts` | Dedupes repeated skill loads per session via `seenSessionIDs` |
| `types.ts` | `SkillArgs`, `SkillInfo`, `SkillLoadOptions` |
| `constants.ts` | Tool name and description prefix |
| `index.ts` | Barrel exports |

## EXECUTION FLOW

```
skill(name="git-master")
  -> matchSkillByName()    # exact, then short-name
  -> ask(permission)       # host skill permission gate
  -> extractSkillBody()    # load SKILL.md content
  -> formatMcpCapabilities()  # if skill has mcpConfig
  -> return "## Skill: ..." + body + MCP info
```

## SCOPE PRIORITY

Project configs override user configs, which override opencode builtins. `sortByScopePriority` applies to both skills and slash commands in the `<available_items>` listing; ties within a scope break by name in code-unit order, so the listing (and the cached tool prefix) does not depend on discovery order (#9432).

## TEST MOCKS

`zauc-mocks-skill-tools/` -- `mock.module()` setup for skill tool tests. Loads alphabetically before consuming tests via the `zauc-` prefix sort-order hack.

## INTEGRATION

- Discovery: `opencode-skill-loader` feature module scans `.opencode/skills/`, `~/.config/opencode/skills/`, and built-in paths
- MCP spawn: `skill-mcp-manager` feature module starts embedded MCP servers per session on demand
- Commands: `slashcommand/` module feeds discovered commands into the tool description
