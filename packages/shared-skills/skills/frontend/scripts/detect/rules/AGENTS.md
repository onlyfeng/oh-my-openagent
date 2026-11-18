# detect/rules/ -- Frontend Detection Rule Catalog

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 13 (223 rule/support files across 12 subdirs, own barrel, distinct domain). This file covers the aggregate; each category dir has its own AGENTS.md.

## OVERVIEW

The frontend-slop detection catalog: 79 rules in six categories, aggregated by `index.ts` into `ALL_RULES: readonly Rule[]` - the default ruleset `detect/lint.ts` runs (page pass + source pass) and `detect/index.ts` re-exports.

## STRUCTURE

| Category | Count | Aggregate type |
|----------|-------|-----------------|
| `color/` | 9 | `PageRule[]` |
| `typography/` | 14 | `Rule[]` |
| `layout/` | 14 | `Rule[]` |
| `decoration/` | 17 | `Rule[]` |
| `copy/` | 9 | `PageRule[]` |
| `source/` | 16 | `SourceRule[]` |

`ALL_RULES` concatenates in exactly that order. Every category dir holds one file per rule plus a `support/` dir of shared non-rule helpers (e.g. `color/support/` contrast pairs, gradients, palette roles).

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add a rule | Category dir → one `<rule-name>.ts` + entry in that category's `index.ts` aggregate |
| Rule types (`Rule`, `PageRule`, `SourceRule`, `RuleMeta`) | `detect/types.ts` (`../rule` re-export) |
| Default ruleset consumption | `detect/lint.ts` (`options.rules ?? ALL_RULES`, split by `isPageRule` / `isSourceRule`) |
| Per-rule overrides (`rules`, `allowFonts`, `allowEasings`, `designTokens`) | `DetectConfig` in `detect/types.ts` |

## CONVENTIONS

- `RuleMeta` carries `id`, `name`, `category` (`slop` | `quality`), `severity` (`error` | `warning` | `advisory`), `engine` (`page` | `source`), `description`, `fixHint`; page rules read a rendered `PageSnapshot`, source rules read raw `SourceFile` text.
- Within a category index the order is deliberate (lint output order follows it); do not reorder casually.
- Some detectors ship both engines - e.g. `ai-color-palette`, `em-dash-overuse` exist as page rules in their category AND as source twins in `source/`.
- Rule semantics are adapted from Impeccable's detector (Apache-2.0, Copyright 2025 Paul Bakaus); keep `ATTRIBUTION.md` §6-7 in sync when porting more.

## ANTI-PATTERNS

- NEVER add a rule to `index.ts` without its category aggregate - it would be unreachable.
- NEVER put shared helpers next to rules; category `support/` dirs exist for that (they hold no `Rule` exports).
- Tests are co-located (`<rule>.test.ts`) with fixture snapshots; a rule without tests does not land.
