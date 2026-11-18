# src/ - Device Service Core (direct files)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 22 (20 direct TS files + 4 subdirs; barrel consumed by the omo-senpi `device-service` component and the desktop app). This file covers ONLY the directory's own files; subdirs are pointers.

## OVERVIEW

Vendored device-service core (`@oh-my-opencode/device-service-core`): device identity, delivery consumption, cold-wake installers, agent-dir registrant, and release acquisition over the vendored local API.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | Re-exports everything below |
| Service lifecycle | `device-service.ts`, `bootstrap.ts` | `connectDeliveryConsumer`, `bootstrapDeviceService` / `forgetDeviceService` |
| Install | `install.ts` | `installDeviceService`, `runCommand` |
| Keys | `hand-device-keys.ts` | `handDeviceKeys`, `HeldDeviceCredential` |
| Registrant | `registrant.ts`, `registrant-watch.ts` | `startAgentDirRegistrant`, `RegistrantEvent` |
| Account | `account-state.ts` | `classifyAccountShow`, `accountStateNotice` |
| Release | `release/` | `deviceServiceHost` (`acquire.ts`) |
| Cold wake | `cold-wake/` | `installColdWake` / `readColdWake` / `uninstallColdWake`; `service-table.ts` (only the `omo` entry is owned; an unparseable table is never rewritten) |
| Vendored wire | `local-api/` (no own AGENTS.md) | The vendored `sisyphuslabs/omo-relay-daemon` local API; `PIN.json` permits only pinned upstream files plus ported `spec-proto.ts` |
| Crypto | `strict-ed25519/` | Strict Ed25519 helpers |
| Support | `wire.ts`, `pin.ts` (`DEVICE_SERVICE_VERSION`), `timers.ts` (`realTimers`), `state-dir.ts`, `background-transport.ts`, `delegated-channel.ts`, `listing-rows.ts`, `local-only.ts`, `minimum-in-effect.ts`, `not-sent-error.ts` | Wire shapes, pin, injectable timers, state dir, transports, error semantics |

## CONVENTIONS

- Wire properties are `readonly`; literal tuples derive union types.
- Time is injectable (`Timers`, `realTimers` default) - tests never sleep.
- Vendored provenance lives in source headers and `local-api/PIN.json`; the parity test rejects any extra file in that directory.

## COMMANDS

```bash
bun run --cwd packages/device-service-core typecheck   # bun check --no-script --all
bun run --cwd packages/device-service-core test        # bun test test (includes local-api conformance)
```
