# src/features/builtin-commands/ -- Built-in Slash Commands

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Registry of built-in commands shipped inside the plugin. Each command is a template literal with title, description, and instructions. Built by the private `createBuiltinCommandDefinitions()` in `commands.ts`; the public entry is `loadBuiltinCommands()`. Definitions use the `CommandDefinition` type from `claude-code-command-loader`.

## FILE CATALOG

| File | Purpose |
|------|---------|
| `commands.ts` | private `createBuiltinCommandDefinitions()` + exported `loadBuiltinCommands()` filter |
| `index.ts` | Barrel exports |
| `types.ts` | `BuiltinCommandName` union type + `BuiltinCommandConfig` |
| `templates/` | One `.ts` file per command; `refactor-sections/` splits the refactor prompt |

## TEMPLATES

| Command | Source File | Notes |
|---------|-------------|-------|
| `goal` | `templates/goal.ts` | Persistent thread objective (set / pause / resume / clear); replaces removed `/ralph-loop`, `/ulw-loop`, `/cancel-ralph` |
| `refactor` | `templates/refactor.ts` | LSP + AST-grep refactoring |
| `ulw-execute` | `templates/ulw-execute.ts` | Prometheus plan executor |
| `stop-continuation` | `templates/stop-continuation.ts` | Stop all continuations (todo enforcer, ralph loop, goal, boulder) |
| `handoff` | `templates/handoff.ts` | Session context summary |
| `remove-ai-slops` | `templates/remove-ai-slops.ts` | AI code smell cleanup |
| `hyperplan` | `templates/hyperplan.ts` | Adversarial team-mode planning |

## STRUCTURE

Each template exports a string constant containing the command's system prompt. `commands.ts` wraps it in `<command-instruction>` XML and injects `$ARGUMENTS`, `$SESSION_ID`, and `$TIMESTAMP` where needed. Some commands append a team-mode addendum when `teamModeEnabled` is true.

## LOADING

Phase 6 of config loading (`command-config-handler.ts`) merges built-ins with user-installed commands from `.opencode/commands/` and Claude Code plugins. `disabled_commands` in config filters out specific built-ins by name. The `autoSlashCommand` hook in `src/hooks/` executes these on user input.

## TESTS

`commands.test.ts` (registry + team-mode addenda) and `init-deep-migration.test.ts` at the root; `templates/goal.test.ts` and `templates/stop-continuation.test.ts` cover those templates.
