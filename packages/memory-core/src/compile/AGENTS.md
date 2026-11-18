# src/compile/ — Memory Prompt-Block Compilation

**Generated:** 2026-11-18 | c2691818a9

Compiles a committed memory revision into the `<memory>` system-prompt block — persona/identity tree, agreements section, `<external_projection>` listings, expiry evaluation — and caches the result by (revision, template hash, expiry slot). Distinct domain: the only place memory bytes become prompt bytes. Package invariants live in [`packages/memory-core/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Path | Role |
|------|------|
| `compile.ts` | `compileMemoryBlock(WithReads\|AtRevision)`: reads the committed tree, renders `system/persona.md` + `system/identity.md`, injects the agreements section and the memory REMINDER. `asOfMs` lets a session pinned to a revision keep byte-identical output when a file expires mid-session; returns per-read degraded reasons. |
| `cache.ts` | `MEMORY_TEMPLATE_STRUCTURE_VERSION` (`senpi-memory-v2`), `MEMORY_BLOCK_COMPILER_VERSION` (2), `hashMemoryTemplate`; in-memory cache plus optional lazily-loaded persisted store. |
| `block-persist.ts` | Persisted blocks under the repo's `.git/` (never committed, pushed, or synced): keyed by a digest of every input, `KEEP_ENTRIES = 32`, atomic temp+rename; a missing/corrupt/foreign entry reads as absent and is rewritten. Saves re-listing the tree at relaunch (~94 ms warm vs 3.5 s cold at ~6.9k notes). |
| `stored-block.ts` | `StoredBlock` (`block`, `slot`, `expiries`) JSON encode/decode; undecodable entries read as `undefined`. |
| `expiry-slot.ts` | The clock slice a block depends on: which `expires` instants have passed. Same slot → same bytes, so a block is cached per (revision, slot). |
| `external-projection*.ts` | `<external_projection>` bounded directory listings with `$MEMORY_DIR` pointers: grouping (`-dirs`), naming (`-names`), byte/item budgets (`-limits`), precision; newest-first ordering degrades to name order when commit times are unreadable (`recencyUnavailable` + stats). |
| `changes.ts` | `ProjectedChanges` (added/updated/removed): which path changes actually alter a compiled block — `system/*.md` edits count (bodies are projected), other paths count only on arrival/removal (name-only projection). `Omo-Session` trailer exclusion. |
| `in-flight-reads.ts` | Tracks started-but-unfinished repo reads; `settled()` must complete before an identity directory is removed (Windows EBUSY, #11135). |
| `render.ts` | `renderSystemTree`: the indented `system/` tree with per-file descriptions. |
| `fixtures/` | Golden compiled blocks: `empty`, `full`, `persona-only`, `persona-identity`. |

## CONVENTIONS

- Bump `MEMORY_BLOCK_COMPILER_VERSION` with ANY change to the bytes `compileMemoryBlockAtRevision` produces for the same inputs — persisted blocks are keyed by it, and a block compiled by older code must never be served.
- Secret-like material is redacted (`sync/redact`) before projection.
- Co-located tests cover cache, persist, changes, projections, expiry, and metadata; fixtures pin golden output.

## ANTI-PATTERNS

- A degraded block (git read timeout, lock contention) is right for this moment only — never persist it as the block of its commit.
- Never hand-edit a persisted block entry; it is a derived cache, not state.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/compile
bun run --cwd packages/memory-core typecheck
```
