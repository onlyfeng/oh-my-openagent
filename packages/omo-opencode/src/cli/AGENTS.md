# src/cli/ - CLI: install, run, doctor, config, mcp (oauth), login/logout/whoami, refresh-model-capabilities, get-local-version, version, boulder, worktree-sweep, cleanup, ulw-loop

**Generated:** 2026-07-03 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

Commander.js CLI (`sparkshell` removed 2026-07). Entry: `index.ts` (bun shebang) -> `runCli()` in `cli-program.ts`. Registration is split: `cli-program.ts` registers install, run, get-local-version, doctor, config migrate, plus `configureCleanupCommand()` (`cleanup-command.ts`), `configureRuntimeCommands()` (`runtime-commands.ts`: refresh-model-capabilities, version, boulder, worktree-sweep, ulw-loop), `mcp-oauth/` (`mcp oauth login|logout|status`) and `configureServiceAuthCommands()` (`service-auth/commands.ts`: login, logout, whoami).

## COMMANDS

| Command | Purpose | Key Logic |
|---------|---------|-----------|
| `install` | Interactive/non-interactive setup | Provider selection → config gen → plugin registration |
| `run <message>` | Non-interactive session launcher | Agent resolution (flag -> `OPENCODE_DEFAULT_AGENT` -> config -> Sisyphus); see [run/AGENTS.md](run/AGENTS.md) |
| `doctor` | Health checks | 9 OpenCode checks + 3 Codex checks; see [doctor/AGENTS.md](doctor/AGENTS.md) |
| `config migrate` | Migrate legacy OMO config into `~/.omo/omo.jsonc` | `config-migrate.ts` over `runOpenCodeStartupMigration`; `--dry-run` prints transform + backup move plan, `--json` machine-readable |
| `get-local-version` | Version detection | Installed vs npm latest |
| `version` | Print plugin version | Trivial 2-line subcommand |
| `mcp` | MCP management; nested `oauth` group | `mcp oauth login <server-name>` (PKCE), `logout`, `status` |
| `refresh-model-capabilities` | Refresh models.dev cache | Model capabilities refresh |
| `boulder` | Boulder state inspector | Format work-state + tasks from `.omo/boulder-state/` |
| `worktree-sweep` | Sweep stale git worktrees | `worktree-sweep/`: classify + delete merged/stale worktrees; excludes externally-owned roots (`~/.codex/worktrees`, `~/.codex-gui-cli-remote/worktrees`) by default |
| `cleanup` (alias `uninstall`) | Remove Codex Light state | Clean managed Codex cache/marketplace + repair project-local legacy Codex artifacts |
| `ulw-loop` | Codex ulw-loop CLI | Run the Codex LazyCodex ulw-loop CLI (`codex-ulw-loop.ts`) |
| `login` / `logout` / `whoami` | OmO service sign-in | `service-auth/`: browser loopback or `--device` code flow, OS credential store keystore, device keys; origins via `--api` / `--accounts` or `OMO_SERVICE_API_URL` / `OMO_SERVICE_ACCOUNTS_URL` (default sign-in origin `https://signin.omo.dev`) |

`install` accepts `--platform=opencode|codex|both|native` (default `opencode`). `codex`/`both` route through `install-codex/` to install the Codex CLI Light edition (also `npx lazycodex-ai install`). See `packages/omo-codex/AGENTS.md`. `native` is public: it routes through `install-native/`, which clears a stale global `omo` left by a pre-rename oh-my-openagent/oh-my-opencode release (`legacy-omo-bin.ts` scan + `repair-legacy-omo-bin.ts`, only the conflicting bin, never the package), runs the real OmO Native install (`bun add -g omo-ai@beta`, npm fallback with bun stated as recommended) through an injected spawn, verifies the `omo` PATH resolves is omo-ai (`verify-omo-command.ts`, with the exact `export PATH=...` fix when it is not), and then points at `omo setup`. Every user-facing surface advertises that installer command (`formatNativeInstallEntryCommand`), not the raw package install, because it is the only spelling that is correct on both kinds of machine. A `native-dev` choice appears (hidden from help) only when `OMO_ENABLE_NATIVE_DEV_PLATFORM=1` (legacy `OMO_ENABLE_SENPI_PLATFORM=1` still accepted) is set from a source checkout; it routes through `install-native-dev/`, which wraps the in-repo engine adapter installer. `install-ast-grep-sg.ts` provisions the `sg` binary at install time.

## STRUCTURE

```
cli/
|-- index.ts, cli-program.ts, runtime-commands.ts, cleanup-command.ts   # entry + command registration
|-- install.ts, install-command-options.ts, install-validators.ts       # routes to TUI or CLI installer; native-dev gate
|-- cli-installer.ts / tui-installer.ts, tui-install-prompts.ts        # non-interactive / @clack/prompts installers
|-- config-manager.ts + config-manager/   # facade + install utilities, see config-manager/AGENTS.md
|-- model-fallback*.ts, fallback-chain-resolution.ts, fallback-lane-policy.ts, provider-*.ts, openai-only-model-catalog.ts
|-- config-migrate.ts, cleanup.ts, codex-ulw-loop.ts, refresh-model-capabilities.ts, star-request.ts
|-- doctor/              # see doctor/AGENTS.md
|-- run/                 # see run/AGENTS.md
|-- install-codex/       # one-line re-export shims over packages/omo-codex/src/install/, see install-codex/AGENTS.md
|-- install-native/      # --platform=native: legacy omo bin repair, omo-ai install, verify, setup offer
|-- install-native-dev/  # wraps @oh-my-opencode/omo-senpi/install runSenpiInstaller
|-- service-auth/        # login/logout/whoami device + browser flows, keystore
`-- boulder/, get-local-version/, worktree-sweep/, mcp-oauth/
```

## MODEL FALLBACK SYSTEM

No single global priority. CLI install-time resolution uses per-agent fallback chains from `model-fallback-requirements.ts`.

Common patterns: Claude/OpenAI/Gemini are preferred when an agent chain includes them, `librarian` follows its fallback chain before GLM providers, `sisyphus` falls back through Kimi then GLM-5, and `hephaestus` requires OpenAI-compatible providers.

## INSTALL-NATIVE RULES

- Ownership of a stale `omo` bin comes from the installed package actually launched, never from shim text alone; a source-checkout-linked `omo` is foreign and never removed (`legacy-omo-bin.ts`).
- Setup is offered only for a verified install, through the exact verified binary path; never a bare `omo` lookup (`offer-native-setup.ts`, `run-native-install.ts`).

## HOW TO ADD A DOCTOR CHECK

See [doctor/AGENTS.md](doctor/AGENTS.md) "HOW TO ADD A CHECK".
