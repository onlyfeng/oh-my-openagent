# oh-my-openagent - Multi-Harness Agent OS

> **REPO MID-REFACTOR TOWARD A MULTI-HARNESS AGENT OS (OpenCode, Codex, Senpi, native). READ [`ROADMAP.md`](./ROADMAP.md) BEFORE TRUSTING ANY LAYOUT CLAIM BELOW.**

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh | **Release:** v5.1.75

## QA GATE (BINDING, EVERY CHANGE)

Any change wired into OpenCode (`packages/omo-opencode/`), Codex Light (`packages/omo-codex/`), or Senpi (`packages/omo-senpi/`, `packages/senpi-task/`) MUST run the matching `.agents/skills/{opencode-qa,codex-qa,senpi-qa}` skill and WRITE EVIDENCE TO DISK. "It typechecks" and "`bun test` is green" are NOT QA. Docker QA (`script/agent/qa-docker.sh`) is the default; on Windows (qa-docker exits 3) use the skills' local scripts.

- Evidence lives under `.omo/evidence/<YYYYMMDD>-<slug>/` (gitignored; `script/tracked-evidence-paths-audit.test.ts` fails the build on any tracked evidence path). Senpi evidence dirs resolve ONLY via `.agents/skills/senpi-qa/scripts/resolve-evidence-dir.mjs` (`.omo/evidence/omo-senpi-adapter/<slug>/`).
- Isolation is absolute: isolated XDG homes / `CODEX_HOME` / `SENPI_CODING_AGENT_DIR`; never write the real `~/.local/share/opencode`, `~/.codex`, `~/.senpi`. Prove it in the evidence.
- The PR body carries the QA summary (tested / observed / why-enough / omitted, plus one sha256 line per artifact). NO EVIDENCE FILE == NO QA == NO COMMIT == NO PUSH.

## WORKFLOW (BINDING)

User-ordered patches run through the `work-with-pr` skill: explore first, plan on disk BEFORE editing, ultra-detailed todos, fresh task-owned worktree, PR worked until MERGED. Never hand-commit to `dev`. Resolve conflicts via `smart-rebase`. Hermetic gates before live QA: `bun run test:codex` (Codex side), `tsgo --noEmit -p packages/omo-senpi/tsconfig.json` + `bun run test:senpi` (Senpi side).

## OVERVIEW

npm `oh-my-opencode`, dual-published as `oh-my-openagent` (Codex alias `lazycodex-ai`, native dist `omo-ai`). Bun 1.4.4 monorepo: 44 workspaces / 59 packages plus a Rust `crates/` workspace. The OpenCode adapter ships 11 agents, 61 hook dirs (56 active on default config / 63 with team mode / 64 with monitor), 12-38 config-gated registry tools, and a 3-tier MCP system. Two editions of one product: Ultimate (OpenCode plugin) and Light (Codex CLI, `lazycodex`).

## STRUCTURE

