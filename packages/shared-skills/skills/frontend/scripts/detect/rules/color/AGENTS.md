# color/ — Color Detection Rules

**Score 11** (34 files, ~3.3k LOC, own barrel + `support/` helpers; distinct detector domain). One of the six rule categories in [`../AGENTS.md`](../AGENTS.md) (which owns the catalog-level facts: `ALL_RULES` order, rule types, overrides, attribution). This file maps THIS category's rules and helpers.

## RULES (`index.ts` -> `COLOR_RULES: readonly PageRule[]`, in registration order)

| Rule | Sev | Detects |
|------|-----|---------|
| `low-contrast.ts` (`low-contrast`) | error | Text below WCAG AA contrast for its size (4.5:1, 3:1 large); skips mockup-declared nodes, disabled/`aria-disabled`, sub-9px bare text, and placeholder-style text (checked via `support/placeholder-contrast`) |
| `gray-on-color.ts` (`gray-on-color`) | warning | Mid-gray ink losing clarity on a chromatic surface |
| `ai-color-palette.ts` (`ai-color-palette`) | warning | Stock violet/cyan ramps or paired neon inks standing in for a palette; has a source-engine twin in `../source/` |
| `cream-palette.ts` (`cream-palette`) | warning | Page defaulting to a warm cream/beige ground |
| `gradient-text.ts` (`gradient-text`) | warning | A color ramp clipped to text glyphs |
| `dark-glow.ts` (`dark-glow`) | warning | A colored blurred shadow reading as a decorative glow |
| `radial-halo.ts` (`radial-halo`) | warning | A saturated radial fade haloing a dark surface |
| `radial-spotlight-glow.ts` (`radial-spotlight-glow`) | warning | A translucent radial accent floating behind section copy |
| `design-system-color.ts` (`design-system-color`) | advisory | A painted color outside the project's declared palette/tokens |

`placeholder-contrast.test.ts` (top level) pins the placeholder-exemption behavior of `low-contrast` against `support/text-colors` — it is the contract that placeholder text may sit below AA without failing, while real text may not.

## SUPPORT (`support/`, shared non-rule helpers — no `Rule` exports)

`contrast-pairs.ts` (`distinctPairs`, `ContrastHit`), `gradients.ts` (`gradientStops`, gradient parsing/compositing), `painted-text.ts` (`textPainted` — decides whether an element actually paints text), `contrast-role.ts`, `palette-roles.ts`, `text-colors.ts` (`textInk`, `textBackgrounds`, `solidGround`, `clipsText`), `color-path.ts` (`bareText`), `mockup-roots.ts` (mockup region detection), `placeholder-contrast.ts`, `glow-forms.ts`, `gradient-samples.ts`, `stock-accent.ts`, `animation-opacity.ts`.

Color/page primitives come from `../../core/color` (`blend`, `contrastRatio`) and `../../core/page` (`ancestors`, `fontSizePx`, `styleOf`, `shortSnippet`); the `PageRule`/`PageHit` types from `../../rule`.

## CONVENTIONS

- Every rule is a pure `PageRule`: `check({ snapshot, config })` over the rendered `PageSnapshot` — rules own no browser access; the harness (`../../lint.ts`) supplies the snapshot.
- One file per rule plus a colocated `<rule>.test.ts` built on fixture snapshots (`../../testing/build-snapshot`); a rule without tests does not land.
- Registration order inside `COLOR_RULES` is deliberate (lint output follows it); `low-contrast` runs first so the highest-severity hits lead.

## ANTI-PATTERNS

- NEVER add a rule file without registering it in `index.ts`'s `COLOR_RULES` — it would be unreachable.
- NEVER put shared logic in a rule file; it goes in `support/` (and support modules export no `Rule`).
- Don't duplicate a check that belongs to the project's declared-palette config path — `design-system-color` already covers out-of-palette paint.

## COMMANDS

```bash
bun test packages/shared-skills/skills/frontend/scripts/detect/rules/color
```

Parent: [`../AGENTS.md`](../AGENTS.md).
