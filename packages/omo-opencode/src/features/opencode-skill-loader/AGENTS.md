# src/features/opencode-skill-loader/ — 4-Scope Skill Discovery

**Generated:** 2026-05-15 / refreshed 2026-11-18

## OVERVIEW

This module discovers, parses, merges, and resolves SKILL.md files from 4 scopes with priority deduplication. Harness-neutral loader, builtin skill, runtime skill, and skill matching primitives are extracted to [`packages/skills-loader-core/`](../../../../../packages/skills-loader-core); this directory is the OpenCode adapter shim.

## 4-SCOPE PRIORITY (highest → lowest)

```
1. Project (.opencode/skills/)
2. OpenCode config (~/.config/opencode/skills/)
3. User (~/.config/opencode/oh-my-opencode/skills/)
4. Global (built-in skills)
```

Same-named skill at higher scope overrides lower.

## KEY FILES

| File | Purpose |
|------|---------|
| `index.ts` | Re-export shim over `@oh-my-opencode/skills-loader-core/opencode-skill-loader` |
| `loader.ts` | Main `loadSkills()` — orchestrates discovery → parse → merge |
| `merger.ts` + `merger/` | Priority-based deduplication across scopes; the subdir holds the complex name/MCP overlap cases (see MERGER SUBDIRECTORY below) |
| `skill-content.ts` | YAML frontmatter parsing from SKILL.md |
| `skill-discovery.ts` / `skill-directory-loader.ts` | Find SKILL.md files in directory trees; load one directory |
| `loaded-skill-from-path.ts` / `loaded-skill-template-extractor.ts` | Build a loaded skill from a single path; extract template bodies |
| `skill-template-resolver.ts` | Variable substitution in skill templates |
| `skill-mcp-config.ts` | Extract MCP configs from skill YAML |
| `allowed-tools-parser.ts` | Parse `allowed-tools` frontmatter |
| `skill-deduplication.ts` / `skill-definition-record.ts` | Post-merge dedup; per-skill definition records |
| `git-master-template-injection.ts` | Git-master commit-attribution template injection |
| `opencode-shell-skill-content.ts` | OpenCode shell-skill content shaping |
| `skill-resolution-options.ts` | Resolution options plumbed through discovery/merge |
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

Handles complex merge logic when skills from multiple scopes have overlapping names or MCP configs.

## TEMPLATE RESOLUTION

Variables like `{{directory}}`, `{{agent}}` in skill content get resolved at load time based on current context.
