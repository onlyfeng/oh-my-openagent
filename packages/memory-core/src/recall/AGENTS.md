# src/recall/ - Recall Candidate Engine

**Generated:** 2026-11-18 | c2691818a9

Read-only lexical recall over committed memory: corpus loading, query planning, substring/BM25/hybrid ranking, the session ledger, and the Kibitzer nudge gate. Consumed by the Senpi Kibitzer sidecar (`packages/omo-senpi`); this package stays harness-neutral (parent invariant). Earned by score: 41 files, own barrel, distinct ranking domain.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `provider.ts` | Corpus from the HEAD committed tree only - the working tree is never consulted. One `ls-tree` + at most one `cat-file --batch` per load, never one git process per file. Excludes only the ROOT `system/` tree (`reference/system/deploy.md` is recallable). `RecallCorpusCache`, `recallMemoryFigures`. |
| `planner.ts` | Newest-first conversation text → ≤4 tiny AND-semantics queries (`MAX_RECALL_QUERIES = 4`): 1-2 distinctive single terms plus verbatim-adjacent bigram phrases. Unicode tokenization; 3-char ASCII / 2-char non-ASCII floor (Korean two-syllable words like 리콜). Pure and deterministic. |
| `strategy.ts` | `chooseRecallStrategy`: `substring` (English query, small corpus), `bm25` (any CJK query or corpus ≥ `CJK_CORPUS_MIN_SHARE` 0.1), `hybrid` (≥ `LARGE_CORPUS_MIN_DOCUMENTS` 200; reciprocal-rank fusion, takes precedence over bm25). Thresholds tuned by `recall-ranker-bench.mjs` (omo-senpi scripts/qa). |
| `bm25*.ts` | Okapi BM25 (k1=1.5, b=0.75) over `description\nbody` with CJK bigram tokenization (Hangul/Han/Kana runs; Han chars are standalone terms) and English stemming (`english-stem.ts`). `-index.ts` term memoization, `-store.ts`/`-persist.ts` persisted terms under the repo `.git/` keyed by text SHA-1 + tokenizer fingerprint (derived cache; corrupt reads as empty), `-terms-transfer.ts` seeding/snapshotting. |
| `select.ts` / `excerpt.ts` / `haystack.ts` | Scores and caps candidates; excerpts (≤200-char windows centered on the first term match) are built lazily only for returned candidates; normalized haystacks memoized per document object (a changed file gets a fresh object and drops the memo). |
| `gate.ts` | Kibitzer output contract: the sidecar speaks ONLY through the nudge tool; the parent re-validates every collected nudge against the candidate set, session ledger, hint shape (`NUDGE_HINT_MAX_CHARS` 200) and cap. Accepted nudges wait in a self-describing per-session pending file; compaction retracts via `delete`; decision-commentary hints are rejected. |
| `ledger.ts` | Per-session surfaced-paths ledger (one JSON per session): fail-closed reads, atomic `.tmp`→rename at mode 0600. |
| `head-signature.ts` | FS fingerprint of what `git rev-parse --verify HEAD` would read (HEAD file content + ref mtimes/sizes + packed-refs); `undefined` falls open to the git call. |
| `render.ts` | The late-hidden nudge hint block; bilingual `RECALL_HINT_HEADER` (en/ko); redacted before render. |
| `project-corpus.ts` | Project-scoped recall (#10044): a child project's notes join under `@project/<identity-id>/<path>` - unique per identity, so ledgers and surfaced sets keep one key per note across identities. |
| `assets/` | `kibitzer-persona.md` + loader; uses `import.meta.url` (not Bun-only `import.meta.dir`) because the senpi extension bundle loads under plain Node via jiti. |

## CONVENTIONS

- Recall never mutates memory state: the only writes are derived caches in `.git/` and ledger/pending files under the identity dir.
- Secret-like material is redacted (`sync/redact`) before any hint is rendered or persisted.
- `EXCERPT_CHARS`, `NUDGE_HINT_MAX_CHARS`, and the strategy thresholds are internal on purpose - not config knobs.

## ANTI-PATTERNS

- Never read the working tree for recall candidates - uncommitted edits must not leak into a corpus.
- Never persist a nudge payload that is not self-describing; a filename collision must never hand one session another session's nudges.
- The nudge block is reference material about a stored note, never an instruction to the primary agent; hints that speak TO the agent are rejected.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/recall
bun run --cwd packages/memory-core typecheck
```
