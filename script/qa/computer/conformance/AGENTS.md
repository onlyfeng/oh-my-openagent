# script/qa/computer/conformance/ -- GUI Conformance Matrix

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Catalog-driven conformance for computer-use cells (OS x persona/toolkit): a driver replays catalog actions against fixture apps on every platform, and a report layer compares observed fields to declared oracles.

## STRUCTURE

| Module | Role |
|---|---|
| `vocabulary.ts` | Closed vocabulary: `SIGNAL_IDS`, `EngineCode`, `OS_NAMES`, persona/toolkit/targeting/mode unions, `Cell`/`CellResult`/`SignalSummary`, Zod `RUN_SCHEMA` for serialized runs |
| `declarations.ts` | `PERSONA_OS`, `TOOLKITS`, `ACTION_TARGETINGS`, `MODES`, `declare` -- what a cell declares |
| `catalog.ts` | `SESSIONS`, `cellId`, `catalogFor` over the declarations |
| `driver.ts` | `Driver`/`isDriver`; `run.ts` selects `engine` or `legacy` via `../sentinel/sentinel.ts` |
| `preflight.ts` | `preflight`, `repoPath`, `executable`, `fakeStdio` environment probes |
| `report.ts` | `buildReport`/`summaryLine`; compares observed fields to the oracle -- THE APP NEVER JUDGES ITSELF |
| `taskset/` | Model-free tasks: `tasks.ts` (`TASKS`, `MAX_REFERENCE_CELLS`), `judge.ts` outcomes |
| `fixtures/` | Seven fixture apps across OSes (GTK3, WinForms, AppKit, Electron, WebKitGTK, WKWebView, WebView2 around shared `web/`), each with `build.sh`/`build.ps1`; own README |

## COMMANDS

```sh
bun script/qa/computer/conformance/run.ts --all | --taskset <name>
bun script/qa/computer/conformance/fixtures/self-test.ts --fixture <name> --app-key <key> \
  [--sabotage <target>] [--a11y '<cmd> {pid}'] [--geometry] -- <app command>
```

## CONVENTIONS

- Fixture state is redirected per run: `CONFORMANCE_FIXTURE_DATA_DIR`, XDG homes, `CFFIXED_USER_HOME`; self-test cleans its run dir.
- Fixture reports use schema `omo-conformance-fixture/1`, written atomically (temp file + rename) after every change.
- Apps are selected by app key (`dev.omo.conformance-fixture.{native,electron,webview}`); a window is selected by TITLE, never by the app display name.
- Fixture installs on hosted images retry three times; a failure is an ENVIRONMENT failure, never a fixture failure.

## ANTI-PATTERNS

- Hosted macOS proves build/self-test only; live macOS cells need owned guests and are recorded as drops -- never as passes or failures.
- GUI fixture code runs only on hosted runners or owned guests, never a person's session.
