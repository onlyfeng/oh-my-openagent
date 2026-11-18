# src/runners/rpc-host/ — Daemon-Hosted Session Runner Internals

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 13 (118 files, ~14.6k LOC, distinct domain: host session protocol)

## OVERVIEW

Internals of `RpcHostRunner` (facade in sibling `rpc-host.ts`; the host execution mode is specified in [`../../../AGENTS.md`](../../../AGENTS.md) §EXECUTION MODES — not repeated here). This directory owns daemon provisioning, the session handle lifecycle, the wire/socket boundary, and turn reconciliation.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Daemon | `daemon.ts`, `daemon-single-flight.ts`, `daemon-contract.ts`, `launch-spec.ts`, `launch-options.ts` | `ensureTaskDaemon`/`forgetTaskDaemon`; single-flight ensure; contract constants — `TASK_HOST_SOCKET_ENV_NAMES`, `resolveTaskHostSocket` (`<agentDir>/rpc/rpc.sock` or override), `TASK_DAEMON_REQUIRED_CAPABILITIES` (incl. `retry_fallback_profile`, #12006); Zod-validated launch spec (`omo.json` cannot override the `core` launch config) |
| Session handle | `handle.ts` (coordinator) + `handle-{recovery,teardown,heartbeat,listeners,exit,port,reattach,reconciliation,session-identity,stop,waiters}.ts`, `handle-client-binding.ts` | Handle construction, type guards, recovery/reattach, bounded teardown, heartbeats, stop/identity/waiters |
| Session client | `session-client.ts` + `session-client-{connection,opening,contract}.ts`, `session-{open,close-proof,transport,role,context}.ts` | `HostSessionClient`; opening/records/model-records; role and context assembly |
| Wire + auth | `session-wire.ts`, `socket-auth.ts`, `socket-client.ts` | Typed wire refusals (`SessionHeldElsewhereError`, `HostSessionOpenError`); `RpcIdentityError`; `socket-client.ts` is the public `./rpc-socket-client` subpath |
| Sharding | `shard-socket.ts`, `shard-sidecar.ts`, `generation-view.ts` | `p-<shardKey>.sock` resolution, sidecar ownership, generation views |
| Close | `close.ts` | THE single writer ending a session this process holds no handle for: cancel, TTL, and reconciliation all route here |
| Lifecycle edges | `admission.ts`, `busy-host.ts`, `child-endpoint.ts`, `endpoint-failure.ts`, `ensure-failure.ts`, `exit-mapping.ts`, `reconcile-turn.ts` | Admission, host-busy refusal, child endpoint resolution, typed failures, exit mapping |
| Host edges | `host-{request,warmup,warm-reply,notice,command-dispatch,endpoint-port,daemon-dir}.ts`, `liveness.ts`, `live-children.ts`, `owner-claim.ts`, `own-endpoint.ts`, `open-session.ts`, `input-{command,receipt}.ts` | Host request lifecycle, warmup, liveness, ownership claims, input receipts |
| Durability | `durable-json.ts` | Durable JSON + store index |
| Tests | 50+ colocated `*.test.ts` + `*.test-support.ts` + `detached-*-probe` supports | Daemon, liveness, recovery, reattach, shard, auth, reconciliation races |

## CONVENTIONS

- Typed ports/options at process, socket, store, and host boundaries; tests supply fake transports and deterministic clocks (`bun:test`).
- Protocol decisions belong to the engine: this code never signals, replaces, or takes over a host (invariant I1).
- Frames are parsed only at `session-wire.ts`; downstream code consumes typed errors and reason codes, never raw frames.

## ANTI-PATTERNS

- NEVER signal a pid to close a session — route through `close.ts` and the session protocol.
- NEVER pass forbidden env keys (`PATH`, `*_API_KEY`, `SENPI_*`, `OMO_*`, `PI_*`) into generated session context (`session-context` tests pin this).
- Preserve one writer per JSONL file across handoff; keep teardown bounded when a daemon does not answer.
- Pinned operator-host state must not be applied to task hosts.

## COMMANDS

```bash
bun test packages/senpi-task/src/runners/         # focused
bun run --cwd packages/senpi-task typecheck       # tsgo --noEmit
```
