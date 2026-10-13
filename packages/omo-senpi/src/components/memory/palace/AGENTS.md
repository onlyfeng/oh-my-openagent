# memory palace

Self-contained HTML memory viewer: one generated artifact with embedded data, script, and styles - no external assets, no server. 25 files (~2.6k LOC: 15 modules, 8 colocated tests, 2 test-support files). Distinct domain (pure generation/rendering, no git mutation), score 8.

## Anatomy

| Path | Purpose |
|------|---------|
| `index.ts` | Barrel: `registerPalaceCommand` / `runPalaceCommand`, collector APIs/types, `collectPalaceData` / `encodePalaceData` / `generatePalaceHtml` / `renderPalaceHtml`, `collectPeople` + primary-human constants, template constants. |
| `command.ts` | `/palace` registration shell (`registerPalaceCommand`, registered from memory `wiring-static.ts`); its handler loads the runtime sidecar on first use and reports a load failure as an actionable error. |
| `report.ts` | The shared context types and `reportPalace`, used by both the shell and the sidecar runtime; it must not import `runtime-loader.ts`, so the sidecar carries no loader of its own. |
| `command-runtime.ts` | `runPalaceCommand`: generates the artifact and reports/opens it. Command output text NEVER enters model context when no UI is attached. |
| `runtime.ts` / `runtime-loader.ts` | Sidecar entry (`#omo-memory-palace-runtime` -> `extensions/omo-memory-palace.js`) exporting the `/palace` and `/people` runtimes, and the shared once-per-process loader both registrations use. Nothing in the entry bundle may import `runtime.ts`, `command-runtime.ts`, `generator.ts` or the collectors statically. |
| `generator.ts` | HTML assembly: `generatePalaceHtml` / `renderPalaceHtml` / `encodePalaceData` over the collected data. |
| `template.ts` | Page skeleton with machine-gated inline-JSON embedding points. |
| `client-script.ts` / `styles.ts` | Inlined client behavior and styling - no external requests. |
| `collectors.ts` | Orchestration: `collectPalaceData` over the memory repo. |
| `people.ts` | People graph collection (`collectPeople`, `PRIMARY_HUMAN_CARD_PATH` / `PRIMARY_HUMAN_SLUG`) backing the relationship view. |
| `entry-collector.ts` | Memory entry collection. |
| `reflection-collector.ts` | Reflection run/outcome collection. |
| `history-collector.ts` | Git history collection, hard-capped: `HISTORY_MAX_COMMITS = 500`, `HISTORY_RECENT_DIFFS = 50`, plus per-diff and total payload caps. |
| `palace.test-support.ts` | Shared fixtures. |
| `lazy-runtime-probe.test-support.ts` | Fresh-process probe for `lazy-runtime.test.ts`: counts sidecar evaluations around startup, a settled turn and the first `/palace` and `/people`. |

## Conventions

- Artifact permissions 0600/0700; inline-JSON embedding is machine-gated with escaping/injection tests.
- Consumers import the barrel or the narrow `./command` / `./people` paths; the command is registered from memory `wiring-static.ts`, and generator exports also feed memory worker completion code.
- Colocated Bun tests (`*.test.ts`) pin escaping, caps, and rendered structure.

## Anti-patterns

- Never add external assets, a server, or a network request; the artifact must stay self-contained.
- Never bypass the machine-gated inline-JSON embedding points.

## Commands

```bash
bun test packages/omo-senpi/src/components/memory/palace
```
