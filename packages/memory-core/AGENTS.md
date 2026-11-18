# memory-core - Harness-Neutral Agent Memory Engine

**Generated:** 2026-11-18 / c2691818a (update pass)

## OVERVIEW

`@oh-my-opencode/memory-core` owns the shared memory domain used by harness
adapters: a git-backed markdown MemFS, atomic memory tools, prompt compilation,
reflection scheduling, transcript search, synchronization, and seed content.
The public API is the barrel at `src/index.ts`.

## ANATOMY

| Directory | Responsibility |
|-----------|----------------|
| `src/git/` | Git command boundary, clean-tree checks, commits, merges, remotes, and typed git errors. A stranded git lock is reclaimed only with proven-dead ownership (`repo-lock-sweep.ts`); a kept one surfaces `StrandedGitLockError` with the way out, `inspectStrandedGitLock` / `unlockStrandedGitLock` (`repo-lock-unlock.ts`: refuse on any visible live holder, remove one confirmed lock after a fresh re-check). |
| `src/identity/` | Memory identity resolution and the `OMO_MEMORY_HOME` directory layout. `resolveAutoIdentity(cwd, env, mode, probe?)` is the one entry point for the auto identity; its opt-in `"git"` mode (`git-layout.ts`) maps a linked worktree's cwd onto the main working tree by reading the `.git` pointer, `commondir`, the admin `gitdir` back-pointer and `core.worktree`/`core.bare` (no `git` spawn), and any unreadable, forged or ambiguous layout keeps the path-derived id. |
| `src/locks/` | Cross-process locks for memory writes, reflection scheduling, and transcript state, plus the machine-wide `recall-wake` counting lease (`recall-wake.slot-<n>.lock` per slot, FIFO `recall-wake.tickets/`, default 2 slots, proof-based stale recovery for slots and tickets alike, bounded wait ending in `RecallWakeBusyError`). |
| `src/memfs/` | Memory-path validation, markdown frontmatter parsing, and hook-script installation. |
| `src/tools/` | `memory` and `memory_apply_patch` operations, patch parsing, typed tool errors, and auto-commit behavior. `leaked-arguments.ts` splits an argument that leaked into its sibling (`summary</description>\n<parameter name="file_text">body`) back apart before `runMemoryTool` validates. |
| `src/journal/` | Per-conversation transcript cursors, reflection snapshots, and durable journal state. |
| `src/facts/` (own AGENTS.md) | Durable fact pipeline: queue + cursor watermarks, failure backoff/store, payload capping, person routing, recovery, mutation planning. |
| `src/people/` | People-card grammar: parse/serialize, slug rules, reserved slugs, observations. |
| `src/soul/` | Soul-file paths and identity-scoped soul-notice watermark consumption. |
| `src/agreements/` | The working-agreement ledger at `system/agreements.md` (#10780): one line per agreement with `id`/`by`/`n`/`since`/`last`/`status` in a trailing comment, id = sha256 of the normalized text (first 10 hex). `mutate.ts` counts repeat corrections, flags `needs-human` at the threshold, and at the cap evicts only the oldest never-reaffirmed, unflagged agent item (reported as an `AgreementEvent`, named in the facts commit as `Omo-Agreement-Evicted`) or refuses the new one - it never merges distinct rules; `guard.ts` refuses automation edits to `by: human` items; `render.ts` builds the compiled `<agreements>` section; `notice-watermark.ts` announces each newly flagged item once per identity. |
| `src/recall/` (own AGENTS.md) | Recall candidate engine: BM25 store and term transfer, the query planner, excerpt selection, and the strategy chooser (`CJK_CORPUS_MIN_SHARE`, `LARGE_CORPUS_MIN_DOCUMENTS`). Consumed by the Senpi Kibitzer sidecar, read-only. |
| `src/receipts/` | Durable per-mutation receipts (`appendMemoryReceipt` / `appendMemoryReceiptOnce` / `readMemoryReceipts`, identity by `receiptIdentity`, kill-point accounting) so a caller can prove a write landed. |
| `src/audit/` | `auditMemoryRepo`: read-only repository audit returning `MemoryAuditReport` with typed `MemoryAuditCode` issues (`/doctor` and `/memory` surfaces). |
| `src/personas/` | `PERSONA_ASSET_FILENAMES` (reflection/dream/facts/kibitzer persona markdown) + `loadPersonaAsset`; every consumer derives filenames from this record so one rename cannot strand a reference. |
| `src/crdt/` | Loro CRDT runtime for concurrent memory edits: `loro-runtime.ts` owns the wasm load and packaging contract, plus the test fixture. |
| `src/reflection/` (own AGENTS.md) | Reflection-run lifecycle: reservations, state machine, worktrees, completion validation, sweeps, stale-lock reaper, park policy — see [`src/reflection/AGENTS.md`](src/reflection/AGENTS.md). |
| `src/compile/` (own AGENTS.md) | Compile committed memory revisions into marked system-prompt blocks and cache them by template hash. |
| `src/search/` | Query parsing, transcript providers, and ranked memory/session search. |
| `src/sync/` (own AGENTS.md) | Remote mirror synchronization and secret redaction. |
| `src/reminders/` | Reflection and memory-maintenance reminder generation. |
| `src/seeds/` | Default memory blocks and first-run repository seeding. |
| `src/concurrency/` | Subprocess fixtures for lock and multi-writer tests; not a public runtime module. |

## CORE INVARIANTS

- **Harness-neutral:** no Senpi/Pi/OpenCode adapter imports in production code or deps (`src/harness-neutrality.test.ts` enforces it).
- **Transactional state:** write tools take the `memory-write` lock, require a clean repository, validate paths, apply one operation, commit only the affected paths. Never bypass `GitMemoryRepo`, lock domains, or the tool entry points.
- **Compile from committed state:** compilation targets `HEAD` or an explicit revision; uncommitted working-tree content is not authoritative memory.
- **Markdown contracts:** frontmatter with a non-empty `description`; `read_only: "true"` blocks mutation; UTF-8, normalized repository-relative paths, LF output.
- **Strict YAML frontmatter, one grammar everywhere:** `renderMemoryFile` is the ONLY writer — it quotes any scalar the `yaml` package would not read back verbatim and re-parses its own header; `describeFrontmatterViolation` gates the pre-commit hook rules, `validateCompletion`, and `normalizeMemoryFrontmatter` (one-time legacy repair). Never emit an unquoted `description:` by hand.
- **Deterministic reflection:** manual triggers outrank compaction, which outranks step-count triggers; only one active run and one merged pending reservation.
- **Domain-specific locks** (`memory-write`, `reflection-scheduler`, transcript) — never one global lock, never timing-based tests.
- **Redact before external synchronization;** never copy secret-bearing raw logs or configuration into committed memory.
- **Facts state is durable and fail-closed:** atomic `.tmp`+rename writes (mode `0o600`) under an identity-scoped lock; malformed JSON parses to empty; enqueue/consumed watermarks NEVER regress; ordering follows canonical journal position / snapshot boundary, never lexical message-ID comparison.
- **Agreements the person wrote are immutable to automation:** only the adapter's `/agreements` command (`commitMemoryWrite` with `writer: "human-command"`) may create, reword, ack, or drop a `by: human` item — the tools and `memory_apply_patch` refuse (`tools/agreements-write-guard.ts`), reflection/dream completion fails on any `system/agreements.md` change, move or delete included (diff `--no-renames`), and automation cannot remove a counted agent item (`n > 0`). With `agreements.enabled: false` the ledger is still projected.
- **Behavioral patterns are NEVER stored in people cards:** lines limited to `IDENTITY` / `ATTRIBUTE` / `RELATIONSHIP` / `INSTRUCTION` prefixes with bounded metadata.

## PUBLIC SURFACES

`runMemoryTool()` (`create`/`str_replace`/`insert`/`delete`/`rename`/`update_description`), `runMemoryApplyPatch()` (multi-file Codex-style patches), `GitMemoryRepo` (init, clean checks, commits, revisions, merge worktrees, remotes), `evaluateTransitions()`/`reserveTransition()`/`completeTransition()` (pure reflection state machine), `compileMemoryBlock()`/`compileMemoryBlockAtRevision()`, and `FactsQueue`/`applyFactsBatch()`/`planFactsMutation()`/`consumeSoulNoticeDelta()`.

## CONSUMERS

Harness adapters provide identity, lifecycle events, tool registration, and sync orchestration; the Senpi adapter lives in `packages/omo-senpi/src/components/memory/` — adapter-specific behavior stays there, never imported back.

## QA

```bash
bun test --timeout 20000 packages/memory-core/src/
bun run --cwd packages/memory-core typecheck
```

Run the nearest co-located `*.test.ts` first, then the package suite. Concurrency tests await exact state or process events with bounded timeouts — never fixed sleeps or retry loops.

## ANTI-PATTERNS

- Importing a harness package into `memory-core`.
- Writing memory markdown without path validation, frontmatter parsing, locking, and an atomic git commit.
- Reading the dirty worktree as compiled memory; mutating `read_only` blocks; accepting non-UTF-8 memory files.
- Swallowing git, lock, merge, or reflection failures.
- Testing cross-process behavior with timing luck.
