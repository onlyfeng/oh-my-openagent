# testing/ — Thread Gateway Test Harness

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 18 (14 files; test-only — no production barrel). Gateway semantics live in [`../AGENTS.md`](../AGENTS.md).

## OVERVIEW

Reusable harness + fake runtime driving real subprocess extensions (`.mjs` drivers) against the gateway: the shared seam for store, lifecycle, race, and callable-operation suites.

## WHERE TO LOOK

| File | Role |
|------|------|
| `harness.ts` | `createGatewayHarness` — `GatewayHarness`, `HarnessSession`, `SessionOptions` |
| `fake-runtime.ts` | `FakeSessionRuntime` — the fake session runtime |
| `sender-process.ts` + `job-control.ts` | Subprocess sender + job control |
| `settled.ts` | `settled()` — await helper for Bun 1.4.2 worker-thread promises |
| `extension-lifecycle.mjs` / `extension-lifecycle-driver.mjs` | Lifecycle fixture + driver |
| `extension-wake-driver.mjs` / `extension-racer.ts` | Wake and race drivers |
| `session-callable-extension.mjs` | Session-callable operations fixture |
| `store-extension.mjs` | Store extension fixture |
| `extension-sdk-smoke.mjs` | SDK smoke driver |
| `v4/` | Pinned schema-v4 worker fixture (`gateway-store-worker.mjs` + `README.md` with the pinned source commit and build recipe) |

## ANTI-PATTERNS

- Bun 1.4.2 worker-thread reply promises may settle without waking `expect(...).resolves/.rejects` — use `settled()` and assert its returned value/error.
- The v4 worker is an immutable historical compatibility fixture — never regenerate or edit it; rebuild path is documented in its README.
- Drivers are `.mjs` on purpose (plain Node subprocesses, dynamic imports) — do not convert them to bundled TS.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/thread/gateway     # consumers (via test:senpi)
bun run test:senpi                                            # full gate
```
