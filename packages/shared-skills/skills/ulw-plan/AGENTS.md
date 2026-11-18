# shared-skills/skills/ulw-plan/ — Cross-Harness Planning Skill

**Score ~8** (7 files; distinct domain — approval-gated planning, no implementation).

## OVERVIEW

The shared/cross-harness `ulw-plan` skill (OpenCode + shared pool). Same planning-only, approval-gated contract as the Senpi copy: read-only planning that writes guarded `.omo` plan artifacts and never implements product code. NOTE: this skill is dual-maintained by hand — the Codex edition keeps its own copy under `packages/omo-codex/plugin/components/ultrawork/skills/ulw-plan/`, and `sync-skills.mjs` does NOT copy this shared version to Codex. Keep both in step when editing.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `SKILL.md` | Planning lifecycle + approval-gate contract |
| `scripts/scaffold-plan.mjs` | Artifact producer: `<slug>`, `--clear|--unclear`, `--draft-only`, `--review-required`, `--reset [--force]` |
| `scripts/plan-templates.mjs` | Deterministic template constants/builders |
| `references/full-workflow.md` | Planning lifecycle reference (~256 LOC) |

## CONVENTIONS

- Scaffold artifacts through `scaffold-plan.mjs`; append todo rows, never rewrite generated headers; preserve `Must NOT have`; record `review_required`.
- Scaffold refuses unsafe paths, symlink escapes, non-Markdown targets, invalid slugs; preserves edited artifacts unless reset/force.

## ANTI-PATTERNS

- Never self-activate on a bare `ultrawork`; never implement product code or delegate implementation; never skip the approval gate.
- Do NOT hand-build plan artifacts — use the scaffold script.
- Do not dispatch ordinary lanes with `category=`; only documented advisory/reviewer exceptions apply.
- Do not convert override phrases into authorization for irreversible/destructive/spending decisions.

## COMMANDS

- `node packages/shared-skills/skills/ulw-plan/scripts/scaffold-plan.mjs <slug> [--clear|--unclear] [--draft-only] [--review-required] [--reset [--force]]`
