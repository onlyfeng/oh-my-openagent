# skill-commands - Bare Skill Slash Commands

**Generated:** 2026-11-18 / c2691818a9

**Score:** 9 (7 files, ~0.9k LOC, own barrel; distinct domain - `/skill-name` aliasing for bundled skills).

## OVERVIEW

Registers the bare-skill command surface: bundled skills become invocable `/name` commands and host autocomplete is wrapped to complete them. Subscribes `input` + `session_start`.

## WHERE TO LOOK

| Task | File |
|------|------|
| Component factory | `index.ts` - `SKILL_COMMANDS_COMPONENT_NAME`, `createSkillCommandsComponent`, `SkillCommandsComponentOptions` (`skillsDir` override + `env`, the fixture seams) |
| Bare-command parse/resolve | `bare-skill-command.ts` - `parseBareSkillCommand`, `resolveBareSkillCommand`, `readBundledSkillNames`, `HostCommandInfo` |
| Autocomplete wrapping | `autocomplete.ts` - `wrapWithBareSkillCommands`, `AutocompleteProviderLike` |

## CONVENTIONS

- Bare command names: lowercase alphanumeric + hyphen only.
- Resolution covers BUNDLED skills only - no user/project skill directories.
- Structural interfaces (`AutocompleteProviderLike`, `HostCommandInfo`), never concrete host command types.

## ANTI-PATTERNS

- Do not widen name validation beyond the bundled-skill grammar.
- Do not read host state directly; `skillsDir`/`env` options are the injection seams.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/skill-commands
```
