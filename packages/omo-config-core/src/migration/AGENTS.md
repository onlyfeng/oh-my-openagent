# src/migration/ — Config Migration Engine

**Score ~17** (24 files, ~2.5k LOC; barrel + journaled transaction domain).

## OVERVIEW

Harness-neutral config-migration engine for `omo-config-core`: runs declarative migration plans as journaled, resumable, no-clobber transactions over an injected filesystem. Pure logic — all IO sits behind injected ports so it runs identically under any harness.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: re-exports batch, engine, journal, lock, merge, predicate, recovery, types |
| `engine.ts` | `runMigration()` — single-plan orchestration |
| `batch.ts` | `runMigrations()` — multi-plan runs (~287 LOC, the largest file) |
| `commit.ts` | Atomic/comment-preserving target writes (~219 LOC) |
| `journal.ts` | Write-ahead journal for resumable transactions |
| `lock.ts` | Owner-aware lease lock (~181 LOC) |
| `recovery.ts` | Crash recovery; runs BEFORE predicates |
| `merge.ts` | `mergeWithoutClobber`, `collectMigrationEdits` no-clobber merge |
| `predicate.ts` | `shouldRun` content gate |
| `types.ts` | Injected ports + result contracts |

## CONVENTIONS

- Injected ports only: `MigrationFileSystem`, `MigrationClock`, `MigrationProcess`, target writer. Defaults provided; tests substitute fakes.
- Plans support `merge` and `replace-target`. A false `shouldRun` predicate skips the whole operation with NO journal, backup, target write, or marker — preserves byte identity.
- Recovery runs before predicates; per-target/per-migration `_migrations` markers record completion.
- Result types: `MigrationRunResult`, `MigrationPreview`, `RunMigrationOptions`, `MigrationBatchRunResult`, `MigrationValidationError`, `MigrationTransactionError`.

## ANTI-PATTERNS

- Do NOT import OpenCode/Codex/Senpi/Pi or adapter code here — keep IO behind injected ports.
- Do not bypass the journal/lock/backup path with a direct write.
- Do not let a false predicate leave any side effect (journal, backup, marker).

## COMMANDS

- `tsgo --noEmit -p packages/omo-config-core/tsconfig.json`
- `bun test packages/omo-config-core`
