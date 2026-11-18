# lifecycle — residency, shutdown, reconcile, revival, TTL

**Score 17** (49 impl + 61 test modules, ~5.9k impl LOC). Owns the durable side of every task child: admission, suspension, crash reconciliation, revival, and expiry. Suspend/resume SEMANTICS (revival populations, admission-lease protocol, deferred outcomes, no-rerun rules) live in the parent [`../../AGENTS.md`](../../AGENTS.md) — not repeated here.

## OVERVIEW

`createTaskLifecycle` (`create.ts`) binds one store + residency registry + config into the lifecycle object; it is the only sanctioned way the rest of the package triggers destroy/suspend/reconcile/TTL. Public surface is the `index.ts` barrel: `createTaskLifecycle`, `selectRevivalBatch`/`RevivalSelection`, `AgentLimitReached`/`ResidentSummary`, host-session probe helpers (`createHostSessionProbe`, `DEFAULT_HOST_SESSION_RETRY_POLICY`, `NO_HOST_ENDPOINT`) and types, reattach + detached-revival rollback port registration/lookup, and the lifecycle result/deps contracts (`types.ts`).

## WHERE TO LOOK (module families)

| Area | Modules |
|------|---------|
| Composition | `create.ts`, `context.ts` (`resolveContext` supplies port defaults), `port.ts` (widest DI contract: `LifecycleDeps`, signaller, respawn/reattach ports), `types.ts`, `errors.ts` |
| Admission / residency | `residency.ts` (`admitResident`, `reclaimIdleResidents`, idle reclaimer), `admission-lease.ts` + `admission-lease-wait.ts` (renewable lease + bounded acquisition), `batch-admission.ts` (+ race tests), `revival-selection.ts` |
| Reconcile | `reconcile.ts` (session-start), `reconcile-revival.ts` (+race), `reconcile-reclamation.ts`, `reconcile-crashed-resident.ts`, `reconcile-loss.ts`, `reconcile-owner.ts`, `reconcile-terminal.ts`, `deferred-owner-reconcile.ts` |
| Revival | `revive-detached.ts`, `revive-policy.ts`, `revive-rollback.ts` (+epochs tests), `revival-claim-fence.ts`, `deferred-revival.ts` (+lifecycle tests, `deferred-revival-reasons.ts`), `idle-cold-revive.ts` (+trace), `respawn-failure-code.ts`, `run-paused-revival` tests |
| Host sessions | `host-session.ts`, `host-session-endpoint.ts` (probe/endpoint), `host-session-revive.ts` (park on daemon loss), `host-session-cancel-driver.ts`/`-draining`/`host-session-cancel-draining`, `host-session-close.ts`, `host-session-end.ts` (+ownership), `host-session-teardown`, `host-session-default.ts`, `host-session-record.ts` (adds `suspension_reason` to records) |
| Shutdown / destroy / TTL | `shutdown.ts` (reason-aware scoped suspension), `destroy.ts`, `ttl.ts` (two-phase sweep), `expunge-attempts.ts`, `pending-cancel.ts`, `park-terminal-resident.ts`, `stopped-owner.ts` (+records/shutdown tests), `teardown-budget.ts`, `suspended-closures.ts`, `suspended-expiry.ts` |
| Liveness / crash | `pid-liveness.ts`, `provisional-exit.ts`, `fallback-closing-child.ts` (+obligation), `fallback-handoff.ts` (+reconcile), `live-parent-recovery.ts` (+guards), `crash-recovery`, `own-crash-reconcile` tests |

Shared child fixtures under `__fixtures__/` (cold-revive harness/team/trace, batch-admission race child).

## INVARIANTS (stated in module headers — preserve them)

- `ttl.ts`: a suspended child whose work is in flight is NEVER TTL-deleted; a `lost` process record is NEVER deleted without pid-dead proof.
- `port.ts`: a child with a pending owner is NEVER lost, by construction.
- `host-session-revive.ts`: a host-session child record is NEVER lost — losing it would discard a transcript the daemon still owns.
- `shutdown.ts`: pre-dispose steps are best-effort so dispose ALWAYS runs.
- Claim/epoch fences and the admission lease are the sanctioned ownership/capacity boundaries — never bypass them in reconcile or revival paths.

## CONVENTIONS

- Persisted task fields are snake_case (`task_id`, `host_pid`, `residency_state`, `run_epoch`); the TS API is camelCase; residency and reconcile outcomes are string unions.
- All effects (spawn, signal, timers, store) arrive via injected ports resolved in `context.ts`; tests use fake stores/runners, controlled clocks, and temporary session files.
- Tests are `bun:test` with `#given/#when/#then` names; race behaviors get paired impl+test modules (`admission-lease-handoff`, `reconcile-revival-race`).

## COMMANDS

- `bun run --cwd packages/senpi-task typecheck`; focused `bun test packages/senpi-task/src/lifecycle`; full gate `bun run test:senpi`.
