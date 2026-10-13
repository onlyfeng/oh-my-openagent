# src/cli/doctor/ - Health Diagnostics (28 Check Files)

**Generated:** 2026-08-10 / 38d268995 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

`bunx oh-my-opencode doctor` runs parallel diagnostic checks. `getAllCheckDefinitions()` registers **9** checks; a second function `getCodexCheckDefinitions()` registers **3** Codex-only checks. System, Config, Tools and Models are aggregators that call nested check functions; the rest register standalone. Catches broken installs, config typos, missing dependencies, provider misconfigurations before they become runtime errors. 31 test files live beside the sources.

## COMMAND FLAGS

```bash
bunx oh-my-opencode doctor              # Full diagnostics
bunx oh-my-opencode doctor --status     # Compact dashboard (status only)
bunx oh-my-opencode doctor --verbose    # Deep details (model resolution traces)
bunx oh-my-opencode doctor --json       # Machine-readable output
```

## CHECK CATEGORIES

Registered by `getAllCheckDefinitions()` (9, in this order):

| Check | File | Validates |
|----------|------|-----------|
| `browser-provider` | `checks/browser-provider.ts` | Browser automation provider (literal id, name "Browser Provider", NOT in `CHECK_IDS`) |
| **SYSTEM** | `checks/system.ts` | OpenCode binary found + version >= `MIN_OPENCODE_VERSION` (`1.4.0`), plugin registered in opencode.json, loaded plugin version matches installed |
| **CONFIG** | `checks/config.ts` | JSONC validity, Zod schema passes, no unknown keys, model override syntax correct |
| **TUI_PLUGIN** | `checks/tui-plugin-config.ts` | TUI sidebar plugin entry resolvable |
| `deprecated-reasoning-keys` | `checks/deprecated-reasoning-keys.ts` | Scans `~/.omo/omo.json[c]` for deprecated `variant` / `reasoningEffort` / `thinking` / `textVerbosity` / `fallback_models` keys, reporting file + dotted path and a `config migrate` hint. Skips the `[opencode]` block and passthrough containers (`provider_options`). Registered with a literal id, NOT in `CHECK_IDS`. |
| **TOOLS** | `checks/tools.ts` | AST-Grep CLI + NAPI, comment-checker binary, LSP servers reachable, GitHub CLI auth, built-in MCP reachability |
| **MODELS** | `checks/model-resolution.ts` | models.json cache exists, per-agent fallback resolution, category overrides valid, provider availability |
| **TELEMETRY** | `checks/telemetry.ts` | Telemetry configuration state |
| **TEAM_MODE** | `checks/team-mode.ts` | Team-mode dependencies |

Registered by `getCodexCheckDefinitions()` (3): **CODEX** (critical, `checks/codex.ts`), **CODEX_COMPONENTS** (`checks/codex-components.ts`), `codex-runtime-wrapper` (literal id, `checks/codex-runtime-wrapper.ts`).

`checks/legacy-config-leftovers.ts` is not registered standalone; the Config aggregator invokes it.

`checks/latest-version.ts` is not a check either: `runner.ts` runs `gatherEditionDistTags(target)` in parallel with the checks (npm dist-tags for `oh-my-openagent` / `lazycodex-ai` via `config-manager/npm-dist-tags.ts`, bounded by `NPM_FETCH_TIMEOUT_MS`, `null` when unreachable) and `resolveLatestVersion()` picks the tag matching the installed channel (`extractChannel` from `hooks/auto-update-checker`). The result lands in `DoctorResult.latestVersion`, which the default formatter prints on the OK summary line together with the edition label and the per-edition update command (`framework/constants.ts` `EDITION_LABELS` / `UPDATE_COMMANDS`; `null` renders as `could not check`).

## SUPPORTING CHECK FILES (28 total)

All in `checks/`: `index.ts` (registration); `browser-provider.ts`; System aggregator `system.ts` + `system-binary.ts` (PATH + desktop app), `system-plugin.ts` (opencode.json entry), `system-loaded-version.ts` (cache vs npm latest); Config aggregator `config.ts` (+ `legacy-config-leftovers.ts`, invoked, not registered); Tools aggregator `tools.ts` + `dependencies.ts` (AST-Grep CLI/NAPI, comment-checker), `tools-gh.ts`, `tools-lsp.ts`, `tools-mcp.ts`; Models aggregator `model-resolution.ts` + `model-resolution-{cache,config,effective-model,variant,details,types}.ts`; `tui-plugin-config.ts`; `deprecated-reasoning-keys.ts`; `telemetry.ts`; `team-mode.ts`; `latest-version.ts` (dist-tags for the summary line, not registered); Codex: `codex.ts` (critical), `codex-components.ts`, `codex-runtime-wrapper.ts`.

## EXECUTION FLOW

```
doctor command
  -> runner.ts: checks + system/tools/codex summaries + dist-tags raced against one 30s DOCTOR_TIMEOUT_MS (timeout yields a single "Timeout" fail result)
  -> checks/index.ts: getAllCheckDefinitions() (9) + getCodexCheckDefinitions() (3)
  -> checks/latest-version.ts: gatherEditionDistTags(target) runs alongside the checks
  -> each check returns CheckResult: { name, status, message, details?, issues, duration? }
  -> formatter.ts: render to stdout (text/status/json)
  -> exit code: EXIT_CODES.SUCCESS (0) | EXIT_CODES.FAILURE (1)
```

`CheckStatus` is `pass` / `fail` / `warn` / `skip` (`framework/types.ts`). There is no `"ok"` or `"error"` status and no `detail` string field.

`framework/` holds `constants.ts`, `doctor-target.ts` (`resolveDoctorTarget`), `types.ts`, `spawn-with-timeout.ts`, and the `format-{default,status,verbose,shared}.ts` + `formatter.ts` renderers.

## KEY FILES

| File | Purpose |
|------|---------|
| `index.ts` | Barrel: `doctor`, `formatDoctorFailure`, `runDoctor`, `resolveDoctorTarget`, formatters |
| `runner.ts` | Orchestration under one 30s `DOCTOR_TIMEOUT_MS` race |
| `framework/formatter.ts` | Pretty printing: colored status, hierarchical output |
| `framework/types.ts` | `CheckDefinition`, `CheckResult`, `DoctorResult` types |

## HOW TO ADD A CHECK

1. Create `src/cli/doctor/checks/{name}.ts` exporting a check function for a `CheckDefinition`
2. Register in `checks/index.ts`: either standalone in `getAllCheckDefinitions()`, or have a category aggregator (system/config/tools/model-resolution) invoke it
3. Return a `CheckResult` (`{ name, status, message, issues }`); a thrown check is mapped to a fail result by the runner
4. Add the id/name to `framework/constants.ts` `CHECK_IDS`/`CHECK_NAMES` unless registering with a literal id

## ANTI-PATTERNS

- `codex-components.ts` keeps a deliberately duplicated bundled-component manifest; never swap in source-tree data for installed bundle data.
- `deprecated-reasoning-keys.ts`: never flag passthrough-container children as deprecated top-level keys.

## EXIT CODES

`framework/constants.ts` `EXIT_CODES` defines only:

- `SUCCESS: 0` — all checks passed
- `FAILURE: 1` — one or more failures
