# test/ — ULW-Loop Regression Suite

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 10 (67 spec files, ~12.2k LOC — the component's behavioral contract)

## OVERVIEW

67 colocated Vitest specs (~12.2k LOC) exercising the sibling `../src/` implementation: CLI dispatch, checkpointing/continuation, steering, quality gates, plan I/O, goals, spawn guards, Codex hooks, evidence, and the published surface. Component layout, build, and test discipline (Vitest, `#given/#when/#then`, fixtures live in `fixtures/`) are defined in [`../AGENTS.md`](../AGENTS.md) and not repeated here.

## SPEC MAP (by prefix)

| Prefix (count) | Covers |
|----------------|--------|
| `cli-*` (14) | `ulwLoopCommand` dispatch, subcommands, flag/value parsing, error surfaces |
| `quality-*` (11) | `validateQualityGate`, defect aggregation, artifact refs, rejection paths |
| `plan-*` (7) | Plan persistence, discovery, recovery, append-only ledger |
| `checkpoint-*` (5) | `checkpointUlwLoop`, `checkpointAndContinue`, resume semantics |
| `sdk-*` (5) | SDK contracts / session context shapes |
| `spawn-*` (4) | Spawn role guards and budget |
| `steering-*` (3) | UserPromptSubmit steering injection + mutations |
| `codex-*` (3) | Codex hook wiring (`user-prompt-submit`, stop-resume) |
| `evidence-*` (2), `validation-*` (2) | Evidence containment, validation batches |
| singles | `goal`, `guided`, `package` (public-surface smoke), `paths`, `review`, `state`, `status`, `stop`, `surface`, `types`, `ultrawork` (directive mirror pin) |

## FIXTURES (`fixtures/`)

- `checkpoint-builders.ts` — `goal`, `plan`, `snapshot`, `passGoal`, `NOW` (deterministic clock).
- `cli-session.ts` — `CLI_TEST_SESSION_ID` / `CLI_TEST_SCOPE` scoping for the CLI suites.
- `quality-gate-builder.ts` — `qaDirFor()` plus the artifact path constants (`QA_DIR`, `CODE_REVIEW_PATH`, `GATE_REVIEW_PATH`, `CLI_PASS_PATH`, `REJECTION_LOG_PATH`, `MISSING_ARTIFACT_PATH`).
- Detached-process probes (one-writer-per-JSONL contract): `commit-writer.ts`, `commit-child.ts`, `commit-crash.ts`, driven by `commit-cli-qa.mjs` / `commit-mutations.mjs`.
- Synchronization: `barrier.ts` (cross-process barrier), `lease-clock.ts`.
- Static payloads: `sample-plan.json`, `sample-quality-gate.json`, `sample-brief.md`, `steering-proposal.json`, `user-prompt-submit.json`, `codex-goal-snapshot.json`.

## CONVENTIONS

- Implementation imports come from sibling `../src/` with explicit `.js`/`.ts` suffixes.
- Extend a fixture builder before duplicating its setup inline; shared builders exist precisely so scenario shapes stay consistent.

## COMMANDS

```bash
npm test                               # vitest --run (CI invokes npm here — see parent)
bunx vitest run test/<file>.test.ts    # focused single spec
```
