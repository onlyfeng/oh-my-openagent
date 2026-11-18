# claude-code-compat-core - Claude Code Compatibility Loaders (Core)

**Generated:** 2026-06-16 · **Updated:** 2026-11-18

[packages/AGENTS.md](../AGENTS.md)

## OVERVIEW

Harness-neutral Core that discovers and translates Claude Code plugins, commands, agents, skills, and MCP configs into structures that harness adapters can consume.

Npm package: `@oh-my-opencode/claude-code-compat-core` (private). 78 TS files (58 prod, ~2.8k prod LOC). Barrel-exported from `src/index.ts`; `package.json` `exports` also maps each feature dir as a subpath (`./claude-code-plugin-loader`, `./claude-code-mcp-loader`, `./claude-code-command-loader`, `./claude-code-agent-loader`, each plus a `/*` wildcard). Runtime deps: `@oh-my-opencode/model-core` + `@oh-my-opencode/utils` only. Consumed by `omo-opencode` via thin shim re-exports under [`packages/omo-opencode/src/features/claude-code-*/`](../omo-opencode/src/features/).

## LOADERS

| Feature dir | What it loads | Key entry |
|-------------|---------------|-----------|
| `claude-code-plugin-loader/` | Claude Code plugins (commands, skills, agents, MCP servers, hooks) | `loader.ts` (`loadAllPluginComponents`) - own [AGENTS.md](src/features/claude-code-plugin-loader/AGENTS.md) |
| `claude-code-mcp-loader/` | `.mcp.json` configs (user/project/local) into OpenCode SDK format | `loader.ts` |
| `claude-code-command-loader/` | Slash commands from markdown files in project and config dirs | `loader.ts` |
| `claude-code-agent-loader/` | Agent definitions from markdown and JSON configs | `loader.ts` |

## SHARED

`src/shared/` (20 modules) has 17 barrel-exported helpers for config-dir resolution, frontmatter parsing, JSONC parsing, model sanitization/normalization, path containment, and Bun shims. Key entry: `shared/index.ts`. Tests are co-located as `*.test.ts` inside each feature dir.

## NOTES

- **Env allowlist:** The MCP loader expands `${VAR}` placeholders only for variables listed in `mcp_env_allowlist` (user-only, security). Implemented in `env-expander.ts` and `configure-allowed-env-vars.ts`.
- **Scope filtering:** Both plugin and MCP loaders filter by scope (`user`/`project`/`local`). See `scope-filter.ts` in each loader dir.
- **Plugin discovery:** `discovery.ts` re-exports `discoverInstalledPlugins` / `resolveActualInstallPath` / `loadPluginManifest`; see the plugin-loader [AGENTS.md](src/features/claude-code-plugin-loader/AGENTS.md) for the pipeline. `discovery.test.ts` (1,115 lines) is the package's largest test.
- **Commands:** `bun run --filter @oh-my-opencode/claude-code-compat-core typecheck` (tsgo --noEmit) / `bun run --filter @oh-my-opencode/claude-code-compat-core test` (bun test src/*.test.ts src/**/*.test.ts).
