# claude-code-plugin-loader

Discovers installed Claude Code plugins and translates each into component records (commands, skills-as-commands, agents, MCP servers, hooks) for harness adapters. Largest feature of [claude-code-compat-core](../../../AGENTS.md): 29 TS files (19 prod, ~1.2k prod LOC). Earned by score 11 — file count >20, ~47 exported symbols, own `index.ts` boundary; the sibling loaders stay covered by the package file.

## PIPELINE (`loader.ts`)

`loadAllPluginComponents(options)` / `loadAllPluginComponentsWithDeps(options, deps)`:

1. Short-circuits to empty when `OPENCODE_DISABLE_CLAUDE_CODE` or `OPENCODE_DISABLE_CLAUDE_CODE_PLUGINS` is `"true"`/`"1"`.
2. Caches per key = sorted `enabledPluginsOverride` + `anthropicProvider`; every return is a `structuredClone`, so callers may mutate. `clearPluginComponentsCache()` resets.
3. `discoverInstalledPlugins(options)` → `LoadedPlugin[]` + errors.
4. Five component loaders in `Promise.all`: `loadPluginCommands`, `loadPluginSkillsAsCommands` (skills become slash commands), `loadPluginAgents` (takes `anthropicProvider`), `loadPluginMcpServers`, `loadPluginHooksConfigs`.
5. Result `PluginComponentsResult`: `commands`, `skills`, `agents`, `mcpServers`, `hooksConfigs`, `plugins`, `errors`.

All five loaders are injectable via `PluginComponentLoadDeps` — that is the test seam.

## DISCOVERY

- `discovery.ts` is a barrel: `discoverInstalledPlugins` (`discovery-core.ts`), `resolveActualInstallPath` (`install-path-resolver.ts`), `loadPluginManifest` (`plugin-manifest.ts`).
- `installed-plugin-database.ts` reads `<pluginsBaseDir>/installed_plugins.json` (v3 entries normalized to `PluginInstallation`); a missing/corrupt DB logs and yields no plugins.
- `plugin-settings.ts` reads `~/.claude/settings.json` `enabledPlugins`; `isPluginEnabled(pluginKey, settings, override)` decides per plugin.
- `discovery-paths.ts`: `CLAUDE_PLUGINS_HOME` / `CLAUDE_SETTINGS_PATH` env overrides; tests pass `pluginsHomeOverride` / `loadPluginManifestOverride` / `enabledPluginsOverride` instead of env.
- `scope-filter.ts` `shouldLoadPluginForCwd`: `project`/`local` installs load only for a cwd inside their tilde-expanded `projectPath` (`contains-path` from shared).
- `plugin-key.ts` `derivePluginNameFromKey` strips `npm:`, `@scope/name@version`, and `file://` forms.

## HOOK LOADER DETAIL

`hook-loader.ts` stamps every command/http action with the plugin's `installPath` (`pluginRoot`) so the downstream dispatcher can set `CLAUDE_PLUGIN_ROOT` on the spawned hook process (#4458) — the association is lost once configs are merged.

## TESTS

`discovery.test.ts` (1,115 lines) is the package's largest test; each component loader has its own colocated `*.test.ts`. Run: `bun run --filter @oh-my-opencode/claude-code-compat-core test`.
