# bin/lib - omo launcher runtime library

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 12 (83 files, distinct CLI-runtime domain)

## OVERVIEW

Hand-written ESM modules behind `bin/omo.js` and the compiled entry (`runCompiledLauncher`, package-root `compile-entry.ts`); `launcher.js` `runLauncher()` dispatches every omo verb into this directory. Package-level concerns (publishing, brand, agent-dir invariant, commands, QA) live in [`../../AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Area | Modules |
|------|---------|
| Dispatch | `launcher.js` (`runLauncher`, `runAttachTui`, brand/update routing), `engine-commands.js`, `retired-command.js`, `thread-args.js` / `daemon-args.js` / `update-args.js` (argv parsing + exit-code tables) |
| Agent state | `agent-dir.js` - `canonicalAgentDir()`, `adoptLegacyFlatState()`; env precedence `OMO_CODING_AGENT_DIR` → `SENPI_CODING_AGENT_DIR` → `PI_CODING_AGENT_DIR` → `~/.omo/agent`. Never recompose the default elsewhere. |
| Setup | `setup-detect.js` + `setup-detect-cache.js` / `setup-detect-refresh.js` (fail-open suggestion cache; the detached refresh child is the only writer; `omo setup` / `omo doctor` always live-detect), `setup-stream.js` / `setup-stream-items.js` (plan/classify/consent/write stages, `SetupStaleError` stale-write protection), `setup-import.js`, `setup-opencode-{assets,providers,models}.js`, `setup-{commandcode,credentials,models,guidance,report,summary}.js`, `setup-{assets,model-choices,providers}-import.js` |
| Doctor | `doctor.js` (`runDoctor`, dist-tag/model warnings, `classifyEngineProcesses` + `reapStaleEngines`), `doctor-{migration,memory-runtime,pi-config,reflection-lock}.js`, `config-doctor.js` (dropped-key/file WARN lines), `computer-use-doctor.js`, `keychain-helper-doctor.js`, `category-coverage.js` (offline ModelRuntime classification, fail-open), `model-chains.js` |
| Daemon | `daemon.js` + `daemon-{adopt,config,operations,rollback,status,stop-budget,doctor-report}.js` - see below |
| Thread | `thread.js` + `thread-args.js` / `thread-output.js` - see below |
| Runtime / process | `bun-runtime.js` (`maybeReexecUnderBun`, `findBunBinary`, `probeBunVersion`; `BUN_MIN_VERSION` 1.4.0 - node:sqlite + worker_threads. A bun the USER chose (bun-global install or `OMO_RUNTIME=bun`) below min fails at startup with the `bun upgrade` message instead of silently switching runtimes; any other install probes and stays on node; an unchosen old bun (`resolveBunGuard`) hands off to a real node pinned `OMO_RUNTIME=node` or fails the same way (#9563); `OMO_RUNTIME=node` always stays on node), `child-process.js` (`spawnNode` / `runChild`), `bun-bin-shim.js` (POSIX bun-global direct-start prelude, fail-open, #9293; only marked legacy wrappers are restored to links #11019; a first manual upgrade from an older shim can fail - one new-version launch migrates it), `standalone-binary.js`, `engine-{boot,dependency,environment,models,prepare}.js` (`.omo-engine-prepared` stamp), `compile-safe-engine.js`, `memory-sync-boot.js`, `process-list.js`, `private-fds.js` / `private-fd-table.js` (fd handoff) |
| Update | `self-update.js` / `update-lock.js` (`omo update`; Windows held-file retry with holder report #10090), `npm-dist-tags.js`, `same-version.js`, `legacy-bun-global-migration.js`, `legacy-service-credentials.js` |
| Trust / RPC | `launch-spec-mode.js` (strips group/world write bits from `daemon-launch-spec.json`, #9208), `rpc-stream-errors.js` (engine stdio serializer guard → `invalid_stream_event`, exit 1), `engine-prepare.js` `preparePluginLaunchSpec` |
| Misc | `jsonc.js` / `jsonc-edit.js`, `sqlite-rows.js`, `package-paths.js`, `provider-map.json`, `host-status.js`, `herdr-agent-hint.js`, `auth-store.js`, `remote-attach-launch.js` |

## GENERATED BUNDLES - never hand-edit

| File | Built by | Consumed as |
|------|----------|-------------|
| `service-auth.generated.js` | `bun script/build-service-auth.ts` | `bin/omo.js` named dynamic imports: `runServiceAuthCommand`, `runSyncCommand`, `runAccountCommand`, `runRemoteAttachCommand`, `runRemoteCommand` |
| `reflection-lock.generated.js` | `bun script/build-reflection-lock.ts` | memory runtime + reflection-lock diagnostics |
| `brand-logo.generated.js` | build pipeline (`.d.ts` colocated) | banner rendering |

## omo daemon

`daemon.js`: `run|adopt|status|stop|handoff|gc|rollback-prepare`, a thin wrapper over the engine's `senpi host`. `run` ensures the operator daemon on `rpc.sock` then warms it through the thread SDK (refused/failed warm → one stderr line, exit stays 0). `status` / `gc` / `handoff` / `stop --all` / `rollback-prepare` cover every endpoint (operator daemon, per-session `p-*` task hosts, Desktop `i-*` thread hosts). `--foreground` exits 2 (the host always detaches); `--persistent` is accepted and ignored. Exit codes 2/3/4/5 live in `daemon-args.js`. Terminal (`tui`) rows render in `daemon-status.js` but never take lifecycle commands (`daemon-operations.js` `operable()`). `adopt` (`daemon-adopt.js`): locate via thread SDK, `release_session`; refusals → 4, not-found/unserved → 3, `release_failed` twice → 5; the launcher then runs the interactive launch on `--session <path>`. Contract: `docs/reference/omo-daemon.md`.

## omo thread

`thread.js`: `list|send|read|bind|unbind|rebind|bindings|report|answer|outbox|ack [--json]`, a thin wrapper over the plugin thread SDK (`plugin/runtime/thread-sdk/sdk.js`) run as `cli:<uid>`; `loadThreadSdk` hands it the client sidecar's remote surface (alias / other-device reads, local-only read kept when the sidecar lacks it). `node:sqlite` is probed with a lazy import before the SDK loads (missing → exit 4). `answer` requires `--binding` (never inferred); `send --binding` is the connector inbound path (`--mode auto|follow_up` capped by the binding, `--author-*` flags). Exit codes: 0 ok, 1 refused-as-data, 2 usage, 3 `host_unavailable`, 4 win32 or no `node:sqlite`, 5 `internal_error`. `--json` shapes are a contract: `docs/reference/omo-thread.md`.

`host-status.js`: `omo host status --all` = engine inventory + `last_activity_at` per `tui` row; the compiled entry self-marks `OMO_HOST_STATUS_RAW=1`.

## CONVENTIONS

- ESM; local imports carry explicit `.js`; built-ins via `node:`. Node >= 24 runtime; tests under Bun.
- CLI modules return exit codes and write JSON/human output via injected streams; fs/env/clock/SDK collaborators are injectable through options for deterministic tests.
- `daemon.js` and `thread.js` refuse win32 (Unix-socket endpoints), exit 4.
- Never `spawnSync` for long-lived handoffs: POSIX `execve` (argv[0], PID preserved, no resident wrapper); async fallback forwards `SIGTERM`/`SIGHUP`, waits `OMO_SIGNAL_GRACE_MS` (default 10 s), waits `SIGINT` without double-forwarding.
- Pattern-killing forbidden: `reapStaleEngines` signals ONLY explicitly named PIDs revalidated stale (interactive, PPID 1); managed/live engines are refused.
- Setup planners are read-only: merge config layers by precedence, convert representable values, report refused ones - no protocol guessing, no command substitution, no unresolved placeholders, no sockets, no model calls. SQLite inputs are read-only.
- Diagnostic/advisory paths fail open; state migration uses explicit allowlists and never overwrites existing canonical files or silently changes directory permissions.
