# src — skills-loader-core source root

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 16 (137 files, 19 subdirs, barrel + subpath export surface)

## OVERVIEW

Source root of `@oh-my-opencode/skills-loader-core` (harness-neutral). Package role and consumer map live in [`../AGENTS.md`](../AGENTS.md); the two deep feature areas carry their own files. This file covers the barrel, the shared contracts, and the areas without their own AGENTS.md.

## STRUCTURE

```
src/
├── index.ts           # barrel: `export *` from 6 areas + `export type * from "./types"`
├── types.ts           # shared readonly contract types
├── text-imports.d.ts  # ambient `*.md` default-string module declarations
├── config/            # git-env-prefix schema/validation, skills config
├── shared/            # skill-path-resolver (re-export from @oh-my-opencode/utils)
├── tools/skill/       # name matcher + scope priority
├── hooks/auto-slash-command/
└── features/          # builtin-skills (own AGENTS.md), opencode-skill-loader (own AGENTS.md), opencode-runtime-skills
```

## WHERE TO LOOK

| Area | Facts |
|------|-------|
| Barrel + subpaths | `index.ts` re-exports `features/opencode-skill-loader`, `features/builtin-skills`, `features/opencode-runtime-skills`, `tools/skill`, `shared/skill-path-resolver`, `config/git-env-prefix`, and all types. package.json subpath exports (`.`, `./opencode-skill-loader(/*)`, `./builtin-skills(/*)`, `./opencode-runtime-skills(/*)`, `./skill`, `./auto-slash-command`, `./shared/*`, `./config/*`) are authoritative — new public surface needs barrel and subpath kept aligned. |
| `types.ts` | `SkillDefinition`, `SkillSource`, `SkillsConfig`, `RuntimeSkillConfig`, `SkillMcpConfig`, `BrowserAutomationProvider`, `GitMasterConfig`, `CommandInfoLike` (+ `CommandDefinition` re-export). Readonly fields, SKILL-frontmatter-shaped metadata. Every feature plus ~70 consuming `omo-opencode` files compile against these — widen carefully, never loosen field types in place. |
| `tools/skill/` | `matchSkillByName`: exact case-insensitive name → unambiguous short name (last `/` segment); zero or multiple candidates → `undefined` — ambiguity deliberately refuses to resolve instead of guessing. `scope-priority.ts` `sortByScopePriority` orders skills/commands for picking. |
| `hooks/auto-slash-command/` | `detector.ts` + `processed-command-store.ts` + `constants.ts` + `types.ts`: auto slash-command detection over the processed command stream; consumed by omo-opencode's hook of the same name. |
| `features/opencode-runtime-skills/` | `runtime-skill-config.ts` (source string / `{ path, recursive?, glob? }` resolution), `source-server.ts` `createRuntimeSkillSourceServer`, `skill-markdown.ts`. Small enough not to warrant its own file. |
| `config/` | `git-env-prefix.ts`: `GitEnvPrefixSchema`, `isValidGitEnvPrefix` / `assertValidGitEnvPrefix`, `GIT_ENV_PREFIX_VALIDATION_MESSAGE`; `skills.ts` skills-config assembly. |
| `shared/` | `skill-path-resolver.ts` is a pure re-export of `@oh-my-opencode/utils/skill-path-resolver` — keep utils the single implementation home. |

## CONVENTIONS

- Harness-neutral: no OpenCode / Senpi / Codex imports anywhere under `src/`; adapters consume this package.
- 39 colocated `*.test.ts` (Bun) under `src/` — test new modules beside their implementation.
- Skill configuration accepts source strings or `{ path, recursive?, glob? }` plus `enable` / `disable` lists and per-skill definitions; preserve that tolerance when extending the schema.