```
oh-my-openagent/                  # Bun monorepo, 44 workspaces / 59 packages; NO root src/
├── packages/                     # role map in packages/AGENTS.md
│   ├── omo-opencode/             # OpenCode adapter: agents(11), hooks(61 dirs), tools(15 dirs), features(24), cli, config, mcp
│   ├── omo-codex/                # Codex Light (lazycodex): vendored plugin `omo` + TS installer + telemetry
│   ├── omo-senpi/                # Senpi native adapter: 35 components (task, memory, thread, side-panel, ...)
│   ├── senpi-task/               # task engine: state, store, runners, dag/, team/, steering/, workpool/
│   ├── omo-native/               # npm omo-ai: launcher + canonicalAgentDir() (~/.omo/agent)
│   ├── *-core (21)               # harness-neutral core pkgs (utils, model-, prompts-, rules-, memory-, skills-loader-, omo-config-, ...)
│   ├── lsp-tools-mcp/ git-bash-mcp/ ast-grep-mcp/  # MCP layer; lsp-daemon + lsp-tools-mcp are Node/npm, outside workspaces
│   ├── senpi-desktop-* / omo-computer-* / device-service-core   # desktop + computer-use TS family (14 pkgs)
│   ├── shared-skills/ web/ get-worker/  # cross-harness skills; Next.js site (own lock, only @/* zone); worker
│   └── oh-my-opencode-<os>-<arch>* (12)  # generated platform launchers, never hand-edit
├── crates/                       # Rust (toolchain 1.99.0, exact-pinned): senpi-desktop-* (v2, shipping) + omo-computer-* (v3 port)
├── script/ | scripts/            # singular: Bun/TS build+publish+QA automation (337 files) | plural: Node ESM packaging gates
├── bin/                          # CLI shim, 5 aliases: oh-my-opencode, oh-my-openagent, omo-agent-toolkit, lazycodex, lazycodex-ai
├── assets/ changes/ tests/ test-support/  # generated schemas | changelog fragments (one per PR) | cross-pkg invariants + hashline live suite | shared test helpers
└── docs/ .agents/ .opencode/ .github/ .omo/  # docs; authoritative skills; frozen mirror; 47 workflows; agent workspace + evidence
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add agent / hook / tool / feature | `packages/omo-opencode/src/{agents,hooks,tools,features}/` | `hooks/AGENTS.md` owns the 61-dir roster + 5-tier composition (58 HookNames) |
| Config schema / loader / migration | `packages/omo-config-core/src/` + `omo-opencode/src/config/` | `assets/omo.schema.json` has a freshness test |
| Task engine internals | `packages/senpi-task/src/` | per-dir AGENTS.md throughout (27 tree-wide); `dag/` = 69 files ~20.8k LOC |
| Senpi components | `packages/omo-senpi/src/components/` | 35 dirs; live QA drivers in `packages/omo-senpi/scripts/qa/` |
| Desktop / computer-use | `crates/` + senpi-desktop-* + omo-computer-* | v2 ships, v3 in port; parity ledger via `script/computer-v3-guards.ts` |
| Codex installer / plugin | `packages/omo-codex/src/install/`, `plugin/components/` | 10 live components; marketplace sync automated in publish.yml |
| Build / publish / QA automation | `script/` (hub: `script/AGENTS.md`, `script/qa/`) | real-artifact drivers; terminal visual QA is browser-rendered, never tmux |
| Release mirror / channels | `script/installer/` | GitHub release to R2 mirror; channel pointer moves only after mirror success |
| Internal prompt dispatch | `packages/utils/src/prompt-async-gate/` | the ONLY sanctioned session.prompt route |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `createPluginModule()` | fn | `packages/omo-opencode/src/testing/create-plugin-module.ts` | Staged plugin init; build entry delegates here |
| `dispatchInternalPrompt()` | fn | `packages/utils/src/prompt-async-gate/` | Only sanctioned internal `session.prompt*` route |
| `canonicalAgentDir()` / `resolveAgentHome()` | fn | `packages/omo-native/bin/lib/agent-dir.js` / `packages/omo-senpi/src/components/agent-home/` | Single canonical `~/.omo/agent` resolution |
| `isHookEnabled` / `safeHook` | fn | `packages/omo-opencode/src/plugin/hooks/` | Sole owners of hook enable/compose |

## ARCHITECTURE INVARIANTS

- Internal message injection goes ONLY through `dispatchInternalPrompt()`; raw `session.prompt`/`promptAsync` outside the shared gate is forbidden, and a new internal-message route adds a row to the gate's owner regression test (the raw-prompt audit stays).
- Canonical agent order Sisyphus, Hephaestus, Prometheus, Atlas (`installAgentSortShim()`); every `Read` output carries LINE#ID hashes and `hashline_edit` rejects on mismatch.
- Two INDEPENDENT fallback systems: `model-fallback` (proactive, chat.params) vs `runtime-fallback` (reactive, session.error). No single global model priority.
- Tier-3 skill MCPs are per-session, keyed `${sessionID}:${skillName}:${serverName}`.
- Agent state has ONE canonical location `~/.omo/agent`, resolved through `canonicalAgentDir()` / `resolveAgentHome()` by every entry point; explicit `OMO_CODING_AGENT_DIR` (or legacy `SENPI_`/`PI_` variants) wins.
- `~/.omo/desktop*` belongs to the desktop app: never create, reset, migrate, prune, or walk it. When a project dir IS `$HOME`, its `.omo` is the OmO home: copy/migrate/sync code must refuse (#9727).
- Transcript journal caps are cross-adapter: tool args <=300 B, tool results <=4096 B, reasoning stored only as `[REDACTED REASONING]`.

## CONVENTIONS

- Bun 1.4.4 only (pinned in CI + Dockerfile). npm only in vendored Node pkgs (`lsp-tools-mcp`, `lsp-daemon`).
- Dev env: `script/agent/setup.sh` bootstraps; `script/agent/cleanup.sh` / `script/agent/cleanup-hook.sh` tear down; source `script/agent/qa-sandbox.sh`; copy `.env.example` for creds; wiring: `.devcontainer`, `.cursor/environment.json`, `.claude/settings.json`, `.codex/setup.sh`.
- TS strict, ESNext, bundler resolution, `bun-types` (Node-targeted pkgs add `node`); bump root `@types/node` in devDeps AND overrides together (#10646).
- Tests: `bun:test` co-located, given/when/then style (never Arrange-Act-Assert); authoring gate in `.omo/rules/test-discipline.md`; CI shape = serial quarantine first, then parallel shards.
- Preload order is load-bearing: `bunfig.toml` `preload = ["./test-env.ts", "./test-setup.ts"]`; XDG redirect stays in `test-env.ts` (import-time constants); 26 packages also preload `test-support/*.preload.ts`.
- kebab-case files; barrel `index.ts` (no business logic); 200 LOC soft cap; no catch-all `utils.ts`/`helpers.ts`/`service.ts`; no path aliases in package src (only `packages/web` uses `@/*`).
- Config: JSONC + Zod v4, snake_case keys; `mcp_env_allowlist` and `browser_automation_engine.playwright_mcp_args` are user-layer-only.
- knip is per-directory: entries go in the dir's `knip.workspace.jsonc`, composed by `script/knip-config.ts` (#13200); `bun fmt` covers JS/TS only (120 cols, no semicolons), Markdown stays hand-formatted.
- Rust: exact-pinned deps, `cargo clippy --workspace --all-targets --locked -- -D warnings`.
- Publishing is GitHub Actions only (OIDC): never `bun publish`, never bump `version` locally.
- `.agents/` is the authoritative skills/commands superset; `.opencode/` is a frozen mirror; new skills land in `.agents/` only; mirrored pairs stay byte-identical. Factories `createXXX()` for tools, hooks, agents.

## CHANGELOG FRAGMENTS (BINDING)

Never edit `CHANGELOG.md`, `changes.md`, or any `packages/<name>/changes.md` in a normal PR. A PR touching a shipped path adds exactly one `changes/unreleased/<pr-number>.md` (preamble + `<!-- changes-log: <path> -->` blocks; spec: `docs/reference/changelog-fragments.md`). The `changelog-gate` workflow refuses anything else; the release PR consumes and deletes fragments.

## ANTI-PATTERNS (BLOCKING)

- Never `as any`, `@ts-ignore`, `@ts-expect-error`; never suppress lint/type errors.
- Never write an existing file without reading it first; never empty catch blocks.
- Never delete or weaken a failing test; never bypass a red required check (`--admin`, skips, retry masking, platform/shell exclusion).
- Prompt/prose contract tests are forbidden: never assert authored prompt, SKILL.md, or AGENTS.md wording; test machine-consumed fields, shipped-copy equality, or runtime behavior only.
- Never `background_cancel(all=true)`; cancel by `taskId`.
- Never commit unless explicitly requested.
- No emojis in code/comments; no em/en dashes or AI filler ("simply", "obviously", "clearly") in generated content.
- Prometheus edits `.md` files only (`prometheus-md-only` hook).
- Never hand-edit generated output: `assets/`, platform launchers, `omo-senpi/plugin/extensions/`, `install-dist` bundles, generated crates schemas. Regenerate.
- Never PR/push against `code-yeongyu/lazycodex` (distribution mirror); source changes land in this repo.

## UNIQUE STYLES

- Singular `script/` (Bun/TS automation) vs plural `scripts/` (Node ESM gates); both at root, never merge or confuse.
- All-caps rule blocks in AGENTS.md / SKILL.md are binding contracts, not style noise.
- Skill precedence is numeric: opencode-project(6) > project(5) > opencode(4) > user(3) > config(2) > builtin=shared(1).
- `zauc-mocks-*` dirs are sort-order test hacks for `bun:test` discovery, not hooks/tools.

## COMMANDS

```bash
bun test                          # root suite (--timeout 20000)
bun run test:fast                 # CI shape locally: serial quarantine, then parallel shards
bun run test:codex                # Codex Light hermetic gate
bun run test:senpi                # Senpi adapter unit gate (live QA via senpi-qa skill)
bun run test:desktop              # senpi-desktop vitest suites + cargo pinned-deps check
bun run test:computer             # omo-computer v3 gate
bun run build && bun run build:all  # ESM bundle + .d.ts + cli + schemas; + 12 generated platform launchers
bun run build:schema              # assets/{omo,oh-my-opencode}.schema.json + dist copy
bun run build:senpi-plugin        # Senpi Pi plugin bundle (chains ast-grep-mcp)
bun run typecheck                 # tsgo --noEmit: root + script + every package (NOT tsc)
bun run lint && bun run fmt:check && bun run knip:check
cargo test --workspace --locked   # Rust suite; clippy: cargo clippy --workspace --all-targets --locked -- -D warnings
bunx oh-my-openagent install | doctor | run <msg>   # wizard / diagnostics / non-interactive session
bunx oh-my-openagent mcp oauth login <server>       # tier-3 MCP OAuth (PKCE + DCR)
```

## CI / PR POLICY (BINDING)

- PRs target `dev` (PRs into `master` are hard-blocked). Merges are MERGE COMMITS (`gh pr merge <n> --merge --delete-branch`); never squash, never rebase-merge, never `--admin`. A red required check on `dev` is a base defect: root-fix it, rerun every check.
- PRs run Linux-only unless the `ci:full-matrix` label is honored (platform-path diff, or a `Platform-reason:` line in the PR body); pushes and merge groups run the full OS matrix.
- Review claim labels `will-review` / `in-review` block merge until the claimer reviews; `stale-review` means the PR needs a new claimer. Never hand-remove someone else's claim label.
- `publish.yml` (manual dispatch): dual npm publish + `lazycodex-ai` alias + automatic Codex marketplace sync on every stable release; Darwin artifacts are signature-verified on Linux (`operator-darwin-verify.ts`) before publish.

## NOTES

- `.opencode/background-tasks.json` is git-tracked and contains `ses_*` session IDs; untrack/scrub is pending hygiene work.
- Never create `docs/guide/AGENTS.md`: case-insensitive collision with the user-facing `docs/guide/agents.md` page.
- `tests/hashline/` exercises live paid models: run only from a disposable copy in an isolated environment.
- Root `test-support/` is the shared helper layer; `packages/omo-senpi` and `packages/senpi-task` ship their own package-local `test-support/` dirs. Do not conflate.
- Windows: tests use the `bunfig.win2.parallel.windows.toml` shard; `qa-docker.sh` exits 3, so QA runs via the skills' local scripts.
