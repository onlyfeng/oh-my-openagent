# src/shared/ — Config-Dir, Discovery & Shell-Env Helpers

**Score:** 11 (11 source files, ~870 LOC; one of the highest-fan-out module families in the repo — importers across omo-opencode, boulder-state, claude-code-compat-core and more).

## OVERVIEW

Config-directory resolution, project skill/command discovery, shell detection/escaping, and product identity constants. Harness-neutral; several modules are deliberate pure re-exports keeping `@oh-my-opencode/utils` the single implementation home.

## WHERE TO LOOK

| File | Exports |
|------|---------|
| `opencode-config-dir.ts` | `getOpenCodeConfigDirs` (plural), `getOpenCodeConfigDir`, `getOpenCodeConfigPaths`, `detectExistingConfigDir`, `TAURI_APP_IDENTIFIER(_DEV)`, `isDevBuild` |
| `claude-config-dir.ts` | `getClaudeConfigDir` |
| `opencode-command-dirs.ts` | `getOpenCodeCommandDirs`, `getOpenCodeSkillDirs` |
| `project-discovery-dirs.ts` | `findProjectClaudeSkillDirs`, `findProjectAgentsSkillDirs`, `findProjectOpencodeSkillDirs`, `findProjectOpencodeCommandDirs`, `findProjectOpencodePluginConfigFiles`, `detectWorktreePath` / `clearWorktreeCache` |
| `shell-env.ts` | `detectShellType` / `ShellType`, `shellEscape`, `shellSingleQuote`, `buildEnvPrefix`, re-export `shellEscapeForDoubleQuotedCommand` |
| `plugin-identity.ts` | Identity constants derived from one `PRODUCT_IDENTITY` record: `PLUGIN_NAME`, `LEGACY_PLUGIN_NAME`, `PUBLISHED_PACKAGE_NAME`, `ACCEPTED_PACKAGE_NAMES`, `CONFIG_BASENAME`, `LEGACY_CONFIG_BASENAME`, `LOG_FILENAME`, `CACHE_DIR_NAME` |
| `skill-path-resolver.ts`, `internal-initiator-marker.ts`, `jsonc-parser.ts` | Pure re-exports from `@oh-my-opencode/utils` (jsonc module also carries the config-file detection cache) |

## CONVENTIONS

- Desktop OpenCode config dirs are detected via Tauri identifiers (`ai.opencode.desktop` / `.dev`) with `isDevBuild(version)`.
- `detectShellType` checks PowerShell profile vars BEFORE `PSModulePath` — `PSModulePath` is always set on Windows and would misdetect (step scoped to Windows for that reason).
- Identity constants must be derived from `PRODUCT_IDENTITY`, never hand-duplicated — one rename must not strand a reference.
- New helpers here ripple repo-wide (highest import fan-out in the package): keep signatures stable, prefer additive exports, and route through `./shared/*` subpath exports.

## ANTI-PATTERNS

- Do not reimplement utils here — `skill-path-resolver` / `internal-initiator-marker` re-export on purpose.
- Do not cache worktree detection outside the `detectWorktreePath`/`clearWorktreeCache` pair (the cache pair is the contract).
