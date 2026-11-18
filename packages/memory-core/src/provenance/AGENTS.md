# memory-core/src/provenance/ - Claim & Consent Provenance

**Score ~10** (8 files, ~1.6k LOC incl. review corpus; distinct domain - claim-receipt + consent rules).

## OVERVIEW

Harness-neutral provenance contracts for memory writes: claim receipts (who asserted what, with a verbatim user quote) and consent-exchange rules. Pure functions over validated inputs; no harness imports. These rules are a public contract consumed by the memory tooling and `claim-receipt` components.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: claim constants/rules + parse/render/resolve + types |
| `claim.ts` | `CLAIM_SOURCES`, `claimReceiptTrailers`, `isClaimSource`, `parseClaimReceipt`, `renderClaimReceipt`, `resolveClaimReceipt`, `proposalId`, `MEMORY_AUTHORITY_RULE`, `USER_QUOTE_RULE`; types `ClaimInput`, `ClaimReceipt`, `ClaimSource`, `ConsentExchange` |
| `consent.ts` | Consent-exchange logic |
| `consent-corpus.shapes.ts` + `consent-corpus.review-rows.json` | Review corpus shapes + data (~1.3k LOC JSON fixture) |

## CONVENTIONS

- Claim sources are a closed set (`CLAIM_SOURCES`, `isClaimSource`); `USER_QUOTE_RULE` requires a verbatim user quote for user-sourced claims.
- Remain harness-neutral - no OpenCode/Codex/Senpi/Pi imports.

## ANTI-PATTERNS

- Do not bypass `GitMemoryRepo`, domain locks, path validation, frontmatter parsing, or atomic commits (memory-core boundary).
- Do not mutate `read_only` blocks; do not store behavioral patterns in people cards.

## COMMANDS

- `bun test --timeout 20000 packages/memory-core/src/` · `bun run --cwd packages/memory-core typecheck`
