# script/qa/ -- QA Drivers

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

End-to-end QA drivers run against real built artifacts (packaged omo binary, senpi, codex, desktop engine) -- not unit tests. Owns `tsconfig.json`, which extends `script/tsconfig.json` with stricter flags (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noFallthroughCasesInSwitch`, `verbatimModuleSyntax`), includes `dependency-audit*.ts`, `dependency-audit/**/*.ts`, `fixtures/dependency-audit/*.ts`, and maps `typebox` from `packages/omo-senpi/node_modules` plus `@earendil-works/pi-tui` to `types/senpi-pi-tui.d.ts`.

## WHERE TO LOOK

| Driver family | Entry | What it proves |
|---|---|---|
| Telemetry privacy | `omo-native-telemetry-qa.mjs --evidence-dir <dir> --senpi-bin <path>` | End-to-end telemetry privacy. `-provider/-drive/-capture/-assertions/-evidence.mjs` own the scripted provider, isolated RPC lifecycle, capture server, privacy checks, evidence output. Progression waits for `agent_settled`; failure termination reserves a one-second grace inside the 120 s drive budget, then escalates to SIGKILL without losing the original error. Evidence: session JSONL, redacted payloads, opt-out checks, cleanup receipts |
| Eval smoke | `omo-native-eval-smoke.mjs` | Packaged binary driven through eval end to end with the source checkout hidden: `-sandbox.mjs` builds the sandbox, `-host.mjs` hides the source and owns processes, `-run.mjs` drives one run, `-assertions.mjs` checks each leg, `-stop.mjs` tears down |
| Distribution parity | `omo-native-parity-smoke.mjs` | Standalone binary vs npm launcher through the same scripted session, `omo doctor`, `omo setup --dry-run` in isolated sandboxes; fails on any difference that is not an expected distribution line (`-compare.mjs` diff core, `-provider.mjs` script source) |
| Marketplace e2e | `codex-marketplace-e2e.sh` | Real-codex install/upgrade scenario; dodges user shell wrappers (`command codex`, absolute resolved binary via `env`, shim PATH with only node); logs to `.omo/evidence/` |
| Dependency diet | `dependency-audit-capture.ts` / `dependency-audit-compare.ts` | CLI pair over the `dependency-audit/` subsystem (own AGENTS.md) |
| Member boot | `member-boot-diff.ts` / `member-boot-profile.ts` | Member boot-graph profile comparison; `compareMemberBoot` refuses to compare across platforms or Bun versions |
| Plan-item drivers | `omp-item2.ts` / `omp-item6.ts` / `omp-item9.ts` | Plan-item QA legs against `packages/senpi-task` lifecycle fixtures (admission/eval/measure; scope/denials/pool-stale/revive; cold-revive). Entry takes `--case <scenario> --out <file>`; per-leg modules (`omp-item*-*.ts/.mjs`) live beside each |
| Terminal visual QA | `web-terminal-visual-qa.mjs` | TUI evidence rendered through real xterm.js + node-pty in a browser, true color; NEVER tmux capture-pane. `xterm-live-terminal.mjs` (live capture core), `strip-ansi.mjs`, `web-terminal-redaction.mjs`; `.d.mts` companions type the mjs modules for importers |
| Kernel warmup | `python-kernel-warmup.mjs` | Python kernel warmup behavior (+ own test) |

## SUBDIRS

- `computer/` -- computer-use QA: corpus replay, cross-platform GUI conformance, hosted/owned live pools (own AGENTS.md).
- `dependency-audit/` -- capture/compare receipt subsystem and its gates (own AGENTS.md).
- `desktop/` -- live desktop-engine QA drivers per OS (own AGENTS.md).
- `fixtures/` -- dependency-audit fixtures; `dependency-audit/baseline-76e54b0-806f8e0/` records the exact audited dev artifact (copied, never rebuilt) and `graph-artifacts/meta.json` the Bun `--metafile` graph snapshot.
- `types/` -- `senpi-pi-tui.d.ts`, the QA-only typing of `@earendil-works/pi-tui` wired by this dir's `tsconfig.json`.

## ANTI-PATTERNS

- These drivers target REAL built artifacts; do not point them at a source checkout (eval smoke hides the checkout on purpose).
- Keep each `.d.mts` companion in sync when its mjs module's signatures move.
