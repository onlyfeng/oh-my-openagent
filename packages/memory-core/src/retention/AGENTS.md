# retention — Runtime Memory Retention

**Score:** 18 (26 files, 5.9k LOC, >20 files, index barrel, runs inside the memory runtime). Parent: [`packages/memory-core/AGENTS.md`](../../../AGENTS.md).

## OVERVIEW

Evicts expired runtime state: the retention pass prunes by policy, the cap tier evicts by measured bytes, the host sweep step integrates it into scheduled maintenance, and sidecar pruning clears stale Kibitzer artifacts. Barrel: `index.ts`.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Run a retention pass | `pass.ts` — `runRuntimeRetention`, `summarizeRuntimeRetention` |
| Byte-cap tier (measure + evict) | `cap.ts` — `describeRuntimeCapExcess`, `measureRuntimeCapEvictableBytes` (562 LOC, largest prod file) |
| Scheduled maintenance integration | `host-step.ts` — `createRetentionHostSweepStep`, `resolveForeignIdentityDiscardUnreflected` |
| Policy defaults | `policy.ts` — `DEFAULT_RUNTIME_RETENTION`, `RuntimeRetentionPolicy` |
| Cursor/byte budget types | `budget.ts` (`RetentionCursor`, `RetentionStore`) |
| Resume stamping | `stamp.ts` — `RETENTION_STAMP_CONFIG_KEY`, `retentionCursorPath`, `retentionResumePending` |
| Last-run log | `log.ts` — `readLastRetentionLogEntry` |
| Unreflected gate signal | `reflection-health-signal.ts` — `readUnreflectedRetentionGate` |
| Kibitzer sidecars | `sidecars.ts` — `pruneKibitzerSidecars` + `KIBITZER_*` constants |
| Runtime size measurement | `size.ts` — `measureRuntimeSize`, `DEFAULT_RUNTIME_MAX_BYTES`, `formatRuntimeBytes`, `runtimeSizeStanding` |
| Past-retention accounting | `past-retention.ts` — `countRuntimePastRetention`, `measureUnreflectedJournalBytes` |
| Supporting scans | `journals.ts`, `recall-ledgers.ts`, `reflection-runs.ts`, `completions.ts`, `legacy.ts`, `guard.ts` |

## CONVENTIONS

- `../reflection/reflection-lock-bundle.ts` re-exports the host-step, cap, and log helpers for the reflection lock bundle — keep that surface aligned when renaming.
- `retention.test-support.ts` is shared by the cap, host-step, log, main retention, and scan-robustness suites; change it with all five in view.
- Pass → policy/store pruners; host-step → pass + cap. Do not call `cap.ts` directly from host code.

## ANTI-PATTERNS

- Never regress the retention stamp/cursor (`retentionResumePending` exists to resume, not restart).
- Never evict unreflected runtime state past the gate without `readUnreflectedRetentionGate`'s verdict.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/retention
bun run --cwd packages/memory-core typecheck
```
