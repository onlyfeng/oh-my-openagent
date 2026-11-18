# src/cli/doctor/ - Health Diagnostics (28 Check Files)

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

`bunx oh-my-opencode doctor` - parallel diagnostic checks. `getAllCheckDefinitions()` registers 9 checks; `getCodexCheckDefinitions()` registers 3 more for the Codex target. Four of the nine are category aggregators (System, Config, Tools, Models). Catches broken installs, config typos, missing dependencies, and provider misconfigurations before they become runtime errors.

## COMMAND FLAGS

```bash
bunx oh-my-opencode doctor                   # problems only (default mode)
bunx oh-my-opencode doctor --status          # compact dashboard
bunx oh-my-opencode doctor --verbose         # deep diagnostics
bunx oh-my-opencode doctor --json            # machine-readable output
bunx oh-my-opencode doctor --platform=codex  # Codex/LazyCodex checks only
```

## CHECK ROSTER

`getAllCheckDefinitions()` (9):

| Check | File | Validates |
|------|------|-----------|
| `browser-provider` | `checks/browser-provider.ts` | Browser automation provider availability (literal id, not in `CHECK_IDS`) |
| **SYSTEM** (critical) | `checks/system.ts` | Aggregator: `system-binary.ts` (PATH + desktop app), `system-plugin.ts` (opencode.json entry), `system-loaded-version.ts`; version >= `MIN_OPENCODE_VERSION` (`"1.4.0"`) |
| **CONFIG** | `checks/config.ts` | JSONC validity, Zod schema, unknown keys; invokes `legacy-config-leftovers.ts` (not registered standalone) |
| **TUI_PLUGIN** | `checks/tui-plugin-config.ts` | TUI sidebar plugin entry resolvable |
| `deprecated-reasoning-keys` | `checks/deprecated-reasoning-keys.ts` | Deprecated keys in `~/.omo/omo.json[c]`, reports file + dotted path with a `config migrate` hint (literal id) |
| **TOOLS** | `checks/tools.ts` | Aggregator: `dependencies.ts` (ast-grep CLI/NAPI + comment-checker), `tools-gh.ts`, `tools-lsp.ts`, `tools-mcp.ts` |
| **MODELS** | `checks/model-resolution.ts` | Aggregator over the `model-resolution-*` helpers (cache, config, effective-model, variant, details, types) |
| **TELEMETRY** | `checks/telemetry.ts` | Telemetry configuration state |
| **TEAM_MODE** | `checks/team-mode.ts` | Team-mode dependencies |

`getCodexCheckDefinitions()` (3): **CODEX** (critical, `checks/codex.ts`), **CODEX_COMPONENTS** (`checks/codex-components.ts`), `codex-runtime-wrapper` (literal id, `checks/codex-runtime-wrapper.ts`).

`checks/latest-version.ts` is not a registered check: `runDoctor()` races `gatherEditionDistTags()` / `resolveLatestVersion()` alongside the checks; the result lands in `DoctorResult.latestVersion` (`string | null`, null when the registry lookup failed) and prints on the summary line with the edition label and per-edition update command (`framework/constants.ts` `EDITION_LABELS` / `UPDATE_COMMANDS`).

## FRAMEWORK

| File | Purpose |
|------|---------|
| `framework/constants.ts` | `CHECK_IDS`, `CHECK_NAMES`, `EXIT_CODES`, `MIN_OPENCODE_VERSION`, `EDITION_LABELS`, `UPDATE_COMMANDS` |
| `framework/types.ts` | `DoctorCheck`, `CheckResult` (`{ name, status, message, details?, issues, duration? }`), `DoctorResult`; `CheckStatus = "pass" / "fail" / "warn" / "skip"` |
| `framework/formatter.ts` | Mode dispatch to `format-default` / `format-status` / `format-verbose` (+ `format-shared`); `formatJsonOutput()` |
| `framework/doctor-target.ts` | Resolves doctor target (opencode vs codex) |
| `framework/spawn-with-timeout.ts` | Subprocess spawn with timeout |

## EXECUTION FLOW

```
index.ts doctor(options)        # mode: default | status | verbose; json flag
  → runner.ts runDoctor()       # Promise.all over all checks (+ dist-tag fetch,
    # + gatherCodexSummary() when target = codex); GLOBAL 30s budget
    # (DOCTOR_TIMEOUT_MS) - on expiry prints a Timeout result and exits
  → framework/formatter.ts      # text/status/verbose/json rendering
  → exit code: EXIT_CODES.SUCCESS (0) | EXIT_CODES.FAILURE (1)
```

The 30s timeout is a whole-run budget, not per-check; individual subprocesses time out via `framework/spawn-with-timeout.ts`.

## HOW TO ADD A CHECK

1. Create `checks/{name}.ts` exporting a check function matching `DoctorCheck`; return a `CheckResult` - no throws.
2. Register in `checks/index.ts`: standalone in `getAllCheckDefinitions()`, or invoked by a category aggregator.
3. Add id/name to `framework/constants.ts` `CHECK_IDS`/`CHECK_NAMES` unless registering with a literal id.
