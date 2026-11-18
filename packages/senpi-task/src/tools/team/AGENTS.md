# src/tools/team/ — Lead Team Tool Family

**Score ~13** (18 files; distinct domain — lead-facing `team_*` tool factories).

## OVERVIEW

The lead-side team tool surface for senpi-task: factories for team lifecycle (`team_create` / `team_delete`), team task CRUD (`create/get/list/update`), member messaging, and shutdown approval. Consumed via `buildLeadTeamTool...` and registered by the omo-senpi `dag`/team wiring. Host access goes through injected ports (`LeadTeamToolDeps`, `TeamToolsService`), never direct host calls.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: re-exports factories, `run*` handlers, param schemas, types; `LeadTeamTool` union |
| `lifecycle.ts` | `createTeamCreateTool` / `createTeamDeleteTool`, `runTeamCreate` / `runTeamDelete` |
| `tasks.ts` | Team task create/get/list/update factories + `run*` + params |
| `messaging.ts` | `runTeamSend` — member/lead delivery views |
| `shutdown.ts` | `runTeamShutdownRequest` / `runTeamApproveShutdown` / `runTeamRejectShutdown` |
| `classify-error.ts` | `classifyMailboxError`, `isMissingStateError`, `MailboxErrorKind` |
| `renderers.ts` | renderCall/renderResult formatting |
| `types.ts` | `LeadTeamToolDeps`, `TeamToolDeps`, `TeamToolsService`, status types |
| `__fixtures__/team-tool-fakes.ts` | Test fakes |

## CONVENTIONS

- Each factory is generically typed so its `renderCall`/`renderResult` keep their own arg/detail types; the family is published as a union (`LeadTeamTool`) and registration spreads each one (same pattern as the task tools).
- Tests are behavior-pinned per concern: `lifecycle-collision`, `lifecycle-precedence`, `tasklist-exposure`, `messaging`, `shutdown`, `allowlist`.

## ANTI-PATTERNS

- Parameter/validation errors are data outcomes, never thrown exceptions (matches the thread-contract rule).
- Do not bypass the injected service/ports with direct host or filesystem access.

## COMMANDS

- `bun test packages/senpi-task` · `tsgo --noEmit -p packages/senpi-task/tsconfig.json`
