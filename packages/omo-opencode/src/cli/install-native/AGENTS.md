# install-native/ - `install --platform=native` pipeline

**Generated:** 2026-11-18 / c2691818a9

14 TS files / ~1.75k LOC, 7 colocated suites. Score 16 (barrel, >10 exports, >30 symbols). Distinct domain: the parent [../AGENTS.md](../AGENTS.md) narrates the user-facing `install --platform=native` flow; this file maps the modules and the load-bearing orderings underneath it.

## WHERE TO LOOK

| Task | File |
|------|------|
| Install plan + advertised commands | `plan.ts` - `resolveNativeInstallPlan(bunAvailable)` (bun preferred, npm fallback), `nativePackageSpec()` (prerelease build → `omo-ai@beta`, stable → bare `omo-ai`), `formatNativeInstallEntryCommand`, `NATIVE_SETUP_COMMAND` (`omo setup`) |
| Stale pre-rename `omo` bins | `legacy-omo-bin.ts` (`scanOmoBins`, `resolveOmoBinEnvironment`) + `repair-legacy-omo-bin.ts` (`repairLegacyOmoBins`, `removeFileCommand`) |
| The install run | `run-native-install.ts` - `runNativeInstall(deps)`; dependencies fully injectable (`isBunAvailable`, `spawn`, `environment`, `probeVersion`) |
| PATH verification | `verify-omo-command.ts` - `verifyOmoCommand`, `pathOrderFix` (the exact `export PATH=...` line when the resolved `omo` is not omo-ai's) |
| Setup offer (TUI nudge) | `offer-native-setup.ts` + its wording constants |
| Test fixtures | `omo-bin-test-fixtures.ts` |

## LOAD-BEARING (from source comments)

- Legacy-bin repair runs BEFORE the package manager: a pre-rename release owns the global `omo` name, npm refuses to overwrite it (`EEXIST`) and bun installs beside it - either way the stale bin wins until removed. Only the conflicting bin file is removed, never the whole package.
- `NativeInstallOutcome.omoBinPath` is set only when `verified` is true - onboarding never runs a bare `omo` lookup.
- The advertised entry command follows the running plugin's channel (prerelease → `@beta` tag) because `latest` rejects `--platform=native`.

## CONSUMERS

`cli-installer.ts`, `tui-installer.ts` and `features/native-edition-nudge/tui.ts` import the barrel `index.ts` - keep new surface exported from there. npm-fallback output carries `NATIVE_RECOMMENDED_RUNTIME_NOTE` (bun is the tested runtime).

## TESTS

`bun test packages/omo-opencode/src/cli/install-native` (plan, legacy-omo-bin, repair-legacy-omo-bin, run-native-install, verify-omo-command, offer-native-setup suites). Installer changes reach OpenCode: run the `opencode-qa` skill and record evidence (repo rule).
