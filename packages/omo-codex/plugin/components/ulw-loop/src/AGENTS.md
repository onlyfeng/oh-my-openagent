# components/ulw-loop/src/ — ULW Loop Implementation Modules

## OVERVIEW

60 TypeScript files, ~6,000 LOC. The implementation tree of the Codex
`ulw-loop` component: CLI dispatch, runtime, SDK surface, plans, checkpoints,
evidence, steering, quality gates, and Codex hooks. Style, build, and command
conventions live in the parent
[`components/ulw-loop/AGENTS.md`](../AGENTS.md) — not repeated.
**Score 18** (60 files, module density, state-transition hotspots).

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| CLI | `cli.ts`, `cli-commands.ts`, `cli-subcommands.ts`, `cli-arg-parser.ts`, `cli-output.ts`, `cli-steering.ts` | Bin entry and subcommand dispatch (`ulwLoopCommand`) |
| Runtime | `runtime.ts`, `sdk.ts`, `constants.ts`, `domain-types.ts`, `command-types.ts` | Runtime composition, SDK factory/manifest, domain types |
| Plans | `plan-io.ts` + plan modules | `.omo/ulw-loop/` persistence, append-only `ledger.jsonl`, mutation lock |
| Checkpoints | `checkpoint*.ts` | Reconciliation, continuation, Codex validation, templates |
| Evidence | `evidence*.ts`, `evidence-artifacts.ts` | Evidence containment for quality gates |
| Steering | `steering*.ts` | Steering mutations and proposals |
| Quality gates | `quality-gate*.ts` | Gate validation, artifacts, aggregate defect pre-pass |
| Codex hooks | `codex-hook.ts`, `codex-goal-*.ts` | UserPromptSubmit steering injection, `create_goal` budget guard |
| Guards | `spawn-guard.ts`, `state-lock.ts`, `stop-resume-hook.ts` | Spawn guards, state locking, Stop auto-resume |

## CONVENTIONS

- Files stay under 250 pure LOC; split by responsibility (parent rule).
- Kebab-case filenames; no default exports.
- `directive.md` is read at runtime, never inlined (parent).

## ANTI-PATTERNS

- Parent forbidden list applies everywhere here (`as any`, enums,
  non-null assertions, default exports, ...).
- Do not add alternate CLI entry aliases; dispatch flows through
  `cli-commands.ts`.

## QA

```bash
npm test                                   # from components/ulw-loop
bunx vitest run test/<file>.test.ts        # focused
```
