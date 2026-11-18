# model-profile/ — Session-Start Model Profile Component

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 22 (19 files, ~3.7k LOC). Component conventions (factories, `register(pi, ctx)`, structural ports) are owned by [`../../AGENTS.md`](../../AGENTS.md).

## OVERVIEW

Applies a named ordered model profile at session start: a lane of `{ providers, model, variant? }` rungs reusing the delegate-core fallback entry shape, with auth-aware resolution and startup bookkeeping.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `createModelProfileComponent` + the three outcome message types (`MODEL_PROFILE_{APPLIED,UNAVAILABLE,UNKNOWN}_TYPE`) |
| `resolve.ts` | Pure resolver: `mergeModelProfiles`, `resolveModelProfile` — rung matching through the delegate-core resolver; `builtin \| user \| pin` sources |
| `builtin-profiles.ts` / `custom-providers.ts` / `default-profile.ts` | Builtin lanes, custom provider declarations, default profile |
| `models-json.ts` | `<agentDir>/models.json` as JSONC — cached parsed trees deeply frozen and readonly |
| `request-auth.ts` | Credential/request probe (~280 LOC) |
| `describe.ts` | Profile view builder; `notice.ts` — user-facing notice |
| `credential-policy.ts` | Credential policy for profile rungs |

## CONVENTIONS

- A user profile entry replaces a builtin wholesale; a providerless model id intentionally matches ANY registry provider; builtin provider order is significant.
- Session startup applies via the session-only model setter and records outcomes as startup bookkeeping.
- Runtime boundaries are probed structurally from unknown host values.

## ANTI-PATTERNS

- `model_profile` is NOT the unrelated `omo.json` `profiles` overlay — never conflate them.
- No aliases/migrations for retired model-profile ids (the builtin source states none exist).
- Don't infer a builtin rung from an aggregator's vendor-prefixed copy or an undeclared custom provider model; custom-provider fallback requires the exact model id declared in `models.json`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/model-profile
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
