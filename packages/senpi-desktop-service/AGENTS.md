# senpi-desktop-service

One `senpi-desktop-engine` child per agent session over NDJSON JSON-RPC on stdio, plus the `computer.run` runtime that executes model-written JavaScript in a `node:vm` context. Behavior contract (lazy start, cancel/grace restart, heartbeat, vm budget, spawn env): `README.md`. Own file: 37 files, two subsystems (`service/`, `run/`), score 12.

## WHERE TO LOOK

| Path | Purpose |
|------|---------|
| `src/service/service.ts` | `DesktopService`: open/close serialized, calls concurrent, replay of `session.open` + armed chord + stop latch after an engine restart. |
| `src/service/rpc-client.ts` / `parse.ts` / `notifications.ts` | Request multiplexing and `$/cancel`; `parse.ts` is the trust boundary that shape-checks every engine payload; `NotificationHub` for `audit` / `stopPath.changed`. |
| `src/service/child.ts` / `timeouts.ts` / `deadline.ts` | `engineChildFactory` / `acquiringEngineChildFactory` (injectable for script engines); `START_TIMEOUT_MS`, `CLOSE_TIMEOUT_MS`, `GRACE_MS`, `HEARTBEAT_MS`. |
| `src/service/control-grants.ts` | Tracks foreground-control grants this service requested so a revoke targets only live ones (#9651). |
| `src/run/runtime.ts` | `runComputerCode`: per-run vm context, `microtaskMode: "afterEvaluate"`, signal wiring, audit projection. |
| `src/run/facade.ts` / `handles.ts` / `context.ts` | `desktop` global, `WindowHandle` / `ElementHandle` (snapshot identity, every call by id or AX ref), `guardRun` read-only enforcement. |
| `src/run/screenshot.ts` / `observe.ts` / `wait.ts` / `input-target.ts` / `engine-results.ts` | Capture display, one-request observe, predicate waits kept below the run budget, pointer options, engine result guards. |

## CONVENTIONS

- Vitest (`bun run --cwd packages/senpi-desktop-service test`); tests drive `test/fake-engine.mjs` / `fake-desktop.mjs` through node via an injected `ChildFactory`.
- Workspace deps: `-protocol` and `-engine` (README import rules also allow `-prelude`; no boundary test is checked in). Only omo-senpi `computer-use` and `-tool` import this package.
- Policy (capture budget, stop policy, coordinate frames, audit persistence) lives in the engine; the service forwards.

## ANTI-PATTERNS

- Do not treat the vm as a security boundary; it scopes globals and bounds CPU time only.
- Do not resolve a method missing from the protocol tier tables as read-only; unknown counts as `exec`.
- Do not let host tools be reached except through the injected `executeTool`; the engine never calls host tools.
- Do not share facade or signal state across runs; leaked async work must keep its own run's aborted signal.

## COMMANDS

```bash
bun run --cwd packages/senpi-desktop-service test       # vitest suite (fake engine via node)
bun run --cwd packages/senpi-desktop-service typecheck  # tsgo
bun run test:desktop                                    # all five desktop packages
```
