# layout/ — Layout Detection Rules

**Score 10** (39 files, ~3.5k LOC, own barrel + `support/` helpers). One of the six rule categories in [`../AGENTS.md`](../AGENTS.md) (which owns the catalog-level facts: `ALL_RULES` order, rule types, overrides, attribution, and the authoring rules — one rule per file, colocated tests, `support/` for shared helpers). This file maps THIS category's rules.

## RULES (`index.ts` -> `LAYOUT_RULES: readonly Rule[]`, in registration order)

| Rule | Sev | Detects |
|------|-----|---------|
| `nested-cards.ts` | warning | A bordered/raised card inside another card adding an unnecessary surface |
| `monotonous-spacing.ts` | warning | One spacing value dominating the page instead of separating groups from sections |
| `edge-flush-cards.ts` | warning | A card row with a gutter on one side but touching/crossing the opposite clipping edge |
| `text-occlusion.ts` | warning | An overlapping surface or text run covering readable words |
| `first-viewport-column-overflow.ts` | warning | An opening row with one column far below the fold while its sibling fits one screen |
| `cramped-padding.ts` | warning | Rendered text sitting too close to a visible container boundary |
| `body-text-viewport-edge.ts` | warning | Reading text touching the viewport edge without a usable gutter |
| `text-overflow.ts` | warning | Text spilling out of its box into a clip, another element, or the viewport edge |
| `clipped-overflow-container.ts` | advisory | A container cutting off a positioned menu/tooltip/dialog that needs to escape it |
| `heading-rhythm.ts` | warning | Headings sitting closer to preceding content than to the content they introduce |
| `straggler.ts` | advisory | A paragraph leaving one short word on its last line |
| `unbalanced-heading.ts` | advisory | A heading whose last line is much shorter than the line above |
| `content-hidden-at-rest.ts` | error | A large share of page text transparent/hidden instead of appearing as content |
| `viewport-reflow.ts` | error | The document wider than the viewport, requiring sideways scrolling |

## SUPPORT (`support/`, shared non-rule helpers — no `Rule` exports)

`geometry.ts`, `cards.ts`, `clipping.ts`, `padding.ts`, `rhythm.ts`, `line-breaks.ts`, `hidden-interface.ts`, `declared-mockups.ts` (the mockup-region exemption), `layout-cases.ts`, `motion.ts`.

## NOTES

- `content-hidden-at-rest` and `viewport-reflow` are the two `error` severities: both mean content is effectively unreachable, not merely sloppy.
- Several rules intentionally trigger only at specific viewports (e.g. reflow stress at 320 px); fixture snapshots must be built per viewport when testing them.

## COMMANDS

```bash
bun test packages/shared-skills/skills/frontend/scripts/detect/rules/layout
```

Parent: [`../AGENTS.md`](../AGENTS.md).
