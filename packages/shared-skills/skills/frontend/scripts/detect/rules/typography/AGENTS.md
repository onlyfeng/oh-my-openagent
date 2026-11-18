# detect/rules/typography/ — Typography Rule Catalog

**Score ~13** (39 files: 14 rules + 14 colocated tests + support; category aggregate).

## OVERVIEW

The typography category of the frontend `detect` lint engine. Each rule is one `PageRule` module with a colocated `.test.ts`; shared text/font/type-hierarchy helpers live under `support/`. The category `index.ts` collects all rules into `TYPOGRAPHY_RULES: readonly Rule[]`, which `detect/rules/index.ts` spreads into the master catalog.

## WHERE TO LOOK

| Area | Role |
|------|------|
| `index.ts` | Aggregate: imports every rule, exports `TYPOGRAPHY_RULES` |
| `<rule>.ts` + `<rule>.test.ts` | One rule per module, typed `PageRule`, with its test |
| `support/` | Shared text/font/type-hierarchy helpers (never beside rules) |

## CONVENTIONS

- The category index order determines lint output order — append deliberately.
- Each rule is typed `PageRule`; the aggregate is `readonly Rule[]`.
- Rule tests import their rule module directly; the aggregate is consumed by `detect/rules/index.ts`.

## ANTI-PATTERNS (parent rules-catalog)

- NEVER add a rule to `index.ts` without its category aggregate.
- NEVER put shared helpers next to rules — they belong in `support/`.
- A rule without a colocated test does not land.

## COMMANDS

- `bun test packages/shared-skills/skills/frontend/scripts/detect/rules/typography/`
- `bun packages/shared-skills/skills/frontend/scripts/detect/cli.ts --json <file-or-url>`
