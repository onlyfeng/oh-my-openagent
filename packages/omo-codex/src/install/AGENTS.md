# src/install — Codex installer engine (TypeScript source of truth)

**Score 14** (72 source modules + 54 colocated tests, ~17.7k LOC; the package's largest code mass).

## OVERVIEW

Everything the `lazycodex` install/upgrade surface does: plugin cache install, `config.toml` mutation, agent TOML linking, local marketplace snapshot, cleanup, bin/runtime wrappers. Barrel `index.ts` (re-exported by `src/index.ts`; package exposes `./install` + `./install/*`). Runs under Node via the generated bundle in `scripts/install-dist/`; this tree is Bun-strict TS. Install mechanics (targets, bin-dir precedence, Git Bash preflight, MultiAgentV2 semantics) are documented in the package AGENTS.md — not repeated here.

## STRUCTURE (file families)

| Prefix | Domain |
|--------|--------|
| `codex-cache-*` (15) | Plugin cache layout, install, bins/shims, local deps, MCP manifests, prune, runtime wrappers, bundled MCPs, legacy bins, cached marketplace manifest |
| `codex-cleanup-*` (4) | Uninstall, bin cleanup, config cleanup, deletion safety |
| `codex-config-*` (9) + `toml-*` (2) | `config.toml` feature mutators + TOML text primitives |
| `install-*` (5) | Orchestration (`install-codex.ts`), local CLI adapter, ast-grep provisioning, agent install (`install-codex-agents.ts`), test fixtures |
| `lazycodex-*` (5) | Manual update flow, CLI args, delegated commands, version stamping, bun global paths |
| `codex-marketplace*`, `codex-model-catalog`, `codex-multi-agent*` | Snapshot/manifest plumbing, model catalog, MultiAgent guards |
| `link-cached-plugin-agents` + `preserved-agent-settings` + `retired-managed-agent-purge` + `managed-agent-reasoning-defaults` | Agent TOML linking, preservation, purging |
| `git-bash*`, `codex-git-bash-*`, `codex-hook-*`, `lsp-daemon-reaper`(+`-attestation`), `codex-process`, `codex-install*` | Preflight, hook trust/targets, daemon reaping, telemetry, detection |
| `codex-agent-config.ts`, `agent-model-overrides.ts`, `managed-agent-model-defaults.ts`, `default-agent-role.ts`, `codex-package-layout.ts`, `codex-project-local-cleanup*.ts`, `codex-multi-agent-mode-config.ts`, `spawned-script-targets.ts` | Agent TOML config sources, managed model defaults, package layout, project-local cleanup, MultiAgent mode guard, allowed spawn targets |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Installer orchestration | `install-codex.ts` (`runCodexInstaller`; also `findRepoRoot`, `resolveCodexInstallerBinDir`) |
| `config.toml` updates | `codex-config-toml.ts` (`updateCodexConfig`) composing the `codex-config-*` mutators |
| TOML section/key primitives | `toml-section-editor.ts` (459 LOC; shared hotspot — every config path imports it), `toml-setting-reader.ts` |
| Cache install/link | `codex-cache-install.ts`, `codex-cache.ts`, `codex-cache-bins.ts` |
| Managed-artifact markers | `codex-cache-command-shim.ts` (`COMMAND_SHIM_MARKER`), `codex-cache-runtime-wrapper.ts` (`RUNTIME_WRAPPER_MARKER`) |
| Cleanup safety | `codex-cleanup-safety.ts`; regression homes: `codex-cleanup.test.ts`, `codex-cache-dangling-bins.ts` |
| Agent TOML linking | `link-cached-plugin-agents.ts` + preservation/purge trio; config-side: `codex-agent-config.ts`, `agent-model-overrides.ts`, `managed-agent-model-defaults.ts`, `default-agent-role.ts` |
| Manual/npx update flow | `lazycodex-manual-update.ts` (288 LOC) |
| Daemon reaping | `lsp-daemon-reaper.ts` + `lsp-daemon-reaper-attestation.ts` + attestation tests |
| Spawn safety | `spawned-script-targets.ts` — only shipped scripts may be spawned (asserted by `install-codex.test.ts`) |
| Shared type contract | `types.ts` (`isPlainRecord` = most-imported internal guard) |

## CONVENTIONS

- NO generic TOML serializer: sections, dotted keys, and multiline values are edited by text-scanning `toml-section-editor.ts`, so comments, ordering, and unknown user sections survive.
- Writes atomic (`codex-config-atomic-write.ts` temp+rename).
- Tests colocated `*.test.ts`, Bun test, real-filesystem tempdirs, explicit platform cases. Heaviest: `codex-config-toml.test.ts` (864 LOC), `codex-cleanup.test.ts` (796), `codex-cache.test.ts` (758), `codex-multi-agent-v2-config.test.ts` (607).
- Source imports omit `.ts` extensions (Bun/bundler resolution).

## ANTI-PATTERNS

- NEVER remove paths resolving to the filesystem root or outside marker/manifest/path-validated managed targets.
- NEVER treat marker-less binaries/symlinks as managed cleanup targets.
- NEVER rewrite `config.toml` wholesale; preserve user-managed sections/settings.
- Typecheck or `bun test` green is NOT QA for installer changes — package root QA mandate applies.

## COMMANDS

- `bun --cwd packages/omo-codex test` (all) / `bun test packages/omo-codex/src/install/<file>.test.ts` (focused)
- `bun --cwd packages/omo-codex run typecheck` (tsgo)
- Regenerate the published Node bundle: `bun run build:codex-install` (repo root)
