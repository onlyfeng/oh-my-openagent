# script/qa/desktop/ -- Desktop-Engine Live QA

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Live scenario QA for `senpi-desktop-engine` per OS: one runner per platform drives scenarios, an INDEPENDENT observer reads every desktop fact, and teardown receipts close each run.

## WHERE TO LOOK

| Runner | Environment | Contract |
|---|---|---|
| `windows.ts` | windows-latest interactive desktop (`.github/workflows/desktop-windows-qa.yml`) | `--all --json [--engine <exe>] [--out <file.jsonl>]` or `--scenario <name> [--sabotage invalid-chord]`; one JSONL line per scenario `{scenario, pass, facts, observer: {before, after}}`; exits 0 iff every scenario passed AND teardown left no process or temp dir behind |
| `macos.ts` | inside an unlocked Aqua session with Accessibility + Screen Recording granted to the launcher; owns and terminates TextEdit between scenarios | `SENPI_BIN=<path> bun script/qa/desktop/macos.ts --all --json`; `--self-test` checks isolation and launch wiring only, never touches the console or TCC |
| `linux.ts` | ON the Debian-family host under test: X11 under Xvfb + xfwm4, Wayland on `sway --headless` (`desktop-linux-qa.yml` runs `--all` only) | `[--provision] [--workdir] [--engine BIN] [--fake-eis BIN] [--scenario ...] [--sabotage skip-optin]`; engine defaults to `locateDesktopEngine()` (`@oh-my-opencode/senpi-desktop-engine`, repo `target/release/...`) and must pass `engine.hello` ABI handshake before any scenario |
| `gnome.ts` / `hyprland.ts` / `kwin.ts` | inside a live GNOME Wayland / Hyprland / Plasma Wayland (KWin) session on the host under test | compositor-specific window-targeted pointer-input QA (`--json [--engine BIN]`); each provisions nothing, reads facts via independent observers (`gnome-extensions`/`busctl`/`gdbus`, `hyprctl -j`, `kdotool`), and none is part of the hosted sway job |

## STRUCTURE

- `windows/` -- `engine.ts` RPC engine; `scenario-kit.ts` + `scenarios-*.ts` (cursor, pointer, pointer-scroll, scroll, delivery, guard, read); independent `observer.ps1`/`observer.ts`; pointer/wheel/scroll/WPF host scripts; `process-identity.ts`; `expect-sabotage.ts`; `until.ts`.
- `macos/` -- `agent.ts` (drives the engine agent), `scenario.ts` + `scenarios-{input,safety}.ts`, `tcc.ts` (permission handling), `observer.ts`, `provider.ts`, `fixtures.ts`.
- `linux/` -- `engine.ts`, `x11*.ts` / `wayland*.ts` session machinery, `fake-eis/` (Rust fake input-method server, own Cargo project), `provision.ts`, `procs.ts`, `scenario.ts`, `expect-sabotage.ts`.

## CONVENTIONS

- Facts are always read by an independent observer (PowerShell on Windows, toolkit/shell reads elsewhere), never by the code under test.
- Sabotage modes (`--sabotage invalid-chord`, `skip-optin`) are first-class scenario inputs exercising refusal paths.

## ANTI-PATTERNS

- A run whose scenarios all passed but whose teardown left a process or temp dir behind FAILS; do not widen the exit contract.
