# decoration — 17 page-snapshot decoration rules

**Score 11** (45 files, ~3.3k LOC; distinct domain: the frontend detect engine's decoration rule family).

## OVERVIEW

One of the six rule families `../index.ts` aggregates into `ALL_RULES` (COLOR, TYPOGRAPHY, LAYOUT, **DECORATION**, COPY, SOURCE). `index.ts` exports `DECORATION_RULES: readonly Rule[]` — 17 `PageRule`s flagging decorative-CSS anti-patterns (hard offset shadows, stripes, clip-path shapes, marquees, pulsing dots, blinking cursors, bounce easing, ...) from `PageSnapshot` evidence.

## RULE ANATOMY (see `pulsing-dot.ts`)

Each rule file exports a `PageRule` (type from [`../../rule.ts`](../../rule.ts)):

- `meta`: stable `id`, human `name`, `category` (e.g. `"slop"`), `severity`, `engine: "page"`, `description`, optional `fixHint`
- `check(ctx: PageContext)`: pure function of the snapshot -> `readonly PageHit[]` (message + snippet + element). A hit may carry a `severity` override — a project override in `config.rules` still wins, `severityFixed` marks notices whose severity must survive overrides, and a rule set to `"off"` still drops the hit.

## STRUCTURE

| Path | Role |
|------|------|
| `index.ts` | registry: imports all 17 rules, exports `DECORATION_RULES` |
| `<rule-name>.ts` + `<rule-name>.test.ts` | one implementation+test pair per rule (17 pairs) |
| `support/` | 10 shared helpers: `appearance.ts`, `assert-cases.ts`, `borders.ts` (193 LOC), `clip-geometry.ts` (162), `css-candidates.ts`, `curve-samples.ts`, `gradient-pattern.ts`, `keyframes.ts`, `labels.ts`, `raster-state.ts` |

## CONVENTIONS

- Adding a rule: new `<name>.ts` exporting the `PageRule`, colocated `<name>.test.ts`, then import + entry in `DECORATION_RULES` (registry order = `ALL_RULES` order). Never edit `ALL_RULES` directly — it only spreads the family registries.
- Rules are pure snapshot inspectors: no DOM access, no async. Shared logic goes into `support/`, never duplicated across rules (`../../core/page` supplies snapshot traversal).
- Tests assert against snapshot fixtures via `support/assert-cases.ts`.

## COMMANDS

- `bun test packages/shared-skills/skills/frontend/scripts/detect/rules/decoration`
- Typecheck: `bun run typecheck:skill-scripts` (repo root)
