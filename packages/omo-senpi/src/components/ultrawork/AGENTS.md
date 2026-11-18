# components/ultrawork/ — Ultrawork Keyword Arming

**Score ~12** (9 files, ~2.4k LOC; distinct domain — keyword detection + directive arming).

## OVERVIEW

Detects a standalone `ulw`/`ultrawork` keyword in user input and arms the session with the ultrawork directive. Selects model-specific directive content and preserves arming across uncached packaged-extension reloads via a `globalThis` symbol slot keyed to the exact directive strings. Entry `index.ts`; `generated-directive.ts` holds the generated directive text.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `createUltraworkComponent()`, `classifyUltraworkInput()`, `isUltraworkInput()`, `sharedSessionArming()`, `armingSnapshot()` |
| `generated-directive.ts` | `SENPI_ULTRAWORK_DIRECTIVE`, `SENPI_ASTRA_ULTRAWORK_DIRECTIVE` |
| `ultrawork.test.ts` | Behavior suite (~577 LOC — the hotspot) |

## CONVENTIONS

- The keyword arms only as a standalone word: `ulw`, `ulw loop`, a leading `ulw-loop`/`ulw-plan.` do; `ulwfoo`, `ulw_helper`, `mass-ulw-refactor`, `ulw-plan.md`, `.omo/ulw/` do NOT (#9738).
- Quoted, injected, relayed, and negated regions are blanked before the pattern runs (`stripQuotedRegions`, `maskNegatedInvocations`).
- Re-arm uses a reminder WITHOUT the `<ultrawork-mode>` open tag so it never reads as a fresh directive block (avoids re-paying ~17KB of tokens).
- Route/stage types: `UltraworkRoute`, `UltraworkInvocationStage`, `UltraworkSuppressionReason`, `SessionArming`.

## ANTI-PATTERNS

- Do NOT reuse a shared arming ledger when either directive text differs — exact text equality is intentional (a prior 32-bit FNV-1a revision admitted collisions).
- Do NOT re-arm after a rejected compaction: the transcript still holds the directive.
- Never match identifiers/paths (`ulwfoo`, `ulw_helper`, `.omo/ulw/`).

## COMMANDS

- `bun test packages/omo-senpi` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json` · `bun run test:senpi`
