# consumer - Thread Delivery Consumer

**Score:** 9 (7 files, 1.1k LOC, distinct state machine, consumed by `thread/wake`). Parent [`thread/AGENTS.md`](../AGENTS.md) owns the fourteen-tool surface; this directory is the delivery-intake half behind it.

## OVERVIEW

`createDeliveryConsumer` runs the delivery admission state machine: intake, answer, wake, and drain over a device-service port. `ConsumerStart` reports `serving | unsupported | schema_too_new` - schema evolution is a first-class outcome.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Consumer state machine | `consumer.ts` - `createDeliveryConsumer` (`DeliveryConsumerOptions`, `DeliveryConsumer`, `ConsumerStart`) |
| Port shapes + ack decode | `port.ts` - delivery event/ack/port types, `decodeServeAck`, `refusalCode` |
| Device-service port adapter | `device-service-port.ts` |
| Test fake | `testing/` (fake port/consumer harness) |
| Production consumers | `../wake/runtime.ts`, `../wake/consumer-start.ts` |

## CONVENTIONS

- Listeners are installed BEFORE the `serve` acknowledgement - deliveries may arrive immediately after ack, and an early delivery must not be dropped.
- Tracked async work routes failures through `onError`, never as unhandled rejections (`consumer-rejection.test.ts` pins this with a colocated comment).
- Refusals decode to codes via `refusalCode`; do not string-match refusal text.

## ANTI-PATTERNS

- Do not ack `serve` before listeners are wired (see above).
- Do not swallow a `schema_too_new` start - surface it so callers can degrade visibly.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/thread/consumer
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
