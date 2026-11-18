# detect — frontend design-review SDK + CLI

TypeScript detect/lint engine behind the `frontend` skill: renders pages at multiple viewports, snapshots computed styles, and runs a catalog of page/source rules. Earned by score 16 (246 TS files incl. rules/, own tsconfig, SDK barrel, high export density). Rule authoring lives in [rules/AGENTS.md](rules/AGENTS.md); skill packaging rules in [packages/shared-skills/AGENTS.md](../../../../AGENTS.md).

## ENTRY POINTS

| File | Role |
|------|------|
| `index.ts` | The `@public` SDK barrel — the only surface for callers outside this repo (skill scripts, a project's own lint step) |
| `cli.ts` | Thin CLI over the barrel: `bun cli.ts [--json] [--reflow] [--config <file.json>] <file-or-dir-or-url>...` |
| `detect.ts` | `detectPage` / `detectFiles`: render each viewport, lint every snapshot, dedupe |
| `lint.ts` | `lintSnapshot` / `lintSources` / `effectiveSeverity` / `sortFindings` / `hasFailures` |
| `config.ts` | `parseConfig` — parses untrusted JSON, rejects unknown keys |
| `rule.ts` | The `Rule = PageRule \| SourceRule` discriminated contract |
| `report.ts` | `summarize` (severity counts) + `formatFindingsText` |
| `types.ts` | Finding/Severity/Viewport/Snapshot types (re-exported by the barrel) |

## STRUCTURE

- `core/` — pure helpers: color math (`blend`, `contrastRatio`, `parseColor`), page ignore logic (`isIgnored`), source extension dispatch, shadow DOM, characters.
- `render/` — `Renderer` contract (`RenderTarget`, `RenderError`), the default renderer factory, HTML probe placement.
- `rules/` — the catalog: six families (color, copy, decoration, layout, source, typography) folded into `ALL_RULES`; own AGENTS.md.
- `snapshot/` — `collectPageSnapshot` + `STYLE_PROPS` (which computed styles are captured).
- `testing/` — `build-snapshot.ts`: Chromium default computed styles for hand-built test snapshots.

## CLI CONTRACT

Exit 0 = no failing findings; 1 = findings that fail the gate (any error or warning — advisories never fail); 2 = usage or render failure. `http(s)://` targets render as pages; other paths go through `collectFiles` (`../perfection`), HTML renders as a page, remaining files lint as source text. Both CLI catches are `no-excuse-ok` boundaries mapping to exit 2.

## RULE + CONFIG CONTRACT

- `meta.engine: "page"` checks a rendered `PageSnapshot`; `"source"` checks file text filtered by `extensions`. Severity is `error | warning | advisory`.
- `config.rules` maps rule id → severity override (`off` drops hits); a hit's own `severity` override still loses to the project setting unless `severityFixed` (missing-evidence notices an override must not turn into a failure).
- `parseConfig` accepts ONLY `rules`, `allowFonts`, `allowEasings`, `designTokens` (fonts/colors/radii/fontSizes) and returns the first bad field name as the error.

## VIEWPORTS + DEDUPE

`DEFAULT_VIEWPORTS` = 1440x900 + 390x844; `--reflow` adds `REFLOW_VIEWPORTS` (320/375/768/1024/1440) with content stress at 320px. The same rule on the same element at two viewports is one finding (first viewport's location kept). Renderers are injectable (`options.renderer`); the default WebView renderer is created and closed by the callee.

## CONVENTIONS

- Own strict `tsconfig.json` (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `verbatimModuleSyntax`, `bun-types`, `noEmit`), checked by the root `typecheck:skill-scripts` script.
- Rule data is `readonly`; nothing here writes beside itself (bundled-scripts constraint).
- Tests colocated: `config.test.ts`, `lint.test.ts`, per-rule tests under `rules/`.

## COMMANDS

```bash
bun test packages/shared-skills/skills/frontend/scripts/detect/config.test.ts packages/shared-skills/skills/frontend/scripts/detect/lint.test.ts
bun packages/shared-skills/skills/frontend/scripts/detect/cli.ts --json <file-or-url>
```
