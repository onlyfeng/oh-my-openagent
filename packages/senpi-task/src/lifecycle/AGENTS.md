# lifecycle - Residency, Suspension, Revival, Teardown

`createTaskLifecycle` (`create.ts`, 221 refs) binds every lifecycle operation to a store + residency registry + config; it is the only sanctioned way for the rest of the package to admit, suspend, revive, expire or destroy a child. 45 implementation files, 47 tests, 18 fixtures (~13.6k LOC incl. tests). Earned its file by size and centrality; the suspend/resume/lease/TTL contracts themselves are in the package [`AGENTS.md`](../../AGENTS.md) (SESSION SUSPEND AND RESUME REVIVAL) and are not repeated here.

## WHERE TO LOOK

| Concern | Files |
|---------|-------|
| Admission + cap | `residency.ts` (`admitResident`, idle reclaimer), `admission-lease.ts` + `admission-lease-wait.ts` (local release wakes waiters synchronously), `errors.ts` (`AgentLimitReached` names every resident), `idle-reclaimer-scheduler.ts` |
| Shutdown + suspension | `shutdown.ts`, `suspended-closures.ts` (startup never waits for old closes), `suspended-expiry.ts`, `park-terminal-resident.ts` (omo#9785), `teardown-budget.ts` |
| Session-start reconcile | `reconcile.ts` (entry), `reconcile-revival.ts`, `reconcile-reclamation.ts`, `reconcile-terminal.ts`, `reconcile-owner.ts` (`hasForeignLiveOwner`), `reconcile-crashed-resident.ts`, `deferred-owner-reconcile.ts` |
| Revival policy | `revival-selection.ts` (THE population + order), `revive-policy.ts`, `revive-rollback.ts` (three answers only), `revive-detached.ts`, `deferred-revival.ts` + `deferred-revival-reasons.ts` (`deferralOutlookFor`, omo#9498), `live-parent-recovery.ts` (5 min budget) |
| Daemon-hosted children | `host-session.ts` (probe, `NO_HOST_ENDPOINT`), `host-session-revive.ts`, `host-session-record.ts` (the 3 record writes the daemon path owns), `host-session-close.ts`, `host-session-default.ts` (lazy `import()` of the engine client), `host-endpoint-reach.ts` |
| Runtime fallback span | `fallback-handoff.ts`, `fallback-closing-child.ts` (`fallback_closing_child` until close confirmed) |
| Destruction + expiry | `destroy.ts` (`destroyResidentTask`), `ttl.ts`, `expunge-attempts.ts`, `pending-cancel.ts` (omo#9403), `provisional-exit.ts` |
| Ports + context | `port.ts` (`DestroyCause`, reattach/respawn/detached-revival ports registered by the manager), `context.ts` (`defaultSignaller`, `TERMINAL_STATUSES`), `pid-liveness.ts` |
| Fixtures | `__fixtures__/lifecycle-fakes.ts` (`FakeRegistry`, `seedRecord`, `tempStore`: broadest dependency), `host-session-fakes.ts`, `live-parent-fakes.ts`, `real-cold-revive*.ts`, `deferred-revival-harness.ts` |

## CONVENTIONS

- Single writer for teardown: `destroy.ts` is the ONLY function that calls a handle's `dispose()`/`terminate()` for terminal teardown; cancel, LRU eviction, TTL and reconcile all route through it with a `DestroyCause`.
- `process.kill` lives only in `src/lifecycle` (`context.ts` default signaller, `pid-liveness.ts` signal-0 probe); other modules (DAG store, recovery, manager) import `defaultSignaller` (12 refs) instead of calling `process.kill`.
- A host session has no pid: host-session paths never signal; the only answers are attach/reopen or park.
- Effects go through injected `LifecycleDeps` ports so tests swap clocks, signallers and registries; fixtures mix fakes with real store/session paths and two cross into `packages/omo-senpi/src/components/task/` for residency fixtures.

## ANTI-PATTERNS

- NEVER lose a host-owned transcript record during revival (`host-session-revive.ts`).
- `host_draining` is retryable: the child waits for the old daemon generation and is NEVER marked lost (`port.ts`, `types.ts`).
- NEVER TTL-delete a nonterminal suspended child; NEVER delete a `lost` process record without pid-dead proof (`ttl.ts`).
- Pre-dispose shutdown steps are best effort; dispose ALWAYS runs (`shutdown.ts`).
- Claim the stopped run BEFORE any close IO when expiring a suspended child (`suspended-expiry.ts`).
- An unconfirmed exit is interrupted work, never a transcript to replay after losing its owner (`provisional-exit.ts`).
- A revival that cannot proceed may only terminate the old process child, roll back its own claim, or defer (`revive-rollback.ts`); no fourth path.
- Fixtures: cold-revive traces must not advance clocks or release barriers; fake session close uses abort + `close_session`, never a signal; `idle-park-session.ts` must never prompt a provider.

## COMMANDS

```sh
bun test packages/senpi-task/src/lifecycle
bun test packages/senpi-task/src/lifecycle/reconcile.test.ts packages/senpi-task/src/lifecycle/reconcile-revival.test.ts  # largest suites (700+ LOC each)
```
