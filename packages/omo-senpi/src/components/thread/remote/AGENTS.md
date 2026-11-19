# remote/ - Account-Wide Thread Half (device service + relay)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 14 (40 files, ~8.8k LOC). Parent [`../AGENTS.md`](../AGENTS.md) owns the tool contracts and the delivery table; this file maps only the remote modules.

## STRUCTURE

| Group | Files | Role |
|------|-------|------|
| Ports | `ports.ts` (device-service port; `RemoteSessionRow` / `RemoteDevice` / `RemoteListing`; `RemoteCommandOutcome` ok/refused/uncertain; `RemoteLookupOutcome`), `alias-port.ts` + `alias-registry.ts` (frozen AliasRegistry v1), `device-service-lookup.ts` | The only shapes that cross to `@oh-my-opencode/device-service-core` |
| Device wiring | `device-service.test.ts` (1.4k lines), `device-service-watch.test.ts`, `remote-watch.ts` + `watch-queue.ts`, `this-device.ts`, `default-remote.ts`, `account.ts`, `attach-pool.ts` | Watch-driven live state, queued watch processing, pooled attaches |
| Ops | `remote-ops.ts` (`remoteAvailable`, `remoteReady`, `holderDetails` / `holderTarget`, `followRemote`, `deviceOnline`), `lazy-ops.ts`, `outcome.ts` | Readiness, holder/follow resolution |
| Create/plan | `cross-device.ts` (account-wide list and aliases), `plan-create.ts`, `create-folder.ts`, `find-session.ts`, `closed-rows.ts` | Placement uses exact device id, normalized name, then a unique id prefix of at least 6 characters; closed-session reopen obtains the holder's session file instead of opening the listing's cwd. |
| Relay | `relay-transport.ts` (the other-device `DeliveryTransport`: ONE `admit_external_message` per envelope/provenance; `held_draft` retried ~30 min under the same delivery id; a `cli` sender is `sender_not_allowed`), `sender-half.test.ts`, `split-key.test.ts` | |
| Reads | `transcript-tail.ts`, `context-tail.ts`, `address-book.ts`, `session-state.ts`, `monitor-source.ts` | Bounded tails and remote address rows; cached/ended rows are resumable and inactive unless archived, and incomplete listings retain a retryable failure for a previously listed missing target. |
| Contract pins | `route-parity.test.ts`, `remote-anchor-e2e.test.ts`, `wake-gate-per-connection.test.ts`, `reply-target-enforce.test.ts`, `lookup-answers.test.ts`, `remote-preset.test.ts`, `one-resolver.test.ts`, `dialog-mirror.ts`, `testing/closing-ports.ts` | |

## CONVENTIONS

- Outcomes are data: `ok` (optionally `local` / `located`), `refused` (reason + receipt detail), `uncertain` - an uncertain command may have run and is never retried blindly.
- The alias registry schema is frozen at v1; changes need a new versioned shape, not an edit.
- Every watcher drains through `watch-queue.ts`; nothing subscribes to the device service ad hoc.
- A listing row's `path` is its working folder, not its session file. Reopen uses `session_file` or the holder's `find_session` answer; sends probe a non-live holder before choosing a current live key, wake, or refusal.
- A read-only state probe that ends uncertain maps to retryable `host_unavailable`; a remote send receipt is released when no record was sent, while an uncertain command that may have acted must not be retried blindly.
- Held-delivery re-sends run outside the originating call deadline. A queued/admitted receipt without a pending count still reports queue position 1.

## ANTI-PATTERNS

- Do not add a second transport for cross-device sends; `relay-transport.ts` is the one seam the gateway composes.
- Do not collapse `uncertain` into success or failure; surface it and let the caller decide.
- Do not admit `cli` principals through the relay (`sender_not_allowed` is pinned).

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/thread
```
