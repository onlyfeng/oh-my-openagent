# senpi-desktop-tool — Computer Tool + `/computer` Command

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 28 (35 files; 17 modules under `src/`). One-row placement in the parent role map.

## OVERVIEW

The `computer` tool definition, permission tiers, host policy, TypeBox settings, the `/computer` command, and skill materialization. Package `@oh-my-opencode/senpi-desktop-tool` (private, ESM, TS-source; deps: `senpi-desktop-{prelude,protocol,sdk,service}`, `typebox`).

## WHERE TO LOOK

| File | Role |
|------|------|
| `src/index.ts` | Full surface: activation handle, `/computer` command, action schemas/parsers + action tool, host policy, permission parser/tiering, settings schema/resolver, session helpers, skill materialization, tool definition/runner/factory |
| `src/registration.ts` | The deliberately NARROW second entry: command, definitions, host policy, permission, host-context type, settings, skill — omits implementation-facing tool/service action exports so a registration-only bundle never reaches the desktop service or engine |
| `src/tool.ts` / `tool-definition.ts` | Tool runner (`COMPUTER_TOOL_NAME`, ~365 LOC) + definition |
| `src/command.ts` | `/computer` subcommands (`COMPUTER_SUBCOMMANDS`, usage) |
| `src/permission.ts` | `COMPUTER_PERMISSION`, `computerPermissionParser`, `computerTier` |
| `src/host-policy.ts` | `defaultStopHotkey`, `isSupportedHost` |
| `src/settings.ts` / `params.ts` / `action-schema.ts` | TypeBox settings schema/resolver, params, action schemas |
| `src/session.ts` | `ComputerHostContext` |
| `src/cua-actions.ts` / `cua-adapter.ts` / `cua-definition.ts` / `cua-errors.ts` | Codemode (cua) action scripts + adapter |
| `src/skill.ts` | `materializeComputerSkill`, `computerSkillMarkdown` (`COMPUTER_SKILL_NAME`) |
| `src/activation.ts` | Activation handle |
| `test/` | Vitest suites |

## ANTI-PATTERNS

- NEVER broaden `src/registration.ts` to import service/engine implementation — its purpose is keeping the registration-only bundle free of those runtime dependencies.
- Permission and host policy stay isolated in their named modules — don't inline policy checks into the tool runner.

## COMMANDS

```bash
bun run --cwd packages/senpi-desktop-tool typecheck   # tsgo --noEmit
bun run --cwd packages/senpi-desktop-tool test        # vitest --run test
bun run test:desktop                                  # repo desktop suites
```
