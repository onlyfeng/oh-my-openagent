# src/features/opencode-skill-loader/ - Skill Discovery Adapter (shims)

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

This module discovers, parses, merges, and resolves SKILL.md files with numeric scope-priority deduplication. Harness-neutral loader, builtin skill, runtime skill, and skill matching primitives are extracted to [`packages/skills-loader-core/`](../../../../../packages/skills-loader-core); this directory is the OpenCode adapter shim. 23 of 24 non-test `.ts` files (including all 5 in `merger/`) are one-line `export * from "@oh-my-opencode/skills-loader-core/opencode-skill-loader/..."`. The only local logic is `opencode-shell-skill-content.ts` (OpenCode shell selection plus sync/async skill content resolution and git-master template injection). Imported by agents, hooks, `create-hooks.ts`, `create-tools.ts`.

## SCOPE PRIORITY (`merger/scope-priority.ts`, Core)

```
opencode-project 6 > project 5 > opencode 4 > user 3 > config 2 > builtin = shared 1
```

Same-named skill at higher priority overrides lower.

## KEY FILES (all Core-backed shims unless noted)

| File | Purpose |
|------|---------|
| `loader.ts` | Main `loadSkills()`: discovery, parse, merge |
| `merger.ts` | Priority-based deduplication across scopes |
| `skill-content.ts` | YAML frontmatter parsing from SKILL.md |
| `opencode-shell-skill-content.ts` | LOCAL: shell-aware content resolution (tests: `skill-content.test.ts`, `git-master-*.test.ts`) |
| `skill-discovery.ts` | Find SKILL.md files in directory trees |
| `skill-directory-loader.ts` | Load all skills from a single directory |
| `config-source-discovery.ts` | Discover scope directories from config |
| `skill-template-resolver.ts` | Variable substitution in skill templates |
| `skill-mcp-config.ts` | Extract MCP configs from skill YAML |
| `types.ts` | `LoadedSkill`, `SkillScope`, `SkillDiscoveryResult` |

## SKILL FORMAT (SKILL.md)

```markdown
---
name: my-skill
description: What this skill does
tools: [Bash, Read, Write]
mcp:
  - name: my-mcp
    type: stdio
    command: npx
    args: [-y, my-mcp-server]
---

Skill content (instructions for the agent)...
```

## MERGER SUBDIRECTORY

Shims over Core `merger/`: `builtin-skill-converter`, `config-skill-entry-loader`, `scope-priority`, `skill-definition-merger`, `skills-config-normalizer`. Edit Core, not these files.

## TEMPLATE RESOLUTION

Variables like `{{directory}}`, `{{agent}}` in skill content get resolved at load time based on current context.
