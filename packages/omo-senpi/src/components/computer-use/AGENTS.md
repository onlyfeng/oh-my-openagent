# computer-use component

Desktop computer use for the Senpi adapter (#8893): registers the `computer` tool, `computer_actions`, `/computer`, and the computer-use skill, then loads the real desktop runtime only on first use (#9113). Earned by score ~18 (37 files, ~4.2k LOC, lazy runtime alias, >30 exported symbols). User-facing registration rules and settings keys are in `../../../AGENTS.md` (Components: `computer-use`); this file maps the code.

## Boundary

- Tool definitions, `/computer` subcommands, permission parsers, prelude and skill text come from `@oh-my-opencode/senpi-desktop-tool` (`/registration` subpath in the shell). Engine location and ABI from `senpi-desktop-engine`, the JSON-RPC client from `senpi-desktop-service`, protocol types from `senpi-desktop-protocol`. This component only wires them to the Senpi host.
- `index.ts` is the registration shell that ships in `omo.js`. `runtime.ts` is the `#omo-computer-use-runtime` entry, built as `extensions/omo-computer-use.js`; `src/computer-use-lazy-runtime.test.ts` pins the split on the EMITTED artifacts.

## Anatomy

| Path | Purpose |
|------|---------|
| `index.ts` | `createComputerUseComponent`: host-API feature check (`getActiveTools`/`setActiveTools`/`executeTool`, else warn and skip), settings via `loadSenpiOmoConfig`, tool + command + skill registration, telemetry and permission wiring. Options inject platform, env, settings loader, engine child, telemetry observers and runtime loader for tests. |
| `runtime-loader.ts` | `createComputerRuntimeLoader`: imports the runtime once, wires activation into the host active-tool set; a failed import is NOT cached, so the next use retries. |
| `runtime.ts` | `createComputerUseRuntime`: `TrackedDesktopService` + `ComputerHandle` + `createComputerTool`/`createComputerActionsTool`. Creating it starts nothing; the engine starts on first activation. |
| `settings.ts` | `resolveOmoComputerSettings`: maps the `computer` block of `omo.json` onto desktop-tool settings per platform. |
| `engine-source.ts` / `installed-engine.ts` | Engine binary resolution (explicit path, runtime dir, sidecar, package prebuild, dev build, cache) and `/computer status` source text; `omoReleaseVersion` reads `OMO_PACKAGE_DIR` or the `OMO_BIN` launcher manifest. |
| `engine-status.ts` | `TrackedDesktopService` (engine state `not started`/`ready`/diagnostic), `ComputerEngineUnavailableError`, `engineErrorCode` for telemetry. |
| `engine-probe.ts` | `probeComputerUseEngine`: bounded hello/ABI handshake (5 s, detached process group on POSIX); consumed by `packages/omo-native` computer-use doctor. |
| `control-confirm.ts` | `confirmComputerControl`: foreground-control human confirm, timeout 15 s under the default run budget so an unanswered confirm reports `{ active: false }`. |
| `permission-events.ts` / `permission-status.ts` | Permission-required events: root sessions emit `omo.computer.permission_required`; task children forward through `senpi-task` child-extension events with a process-wide latch. |
| `registration-support.ts` | Skill status line, `tool_activated` name parsing, control-event wiring to a started runtime. |
| `telemetry.ts` | Activation, backend and engine-error observers over `../telemetry/omo-native-computer-use`. |

## Tests

Colocated. `engine-source-parity.test.ts` and `installed-source-parity.test.ts` assert `/computer status` and `omo doctor` (`packages/omo-native/computer-use-doctor-runtime`) agree on engine location and failure text without downloading. `index.*.test.ts` split by concern (passive status, unsupported host, telemetry). `*.test-support.ts` / `*.test-fixture.ts` are helpers, not suites.

## Anti-patterns

- Never value-import `runtime.ts`, `senpi-desktop-service`, or the engine locator from the shell path (type-only imports are erased and fine); only `runtime-loader.ts` reaches the runtime, through the alias.
- Never inline prelude or skill text; it is read from the staged `assets.generated.json`.
- Never let `/computer status` and the doctor compute engine location separately; extend `engine-source.ts` and keep both parity tests green.
- Never start the engine at registration; tool_search must index the tool with nothing running.

## Commands

```bash
bun test packages/omo-senpi/src/components/computer-use
SENPI_BIN="$(command -v senpi)" node packages/omo-senpi/scripts/qa/computer-use-e2e.mjs   # live, fake engine backend
```
