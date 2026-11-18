# src/reflection/ — Reflection Run Lifecycle

**Score:** 15 (57 files, ~10.6k LOC; barrel `index.ts` re-exports 21 modules; 33 external importers concentrated in `packages/omo-senpi/src/components/memory/**`).

## OVERVIEW

Reflection-run lifecycle: reservation locking, run state machine, worktree create/finalize, completion validation (secret scan + link repair), park circuit-breaker, quarantine, orphan/host sweeps, stale-lock reaping, run-id high-water marks. Domain invariants (locks, harness neutrality, transactional writes) live in [`../../AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Concern | Files |
|---------|-------|
| Reservation/locks | `reservation.ts`, `reservation-files.ts` (`writeJsonAtomic`, `moveTornReservationAside`), `terminal-claim.ts`, `stale-holder.ts` |
| State machine | `machine.ts` — `evaluateTransitions`/`evaluate`, reserve/complete/promote transitions |
| Worktree runs | `worktree.ts`, `worktree-integration.ts` — create/finalize/discard, `integrateValidatedReflection`, `probeReflectionIntegration` |
| Completion validation | `completion-validation.ts`, `completion-secret-scan.ts`, `completion-links.ts`, `link-rewrite.ts`, `completion-repair.ts` |
| Sweeps | `orphan-sweep.ts`, `host-sweep.ts`, `identity-orphan-sweep.ts` — per-item receipts; one failing item never aborts a sweep |
| Reaper | `stale-lock-reaper.ts` (958 LOC, largest file): `runStaleLockReaper`, `decideStaleReflectionLock`/`decideStrandedRun`, `quarantineStrandedRun`, `reclaimDeadOwnerLocks`; `reflection-lock-bundle.ts` re-exports the lock/sweep cluster |
| Park/quarantine | `park.ts`, `park-file.ts`, `quarantine.ts`, `abandoned.ts`, `description-preflight.ts` |
| Run identity | `run-id-high-water.ts`, `run-ledger.ts`, `run-process.ts` (`classifyRecordedProcess`) |
| Personas | `assets/assets.ts` loads `reflection-persona.md` / `dream-persona.md` prompt payloads |

## CONVENTIONS

- Decision functions are pure, returning typed discriminated unions (`StaleLockDecision`, `StrandedRunDecision`, `CompletionValidation`, `ReservationResult`); side-effectful wrappers (`runStaleLockReaper`, `runTerminalClaim`) consume them. Time/process/git arrive via injected probes (`StaleLockProbe`, `RecordedProcessSeams`, `GitExec`) — never `Date.now()`/`exec` inside decision logic.
- Decision rules carry machine-readable `rule:` tags named by their test; `stale-holder.scenarios.test-support.ts` (663 LOC) drives the tagged cases — extend the scenario tables instead of adding ad-hoc tests.
- Named exported `REFLECTION_*` constants (orphan grace 15 min, torn-lock min age 1 h, park streaks 3/6, probe interval 6 h) instead of inline magic numbers; errors are subclasses (`RunTerminalClaimUnrecoverableError`, `ReflectionParkStateError`).

## ANTI-PATTERNS

- NEVER delete a torn reservation/lock file — move aside as `*.torn-<time>`, bytes unchanged (`reservation-files.ts`).
- NEVER derive a timestamp from a fresh clock when a ledger/reservation timestamp exists (`quarantine.ts`); NEVER lower the run-id high-water mark.
- The secret screen must never see, drop, or squash the link-repair commit; repair is recomputed on the screened tip. Refusal text names the earliest introducing commit, never the raw secret-shaped value.
- An unreadable park file must never block reflection; a diagnostic sink failure must not prevent the park gate from settling (`description-preflight.ts`).
- Lock-holder classification proves "stuck", never "dead" (`stale-holder.ts`); queued runs stay in `pending.json`, never promoted from the CLI (`stale-lock-reaper.ts`).
- Renaming barrel exports breaks the omo-senpi memory wiring broadly (33 importing files).

## COMMANDS

```bash
bun test packages/memory-core/src/reflection/    # suite (package-wide: bun test packages/memory-core/src/)
bun run --cwd packages/memory-core typecheck
```
