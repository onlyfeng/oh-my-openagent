# packages/omo-native

**Generated:** 2026-11-18 / c2691818a9

**Role:** Adapter - distribution package for the senpi-based omo native edition. Publishes npm `omo-ai` (v5.1.75, bin `omo`, Node >= 24, exact-pinned `@code-yeongyu/senpi` 2026.11.15-4) on the channel its version names: prerelease → `beta`, stable → `latest`. The launcher runs the pinned senpi CLI with `--extension <pkgRoot>/plugin`, where `plugin/` is the staged omo-senpi payload produced by `bun run build:omo-native` (gitignored, never committed).

## WHERE TO LOOK

| Path | Purpose |
|------|---------|
| `bin/omo.js` | Launcher entry: dispatch, doctor, setup, senpi passthrough |
| `bin/lib/` | Launcher runtime library - module map, generated bundles, daemon/thread CLI reference: [`bin/lib/AGENTS.md`](bin/lib/AGENTS.md) |
| `compile-entry.ts` | Compiled-launcher orchestration: `runCompiledLauncher`, `answerCompiledFastPath`, `compiledBannerLines`, update selection (557 LOC hotspot) |
| `remote-*.ts` | Remote CLI: session verbs (`runSession`), local parser/dispatch (`runLocalRemoteVerb`), `runRemoteVerb` / `prepareRemoteAttach`, args, bootstrap, attach presentation, renderer |
| `account-*.ts` | Account CLI; `account-show.ts` `showAccount` deliberately never contacts the account service; an uncertain refresh is not "running" when its recovery attempt is ineligible |
| `trust-*.ts` | Engine trust acquisition / generation (`writeTrustGeneration`, `verifyTrustGeneration`) / staging / retention / policy + platform helpers (macOS / Windows split) + `openssl-hash-dir.ts` |
| `keychain-helper/` | Native keychain helper (`src/` + `test-support/`) |
| `test/` | Package-contract + launcher tests; `pty-signal-qa.py` is the real-surface QA harness (boots the real chain on a pty whose session leader outlives the launcher, SIGTERMs the launcher, asserts the engine ran graceful shutdown and left no survivor) |
| `knip.workspace.jsonc` | Package-owned knip entries composed by the root config; its ignored-export annotations are pre-existing, not new defects |

- Brand: the launcher injects a `SENPI_BRAND` profile (name, `~/.omo/agent` home, `OMO_*` env prefix, wire identity, omo-ai update channel of the running version) so the pinned engine presents as omo; `--version` and every self-update spelling are answered by the launcher. See `docs/reference/omo-ai-publishing.md`.
- **Agent state lives in ONE canonical directory: `~/.omo/agent`.** `bin/lib/agent-dir.js` `canonicalAgentDir()` owns that answer; the launcher, `omo doctor`, `omo setup` and the locally installed launcher (`packages/omo-senpi/src/install/local-launcher.ts`) all resolve it from there - never by composing their own default. An explicit `OMO_CODING_AGENT_DIR` (or legacy `SENPI_CODING_AGENT_DIR` / `PI_CODING_AGENT_DIR`) still wins, and `adoptLegacyFlatState` carries pre-unification flat `~/.omo` state forward once, so unifying the location never reads as another reset.
- Engine prep: postinstall (`bin/senpi-patch.mjs`) runs `engine-prepare` (compile-safe css-tree data, RPC stream guard; the Claude Code UA floor rewrite went away with senpi#5131) and stamps the engine tree `.omo-engine-prepared`; the launcher runs `ensureEnginePrepared` before every engine start so an install whose scripts never ran (`ignore-scripts=true`, Bun's blocked postinstalls) is prepared on first launch - a failure warns with the reinstall command and never blocks (#8713).
- The agent toolkit SDK reaches agents through the staged `packages/omo-senpi/plugin/runtime/agent-toolkit-sdk/sdk.js` (env `OMO_AGENT_TOOLKIT_SDK_ROOT`), imported from an eval js cell; there is no `bin/omo-agent-toolkit.js` in this package.

## CONVENTIONS

- ESM (`"type": "module"`); local JS imports use explicit `.js` extensions; Node built-ins via `node:` prefix. Runtime requires Node >= 24; tests run under Bun.
- Paths derive from `import.meta.url` + the agent-dir helpers - never recompose home-directory defaults elsewhere.
- Setup is plan/classify/consent/write oriented; SQLite stores are read-only inputs. The setup-suggestion cache is never a blocker; `omo setup` / `omo doctor` always use live detection.
- Engine ownership/upgrade policy belongs to the engine; retired-payload diagnostics never signal running engines (restart belongs to the session owner). Pattern-based process killing is forbidden.
- Never use `spawnSync` for long-lived runtime handoffs; do not launch Windows `.cmd`/`.bat` directly as executables without a shell.
- The staged `plugin/` payload is generated and gitignored - never commit it; the engine dependency stays exact-pinned.

## COMMANDS

```bash
bun run build:omo-native                 # stage plugin payload (repo root)
bun test packages/omo-native/test        # package tests (repo root)
bunx tsc -p packages/omo-native/tsconfig.json --noEmit
node packages/omo-native/bin/omo.js --version   # entry smoke check
python3 packages/omo-native/test/pty-signal-qa.py packages/omo-native/bin/omo.js /tmp/qa.txt   # launcher signal QA (OMO_RUNTIME=node for the node chain)
```

Release mechanics + beta-channel contract: `docs/reference/omo-ai-publishing.md`. Daemon/thread CLI contracts: `docs/reference/omo-daemon.md`, `docs/reference/omo-thread.md`.
