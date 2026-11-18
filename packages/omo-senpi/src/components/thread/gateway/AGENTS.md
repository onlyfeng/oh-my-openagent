# gateway — thread delivery store and relay

**Score 16** (110 direct files, ~23k LOC, barrel `index.ts`; `createGatewayStore` consumed by the thread component, registration, consumer, and cross-process tests). The persistence + delivery half of the thread component ([`../AGENTS.md`](../AGENTS.md)): every `thread_send` / `thread_handoff` — tool, `omo thread` CLI, or SDK — delivers through it. Store file: `<agentDir>/gateway/gateway.sqlite`.

## STRUCTURE

| Path | Purpose |
|------|---------|
| `store.ts` | Async facade; starts `store-worker.ts` as a worker thread from `gatewayStoreWorkerUrl()` (the built `gateway-store-worker.mjs` sidecar when it exists, else source) |
| `store-worker.ts` | The ONE worker thread owning the SQLite file (WAL, `synchronous=FULL`, 0600, 5 s busy timeout); strictly serial request handling |
| `store-ops.ts` / `store-relay-ops.ts` | Delivery ops (`enqueue`, `reconcile`, `claim`, `recordOutcome`, `placeCausally`) and relay ops (receipts, bind/unbind/rebind/list, report, completion arms, outbox read/ack, answer claim/release, session incarnation); every op one `BEGIN IMMEDIATE` transaction that first moves due bindings to `expired` |
| `engine.ts` | Sender half: one transaction writes row + receipt + budgets + the target's inbox marker, then a best-effort `wake` |
| `drain.ts` | Receiver half — the only path moving a delivery out of `queued`: barrier `reconcile`, two-phase claim, `applied` only once the runtime reports the entry emitted |
| `decision.ts` | `decideDelivery`, the 15-cell target-state x delivery-mode table (rendered in `../AGENTS.md` "Delivery table") |
| `relay.ts` | `createGatewayRelay({ store, engine, endpoints, locate })`: the SDK the tools and the `omo thread` CLI share (`bind`/`unbind`/`rebind`/`bindings`/`report`/`outbox`/`ack`/`answer`/`inbound`/`settle`) |
| `registration.ts` | Receiver entry point: `sessionControlOf(pi)` feature-detects `pi.session`; registers the control endpoint, then records the session incarnation — a failed record disposes the endpoint and fails the registration |
| `bindings.ts` | Closed binding record (`schema_version` 1), `normalizeBindInput` defaults + validation, the HMAC reply-token codec, `hashArgs` |
| `transport.ts` / `pipeline.ts` / `local-transport.ts` | The ONE send path (#12879): `DeliveryTarget` (`local`/`remote`), `OutboundDelivery`, the `DeliveryTransport` port; `createDeliveryPipeline({ local, relay })` routes by target kind and nothing past it reads the device |
| `sql.ts` | SQLite abstraction over `exec()` with `gw_p`/`gw_sink` user functions — no `prepare()` (a Bun 1.4 `node:sqlite` `StatementSync` can keep the database file open after `close()`); placeholder substitution only after tokenizing (quoted text and comments survive); ordered reads apply `ORDER BY` to the outer query and restore order by row-number ordinal |
| `store-extensions.ts`, `store-extension-*.ts`, `extension-registrations.ts`, `extension-session-await.ts`, `extension-sql.ts`, `extension-statement.ts`, `extension-transaction.ts`, `extension-resolver.ts`, `extension-authorizer.ts` | Store extension seam: `registerStoreExtension` validates a `sessionCallable` declaration (`SessionCallableOp`: exported ops, `toolName` `^[a-z][a-z0-9_]{1,40}$` surfacing as `ext_<extension>_<toolName>` <= 64 chars, JSON Schema `additionalProperties: false`, `await {statusOp, expireOp, timeoutMs 1..120000}`, `wake` plain file name) and upserts it into `extension_registrations`; `extensionSessionAwait` arms `fs.watch` on wake files FIRST (no polling; a dropped event costs at most `timeoutMs`), runs `statusOp` once, re-checks per wake, and at the timeout runs `expireOp` — an unresolved expiry answers `await_unresolved` |
| `lock-wait.ts` | One busy timer per protocol; one operation waits for the write lock at most `GATEWAY_LOCK_WAIT_MAX_MS` = 30 s total, failing `gateway_lock_wait_exceeded`; callers that must not give up (inbox drain, completion write, answer release) re-arm exactly one retry after the busy timeout, off the worker queue |
| `process-identity.ts` | Claimant liveness `{ pid, process_start_time, instance_id }` (`ps -o lstart=`, recorded `epoch:<seconds>`); win32 has no `ps`, so `process_start_time` is null and liveness falls back to the pid alone — a reused pid is mistaken for the claimant, acceptable only while `omo thread` and the gateway endpoints refuse win32 |
| `store-retention.ts` | Bounded retention (<= 256 rows per table per sweep, hourly unless the last sweep hit the bound) running inside existing write transactions: expired causal roots/edges, idle rate buckets, closed bindings 30 d old with no outbox row/arm/open delivery, unreferenced `session_meta` |
| `answer-shape.ts`, `completion.ts`, `envelope.ts`, `provenance*.ts`, `author.ts`, `intake-admission.ts`, `legacy-mailbox.ts`, `constants.ts`, `paths.ts`, `types.ts`, `adapter.ts` | `answerShape(kind, text)` per-request-kind `extension_ui_response` fields; completion tracker (written at `agent_settled` with the last `agent_end`'s outcome, never at `agent_end`); sender-facts envelope + provenance rendering + `normalizeAuthor`; admission and legacy-mailbox import (invalid legacy targets reported, never imported; provenance-less legacy JSON is never attributed to the receiver); the narrow provisional senpi contract port |
| `index.ts` | Barrel re-exporting adapter, bindings, completion, constants, decision, drain, engine, paths, provenance, registration, relay, result, store, types |

Tables: `deliveries` (the single delivery authority: `queued -> admitting -> admitted -> applied`, or `refused`/`uncertain`), `receipts`, `causal_roots`/`causal_edges`, `rate_buckets`, `session_meta`, `bindings`, `outbox`, `outbox_cursors`, `completion_arms`, `extension_registrations`, `thread_creations`, `gateway_meta` (realm id + token secret). Migrations are the ordered `GATEWAY_MIGRATIONS`; v2-v4 are additive (incarnation, question/answer/outcome columns, `ui_request_kind`, `answer_state` `in_flight|delivered`, `answered_by`); `migrate` takes no write lock on a current store and re-reads the version under the lock otherwise.

## CONVENTIONS

- SQLite lives ONLY in the store worker; the session loop never runs statements, lock waits, retries, or marker writes.
- Mutations are one `BEGIN IMMEDIATE` transaction; idempotent operations persist receipts atomically with their effects (same key + different payload = `idempotency_conflict`; refusals record no receipt).
- Wake markers are ordered with commits: the inbox marker is created under the write lock, the outbox marker via temp-file + rename. A joined extension transaction gets after-commit/rollback callbacks for its marker effects.
- Persisted fields are snake_case; TS types are readonly with literal unions; timestamps UTC RFC 3339; schema versions explicit.
- Every API returns discriminated data (`kind: ok` / `kind: error` + taxonomy + next action); expected failures never throw.

## ANTI-PATTERNS

- Never auto-resend after `idempotency_uncertain` — inspect the target transcript first. Exactly-once delivery is NOT claimed; the receipt layer is the dedupe seam, so do not add engine-side correlation-id dedupe.
- Never wake a receiver before transaction COMMIT. Marker notifications may be early, never late.
- Extension SQL: no `PRAGMA`/`writable_schema`; DDL is one statement; caller text never flows into SQL; unsupported declared schema keywords are refused (registration rejects), never silently skipped; duplicate session-extension entries naming the same `op` cannot apply and are rejected.
- Don't infer row order from subquery order or sink invocation order — pass `orderBy` and sort by the ordinal.
- Session-call caller identity comes from the engine-resolved caller, never from op arguments; the public `extensionCall` channel refuses declared ops with `caller_not_allowed`; `extensionSessionAwait` refuses an empty or `UNKNOWN_CALLER` caller and args carrying `caller_session_durable_id`/`caller_created_target`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/thread/gateway   # also runs as part of the thread suite
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
bun run test:senpi                                          # full gate: build + typecheck + tests
```

Parent: [`../AGENTS.md`](../AGENTS.md) — the fourteen tools, addressing rules, budgets, the delivery table, and the QA harnesses.
