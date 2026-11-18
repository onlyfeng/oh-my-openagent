# src/shared/ - Shared Adapter Utilities

**Generated:** 2026-11-18 (updated)

## OVERVIEW

Cross-cutting adapter utilities used throughout the plugin. Barrel-exported from `index.ts`. Logger writes `oh-my-opencode.log` to the OS temp dir (Node's `os.tmpdir()` - `/tmp` on Linux, `%TEMP%` on Windows, etc.); rotated at 50 MB; up to 2 backups at `.1` / `.2`. Includes runtime shims for `Bun.file`, `Bun.write`, `Bun.hash`, `Bun.which`, `Bun.spawn` to support non-Bun runtimes (Electron-hosted OpenCode). Many former utility implementations are now extracted to Core packages such as `utils`, `model-core`, `tmux-core`, `telemetry-core`, and `skills-loader-core`; keep local files as stable OpenCode import shims when existing wiring depends on their paths.

## CATEGORY MAP

| Category | Files | Key Exports |
|----------|-------|-------------|
| **Model Resolution** | ~22 | `resolveModel()`, `checkModelAvailability()`, `AGENT_MODEL_REQUIREMENTS` |
| **Live server & model cache** | 6+ | `initLiveServerRoute()` (wake routing, #5569), `createConnectedProvidersCacheStore()`, `isModelAvailable()`/`fuzzyMatchModel()`, `promptWithModelSuggestionRetry()`, `getPluginSandboxRoot()` (dot-directory plugin sandboxes) |
| **Tmux Integration** | 8 + `tmux/tmux-utils/` | `createTmuxSession()`, `spawnPane()`, `closePane()`, server health; pane/session/layout/sweep primitives live in [`tmux/tmux-utils/`](tmux/tmux-utils/AGENTS.md) (33 files, runner/injectable-spawn pattern) |
| **Configuration & Paths** | 10 | `resolveOpenCodeConfigDir()`, `getDataPath()`, `parseJSONC()` |
| **Session Management** | 8 | `SessionCursor`, `trackInjectedPath()`, `SessionToolsStore` |
| **Git Worktree** | 7 | `parseGitStatusPorcelain()`, `collectGitDiffStats()`, `formatFileChanges()` |
| **Command Execution** | 7 | `executeCommand()`, `executeHookCommand()`, embedded command registry |
| **Migration** | 6 | `migrateConfigFile()`, AGENT_NAME_MAP, HOOK_NAME_MAP, MODEL_VERSION_MAP |
| **String & Tool Utils** | 6 | `toSnakeCase()`, `normalizeToolName()`, `parseFrontmatter()` |
| **Agent Configuration** | 5 | `getAgentVariant()`, `AGENT_DISPLAY_NAMES`, `AGENT_TOOL_RESTRICTIONS` |
| **OpenCode Integration** | 5 | `injectServerAuth()`, `detectExternalPlugins()`, client accessors |
| **Type Helpers** | 4 | `deepMerge()`, `DynamicTruncator`, `matchPattern()`, `isRecord()` |
| **Misc** | 8 | `log()`, `readFile()`, `extractZip()`, `downloadBinary()`, `findAvailablePort()` |

## MODEL RESOLUTION PIPELINE

```
resolveModel(input)
  1. Override: UI-selected model (primary agents only)
  2. Category default: From category config
  3. Provider fallback: AGENT_MODEL_REQUIREMENTS chains
  4. System default: Ultimate fallback
```

Key files: `model-resolver.ts` (entry), `model-resolution-pipeline.ts` (orchestration), `model-requirements.ts` (fallback chains), `model-availability.ts` (fuzzy matching).

## MIGRATION SYSTEM

Automatically transforms legacy config on load:
- `agent-names.ts`: Old agent names → new (e.g., `junior` → `sisyphus-junior`)
- `hook-names.ts`: Old hook names → new
- `model-versions.ts`: Old model IDs → current
- `agent-category.ts`: Legacy agent configs → category system

## MOST IMPORTED

| Utility | Import Count | Purpose |
|---------|-------------|---------|
| `connected-providers-cache.ts` | 29 | `createConnectedProvidersCacheStore()`, `findProviderModelMetadata()`, `ProviderModelsCache` - the highest-fan-out module in the adapter; changes here have the widest blast radius |
| `logger.ts` | 62 | `oh-my-opencode.log` in `os.tmpdir()` (50 MB cap, rotates to `.1`/`.2`) |
| `data-path.ts` | 11 | XDG storage resolution |
| `model-requirements.ts` | 11 | Agent fallback chains |
| `system-directive.ts` | 11 | System message filtering |
| `frontmatter.ts` | 10 | YAML metadata extraction |

## TEST CONVENTIONS

- Colocated `*.test.ts` (Bun): `require("bun:test")` for `describe/it/expect` plus dynamic `await import("./module")` in `beforeAll` with destructured exports - enables module-reload isolation per test file.
- Structured Given/When/Then comment markers inside tests (`//#given`, `//#when`, `//#then`); issue references cited inline (`(#5569)`).
- Filesystem/env isolation: `mkdtempSync(join(tmpdir(), "opencode-test-"))` + save/restore `process.env.XDG_CACHE_HOME` per test; cache reads neutralized via `spyOn`.
- Test-only exports are `_`-prefixed (`_setFetchImplementationForTesting`, `_setLiveClientForTesting`); module reset hooks like `__resetModelCache`.
- Hotspots: `model-availability.test.ts` (932 LOC), `model-suggestion-retry.test.ts` (748), `connected-providers-cache.test.ts` (552). `live-server-route.ts` is concurrency-critical (wake routing, probe topology, #5569) - treat edits as behavior-critical and QA through the `opencode-qa` skill.
