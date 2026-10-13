# src/cli/config-manager/ - CLI Installation Utilities

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

18 source files + 12 colocated tests, no `index.ts`. Flat utility collection for the installers. The public facade is the sibling `src/cli/config-manager.ts` (imported by `cli-installer.ts`, `tui-installer.ts` and their tests); `ensureTuiPluginEntry` and `refreshOpenCodePluginSandboxes` are imported directly by the installers, and `ensureTuiPluginEntry` also by the doctor TUI check and plugin startup.

## FILE CATALOG

| File | Purpose |
|------|---------|
| `add-plugin-to-opencode-config.ts` | Register `oh-my-opencode` in the OpenCode config plugin array (string and tuple entries) |
| `add-tui-plugin-to-tui-config.ts` | `ensureTuiPluginEntry`: register the TUI plugin in the separate TUI config |
| `backup-config.ts` | `backupConfigFile`: copy to `<path>.backup-<ISO timestamp>` before a config write |
| `bun-install.ts` | Run `bun install` / `npm install` for plugin setup (`runBunInstallWithDetails`, also used by auto-update) |
| `config-context.ts` | Module-level `ConfigContext` (init lazily to OpenCode defaults, get, reset) and config paths |
| `deep-merge-record.ts` | Recursive plain-record merge; arrays/scalars replace, `undefined` skipped, `__proto__`/`constructor`/`prototype` keys dropped |
| `detect-current-config.ts` | Read existing OpenCode config, detect installed plugins |
| `ensure-config-directory-exists.ts` | Create the config dir if missing |
| `format-error-with-suggestion.ts` | Format errors with actionable suggestions |
| `generate-omo-config.ts` | Generate the `[opencode]` OMO view from install selections |
| `npm-dist-tags.ts` | `fetchNpmDistTags` (bounded by `NPM_FETCH_TIMEOUT_MS`); also feeds the doctor latest-version line |
| `opencode-binary.ts` | Detect OpenCode binary location, verify it's installed |
| `opencode-config-format.ts` | OpenCode config format constants and type guards |
| `parse-opencode-config-file.ts` | Parse opencode.json/opencode.jsonc with fallback |
| `plugin-name-with-version.ts` | Resolve `oh-my-opencode@X.Y.Z` for installation |
| `refresh-opencode-plugin-sandbox.ts` | `refreshOpenCodePluginSandboxes`: refresh OpenCode's cached plugin install |
| `version-compatibility.ts` | `checkVersionCompatibility` of an already-registered plugin entry (`extractVersionFromPluginEntry`) against the version being installed |
| `write-omo-config.ts` | Write generated config to `~/.omo/omo.jsonc` |

## USAGE PATTERN

Called through the facade by `src/cli/cli-installer.ts` / `src/cli/tui-installer.ts`, roughly:

```
1. config-context init + ensure-config-directory-exists
2. detect-current-config + version-compatibility (what's already set up, at which version)
3. opencode-binary (verify opencode installed)
4. npm-dist-tags (get latest version)
5. generate-omo-config -> backup-config -> write-omo-config
6. add-plugin-to-opencode-config + add-tui-plugin-to-tui-config
7. refresh-opencode-plugin-sandbox, bun-install
```

## NOTES

- Functions are stateless except disk I/O and the module-level context in `config-context.ts`
- JSONC files go through `jsonc-parser` / `parseJsonc`; NEVER use `JSON.parse` on JSONC files (tests use it only on plain `.json` fixtures)
- `opencode-binary.ts` searches PATH + common install locations (`.local/bin`, `~/.bun/bin`, etc.)
