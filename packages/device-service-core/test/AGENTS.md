# test/ - Device-Service Conformance Suite

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 10 (~3.2k LOC - the protocol's observable contract)

## OVERVIEW

`bun:test` suites cover the public device-service surface, injected client/timer seams, cold-wake units and local-API conformance. Shared fake owners, generated credentials and per-test temporary directories keep behavioral checks isolated; cold-wake suites emulate systemd, launchd and the operator host.

## WHERE TO LOOK

| File(s) | Role |
|---------|------|
| `registrant-support.ts` | The reused harness: `useRegistrantWorld` (fake local-API owner + recorder), `ownerAt`/`stopOwner`, `registers`, `deafWatch`, `credential`, `AGENT_DIR`, `SELF` - consumed by the registrant-*, delivery, timer, and adopt-account-skew suites |
| `release-fixture.ts` | `buildRelease`/`serveRelease` (`ServedRelease`), `ReleaseKey`/`releaseKey`, pinned `RELEASE_ORIGIN`/`PIN_VERSION`/`ENGINE_VERSION` - release/bootstrap/install fixtures |
| `device-service.test.ts`, `device-service-{rows,timers}.test.ts`, `endpoint-lookup-not-sent.test.ts`, `wake-mark-gates.test.ts` | Lifecycle, rows 4 fallback, command budgets, primary endpoint refusal and the boundary between proven not-sent and uncertain wake outcomes |
| `registrant-{device,watch,silent-watch,triggers,support}.test.ts` | Agent-dir registrant behavior against a fake owner |
| `delivery-consumer`, `attach-events`, `acquire`, `endpoint-lookup-not-sent` | Delivery consumer attach, event plumbing, acquire paths, endpoint resolution |
| `bootstrap`, `install`, `release-code-signature`, `hand-device-keys` | Bootstrap/install ceremonies, release signature verification, device key handoff |
| `delegated-channel-forget`, `local-only`, `minimum-in-effect`, `state-dir`, `wake-mark-gates` | Edge/behavioral pins |
| `cold-wake/` | Install, unit rendering, host-aware readback and older-runtime refresh; shared `cold-wake-fakes.ts` owns fake systemd/launchd hosts and temp-home cleanup |

## CONVENTIONS

- Public SDK tests use `../src/index.ts`; client/timer probes and `cold-wake/` tests also import narrow internal modules intentionally to drive routing, deadlines, unit rendering and refresh behavior.
- Fake owners/worlds come from the two shared supports; extend those before duplicating a fake.
- Temp state per test; credentials are freshly generated Ed25519 pairs, never baked in.

## COMMANDS

```bash
bun run --cwd packages/device-service-core test        # bun test test
bun run --cwd packages/device-service-core typecheck   # bun check --no-script --all -p tsconfig.json
```
