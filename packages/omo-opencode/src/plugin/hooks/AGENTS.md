# src/plugin/hooks/ - Hook Composition (5-Tier)

## OVERVIEW

9 files, ~890 LOC. The composers that assemble every lifecycle hook into the
five tiers registered with OpenCode: core (session + tool-guard + transform),
continuation, and skill. This is the ONLY place hook enablement
(`isHookEnabled`) and fail-safe registration (`safeHook`) are decided.
Parent: [`src/AGENTS.md`](../../AGENTS.md) carries the tier counts.

## WHERE TO LOOK

| File | Role |
|------|------|
| `create-core-hooks.ts` | Folds session + tool-guard + transform tiers |
| `create-session-hooks.ts` | Session lifecycle tier composition |
| `create-tool-guard-hooks.ts` | Pre/post tool.execute tier; gates most guard hooks |
| `create-transform-hooks.ts` | `messages.transform` tier |
| `create-continuation-hooks.ts` | Boulder/atlas/babysitter/compaction/notification tier |
| `create-skill-hooks.ts` | Skill awareness tier |
| `model-fallback-title-updater.ts` | Title updater for model-fallback |

## CONVENTIONS

- Registration pattern: `isHookEnabled(<name>) ? safeHook(<name>, factory(deps)) : null`
  - a throwing factory becomes `null`, never a broken plugin.
- Each composer receives its dependencies explicitly (context, managers,
  options); no import of hook internals beyond the `src/hooks` barrel.
- Adding a hook means picking the right composer here plus the hook dir and
  config-schema entry (see parent `src/hooks/AGENTS.md`).

## ANTI-PATTERNS

- Do not wire `task-reminder` or `ralph-loop` casually - both are deliberate
  WIP/unwired (parent hooks AGENTS.md).
- Do not register a hook unconditionally; every hook goes through the
  enabled/safeHook pattern.

## QA

```bash
bun test packages/omo-opencode/src/plugin/hooks
bun run --cwd packages/omo-opencode typecheck
```

Composition changes are OpenCode-wired: run the `opencode-qa` skill and
record local evidence.
