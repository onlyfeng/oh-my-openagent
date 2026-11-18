# agreements — Working-Agreement Ledger

**Score:** 11 (15 files, 1.5k LOC, index barrel, distinct grammar). Domain invariants live in the parent [`packages/memory-core/AGENTS.md`](../../../AGENTS.md); this file maps the modules and the receipt mechanics.

## OVERVIEW

Owns `system/agreements.md`: parse/serialize the one-line-per-agreement grammar, apply human/agent mutations as ownership-checked transactions, and render the compiled `<agreements>` section.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Ledger parse/serialize | `ledger-file.ts` (`parseAgreementsLedger`) |
| Mutations | `mutate.ts` — `applyAgreementRecords`, `addHumanAgreement`, `ackAgreement`, `dropAgreement` |
| Automation guard + violation text | `guard.ts` (`describeAgreementsToolViolation`) |
| Compiled-prompt section | `render.ts` (`renderAgreementsSection`) |
| Event + record types | `events.ts`, `record.ts` |
| Line/format grammar | `format.ts` |
| Flag announcements (once per identity) | `notice-watermark.ts` |
| Barrel | `index.ts` (also re-exports the README-documented surface) |

## RECEIPTS (`README.md` is the spec)

- Modern events carry a durable HTML-comment SHA-256 receipt identifying them by version/agreement/session/message — never by date, text, or tool call.
- Receipt append and item update share ONE ownership-checked transaction.
- Legacy identityless records stay date-keyed; they are read-only history.
- Receipt history has no safe lifetime cap; active items follow `maxItems`.

## ANTI-PATTERNS

- Never synthesize a source id from text or a tool call — ids are derived receipts.
- Never prune receipts by age or count.
- Receipts never enter the compiled prompt or the facts extraction payload, and memory tools cannot edit them.
- Do not emit ledger lines outside `format.ts`/`ledger-file.ts`.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/agreements
bun run --cwd packages/memory-core typecheck
```
