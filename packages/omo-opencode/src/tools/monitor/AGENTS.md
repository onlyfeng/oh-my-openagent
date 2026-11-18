# src/tools/monitor/ — Monitor Tools (OpenCode)

## OVERVIEW

10 files, ~1,000 LOC. Registers the `monitor_list` / `monitor_output` /
`monitor_start` / `monitor_stop` tools (gated on `monitor.enabled`); the
watcher-process backend itself lives in
`src/features/monitor/`. Distinct domain: tool surface over the monitor
feature. Parent: [`src/tools/AGENTS.md`](../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel exporting `createMonitorTools` |
| `create-monitor-tools.ts` | Tool-record assembly and registration |
| `monitor-start.ts` / `monitor-stop.ts` | Start/stop watchers; stop argument/result types |
| `monitor-list.ts` | List registered monitors |
| `monitor-output.ts` | Read captured output |
| `monitor-*.test.ts` | Colocated suites per tool |

## CONVENTIONS

- One factory file per tool; `create-monitor-tools.ts` only composes.
- Tool results are plain JSON-safe values — no live handles cross the tool
  boundary.

## ANTI-PATTERNS

- Do not register these tools without the `monitor.enabled` gate.
- Do not reach into `src/features/monitor/` internals from tool logic; use
  the feature's manager API.

## QA

```bash
bun test packages/omo-opencode/src/tools/monitor
bun run --cwd packages/omo-opencode typecheck
```

Tool changes are OpenCode-wired: drive real OpenCode (`opencode-qa`) and
record local evidence.
