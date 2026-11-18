# thread/tools/ - Thread Tool Surface

**Score ~11** (13 files; thread tool + relay/extension wiring domain).

## OVERVIEW

The omo-senpi thread component's tool implementations: thread read ops, relay/extension tools, gateway service access, and reply-target/permission resolution. Parameter schemas and result contracts live one level over in [`../contracts`](../contracts) (TypeBox; `threadToolParamSchemas`, `parseThreadParams`). Host access goes through injected ports, never concrete senpi types.

## WHERE TO LOOK

| File | Role |
|------|------|
| `read-ops.ts` | Thread read operations |
| `relay-tools.ts` | Relay tool implementations |
| `extension-tools.ts` | Extension-facing tools (+ `extension-tools.test.ts`) |
| `gateway-services.ts` | Gateway service access |
| `ports.ts` | Injected host ports |
| `internals.ts` | Shared internal helpers |
| `reply-target.ts` / `reply-permission.ts` | Reply-target resolution + permission |
| `model-match.ts` / `send-view.ts` / `thread-row.ts` | Model matching, send view, thread row shaping |

## CONVENTIONS

- Invalid parameters are data outcomes, never thrown exceptions (matches the thread-contract rule).
- Colocated `*.test.ts`; Bun `#given/#when/#then`.
- Consume the host only through `ports.ts` structural ports.

## ANTI-PATTERNS

- Keep parameter/validation errors as data outcomes; never throw on bad params.
- Do not import concrete senpi types outside `extension/`.

## COMMANDS

- `bun test packages/omo-senpi/src/components/thread` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json` · `bun run test:senpi`
