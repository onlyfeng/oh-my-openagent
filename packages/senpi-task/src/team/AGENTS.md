# src/team - Senpi Named-Team Runtime

Senpi-side team runtime over `@oh-my-opencode/team-core` primitives: spec discovery and normalization, member spawn/respawn as senpi-task children, durable mailbox messaging, the in-child member extension, tasklist binding, and the shutdown handshake. 75 TypeScript files (about 3.1k non-test LOC) behind the `index.ts` barrel, re-exported by the package barrel. Earned this file: distinct domain with its own invariants against team-core; the delivery model itself is documented in the package AGENTS.md (TEAM DELIVERY MODEL) and is not repeated here.

## MODULE MAP

| Path | Purpose |
|------|---------|
| `normalize.ts` / `member-validator.ts` | `normalizeSenpiTeamSpec` (team-core normalizer + `TeamSpecSchema`, never team-core `validateSpec`), `TEAM_LEAD_SENTINEL`, senpi-local member validation. |
| `registry.ts` | `loadTeamRegistry`: `omo.json` `teams` plus `<projectRoot>/.omo/teams/<name>/config.json`; the project directory wins a name collision; a failing team lands in `errors` and spawns zero members. |
| `storage.ts` | `teamStorageBaseDir` = `<resolveStateDir(cfg)>/teams` (the base passed to team-core path helpers), inbox/runtime dir resolution. |
| `runtime.ts` / `runtime-types.ts` / `runtime-config.ts` | `createTeam` / `deleteTeam`: `max_members` enforced before any spawn, state seeded `creating`, rollback to `failed` on the first spawn failure; `toTeamCoreConfig` pins team-core storage under the state dir. |
| `spawn-members.ts` / `member-respawn.ts` | Members spawn as durable background process children capped by `max_parallel_members` under a wall-clock deadline; respawn launch resolver for reconcile. |
| `member-map.ts` / `member-projection.ts` / `liveness-ownership.ts` | Atomic member -> `st_` task sidecar map next to runtime state; member status projection; ownership checks from task-store liveness. |
| `messaging/` | `sendTeamMessage`, `createLeadPoller`, in-memory lead delivery journal, `session-marker-index.ts` incremental JSONL ack index, `reclaim.ts` + `session-start-reconcile.ts` (stale `.delivering-` reservations back to unread, default TTL 10 min). |
| `member-extension/` | Bundle loaded INSIDE each process member (`MEMBER_EXTENSION_BUNDLE_NAME`, `parse-env.ts` identity env): `self-poller.ts` member inbox poller, `session-scan.ts` delivery ack, `tools.ts` team-scoped `task_send` (static JSON Schema), `qa-inject-hold.ts` QA hook. |
| `tasks.ts` | Team tasklist bound to one run (`createTeamTask`, `claimTeamTask`, `updateTeamTaskStatus`, typed claim/transition errors). |
| `shutdown.ts` / `shutdown-helpers.ts` | `requestShutdown` / `approveShutdown` / `rejectShutdown` with an injected `ShutdownMessenger`; `SenpiShutdownError`. |
| `errors.ts` | `SenpiTeamSpecError`. |

## CONSUMERS

- `src/tools/team/` (lead tool family) and `src/tools/control/send-shutdown.ts` (`SenpiShutdownError`, `TEAM_LEAD_SENTINEL`).
- `packages/omo-senpi` task component wires the lead poller, session-start reconcile, and member extension entry.

## CONVENTIONS

- team-core is consumed only through its subpath exports (`team-core/team-registry`, `team-state-store`, `team-mailbox`, `team-tasklist`, `types`, `config`).
- Sidecar writes are temp-sibling + rename; a missing or malformed sidecar reads as empty.
- Shutdown and messaging transports are injected ports so each protocol is testable alone.

## ANTI-PATTERNS

- `TEAM_LEAD_SENTINEL` is NEVER a spawnable member (`normalize.ts`); never route lead identity through team-core `validateSpec`.
- `<projectRoot>/.omo/teams/...` is discovery-only and NEVER a runtime-write target (`storage.ts`).
- Never decide reclaim or liveness from team-core's opencode session-liveness helper; member names come from the senpi sidecar map and task-store liveness (`messaging/reclaim.ts`).
- Never commit a delivery reservation before the envelope is observed in the recipient session.

## QA

```sh
bun test packages/senpi-task/src/team
```

Parent: [`../AGENTS.md`](../AGENTS.md); package contracts in [`../../AGENTS.md`](../../AGENTS.md).
