# codex-qa — Isolated Codex Light QA Skill

**Generated:** 2026-11-18 / c2691818a9 | Score: 14 (distinct QA domain: 16 files, real harness scripts, 3.7k-LOC E2E hotspot)

## OVERVIEW

QA skill that drives the REAL `codex app-server` against an isolated `CODEX_HOME` + a LOCAL mock model (no real API call) to prove the omo Codex plugin (`packages/omo-codex/`) works — nothing else here is more than supporting docs for that flow.

## WHERE TO LOOK

| Need | Location |
|------|----------|
| Skill entry / golden rules / router table | `SKILL.md` |
| Live app-server hook proof (`hook/started` + `hook/completed`) | `scripts/app-server-drive.sh` (`--self-test` = no-plugin driver check, `--plugin` = full plugin proof) |
| Isolated install + `config.toml` landing | `scripts/install-verify.sh` |
| Deterministic per-component hook pin (no codex spawned) | `scripts/hook-unit-probe.sh` |
| TUI smoke under tmux | `scripts/tui-smoke.sh` |
| Shared harness (isolated home factory `cqa_mk_isolated_home`, mock model `cqa_start_mock`, shasum guard) | `scripts/lib/common.sh` |
| Mock model server (Responses SSE) | `scripts/lib/mock-model.mjs` |
| App-server JSON-RPC client | `scripts/lib/app-server-client.mjs` (exports `parseExpectedHooks`, `summarizeRun`; test: `app-server-client.test.js`) |
| Protocol / isolation / logging deep dives | `references/*.md` (app-server, components-hooks, docker-qa, install-verify, isolation, logging-debug) |

## STRUCTURE

`scripts/lsp-e2e.sh` (3,676 LOC) is the largest file in `.agents/` — LSP end-to-end driver; treat as its own subsystem when editing.

## CONVENTIONS

- Every helper script ships `--self-test` that asserts against the live machine; the scripts are their own regression checks. Run `bash scripts/lib/common.sh --self-check` first.
- Docker (`script/agent/qa-docker.sh`) is the DEFAULT QA surface; local scripts are the fallback (Windows / no Docker).
- Verified against `codex-cli 0.140.0`; confirm with `codex --version` before trusting flags.
- Interactive `codex` is a shell function injecting `--profile quotio`; bash scripts bypass it.

## ANTI-PATTERNS

- NEVER use the real `~/.codex` or a published package; every script shasums `~/.codex/config.toml` before/after — keep that guard.
- NEVER hit a real model API — mock model only.
- NEVER prove a hook via log scraping; the app-server notification stream is first-party proof.
- NEVER trust the interactive codex shell function in scripts.
- Evidence goes under gitignored `.omo/evidence/<YYYYMMDD>-<slug>/`; never commit it.
