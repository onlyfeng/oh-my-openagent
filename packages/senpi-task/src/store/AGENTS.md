# store - Durable Task Record Store

File-backed persistence for task records: `createTaskRecordStore` (`record-store.ts`), record locks, strict parsers, event logs, state-dir resolution, and the host-session rollback migration. 20 implementation files, 14 tests (~5k LOC excl. `__fixtures__/`). Earned its file as the single durability boundary every other subsystem writes through; the cache, fd pool and default state-dir layout are summarized in the package [`AGENTS.md`](../../AGENTS.md) and not repeated here.

## LAYOUT ON DISK

- Records: `<stateDir>/tasks/<taskId>.json`, replaced whole through temp-file + rename (`record-write.ts` `atomicReplace`; win32 retries sharing-violation renames with a bounded backoff, #8050).
- Event logs: `<stateDir>/logs/<taskId>.jsonl`, append-only (`event-log.ts`, `taskEventLogPath` is shared so the DAG activity clock stats the same file).
- TTL tombstone: `<taskId>.json.expunging` plus optional owner marker `<taskId>.json.expunging.owner` (`expunge-owner.ts`); both are invisible to `load`/`list`.
- Removal cleans child sessions, completion spill and the event log before deleting the record.

## WHERE TO LOOK

| Concern | Files |
|---------|-------|
| Store API | `index.ts` barrel, `types.ts` (`TaskRecordStore`, `TombstoneResult`, `PersistedTaskEvent`), `record-store.ts` |
| Locks | `record-lock.ts` (`withTaskRecordLock` / `withTaskRecordLockAsync`, reaper), `lock-owner.ts` (holder identity + proven-dead check) |
| Claims | `claim.ts` (`claimTaskRecord`; `claim-race.test.ts` covers cross-process races) |
| Parsing | `record-parse.ts` (`parseTaskRecord`), `record-blocks-parse.ts` (ownership, spawn spec, steering queue, model chain, epochs, isolation, host session), `run-stats-parse.ts`, `start-failure-parse.ts`, `provisional-exit-parse.ts`, `scalar-read.ts` |
| State dir | `state-dir.ts` (`resolveStateDir`), `project-state-directory.ts` (`projectStateKey`, `IN_PROJECT_STORE_MARKER`) |
| Rollback | `rollback-migrate.ts` (`planHostSessionSocketMigration` / `migrateHostSessionSockets`), `rollback-r0-contract.ts` (R0 = #9027 merge, commit ebd01f84e, persisted vocabulary) |
| Security | `redaction.ts` (`redactEventPayload`), `security.test.ts` (redaction, traversal rejection) |

## CONVENTIONS

- Parsers are absent-tolerant but type-strict: a missing optional field is fine (older records stay valid), a present field of the wrong type always throws, never coerces.
- Legacy records default new intent flags conservatively (no `notify_on_terminal` -> false).
- Sync `node:fs` for record and lock updates; the async lock variant exists only for async critical sections.
- Platform-dependent branches (win32 rename retry) take an injected platform so tests cover them on any OS.

## ANTI-PATTERNS

- Transitions NEVER delete a record; use `transition()` (`types.ts`).
- Once a legacy in-project store holds records it must not move; live sessions may still write there (`project-state-directory.ts`).
- A lock is removed by anyone but its holder only when the holder is PROVEN dead (`lock-owner.ts`); a reaper that lost its recovery lock must not unlink the primary lock, and must not delete a lock another process took afterwards (`record-lock.ts`).
- A missing record is a defined outcome, never an `ENOENT` throw (`record-store.ts`).
- Rollback preparation may remove only values outside the R0 contract (`rollback-r0-contract.ts`).

## COMMANDS

```sh
bun test packages/senpi-task/src/store
bun packages/senpi-task/scripts/manual-qa.ts <evidence-dir>   # store + transitions on a disposable tree
```
