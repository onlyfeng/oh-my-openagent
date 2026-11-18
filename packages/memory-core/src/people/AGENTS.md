# src/people - People-Card Grammar

**Generated:** 2026-11-18 | c2691818a9

People-card markdown grammar: parse/serialize, slug rules, reserved slugs, collision resolution, observation formatting. Distinct domain: the one place a "person" entity gets a structured representation.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `format.ts` | `parsePeopleCard`, `serializePeopleCard`, observation format. |
| `person-slug.test.ts` | Pins `sanitizePersonSlug`, `isReservedSlug`, `resolveSlugCollision` behavior (slug rules live in the module under test). |
| `format.test.ts`, `format-observations.test.ts` | Round-trip and observation rendering coverage. |
| `index.ts` | Barrel re-export. |

## ANTI-PATTERNS

- Behavioral patterns are NEVER stored in people cards - lines are limited to `IDENTITY` / `ATTRIBUTE` / `RELATIONSHIP` / `INSTRUCTION` prefixes with bounded metadata (parent memory-core invariant, enforced here).
- `isReservedSlug` is the only guard against a person slug colliding with a system path; never bypass it.
- `resolveSlugCollision` is deterministic: same input pair always picks the same winner. Do not introduce randomness.

## CONVENTIONS

- Tests are co-located `*.test.ts` under `bun test`.
- Slug rules are pure string functions; no I/O.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/people
bun run --cwd packages/memory-core typecheck
```
