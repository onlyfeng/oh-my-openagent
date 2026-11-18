# components/side-panel/ - TUI Side Panel (direct files)

## OVERVIEW

Top-level files of the Senpi side panel component (nested `data/`, `format/`,
`git/`, `popups/` subdirs are out of this direct-files scope). Entry
`index.ts` exports `createSidePanelComponent`, `SidePanelComponentOptions`,
and `SIDE_PANEL_FLAG`; the controller runtime is lazy-loaded via
`createSidePanelController` on the first enabled session. **Score 14**
(24 top-level modules, lazy runtime seam). Parent: [`src/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Registration shell + flag; epoch-guarded lazy runtime import |
| `runtime.ts` | Controller implementation (loaded per enabled session) |
| `commands.ts`, `host-surface.ts` | Panel commands; host integration surface |
| `rows.ts`, `body.ts`, `width.ts`, `guards.ts`, `links.ts` | Row/body rendering, width handling, guards, link handling |
| `store.ts` (if present) / state modules | Panel state |
| `constants.ts`, `context.ts`, `regular-mode-notice.ts` | Constants, command context, mode notice |
| `*.test.ts` | Colocated Bun suites (`#given/#when/#then` names) |

## CONVENTIONS

- Registration shell + lazy runtime: settings and runtime loaders are
  injectable for tests; the flag has NO default so persistent configuration
  decides when the flag is absent.
- Teardown must run on committed `session_shutdown`; `session_before_switch`
  may be cancelled.
- An in-flight runtime import must not mount after teardown - check the
  epoch before mounting.

## ANTI-PATTERNS

- Do not eagerly import the runtime from the registration shell.
- Do not weaken the epoch guard to simplify mounting.

## QA

```bash
bun test packages/omo-senpi
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
