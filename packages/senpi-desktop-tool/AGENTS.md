# senpi-desktop-tool

The senpi `computer` and `computer_actions` tool definitions, permission tiers, settings schema, `/computer` command, supported-host policy, and activation handle. Full tool, permission, and settings contract: `README.md`. Own file: 34 files, two entry points, score 10.

## WHERE TO LOOK

| Path | Purpose |
|------|---------|
| `src/registration.ts` | `./registration` subpath: what a host needs to register computer use before the implementation loads (#9113). |
| `src/tool-definition.ts` / `tool.ts` / `params.ts` | `computerToolDefinition` (no `execute`), `createComputerTool` + `parseComputerParams`; flat root-object `ComputerParams` narrowed to the per-action `ComputerActionShape` (timeout 1-600 s, default 60). |
| `src/cua-definition.ts` / `cua-adapter.ts` / `cua-actions.ts` / `cua-errors.ts` | Optional `computer_actions` (OpenAI computer-use snake_case actions or `batch`), routed through the same run path; engine failures mapped to `COMPUTER_*` codes with recovery hints. |
| `src/action-schema.ts` | `ComputerArgumentsError` (`COMPUTER_INVALID_ARGUMENTS`) raised before any engine starts. |
| `src/permission.ts` | `computerPermissionParser`: `read` vs `exec` tiers; malformed or unknown is `exec`. |
| `src/activation.ts` / `command.ts` / `host-policy.ts` | `ComputerHandle` (idempotent activate, re-open only on param change), `/computer on/off/status/stop/resume`, `isSupportedHost` + `defaultStopHotkey`. |
| `src/session.ts` / `settings.ts` / `skill.ts` | `session.open` params + coordinate-safe sizing, `resolveComputerSettings` (unknown keys throw), content-addressed skill materialization. |

## CONVENTIONS

- Vitest (`bun run --cwd packages/senpi-desktop-tool test`); `test/fixtures.ts` holds shared fakes (the README's `test/coding-agent-contract.ts` pin does not exist at this commit).
- Imports neither coding-agent nor codemode; types are structural. Workspace deps: `-prelude`, `-protocol`, `-service`; peer `typebox`.
- Published parameter schemas stay one root object: providers reject a root union (Anthropic `type: "object"`, OpenAI strict, Gemini no root `anyOf`).

## ANTI-PATTERNS

- Never evaluate permissions inside `execute`; the permission-system `tool_call` hook is the only evaluator (a second one prompts twice under `ask`).
- Never add a `resume` action; only the user resumes, through `/computer resume` or the host stop surface.
- An "always" approval stores only its own tier; never widen `read` approval to `exec`.

## COMMANDS

```bash
bun run --cwd packages/senpi-desktop-tool test          # vitest suite
bun run --cwd packages/senpi-desktop-tool typecheck     # tsgo
bun run test:desktop                                    # all five desktop packages
```
