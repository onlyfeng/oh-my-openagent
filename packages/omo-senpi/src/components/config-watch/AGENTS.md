# components/config-watch/ - omo Config Watch Registration

**Score ~9** (7 files, ~1.8k LOC; distinct domain - senpi config-watch protocol integration).

## OVERVIEW

Registers omo config surfaces (`.omo/omo.jsonc` and friends) with senpi's optional in-process config-watch protocol so config edits reload with validation. Separates target/path safety (`paths.ts`) from validation (`validate.ts`), and injects resolvers/validators for tests.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `createConfigWatchComponent()` - registration + reload/reject handling |
| `paths.ts` | `resolveOmoConfigWatchTargetResolution`, watch-target globs, `isSenpiRestrictedTarget` |
| `validate.ts` | `createOmoConfigValidator`, validation types; deprecated-key warnings on reload |

## CONVENTIONS

- Protocol events: `config-watch:register` / `ready` / `reloaded` / `rejected`; registration id `"omo"`.
- Exact watch targets (`paths.ts` constants): files `/omo.jsonc` `/omo.json`; dirs `/.omo` `/.omo/omo.jsonc` `/.omo/omo.json`; `isSenpiRestrictedTarget` filters senpi-protected paths out before registering.
- Plan 9 filesystems are detected in target resolution via `statfsSync` (`PLAN9_FILE_SYSTEM_TYPE = 0x01021997`, deliberately distinct from tmpfs `0x01021994`; returns null where `statfsSync` is unavailable).
- Rejected reloads trigger a DEFERRED, BOUNDED retry: `MAX_REJECTION_RETRIES = 3` per distinct rejected payload, reset when the payload changes. Senpi rejects on the same synchronous stack that delivered REGISTER, so an unbounded re-emit would recurse to `RangeError`.
- Resolvers/validators are injectable (`ConfigWatchComponentOptions`) for hermetic tests.

## ANTI-PATTERNS

- Must NOT synchronously re-register after a rejection - deferred retry with a bounded budget, reset on payload change (test-pinned).

## COMMANDS

- `bun test packages/omo-senpi` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json`
