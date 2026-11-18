# device-service - Senpi Bootstrap Component

**Score:** 11 (8 files, ~1k LOC, index barrel, sidecar split, distinct domain). Bootstrap conditions (first TUI `session_start`, absolute `OMO_BIN`, never CI/task-child, `globalThis` latch, `device_service.enabled: false` opt-out) are documented in the parent [`packages/omo-senpi/AGENTS.md`](../../../../AGENTS.md) - not repeated here.

## OVERVIEW

The omo.js entry carries only the component registration; the actual bootstrap loads after first paint from the `#omo-device-service-runtime` sidecar to keep the entry bundle small.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Registration factory | `index.ts` - `createDeviceServiceComponent` (`DeviceServiceComponentOptions` extends the runtime options) |
| Lazy sidecar (ships as `extensions/omo-device-service.js`) | `runtime.ts` - `startAtSession` / `bootstrapAtStart` |
| Registrant path | `registrant-start.ts` - `startRegistrantAtStart` |
| Bootstrap worker | `runner.ts` - `runBootstrapWorker` |
| Tests | `index.test.ts`, `runtime.test.ts`, `registrant.test.ts`, `runner.test.ts` (colocated) |

## CONVENTIONS

- Process scope (CI/task-child/launcher detection) is captured in the component module, NOT in the lazy sidecar - the sidecar must stay importable without repeating environment decisions.
- The once-per-process latch lives on `globalThis` under `Symbol.for("omo.device-service.bootstrap")`, shared by every copy of the sidecar module.

## ANTI-PATTERNS

- Do not move bootstrap work back into the entry bundle; the byte budget is the reason for the split.
- Do not add a second latch key - every sidecar copy must agree on the same symbol.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/device-service
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
