# thread-tools — Cross-Surface Thread QA Suite

**Generated:** 2026-11-18 / c2691818a9

**Score:** 17 (30 `.mjs` files, ~5.9k LOC; the Thread lane of the parent [`qa/AGENTS.md`](../AGENTS.md), which owns the harness/env rules — this file maps the scenarios).

## OVERVIEW

End-to-end scenarios proving thread create/send/steer works across surfaces: CLI, plugin, TUI host, and the real desktop client. `run-all.mjs` runs scenarios SEQUENTIALLY on purpose — each owns ports, sockets and pty terminals, and concurrency would make resource allocation the thing under test.

## SUITES (`run-all.mjs`)

| Suite | Harness | Engine | Notes |
|-------|---------|--------|-------|
| `gateway` (todo 16) | `lib/gateway.mjs` | RELEASED senpi from npm; `THREAD_QA_SENPI_VERSION` REQUIRED — unset/empty exits 2 before anything runs | kit installed once under `THREAD_QA_KIT_DIR` (default `<QA tmp root>/qa-thread-tools-kit`); real pty TUIs, shard hosts, `omo` CLI |
| `legacy` (task 13/14) | `lib/harness.mjs` | SOURCE checkout via `THREAD_QA_SENPI_ROOT` + `THREAD_QA_DESKTOP_ROOT` | unmet prerequisites report `SKIP` with the missing path, never a silent drop |

Flags: `--suite gateway|legacy|all`, `--out-dir`, `--with-mutant` (also runs `loop-guard.mjs --mutant` and REQUIRES it to fail), `--keep-kit`. Output lines starting `DEFECT` are product defects a scenario detected; they are counted in the summary.

## SCENARIO MAP

| Group | Files |
|-------|-------|
| Surface reachability | `cli-surface.mjs`, `plugin-surface.mjs`, `desktop-client.mjs` (real desktop provider client), `terminal-to-ui.mjs` / `desktop-to-cli.mjs` (each surface's sessions addressable from the other), `host-to-tui.mjs`, `tui-to-tui.mjs` |
| Session control | `session-control-qa.mjs`, `adopt.mjs`, `binding-author.mjs` |
| Draft + lifecycle | `draft-preservation.mjs`, `held-then-exit.mjs`, `stopped-target.mjs`, `lost-ack.mjs`, `offline-queue.mjs`, `repeated-wake.mjs` |
| Fault injection | `receiver-crash.mjs`, `sender-death.mjs` |
| Report origin | `report-origin.mjs`, `report-origin-steer.mjs`, `report-origin-auto-steer.mjs` |
| Loop safety | `loop-guard.mjs` (send-cycle check; `--mutant` disables it and must fail) |

## LIB (`lib/`)

`harness.mjs` (legacy: ONE harness, sanctioned `qa-shared/lib/*` modules, target-state assertions, self-cleaning), `gateway.mjs` (released-engine kit), `engine-version.mjs` + `qa-tmp.mjs` (`QA_TMP_ROOT`) — the two `.test.mjs` files sit at the suite root, beside `run-all.mjs`, not in `lib/` — `desktop-shell.mjs`.

## COMMANDS

```bash
THREAD_QA_SENPI_VERSION=<engine> bun packages/omo-senpi/scripts/qa/thread-tools/run-all.mjs    # gateway
bun packages/omo-senpi/scripts/qa/thread-tools/run-all.mjs --suite legacy                      # needs THREAD_QA_*_ROOT
bun packages/omo-senpi/scripts/qa/thread-tools/loop-guard.mjs --mutant                          # must FAIL
```
