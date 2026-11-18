# features/builtin-skills - Built-in Skill Catalog

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Skills shipped inside the plugin (no install). `skills.ts` `createBuiltinSkills()` assembles the catalog; every entry implements `BuiltinSkill` (`types.ts`: name, description, template, optional mcpConfig, agent/model/allowedTools gating, argumentHint). Same-name user-installed skills outrank built-ins (builtin/shared = 1 of 6, [scope-priority](../opencode-skill-loader/merger/scope-priority.ts)).

## STRUCTURE

```
builtin-skills/
├── skills.ts             # createBuiltinSkills() + resolveActiveBuiltinSkills()
├── types.ts              # BuiltinSkill interface
├── skill-file-loader.ts  # createSharedSkillTemplateLoader / loadSharedSkillTemplate
├── skills/               # one module per skill + git-master-sections/
└── git-master/ hyperplan/ security-research/ dev-browser/ frontend/  # resources, see ANTI-PATTERNS
```

## WHERE TO LOOK

| Concern | Facts |
|---------|-------|
| Catalog assembly | `createBuiltinSkills({ browserProvider, disabledSkills, teamModeEnabled, playwrightMcpArgs })` returns ONE browser skill (below) plus frontend, git-master, review-work, remove-ai-slops, init-deep, debugging, security-research, security-review, hyperplan, visual-qa; `disabledSkills` (name Set) filters the result. |
| Browser variants | `browserProvider` picks one: `"playwright"` (default, MCP), `"playwright-cli"` (token-efficient CLI), `"dev-browser"` (persistent page state, inline template + `resolvedPath`). `playwright` and `playwright-cli` both export name `"playwright"` (agents hardcode it); only the implementation swaps. |
| playwright MCP args | `playwrightMcpArgs` appends flags after the default `npx @playwright/mcp@latest` mcpConfig invocation (e.g. `--executable-path`, `--no-sandbox` for sandboxes); absent or empty keeps the `playwrightSkill` singleton byte-identical. |
| team-mode | Included ONLY when `teamModeEnabled && !disabledSkills.has("team-mode")`; documents the 12 `team_*` tools. The interface has NO `shouldLoad` field; gating lives in the factory. |
| MCP collision | `resolveActiveBuiltinSkills({ systemMcpNames })` drops skills whose `mcpConfig` keys collide with system MCP names (only `playwright` declares one today). |
| Template sources | (a) `loadSharedSkillTemplate(name)`: cached frontmatter body from `@oh-my-opencode/shared-skills/<name>/SKILL.md` (debugging, frontend, init-deep, remove-ai-slops, review-work, visual-qa). (b) inline `with { type: "text" }` import of sibling SKILL.md (security-research, hyperplan). (c) security-review aliases security-research's template and description. |
| git-master | Template COMPOSED from `skills/git-master-sections/` constants (overview, commit-workflow, rebase-workflow, history-search-workflow, quick-reference) joined with `---` separators; name/description from `git-master-skill-metadata.ts`. |

## CONVENTIONS

- Adding a skill: create `skills/<name>.ts` exporting a `BuiltinSkill`, register it in `skills.ts` AND the `skills/index.ts` barrel, put resources under a sibling `<name>/` dir. Conditional skills are gated in `createBuiltinSkills()`.
- `createSharedSkillTemplateLoader(readFile?, rootPath?)` stays injectable for tests; ENOENT rethrows (no silent fallback).

## ANTI-PATTERNS

- NEVER edit `git-master/SKILL.md`, `dev-browser/SKILL.md`, or `frontend/SKILL.md` expecting prompt changes: git-master ships composed sections, dev-browser an inline template, frontend the shared-skills copy (the local file feeds `shared-skill-extraction.test.ts` only). Only `hyperplan/SKILL.md` and `security-research/SKILL.md` are imported at runtime.
- NEVER export a duplicate skill `name` (sole sanctioned exception: the browser-variant swap behind `"playwright"`).
