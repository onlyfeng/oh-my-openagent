# rpc-host - Child Sessions on the Shared Task Host

`RpcHostRunner` (parent `../rpc-host.ts`) opens a child as a SESSION of the parent tree's engine host instead of spawning a process. This directory is its machinery: 55 implementation/support files, 30 tests, 11 fixtures (~9.4k LOC incl. tests). No local barrel; the public subset is curated in [`../index.ts`](../index.ts). Earned its file by size (largest runner subtree) and by invariants that differ from process children (no pid, recorded endpoints, transport recovery). Mode selection and shard routing summary: package [`AGENTS.md`](../../../AGENTS.md) (EXECUTION MODES).

## WHERE TO LOOK

| Concern | Files |
|---------|-------|
| Daemon ensure | `daemon.ts` (`ensureTaskDaemon`; `omo daemon run` and a child-triggered ensure reach the same producer), `daemon-contract.ts` (`resolveTaskHostSocket`: override else `<agentDir>/rpc/rpc.sock`, required capabilities, protocol version), `daemon-single-flight.ts`, `owner-claim.ts`, `launch-spec.ts` (rejects group/world-writable specs and `..` extension paths), `launch-options.ts`, `host-daemon-dir.ts` |
| Shards + stores | `shard-socket.ts` (one host per parent tree, `<shardRoot>/p-<shardKey>.sock`), `shard-sidecar.ts` (informational `.meta.json`), `store-index.ts` (append-only agent-dir index of every task store that opened a child), `durable-json.ts`, `child-endpoint.ts`, `own-endpoint.ts` |
| Open + admission | `open-session.ts` (a FRESH open must land on the requested endpoint, #9722), `session-open.ts`, `admission.ts`, `busy-host.ts` (a slow host is not a dead host, omo#9067), `host-warmup.ts` (a host pays for its first session; `HOST_WARMUP_TASK_ID`), `ensure-failure.ts`, `endpoint-failure.ts` |
| Session wire | `session-transport.ts` (engine-client port + senpi adapter, `open_session` payload), `session-wire.ts` (typed records, `SessionHeldElsewhereError`), `session-client*.ts`, `session-context.ts` (`buildChildContext`: role `dag_child`/`member`/`child`), `session-role.ts` (the reader), `host-request.ts` (one connection, one command) |
| Handle | `handle.ts` (`createHostSessionHandle`, same turn semantics as `../rpc/handle.ts`), split into `handle-port.ts`, `handle-listeners.ts`, `handle-waiters.ts`, `handle-stop.ts`, `handle-teardown.ts`, `handle-heartbeat.ts`, `handle-exit.ts`, `handle-client-binding.ts` |
| Transport loss | `reattach.ts`, `reattach-port.ts` (re-ensure the RECORDED endpoint, reopen the same session path), `handle-reattach.ts`, `handle-recovery.ts`, `transport-recovery.ts` (bounded recovery, omo#9403), `transport-error.ts`, `live-children.ts` (all siblings on a lost socket learn at the first loss) |
| End + liveness | `close.ts` (single writer for closing a session this process holds no handle for), `exit-mapping.ts` (session endings mapped onto `ChildExitOutcome`), `liveness.ts`, `host-endpoint-port.ts` (`LifecycleDeps.hostEndpoint`), `host-notice.ts`, `terminal-message.ts` |
| Test support | `__fixtures__/fake-host*.ts`, `host-world*.ts`, `manual-recovery-clock.ts`; `*.test-support.ts` (detached probe/process run outside `bun:test`) |

## CONVENTIONS

- A record names its endpoint; revival and reattach open there and nowhere else. A re-derived endpoint is never used for an existing child.
- Ports, sleeps and clocks are injected (`HostAdmissionOptions`, `ChildEndpointPorts`, session ports) so retry paths are deterministic in tests.
- Notice `detail` names a socket path, never a secret.
- `OMO_RPC_SOCKET*` overrides reach the operator socket only; they never route children.

## ANTI-PATTERNS

- NEVER signal a pid from session close; the machine-wide daemon pid does not belong to the child (`close.ts`). A host-session handle's `pid` is ALWAYS undefined (`handle.ts`).
- NEVER let a per-child session dir become the daemon's (`launch-options.ts`).
- NEVER send a child whose retained session lives at a recorded endpoint to a fresh per-child fallback on ensure/open failure (`endpoint-failure.ts`).
- NEVER treat `host_busy` from a socket that still accepts as host death (`busy-host.ts`).
- NEVER deliver a first turn on a reattached (non-fresh) open; only a fresh child's first turn is sent (`session-open.ts`).

## COMMANDS

```sh
bun test packages/senpi-task/src/runners/rpc-host
bun test packages/senpi-task/src/runners/rpc-host/daemon.test.ts   # ensure/reuse/compat contract
```
