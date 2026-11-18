# src/store/ — Durable Task Record Store

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 24 (48 files, ~6.2k LOC). The parent row in [`../../AGENTS.md`](../../AGENTS.md) owns the summary contract (JSONL records, mtime+size-validated read cache, capped-16 append-fd pool, `resolveStateDir`, redaction); this file maps the internals.

## OVERVIEW

Persistence behind a deliberately narrow barrel: record parsing, locking, claiming, event log, host-session migration, redaction.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | The ONLY stable surface: `claimTaskRecord`, `createTaskRecordStore` (+resident-store types), `TaskRecordCollisionError`, `migrateHostSessionSockets`/`planHostSessionSocketMigration`, `resolveStateDir`, `resolveProjectStateDirectory`/`projectStateKey` |
| Store core | `record-store.ts`, `record-write.ts`, `storable-record.ts` | Store facade + figures, atomic writes, storable-record conversion |
| Locking / claim | `record-lock.ts`, `lock-owner.ts`, `claim.ts` | Cross-process record locks (+reap window), claim with race tests (`claim-race.test.ts`) |
| Parse family | `record-parse.ts`, `record-blocks-parse.ts` (~310 LOC), `host-session-parse.ts`, `input-effects-parse.ts`, `model-refusals-parse.ts`, `provisional-exit-parse.ts`, `run-pause-parse.ts`, `run-stats-parse.ts`, `start-failure-parse.ts`, `record-optional-fields-parse.ts`, `scalar-read.ts` | Parsers split by persisted-record field group |
| Events / expunge | `event-log.ts`, `expunge-owner.ts` | `taskEventLogPath` + persisted events; tombstone/expunge owner types |
| Migration | `rollback-migrate.ts`, `rollback-r0-contract.ts` | Host-session socket migration, planned then executed |
| Security | `redaction.ts`, `security.test.ts` | Redaction + security proofs |
| Paths | `state-dir.ts`, `project-state-directory.ts` | Default state dir + project-state keys |
| Fixtures | `__fixtures__/` | Store fixtures shared by suites |

## ANTI-PATTERNS

- Transitions NEVER delete a record — they go through `transition()` (`types.ts`).
- Never bypass the claim/lock/atomic-write paths for competing record updates; claim-race, lock-recovery, rollback and security suites pin them.
- No new public surface outside `index.ts` — the package export map has no `./store` subpath; helpers stay module-local.

## COMMANDS

```bash
bun test packages/senpi-task/src/store           # focused
bun run --cwd packages/senpi-task typecheck      # tsgo --noEmit
```
