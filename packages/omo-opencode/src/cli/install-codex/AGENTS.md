# src/cli/install-codex/ - Codex Installer Adapter Shims (canonical source in omo-codex)

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Every `.ts` file here except `install-codex-test-fixtures.ts` is a ONE-LINE re-export shim: `export * from "@oh-my-opencode/omo-codex/install/<module>"`. The canonical installer lives in [`packages/omo-codex/src/install/`](../../../../omo-codex/src/install); this directory keeps the OpenCode CLI import paths (`src/cli/install-codex/*`) stable. Entry: `runCodexInstaller()` (re-exported from `install-codex.ts`), triggered by `bunx oh-my-openagent install --platform=codex` (alias `npx lazycodex-ai install`). `install-codex-test-fixtures.ts` (the only real file) builds the isolated `CODEX_HOME` fixture used by installer tests.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Change install/cache/config/uninstall behavior | `packages/omo-codex/src/install/` (this dir only re-exports) |
| Installer test isolation | `install-codex-test-fixtures.ts` (scratch `CODEX_HOME` + project dir) |
| CLI wiring | `src/cli/` commander command for `install --platform=codex` |
| Codex plugin bundle being installed | [`packages/omo-codex/plugin/`](../../../../omo-codex/plugin/AGENTS.md) (marketplace `sisyphuslabs`, plugin `omo`) |

## WHAT INSTALLING WRITES (effect summary; implementation in omo-codex)

- `~/.codex/plugins/cache/{marketplace}/{plugin}/{version}/` - built plugin cache (npm-installed, MCP manifest rewritten, atomic temp-and-rename promotion)
- `~/.codex/.tmp/marketplaces/{marketplace}/` - local marketplace snapshot
- `~/.codex/agents/*.toml` - copied agent TOMLs (roster: `explorer`, `librarian`, `metis`, `momus`, `plan`; retired `reviewer` still tracked for cleanup)
- `~/.codex/config.toml` - string-based TOML mutation (no parser dependency): marketplace block, plugin enablement, feature flags, agent configs, hook trust, MCP policies
- `~/.local/bin/` (or `~/.codex/bin`) - symlinks (POSIX) / `.cmd` shims (Windows) for component CLIs + the `omo` wrapper

## ANTI-PATTERNS

- NEVER implement installer logic in a shim; extend `packages/omo-codex/src/install/` and keep the shim.
- NEVER hand-edit `~/.codex/config.toml` semantics here: the text-manipulation layer (`toml-section-editor`) is intentional to avoid a TOML parser dependency - in the canonical package.
- Codex-side QA: run the `codex-qa` skill (`install-verify.sh`, isolated `CODEX_HOME`; the real `~/.codex` must stay untouched) - see root AGENTS.md.

## COMMANDS

```bash
bun test packages/omo-codex/src/install   # canonical installer suite (fixtures built by install-codex-test-fixtures.ts)
```

Parent: [`../../AGENTS.md`](../../AGENTS.md).
