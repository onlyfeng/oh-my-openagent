# src/locks - Cross-Process Lock Domains

**Generated:** 2026-11-18 / c2691818a9

**Score:** 25 (33 files, ~4.1k LOC, barrel with >10 exports; consumers across agreements, facts, concurrency, git, reflection).

## OVERVIEW

Filesystem locks with owner identity and proof-based recovery. Core API: `acquireLock` / `withLock` / `isHeld` / `releaseLock` + `LockContentionError`. Domain separation (memory-write, reflection-scheduler, transcript, recall-wake, reflection-run) is the design - never one global lock.

## WHERE TO LOOK

| Task | File |
|------|------|
| Acquire / with-lock / release | `acquire.ts` + `acquire-core.ts`; bounded backoff in `lock-wait-backoff.ts`, `retry-delay.ts` |
| Domain names + paths | `domains.ts` (one row per lock domain) |
| Lock record format | `lock-record.ts` - `createLockRecord` / `parseLockRecord` (pid, process-start identity, hostname, nonce, timestamp, purpose) |
| Owner identity + liveness | `process-identity.ts`, `process-start-time.ts` (`getPidLiveness`, `getProcessStartIdentity`), `lock-owner.ts` |
| Dead-owner reclaim | `dead-owner-sweep.ts` (`sweepDeadOwnerLocks`), `proven-dead-takeovers.ts`, `stale-owner.ts` - takeover requires PROOF of dead ownership, never age |
| Recall-wake counting lease | `counting-lease.ts` + `recall-wake-domain.ts` - N slot locks + FIFO ticket queue; bounded wait ends in `RecallWakeBusyError` |
| Reflection-run lease | `reflection-run-domain.ts` - `acquireReflectionRunLease` |
| Sweep hygiene | `candidate-sweep.ts` + `sweep-memo.ts` (sweeps must not block acquisitions) |
| Kernel-held lock | `os-file-lock.ts` - flock(2) via bun:ffi, non-blocking only; liveness answered by the kernel (omo #10322) |
| Test seams | `subprocess-worker.ts`, `process-liveness.test-support.ts` (real subprocess contention, not mocks) |

## CONVENTIONS

- The lock file IS the lock record; publication is atomic. Ownership is checked before age.
- Tombstone names must never match leaked-candidate names (sweep contract).
- Contention tests drive real subprocesses; timing-based tests are forbidden.

## ANTI-PATTERNS

- Never add a global lock or borrow another domain - add a row to `domains.ts`.
- Never delete a lock on age alone - only proven-dead takeover.

Parent: [`../../AGENTS.md`](../../AGENTS.md) (one `src/locks/` row; this file is its detail).
