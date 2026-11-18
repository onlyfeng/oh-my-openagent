# search — Query Parsing + Memory/Session Search

**Score:** 11 (7 files, 1.3k LOC, index barrel, feeds `/search` and recall). Parent: [`packages/memory-core/AGENTS.md`](../../../AGENTS.md).

## OVERVIEW

Ranked search over memory content and session transcripts: query parsing, the search engine, and transcript providers behind one barrel (`index.ts` re-exports `query.ts`, `engine.ts`, `senpi-session-provider.ts`).

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Query parsing (operators, terms) | `query.ts` |
| Ranked search — `searchTranscripts`, engine contracts | `engine.ts` |
| Transcript discovery/provider — `SenpiSessionProvider`, provider contracts | `senpi-session-provider.ts` |
| Barrel | `index.ts` |

## CONVENTIONS

- Providers are injected: the engine takes a provider contract, and `SenpiSessionProvider` is the one adapter-side implementation (it reads Senpi session files; keep harness knowledge inside it).
- Provider and engine are exercised together in `senpi-session-provider.test.ts`; `query.test.ts` pins the grammar in isolation.
- Provider failures degrade to empty results with an error note — search never throws into the memory tools.

## ANTI-PATTERNS

- Do not add a second transcript provider inside memory-core; harness-specific providers belong to the adapter and plug into the contract.
- Do not parse queries ad hoc in the engine — every grammar change goes through `query.ts` and its tests.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/search
bun run --cwd packages/memory-core typecheck
```
