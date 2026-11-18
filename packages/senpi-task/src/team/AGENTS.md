# team/ — Named-Team Runtime (senpi-task)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 14 (78 files, own barrel; largest senpi-task subtree after `dag/`). Parent [`../../AGENTS.md`](../../AGENTS.md) owns the delivery model (injection-driven mailboxes, reservation commit, session-start recovery) and the lead-only tool family; this file maps the modules.

## STRUCTURE

| Group | Files | Role |
|------|-------|------|
| Spec | `normalize.ts` (`normalizeSenpiTeamSpec`, `TEAM_LEAD_SENTINEL`), `member-validator.ts`, `errors.ts` (`SenpiTeamSpecError`) | Spec grammar + validation |
| Durable state | `registry.ts` (team registry), `storage.ts` (runtime dirs, member inbox dirs, project spec path), `member-map.ts` (member↔task map) | Where teams live on disk |
| Lifecycle | `runtime.ts` / `runtime-types.ts` / `runtime-config.ts` (`createTeam` / `deleteTeam`), `spawn-members.ts` (planning; slow-host / error / task-summary suites), `member-respawn.ts` (+`member-revival.test.ts`), `member-projection.ts` (status refresh), `liveness-ownership.ts` (member-task identity/ownership), `shutdown.ts` + `shutdown-helpers.ts` (handshake), `tasks.ts`, `fallback-notification.ts`, `member-extensions.ts` | Create / spawn / respawn / status / shutdown |
| `messaging/` | `send.ts`, `message.ts` (peer envelope), `delivery-journal.ts` + `delivery-events.ts`, `lead-poller.ts` (+ types), `reclaim.ts`, `session-marker-index.ts` (per-path incremental byte-offset JSONL index — O(1) per messageId lookup, rescans on truncation), `session-start-reconcile.ts` | Delivery mechanics under the parent's model |
| `member-extension/` | `index.ts` (the in-child extension), `self-poller.ts` (the member's own inbox poller), `tools.ts` (member-scoped `task_send` only), `parse-env.ts`, `identity.ts`, `session-scan.ts`, `residency.ts`, `member-session.ts`, `qa-inject-hold.ts` | Runs INSIDE each member process; the adapter never polls member inboxes |
| Tests | `import-discipline.test.ts` (import boundaries), `__fixtures__/runtime-fakes.ts` | |

## CONVENTIONS

- `import-discipline.test.ts` pins what may import what — extend it with any new dependency edge.
- Member-side behavior is reachable only through `member-extension/` (loaded in-child); lead-side through `messaging/` + `runtime.ts`.
- `session-marker-index.ts` reads only bytes appended since the last check; do not re-read whole session files per tick.

## ANTI-PATTERNS

- Do not add adapter-side polling of member inboxes — the member extension owns that.
- Do not bypass the reservation states (unread → `.delivering-` → `processed/`); the processed file is the durable exactly-once ledger.
- Do not spawn members ad hoc; `spawn-members.ts` owns planning and its error taxonomy.

## COMMANDS

```bash
tsgo --noEmit -p packages/senpi-task/tsconfig.json
bun test packages/senpi-task/src/team
```
