# src/ - Device Service Core (direct files)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 22 (4 subdirs; barrel consumed by the omo-senpi `device-service` component and the desktop app). This file covers ONLY the directory's own files; subdirs are pointers.

## OVERVIEW

Vendored device-service core (`@oh-my-opencode/device-service-core`): device identity, delivery consumption, cold-wake installers, agent-dir registrant, and release acquisition over the vendored local API.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | Re-exports everything below |
| Service lifecycle | `device-service.ts`, `bootstrap.ts` | `connectDeviceService`, `connectDeliveryConsumer`, `bootstrapDeviceService` / `forgetDeviceService`; `deviceServiceAbsence` distinguishes missing installation, restart/update in progress, and an installed service that is not running |
| Install | `install.ts` | `installDeviceService`, `runCommand` |
| Keys | `hand-device-keys.ts` | `handDeviceKeys`, `HeldDeviceCredential` |
| Registrant | `registrant.ts`, `registrant-watch.ts` | `startAgentDirRegistrant`, `RegistrantEvent` |
| Account | `account-state.ts` | `classifyAccountShow`, `accountStateNotice` |
| Release | `release/` | `deviceServiceHost` (`acquire.ts`) |
| Cold wake | `cold-wake/` | `installColdWake` / `readColdWake` / `uninstallColdWake` / `refreshColdWake`; one host ensure per start (systemd oneshot with a 300 s start limit; launchd RunAtLoad without KeepAlive). Refresh upgrades only an older provisioned runtime's installed unit, preserving its agent dir, PATH and disabled state; `service-table.ts` owns only the `omo` entry. |
| Vendored wire | `local-api/` (no own AGENTS.md) | The vendored `sisyphuslabs/omo-relay-daemon` local API; `PIN.json` permits only pinned upstream files plus ported `spec-proto.ts` |
| Crypto | `strict-ed25519/` | Strict Ed25519 helpers |
| Support | `wire.ts`, `pin.ts` (`DEVICE_SERVICE_VERSION`), `timers.ts` (`realTimers`), `state-dir.ts`, `background-transport.ts`, `delegated-channel.ts`, `listing-rows.ts`, `local-only.ts`, `minimum-in-effect.ts`, `not-sent-error.ts` | Wire shapes, pin, injectable timers, state dir, transports, error semantics |
| Command routing | `engine-endpoint.ts`, `remote-attach.ts`, `refused-before-sending.ts`, `unsettled-command.ts` | Primary engine selection, attach commands, proven pre-send refusals, and `UnsettledCommandError` retaining the command id, last progress and delivery evidence |

## CONVENTIONS

- Wire properties are `readonly`; literal tuples derive union types.
- Time is injectable (`Timers`, `realTimers` default) - tests never sleep.
- Vendored provenance lives in source headers and `local-api/PIN.json`; the parity test rejects any extra file in that directory.
- Endpoint commands ask services >=0.1.11 for the primary engine; only older services may use a lone endpoint. No primary is `NoPrimaryEndpointError`, never an arbitrary first endpoint.
- `endpointCommand` returns its endpoint for reuse and accepts a total `waitMs` budget including lookup; `sessionCommand` delegates to `<device>.<key>` without attaching. Wake commands accept a wait budget too.
- Wake failures are `not_sent` only for `REFUSED_BEFORE_SENDING` codes; `closed` and `protocol_violation` can follow dispatch. Attach timeouts retain progress, and `transport_confirmed` or `engine_dispatched` counts as delivered.
- Listing starts at rows 4 (model and archived), falls back through 3, 2 and legacy only on `invalid_request`, and remembers the accepted version.

## COMMANDS

```bash
bun run --cwd packages/device-service-core typecheck   # bun check --no-script --all
bun run --cwd packages/device-service-core test        # bun test test (includes local-api conformance)
```
