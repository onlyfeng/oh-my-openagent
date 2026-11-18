# test/ — Device-Service Conformance Suite

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 10 (24 files, ~3.2k LOC — the protocol's observable contract)

## OVERVIEW

22 `bun:test` specs plus 2 shared supports, driving the package ONLY through its public surface (`../src/index.ts`). Protocol conformance with fakes throughout: fake local-API owners/worlds, generated Ed25519 credentials, per-test temp directories — no real device, no network.

## WHERE TO LOOK

| File(s) | Role |
|---------|------|
| `registrant-support.ts` | The reused harness: `useRegistrantWorld` (fake local-API owner + recorder), `ownerAt`/`stopOwner`, `registers`, `deafWatch`, `credential`, `AGENT_DIR`, `SELF` — consumed by the registrant-*, delivery, timer, and adopt-account-skew suites |
| `release-fixture.ts` | `buildRelease`/`serveRelease` (`ServedRelease`), `ReleaseKey`/`releaseKey`, pinned `RELEASE_ORIGIN`/`PIN_VERSION`/`ENGINE_VERSION` — release/bootstrap/install fixtures |
| `device-service.test.ts`, `device-service-{rows,timers}.test.ts` | Core service lifecycle, row projection, timers |
| `registrant-{device,watch,silent-watch,triggers,support}.test.ts` | Agent-dir registrant behavior against a fake owner |
| `delivery-consumer`, `attach-events`, `acquire`, `endpoint-lookup-not-sent` | Delivery consumer attach, event plumbing, acquire paths, endpoint resolution |
| `bootstrap`, `install`, `release-code-signature`, `hand-device-keys` | Bootstrap/install ceremonies, release signature verification, device key handoff |
| `delegated-channel-forget`, `local-only`, `minimum-in-effect`, `state-dir`, `wake-mark-gates` | Edge/behavioral pins |

## CONVENTIONS

- Tests import the SDK source entry (`../src/index.ts`): `connectDeviceService`, `connectDeliveryConsumer`, `DeviceService`, `DeviceCredential`, `LocalApiError`, `NotSentError`, release/bootstrap/install APIs — never internal modules.
- Fake owners/worlds come from the two shared supports; extend those before duplicating a fake.
- Temp state per test; credentials are freshly generated Ed25519 pairs, never baked in.

## COMMANDS

```bash
bun run --cwd packages/device-service-core test        # bun test test
bun run --cwd packages/device-service-core typecheck   # bun check --no-script --all -p tsconfig.json
```
