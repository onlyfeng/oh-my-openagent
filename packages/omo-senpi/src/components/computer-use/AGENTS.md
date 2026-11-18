# computer-use — Desktop Control Component (registration shell + lazy runtime)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 14 (51 files, ~7.2k LOC, distinct domain: the `computer` tool surface)

## OVERVIEW

Senpi component owning the `computer` / `computer_actions` tools and the `/computer` command over the `senpi-desktop-*` package family and the Rust engine binary. Registration is split from the runtime: `index.ts` registers a cheap shell at extension load; the desktop client loads through `#omo-computer-use-runtime` on first activation or use (`runtime-loader.ts`). `src/computer-use-lazy-runtime.test.ts` (package root) asserts only the shell stays in `omo.js`.

## WHERE TO LOOK

| Area | Files | Role |
|------|-------|------|
| Component | `index.ts` | `createComputerUseComponent(options)`; `COMPUTER_USE_COMPONENT_NAME`; `ComputerUnavailableReason` + `COMPUTER_UNAVAILABLE_MESSAGE` (unsupported_host / host_api_missing / disabled / invalid_settings) |
| Registration support | `registration-support.ts` | `initialComputerState`, `wireComputerControlEvents`, `hostLifecycleSource`, `skillStatusLine`, `computerFailureText` |
| Runtime | `runtime.ts`, `runtime-loader.ts` | `createComputerUseRuntime`, loader for `ComputerUseRuntimeModule` |
| Engine resolution | `engine-probe.ts`, `engine-source.ts`, `engine-status.ts`, `installed-engine.ts` (+ parity tests) | `probeComputerUseEngine`, `resolveInstalledEngine`, probe phases, packaged-vs-installed source parity |
| Permissions | `permission-events.ts`, `permission-child.test-fixture.ts` | `wireComputerPermissionEvents`; exec-policy and control confirmation (`control-confirm.ts`, #9651) |
| Foreground | `lease.ts`, `foreground-lease*.test.ts` | `createForegroundLease` / `TrackedDesktopService` |
| Full access | `full-access.ts` | `watchFullAccess` |
| Settings | `settings.ts` | `resolveOmoComputerSettings` from omo.json `computer.*` |
| Telemetry | `telemetry.ts` (+ ordering tests) | `createComputerUseTelemetry` (`ComputerUseTelemetryObservers`) |
| Fixtures | `published-parameters.fixture.json`, `screenshot-per-call.test.ts` | Published parameter contract; per-call screenshots |

## CONVENTIONS

- Dependency seams everywhere (settings, runtime loading, engine child creation, lifecycle source, telemetry observers) — tests inject fakes, never a real desktop.
- Component code reaches the host only through `../../extension/types` ports.
- Characterization tests (`control-confirm`, `foreground-lease`, `runtime-path`) pin observed behavior; extend them rather than deleting assertions.

## ANTI-PATTERNS

- Don't import the desktop service eagerly; the lazy `#omo-computer-use-runtime` split is a bundle contract.
- Don't add a second engine probe path — `engine-probe.ts` owns probe phases and refusal reasons.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/computer-use
# live QA (senpi-qa): scripts/qa/computer-use-e2e.mjs, computer-control-rpc-e2e.mjs
```
