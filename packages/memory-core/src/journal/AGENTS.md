# src/journal/ — Transcript Journal (per-conversation durability)

**Generated:** 2026-11-18 / c2691818a9

Earned its file (score ~25): 12 TS files, ~2.3k LOC, the durable journal domain behind a single anatomy row in the parent [packages/memory-core/AGENTS.md](../../AGENTS.md). Barrel + >10 exports + referenced from the reflection/search/facts lanes.

## WHERE TO LOOK

| Task | File |
|------|------|
| Append + projection + lock orchestration | `store.ts` — `TranscriptJournal` (append, state/cursor projection, lock acquisition, atomic replacement, durability via `fsync.ts`) |
| Cursor + reflection snapshots | `cursor.ts` — `captureCursorSnapshot`, `deriveState`, `initialReflectionState`; state schema `v3_assistant_steps` |
| Entry types + truncation | `entries.ts` — `projectTranscriptEntries`, `truncateToolResult`, the caps below |
| Cross-process local lock | `lock.ts` — `withLocalJournalLock`, `JournalLockTimeoutError`, `ACQUISITION_WAIT_MS` |
| Empty-journal sweep | `sweep.ts` — `sweepEmptyTranscriptJournals` + keep reasons |
| Public API | `index.ts` barrel — the only import surface for the rest of memory-core |

## INVARIANTS

- Capture caps are contracts, not tunables: tool args ≤ 300 bytes (`TOOL_ARGS_TRUNCATE_LIMIT`), tool results ≤ 4096 bytes (`TOOL_RESULT_TRUNCATE_LIMIT`) kept as a head+tail window; reasoning text is stored only as `REDACTED_REASONING_TEXT` (`[REDACTED REASONING]`).
- A reflection snapshot is bounded by `REFLECTION_SNAPSHOT_MAX_BYTES` = 131072 serialized bytes; an oversized backlog ships the OLDEST window that fits and leaves the rest for the next run (the cursor stays put while reflection fails).
- Lock contention surfaces as the typed `JournalLockTimeoutError`, never a raw lock-file `EEXIST`; callers may degrade a best-effort write to a no-op instead of failing the extension.
- The sweep is conservative and reason-coded: live, locked, recent, non-empty, unreadable and changed-during-removal journals are all retained.

## ANTI-PATTERNS

- Never append unbounded tool results or raw reasoning into the journal — extend `entries.ts` truncation instead.
- Never bypass `TranscriptJournal` / `withLocalJournalLock` with direct `transcript.jsonl` writes.

## TESTS

Colocated per module (`store`, `cursor`, `entries`, `lock`, `sweep`): `bun test --timeout 20000 packages/memory-core/src/journal`; package-level gates live in the parent AGENTS.md.
