# src/shared/ — Shared Adapter Utilities

**Generated:** 2026-10-13 | **Score:** 25 (138 implementation + 96 test files flat, 7 subdirs, 89-line barrel)

## OVERVIEW

Cross-cutting adapter utilities used throughout the plugin. Barrel-exported from `index.ts`. Logger writes `oh-my-opencode.log` to the OS temp dir (Node's `os.tmpdir()`: `/tmp` on Linux, `%TEMP%` on Windows, etc.); rotated at 50 MB; up to 2 backups at `.1` / `.2`. Runtime shims `bun-file-shim.ts` (`bunFile`, `bunWrite`), `bun-spawn-shim.ts`, `bun-which-shim.ts` re-export `@oh-my-opencode/utils/runtime` so non-Bun runtimes (Electron-hosted OpenCode) work; `opencode-adapter-bun-runtime-audit.test.ts` and `dist-bundle-bun-globals.test.ts` fail the suite on raw `Bun.*` globals. Many former utility implementations are now extracted to Core packages such as `utils`, `model-core`, `tmux-core`, `telemetry-core`, and `skills-loader-core`; keep local files as stable OpenCode import shims when existing wiring depends on their paths.

## CATEGORY MAP

| Category | Files | Key Exports |
|----------|-------|-------------|
| **Model Resolution** | ~22 | `resolveModel()`, `checkModelAvailability()`, `AGENT_MODEL_REQUIREMENTS` |
| **Tmux Integration** | `tmux/` (40 files, ~2.6k LOC) | Adapts `@oh-my-opencode/tmux-core`: `runner.ts` (`runTmuxCommand`), `cmux-detect.ts`, types/constants; pane/session/layout/sweep primitives live in [`tmux/tmux-utils/`](tmux/tmux-utils/AGENTS.md) (runner/injectable-spawn pattern) |
| **Configuration & Paths** | 10 | `resolveOpenCodeConfigDir()`, `getDataPath()`, `parseJSONC()` |
| **Session Management** | 8 | `SessionCursor`, `trackInjectedPath()`, `SessionToolsStore` |
| **Git Worktree** | `git-worktree/` | `parseGitStatusPorcelain()`, `collectGitDiffStats()`, `formatFileChanges()` (re-exported from `@oh-my-opencode/utils`) |
| **Command Execution** | `command-executor/` | `executeCommand()`, `executeHookCommand()` (utils shims) |
| **Migration** | `migration/` | `migrateConfigFile()`, AGENT_NAME_MAP, HOOK_NAME_MAP, MODEL_VERSION_MAP, migrations sidecar |
| **Model Capabilities** | `model-capabilities/` | `getModelCapabilities()` over the bundled snapshot + connected-provider cache (model-core types) |
| **Zip Listing / Mocks** | `zip-entry-listing/`, `zauc-mocks-migrate-legacy-plugin/` | Archive entry listing shims; sort-order `mock.module()` owner for legacy-plugin migration tests |
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

Key files: `model-resolver.ts` (entry), `model-resolution-pipeline.ts` (orchestration), `model-requirements.ts` (fallback chains), `model-availability.ts` (fuzzy matching, largest implementation at ~290 LOC), `model-string-parser.ts` (`"model variant"` strings; consumed by `tools/delegate-task/`).

## MIGRATION SYSTEM

Automatically transforms legacy config on load. Every file except `agent-category.ts` is a re-export of `@oh-my-opencode/utils/migration/*`; `agent-category.ts` binds the utils engine to `DEFAULT_CATEGORIES` from `tools/delegate-task/constants`:
- `agent-names.ts`: Old agent names → new (e.g., `junior` → `sisyphus-junior`)
- `hook-names.ts`: Old hook names → new
- `model-versions.ts`: Old model IDs → current
- `agent-category.ts`: Legacy agent configs → category system
- `migrations-sidecar.ts`: applied-migration sidecar bookkeeping

## MOST IMPORTED

Counts are non-test files under `packages/omo-opencode/src` importing the module by path (barrel imports not counted).

| Utility | Importing files | Purpose |
|---------|-------------|---------|
| `logger.ts` | ~190 | `oh-my-opencode.log` in `os.tmpdir()` (50 MB cap, rotates to `.1`/`.2`) |
| `model-requirements.ts` | ~35 | Agent fallback chains |
| `data-path.ts` | ~13 | XDG storage resolution |
| `system-directive.ts` | ~10 | System message filtering |
| `prompt-async-gate.ts` | - | The ONLY file allowed to call raw `session.prompt*`; route audit in `prompt-async-route-audit.test.ts` |

Test hotspots over 500 lines: `model-availability.test.ts` (993), `model-suggestion-retry.test.ts` (744), `external-plugin-detector.test.ts`, `connected-providers-cache.test.ts`, `live-server-route.test.ts`, `prompt-async-route-audit.test.ts`.
