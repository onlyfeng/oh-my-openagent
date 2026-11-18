# scripts/ - ulw-research runtime (delta over the parent map)

**Generated:** 2026-11-18 / c2691818a9

170 files (plus `tests/`). Score ~30 (file count, symbol density, exports, sibling-import fan-in). The parent [ulw-research/AGENTS.md](../AGENTS.md) already maps the tree, CLI surface, sync overlay, conventions and anti-patterns - read it first. This file covers only clusters and invariants the parent tree omits.

## CLUSTERS NOT IN THE PARENT TREE

| Cluster | Files | Verified role |
|---------|-------|---------------|
| Copy-polish pipeline | `polish-rules.mjs`, `polish-apply.mjs`, `polish-diff.mjs`, `polish-project.mjs` (+ `polish-*`, `copy-returns-polish` tests) | Checked proofread-lane edits: text nodes only, whole-document markup skeleton byte-identical; an edit containing `<>&` or touching a character reference is refused before anything else |
| Meaning verification | `fact-pairs.mjs`, `meaning-check.mjs`, `fact-commands-meaning.mjs` (+ tests) | (source, sentence) pairs - quote, claim, translation - fed to separate reader lanes; drift confirmed only when both tiers name the same `drift_kind`, warned on one; pairs no reader judged land in `unverified` (fail-closed) |
| Fact command splits | `fact-commands-fill.mjs`, `fact-commands-verify.mjs`, `fact-commands-polish.mjs`, `fact-commands-meaning.mjs` | Lazy-loaded halves behind `fact-commands.mjs` |
| Retention bar | `copy-returns.test.ts` | Every shared table/list/regex/classifier has ONE owner export; a reintroduced local copy in a consumer fails (#11156) |
| Fill-map authority | `value-rules-*.test.ts`, `value-mode-unit.test.ts` | The pre-polish fill map stays authoritative for `fact pairs --changed-since`; `fact derive` takes its time from `--now` only (#11290) |

## INVARIANTS (beyond the parent's)

- `polish-rules.mjs` defect codes: `polish_diff_fail` (names the first differing markup token), `polish_hedging_decreased`, `polish_text_churn` (bounded by `POLISH_CHURN_LIMIT`), `polish_link_changed`, `polish_name_changed`. Ambiguous markup is refused before the byte comparison; severity lives only in `contracts.DEFECT_CODES`.
- `polish-apply.mjs`: `before` must occur exactly once (whitespace runs match any run) and must not cut through a fact span; guards are `POLISH_MARKUP_REASON` / `POLISH_REFERENCE_REASON`.
- `fact-derive.mjs` dispatches every arithmetic/generated field through the operation registry (`ruleOf`), verifies transitive trust, and appends only on success - no ad-hoc computation path may bypass it.
- `delivery-barrier.test.ts` enumerates `DEFECT_CODES` itself, so every major/integrity code automatically gets a refuses-delivery test - never hand-maintain that list.

## ANTI-PATTERNS

- Never add a second copy of a shared table, regex or classifier - the retention bar fails it.
- Never let a meaning verdict skip silently: unjudged pairs must surface as `unverified`, not disappear.

## COMMANDS

Same as the parent: `bun test --timeout 20000 packages/shared-skills/skills/ulw-research/scripts`; runtime `node scripts/report-tools.mjs <command>` (and `node scripts/claim-kinds.mjs` with module-defined args).
