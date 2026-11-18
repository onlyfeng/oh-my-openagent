# src/features/hook-message-injector/ - Hook Message Injection

**Score ~14** (15 files, ~1.3k LOC; distinct domain - injecting hook messages into session history).

## OVERVIEW

Injects hook-produced messages into an OpenCode session by locating the right anchor message and writing synthetic parts. Bridges both message backends: the on-disk JSON message store and the OpenCode SDK message API. Used by hooks that need to surface text as if it were a real session message (continuation prompts, reminders, status).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: `injectHookMessage`, lookup/context helpers, `StoredMessage`, `MESSAGE_STORAGE`, `generatePartId`, types |
| `injector.ts` | Injection orchestration |
| `message-injection.ts` | Part/message construction |
| `message-directory.ts` | Message storage directory resolution |
| `json-message-lookup.ts` | On-disk JSON lookup: `findNearestMessageWithFields`, `findFirstMessageWithAgent` (~151 LOC) |
| `sdk-message-lookup.ts` | SDK variants (~114 LOC) |
| `context-resolver.ts` / `sdk-message-context.ts` | `resolveMessageContext`, `findMessageContextFromSDK` |
| `id-generation.ts` | `generatePartId` |
| `constants.ts` / `types.ts` | `MESSAGE_STORAGE`, message types |

## CONVENTIONS

- Two lookup backends (JSON store + SDK) share one injection contract; pick via context resolver.
- Relative imports within the module; barrel imports across modules; no path aliases.

## ANTI-PATTERNS

- Do not bypass the anchor-message lookup and write messages blindly - injection must attach to the correct nearest/first message context.

## COMMANDS

- `bun test packages/omo-opencode/src/features/hook-message-injector`
- Changes here are OpenCode-wired: run the `opencode-qa` skill with isolated XDG + saved evidence (typecheck/`bun test` alone is not QA).
