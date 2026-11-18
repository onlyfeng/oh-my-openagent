# auto-slash-command - Slash-Command Execution Hook

**Score:** 12 (14 files, 2.3k LOC, index barrel, two test files >500 LOC). Parent [`src/hooks/AGENTS.md`](../AGENTS.md): Skill tier, `chat.message` event, composed in `create-skill-hooks.ts`.

## OVERVIEW

Detects a slash command in user input, executes it once, and wraps the result in `auto-slash-command` tags. `detector.ts` is a pure re-export of `@oh-my-opencode/skills-loader-core/auto-slash-command/detector` - detection logic lives in the core package, this directory is the OpenCode hook + executor wiring.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Hook factory | `hook.ts` - `createAutoSlashCommandHook` (`AutoSlashCommandHookOptions`) |
| Execution | `executor.ts` - `executeSlashCommand` (`ExecutorOptions`, `ExecuteResult`) |
| Detection (re-export shim) | `detector.ts` - `detectSlashCommand`, `extractPromptText`, `findSlashCommandPartIndex` |
| Dedup store | `processed-command-store.ts` - per-session processed-command memory |
| Tags + constants | `constants.ts` - `AUTO_SLASH_COMMAND_TAG_OPEN`/`CLOSE` |
| Types / barrel | `types.ts`, `index.ts`; `leak/` holds leak-prevention tests |

## CONVENTIONS

- Execution is deduped by resolved event id plus a short fallback TTL (`COMMAND_EXECUTE_FALLBACK_DEDUP_TTL_MS = 100`) so a command never runs twice for one submission.
- Output is wrapped in the `AUTO_SLASH_COMMAND_TAG_*` markers; keep that envelope stable - downstream consumers parse it.
- Command execution routes through the loaded commands/skills surface (see `src/features/builtin-commands/`), never a shell.

## ANTI-PATTERNS

- Do not reimplement detection here - extend `skills-loader-core` and keep this shim a re-export.
- Do not execute non-slash input; the detector's verdict gates the executor.
- Test hotspots: `executor.test.ts` (596 LOC) and `index.test.ts` (536 LOC) pin resolution and integration; `init-deep-skill.test.ts` pins the init-deep interaction.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-opencode/src/hooks/auto-slash-command
bun run --cwd packages/omo-opencode typecheck
```
