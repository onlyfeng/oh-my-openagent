# skills/ulw-plan/ — Senpi-Native Planning Skill

**Score ~8** (7 files; distinct domain — approval-gated planning, no implementation).

## OVERVIEW

The Senpi-native `ulw-plan` skill: turns an ultrawork request into a guarded, approval-gated plan written only under `.omo/drafts` or `.omo/plans`. Planning is read-only — it never implements product code and never dispatches implementers. `SKILL.md` is the contract; `scripts/scaffold-plan.mjs` is the only sanctioned artifact producer.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `SKILL.md` | Planning lifecycle + approval-gate contract |
| `scripts/scaffold-plan.mjs` | Artifact producer (~190 LOC): `<slug>`, `--clear|--unclear`, `--draft-only`, `--review-required`, `--reset [--force]` |
| `scripts/plan-templates.mjs` | Deterministic template constants/builders (~171 LOC) |
| `references/full-workflow.md` | Planning lifecycle reference (~279 LOC) |

## CONVENTIONS

- Senpi-native: OpenCode-only tool names (`call_omo_agent`, `task`, `background_output`, `team_*`) must be translated to Senpi-native tools.
- Scaffold artifacts through `scaffold-plan.mjs`; append todo rows, never rewrite generated headers; preserve `Must NOT have`; record `review_required`.
- Scaffold refuses unsafe paths, symlink escapes, non-Markdown targets, invalid slugs; preserves edited artifacts unless reset/force.

## ANTI-PATTERNS

- Never self-activate on a bare `ulw`; never implement product code or delegate implementation; never skip the approval gate.
- Do NOT hand-build plan artifacts — use the scaffold script.
- Do not dispatch ordinary lanes with `category=`; only the documented advisory/reviewer exceptions apply.
- Do not convert override phrases into authorization for irreversible/destructive/spending decisions.

## COMMANDS

- `node packages/omo-senpi/skills/ulw-plan/scripts/scaffold-plan.mjs <slug> [--clear|--unclear] [--draft-only] [--review-required] [--reset [--force]]`
