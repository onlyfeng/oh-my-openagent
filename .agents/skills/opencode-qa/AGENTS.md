# opencode-qa — opencode Harness QA Skill

**Generated:** 2026-11-18 / c2691818a9 | Score: 16 (22 files >20 threshold, largest QA surface in `.agents/`, 3.1k-LOC E2E + 994-LOC probe hotspots)

## OVERVIEW

QA skill for opencode itself: CLI (`opencode run`), HTTP server + SSE hook proof, tmux TUI smoke, and SQLite session-DB investigation — each case mapped to a tested helper script plus a per-domain reference doc.

## WHERE TO LOOK

| Need | Location |
|------|----------|
| Skill entry / case router (A=CLI, B=server/SSE, C=TUI, D=DB) | `SKILL.md` |
| Shared harness (DB path, SQL escaping, isolated XDG sandbox, server start/stop, EXIT-trap cleanup) | `scripts/lib/common.sh` |
| Server health/endpoint check | `scripts/server-smoke.sh` |
| Hook/action/event proof via SSE stream | `scripts/sse-hook-probe.sh` |
| Serve-topology wake runner-split regression probe | `scripts/serve-wake-split-probe.sh` (994 LOC; `--expect reproduced\|fixed --evidence-dir`) |
| TUI smoke under tmux | `scripts/tui-smoke.sh` |
| Session lookup by id / name / text; export roundtrip | `scripts/db-session-by-{id,name,text}.sh`, `scripts/export-roundtrip.sh` |
| Fake LLM server + event/branch fixtures | `scripts/lib/fake-openai-{server,events,branches}.mjs` |
| LSP end-to-end driver | `scripts/lsp-e2e.sh` (3,155 LOC — own subsystem) |
| Per-domain deep dives | `references/*.md` (cli-commands, db-investigation, docker-qa, events-hooks, sdk, server-api, testing-harness, tui-tmux) |

## CONVENTIONS

- Every script ships `--self-test` asserting against the live machine; run `bash scripts/lib/common.sh --self-check` first.
- Docker (`script/agent/qa-docker.sh`) is the DEFAULT QA surface; local scripts are the fallback (Windows / no Docker).
- Verified against opencode v1.17.7 (bun 1.3.12); sanity-check flags with `opencode <cmd> --help`.
- `common.sh` requires `opencode`, `sqlite3`, `curl`, `jq`, `tmux` on PATH.

## ANTI-PATTERNS

- NEVER spawn opencode without an isolated XDG sandbox (`XDG_DATA_HOME`/`XDG_CONFIG_HOME`/`XDG_STATE_HOME`/`XDG_CACHE_HOME` → temp dirs); never pollute the real `opencode.db`.
- NEVER run unbounded text search over the `part` table (multi-GB scan); `db-session-by-text.sh` refuses it on purpose — always scope (`--session`, `--recent`, `--since`).
- tmux is SMOKE-only (boots/renders/survives); assert real behavior via `opencode run --format json`, the server API, or SSE events.
- The opencode source repo (`packages/opencode`) tests with `bun test` from ITS root, never from this repo root.
- Evidence goes under gitignored `.omo/evidence/<YYYYMMDD>-<slug>/`; never commit it.
