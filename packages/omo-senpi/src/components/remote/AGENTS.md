# remote — Remote Client Surface (sessions, follow, thread addressing)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 14 (41 direct files, ~8.6k LOC, distinct domain: remote device/session operations)

## OVERVIEW

The public remote-operations module (#13414, design B2): typed clients over remote sessions and devices, thread-address parsing/naming, deadline handling, and the attach/follow surfaces. The barrel is deliberately narrow; `createRemoteClient` stays internal — every caller builds its client through its sender's factory.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | Type-only `RemoteClient`/`LocalSessions`/`types.ts`; `exitCodeOf`, `REMOTE_CODES`, `RemoteCode`; five `remoteClientFor{Cli,Connector,Person,Service,Session}` factories; `canonicalizeThreadHandle`, `describeThread`, `handleOf` |
| Client core | `client.ts`, `factories.ts`, `client-message-id.ts`, `client-roster/member tests` | Client composition and sender-scoped factories |
| Sessions | `sessions.ts` (802 LOC), `sessions-local*.ts`, `session-detail.ts`, `this-device-row.ts`, `unknown-device.ts` | Session listing/detail/members; local-device rows |
| Follow | `follow.ts` (849 LOC) | Follow/wait stream handling |
| Attach | `attach.ts`, `local-attach.ts`, `local-wire.ts`, `local-sessions.ts` | Attach flows; local sessions load lazily (they open sockets) |
| Addressing | `seams.ts`, `aliases.ts`, `resolve.ts` | `canonicalizeThreadHandle` = the ONLY thread-address parser; `describeThread` = the ONLY human naming helper; device/session handles use `device`/`session` with path-like inputs excluded |
| Errors/deadlines | `codes.ts`, `deadlines.ts` | Centralized error codes → exit codes; actionable `next_action` |
| Subdirs | `port/` (device-service, lazy-remote, thread-runtime), `tui/` (`command.ts`, `run-omo.ts`), `testing/` (`fake-service.ts`) | Host ports, TUI commands, test fakes |

## CONVENTIONS

- Typed Outcome/`RemoteFailure` returns; no thrown strings. Failures carry `next_action`.
- Dependencies injected; `LocalSessions` export exists for injection, not direct construction.
- Hotspot tests over 500 LOC (`sessions.test.ts` 1109, `follow.test.ts` 906) pin race-heavy behavior — run them on any touch.

## ANTI-PATTERNS

- Don't call `createRemoteClient` outside this module; add a sender-specific factory instead.
- Don't parse thread handles ad hoc — route through `canonicalizeThreadHandle`/`handleOf`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/remote
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
