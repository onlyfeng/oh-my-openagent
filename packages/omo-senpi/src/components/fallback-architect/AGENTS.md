# fallback-architect/ — Model-Fallback Nudge Component

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 19 (10 files, ~1.4k LOC). Component anatomy and registration order live in `extension/component-list.ts` and [`../../../AGENTS.md`](../../../AGENTS.md).

## OVERVIEW

Detects a refused turn and nudges the model onto a working fallback profile with hidden directives, gated by category/model availability, plus a user-visible notice.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `createFallbackArchitectComponent(options): OmoSenpiComponent` — the factory |
| `detection.ts` | THE single refusal predicate — shared with `components/ulw-execute-continuation/agent-end-eligibility.ts`, so the nudge and the continuation gate can never disagree |
| `architect-gate.ts` | Gate over `senpi-task` category resolution + `loadSenpiOmoConfig` — decides whether a fallback profile is available |
| `directive.ts` | Hidden directive content — rides as custom messages with `display: false`; the custom types are the wire values senpi persists in the transcript (asserted by the live QA driver) |
| `notice.ts` | User-visible upgrade notice via senpi-task `notice-box` |

## CONVENTIONS

- Structured custom message types distinguish `display: false` model prompts from `display: true` TUI notices — keep the distinction when adding messages.

## ANTI-PATTERNS

- Never add a second refusal predicate — `detection.ts` is the single source; both consumers must keep reading it.
- The directive is hidden, not optional: do not surface model-facing nudge text in the TUI, and do not hide the notice.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/fallback-architect
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
