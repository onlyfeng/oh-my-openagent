# src/fs — Resilient Filesystem Seam

**Generated:** 2026-11-18 / c2691818a9

**Score:** 10 (12 files, ~1.1k LOC; distinct low-level domain — the only place memory-core touches `node:fs`).

## OVERVIEW

Resilience wrappers over `node:fs`: EINTR retry, exclusive-flag write policy, rename-contention retry, full-buffer writes, recursive tree removal. `no-direct-node-fs.test.ts` forbids every OTHER memory-core source file from importing `node:fs` / `node:fs/promises` — all fs traffic funnels through this seam.

## WHERE TO LOOK

| Task | File |
|------|------|
| Import fs primitives | `resilient.ts` — the namespace: `resilientNamespace`, `wrapFileHandle`, `rename`, `open`, `writeFile`, `appendFile`, `closeSync`, `existsSync`, re-exported `constants`/`watch` |
| EINTR retry policy | `retry.ts` — `retryOnEintr` / `retryOnEintrSync`, `EINTR_RETRY_CAP` |
| Exclusive-flag open + full writes | `write-all.ts` — `isExclusiveFlag`, `openSyncWithExclusivePolicy`, `writeHandleAll` |
| Contentious rename (Windows/AV) | `rename-contention.ts` — `renameSyncWithContentionRetry` |
| Recursive tree removal | `remove-tree.ts` |
| Test double seam | `spy-scope.test-support.ts` |

## CONVENTIONS

- No barrel: consumers import the module they need; `resilient.ts` is the integration point.
- Async and sync variants stay in lockstep — a fix to one lands in the other.
- Tests colocated; `no-direct-node-fs.test.ts` is the boundary guard and must keep passing when files are added.

## ANTI-PATTERNS

- NEVER import `node:fs` directly outside this directory — the boundary test fails the suite.
- No unbounded retries: EINTR retry is capped, rename contention bounded.

Parent: [`../../AGENTS.md`](../../AGENTS.md) (its ANATOMY table does not itemize `src/fs/` — this file is that row's detail).
