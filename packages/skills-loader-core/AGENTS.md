# skills-loader-core — Skill Loading + Matching (Core)

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Harness-neutral skill loading, builtin skill, runtime skill, and skill matching primitives.
Package `@oh-my-opencode/skills-loader-core`.
Confirmed by grep: consumed by `omo-opencode` in ~49 files — `src/features/opencode-skill-loader` (22), `src/features/opencode-runtime-skills` (4), `src/tools/skill` (9), `src/hooks/auto-slash-command` (5), plus 9 more (plugin skill context, plugin-handlers, config validation, `create-managers.ts`). `src/features/builtin-skills` on the consumer side is gone; the builtin catalog lives HERE, in `src/features/builtin-skills/`.

## SUBSYSTEMS

| Area | Purpose | Sub-AGENTS.md |
|------|---------|---------------|
| `features/opencode-skill-loader/` | Multi-source SKILL.md discovery, parse, merge, deduplication | [src/features/opencode-skill-loader/AGENTS.md](src/features/opencode-skill-loader/AGENTS.md) |
| `features/builtin-skills/` | Built-in skill catalog (git-master, browser variants, frontend, debugging, visual-qa, security, review-work, team-mode) | [src/features/builtin-skills/AGENTS.md](src/features/builtin-skills/AGENTS.md) |
| `features/opencode-runtime-skills/` | Runtime security skill source (security-research, security-review) | — |
| `tools/skill/` | Skill name matching and scope priority sorting | — |
| `hooks/auto-slash-command/` | Slash command detection and processed command store | — |
| `shared/` | Skill path resolver, config dir discovery, plugin identity, shell env | — |
| `config/` | Git env prefix and skills config schema | — |

## NOTES

- **Exports:** barrel root plus subpath exports for `./opencode-skill-loader`, `./builtin-skills`, `./opencode-runtime-skills`, `./skill`, `./auto-slash-command`, `./shared/*`, and `./config/*`.
- **Skill matching:** `tools/skill/skill-matcher.ts` provides `matchSkillByName()`, `matchCommandByName()`, and `findPartialMatches()`; ambiguous short-name matches refuse to resolve.
- **Runtime skill source:** `features/opencode-runtime-skills/` exports `selectRuntimeSecuritySkills()` and `createRuntimeSkillSourceServer()`; dual-runtime (Bun.serve preferred, Node http fallback), loopback-only, `cache-control: no-store`, 404 on unknown paths.
- **126 TypeScript files** under `src/` (87 non-test). Barrel entry `src/index.ts`.

## COMMANDS

```bash
bun test packages/skills-loader-core/src          # co-located suite (package.json: bun test src/*.test.ts src/**/*.test.ts)
tsgo --noEmit -p packages/skills-loader-core/tsconfig.json   # typecheck (package.json: tsgo)
```

Parent: [packages/AGENTS.md](../AGENTS.md)
