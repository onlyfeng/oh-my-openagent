# components/claim-receipt/ — Answer Claim Verification

## OVERVIEW

18 files, ~2,540 LOC. Classifies claims made in an answer, correlates them
against the evidence timeline (tool calls, eval cells, background jobs,
edits, notifications), and renders nudges when a claim outruns its evidence.
Registration/session state lives in `index.ts`; the classifier/matcher
runtime is deferred through the `#omo-claim-receipt-runtime` import.
**Score 17** (18 files, deferred runtime, shadow-mode flag). Parent:
[`src/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `createClaimReceiptComponent`, `CLAIM_RECEIPT_NUDGE_FLAG`, session epochs, verdict records |
| `runtime.ts` | `evaluateAnswer` — the deferred API |
| `runtime-loader.ts` | Shared in-flight import; failed imports cleared so a later answer retries |
| `classify.ts` | Claim extraction regex rules (code/quotes/relayed text masked) |
| `evidence.ts` | `TimelineBuilder` — correlates tool results, eval cells, jobs, edits |
| `match.ts`, `verdict.ts` | Claim-to-evidence matching, verdict constants/renderers |
| `invocations.ts`, `shell-words.ts`, `cell-commands.ts` | Shell/runner/eval-cell command parsing |
| `signatures.ts` | Test/build/typecheck output judges |

## CONVENTIONS

- Nudge flag defaults OFF (shadow mode): verdicts are recorded even when not
  surfaced.
- Korean patterns avoid `\b` (fails after Hangul).
- Shell recognition is conservative: only known command positions count;
  unknown positions fail closed. Cell failures/writes do not count as failed
  checks/edits; lookups establish existence only.
- Evidence strictly before the answer position counts; later evidence cannot
  support an earlier claim.

## ANTI-PATTERNS

- Do not surface nudges by default — shadow mode is intentional pending
  replay precision.
- Do not interpret arbitrary strings in eval source as shell commands.
- Do not treat cell failures or cell writes as authoritative receipts.

## QA

```bash
bun test packages/omo-senpi/src/components/claim-receipt
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
