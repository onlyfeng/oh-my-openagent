# senpi-desktop-service - Desktop Engine Host + `computer.run` (Desktop)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 16 (59 files, two barrels incl. `./testing`, protocol-critical runtime). Runs one `senpi-desktop-engine` child per agent session (stdio JSON-RPC) and hosts the `computer.run` JS runtime. Import direction is enforced by `scripts/desktop-package-boundaries.test.mjs`: `-service` may import `-protocol`, `-engine`, `-prelude` - nothing else imports down here except `-tool`.

## STRUCTURE

| Path | Role |
|------|------|
| `src/index.ts` | Barrel: `DesktopService`, `runComputerCode`, facades/handles, typed run/session/service errors, options/results/timeouts |
| `src/service/` | `service.ts` (`DesktopService` lifecycle), `child.ts` + `rpc-client.ts` (spawn + multiplexed JSON-RPC), `parse.ts`, `session-state.ts`, `notifications.ts` (audit/stopPath forwarding), `timeouts.ts` / `deadline.ts`, `control-grants.ts`, `global-input.ts`, `permission-preflight.ts` |
| `src/run/` | `runtime.ts` + `context.ts` (per-run context), `facade.ts` / `handles.ts` / `hold-call.ts` / `input-target.ts` / `observe.ts` / `screenshot.ts` / `wait.ts`, `engine-results.ts`, `sandbox/` (`quickjs.ts`, `worker.ts`, `limits.ts`, `protocol.ts`) |
| `test/` | `testing.ts` is the public `./testing` export (`fakeEngineFactory`, `FAKE_ENGINE_PATH`); suites cover engine, run, sandbox, control, permissions, holds, zoom, observe, display, forward-compat, launch-refusal |

## SERVICE CONTRACT

Lazy `open` → `engine.hello` ABI check (`DesktopEngineAbiMismatchError`) → `session.open` (`resumeToken` stays inside; only `resume()` sends it back). Requests multiplex by id; abort/timeout send `$/cancel`; if the engine has not answered `GRACE_MS` (750 ms) later the child is killed and restarted - `engineRestarted: true`, captures/AX refs reset, session + armed chord replayed. Heartbeat 500 ms while open; start timeout 10 s; close timeout 1.5 s. The ENGINE owns policy, byte budgets, and audit persistence - the service only forwards notifications.

## RUN CONTRACT (`runComputerCode`)

Model JS runs in a fresh `node:vm` context (no child, no worker) with globals `desktop`, `wait`, `assert`, `console`, and `tool.*` routed to the host `executeTool` - validation, permissions, and lazy activation stay in the host pipeline; the engine never calls host tools. `microtaskMode: "afterEvaluate"` bounds every synchronous stretch by the remaining budget. Read-only runs: methods tagged `exec` in the protocol tier tables throw `read-only run` BEFORE any request; a method missing from the tables counts as `exec`. Each run owns its context and aborted signal - leaked async never borrows the next run's.

## ANTI-PATTERNS

- The vm is NOT a security boundary (global scoping + CPU bounding only).
- Do not call host tools from the engine side, and do not skip `permission-preflight` on the host side.
- Do not widen a method tier without the `-protocol` tables (allowlist drift is pinned by prelude tests).

## COMMANDS

```bash
npm run typecheck   # tsgo --noEmit
npm test            # vitest --run test
```
