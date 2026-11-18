# ulw-research - Research Orchestration + report-tools Runtime

**Generated:** 2026-11-18 / c2691818a9 (update pass)

## OVERVIEW

`SKILL.md` is the research orchestration contract (team-first saturation, claim graph, convergence, delivery). `scripts/` is the deliverable runtime that contract calls: `report-tools.mjs`, a zero-dependency Node CLI for the deliverable phase (outcome manifest, static/fidelity/layout gates, bounded repair, format extraction). `references/` holds the edition-neutral deliverable contract and the defect glossary. Earned this file: the only shared skill whose runtime is overlaid into a second, separately authored edition (the senpi native skill) at sync time.

## STRUCTURE

```
ulw-research/
├── SKILL.md                     # shared-edition prose contract (router to the references)
├── references/                  # 10 files:
│   ├── deliverable-phase.md / report-gates.md / report-visualization.md   # lanes + state + manifest; one row per defect code; which visual fits which finding
│   ├── fact-ledger.md / fact-ledger-grammar.md / fidelity-claim-kinds.md / unit-rules.md   # ledger + claim-kind + unit contracts (grammar tables rendered from KIND_RULES / CLAIM_KINDS)
│   ├── markdown-blocks.md       # the light Markdown block model's CommonMark paragraph-interruption rule
│   └── research-collection.md / research-delivery.md   # phase contracts: decompose + journal before spawning; settle contested claims by running code
└── scripts/                     # 88 zero-dependency .mjs modules (+ co-located *.test.ts and tests/fixtures/, unshipped)
    ├── report-tools.mjs / report-tools-commands.mjs    # CLI entry + frozen COMMANDS dispatch; one function per subcommand
    ├── cli-support.mjs / entry-guard.mjs / contracts.mjs   # flags, CliError, exit codes, atomic JSON, symlink-safe entry; frozen vocabulary (DEFECT_CODES, FIDELITY_CODES, GATE_NAMES, manifest + repair-state validators)
    ├── outcome.mjs / outcome-start.mjs / repair-tracker.mjs   # manifest lifecycle + run clock (--started-at, dir-name fallback); bounded repair (attempts, plateau, oscillation, wall clock, integrity codes)
    ├── gates-static.mjs (+ gates-text/figures/structure/fidelity.mjs) / sources-citation.mjs   # static gates G1-G17; the one Sources predicate (claim segments vs citation fields)
    ├── gates-layout.mjs / layout-probe.mjs    # layout gates L1-L5 over the in-page probe source
    ├── fidelity-*.mjs (14)     # locked-brief grammar (`## Fidelity` of brief.md) + policy, records/delivery barrier, per-sentence repair state, csv/lexicon/trace/value-unit machinery
    ├── fact-commands.mjs + fact-commands-{fill,verify,meaning,polish}.mjs   # `fact add|quote|retract|export|fill|verify|derive` dispatch; fill/verify are reader-facing; meaning = `fact pairs`/meaning-check halves; polish = proofread-gate halves
    ├── fact-ledger{,-schema,-kinds}.mjs / fact-derive.mjs / fact-recompute.mjs / fact-locate.mjs / fact-pairs.mjs / fact-fill.mjs   # ledger IO + writers + KIND_RULES, script-computed derived rows + tamper detector, deterministic snapshot locators, meaning-verifier pairs
    ├── unit-rules{,-shapes}.mjs / unit-{pairs,algebra,precision,metadata,errors}.mjs / units.mjs / fact-expr.mjs / numerals.mjs   # OPERATION_RULES, typed operand pairs, canonical unit text, the expr parser/evaluator
    ├── source-{commands,refresh,snapshot,view}.mjs / raw-{decode,locator}.mjs / html-text-map.mjs   # snapshot IO (`raw.bin` + locator text) and the one snapshot reading (charset decode, byte map, visible text)
    ├── claim-kinds.mjs / claims-index.mjs / placeholder.mjs / meaning-check.mjs   # CLAIM_KINDS (+ --render-doc), claim-graph.md index + digest, placeholder marks, meaning-verifier verdict ingest
    ├── polish-{rules,apply,diff,project}.mjs   # proofread gate: the one rule set (per-edit + whole document) over a protected markup projection
    ├── html-lite.mjs / html-markup.mjs / html-foreign-object.mjs / md-lite.mjs / markdown-closers.mjs / entities.mjs   # HTML tokenizer/selector paths, byte-preserving polish boundaries, light Markdown block model, CommonMark closer runs
    ├── css-lite.mjs / css-hiding.mjs / design-spec.mjs / format-extract{,-css}.mjs   # CSS scanning, the hiding cascade, palette + lineage, design-spec draft from a pointed-at HTML/Markdown document
    ├── conversion-probe.mjs / number-grammar.mjs   # `source probe-conversion` numeric diff; the grouped-number grammar
    └── sentence-split.mjs / text-{normalize,similarity}.mjs / particles-ko.mjs / rational.mjs / lexicon.mjs   # text plumbing; Korean particle normalization as \u escapes
```

## SURFACE

`node "$SKILL_DIR/scripts/report-tools.mjs" <command>` - the frozen `COMMANDS` list (23): `check`, `layout-probe`, `repair decide`, `outcome init|set|gate|render|state|verify|finish|briefing`, `format-extract`, `fact add|quote|retract|export|fill|verify|derive`, `source snapshot|refresh|probe-conversion`, `claims index`, plus `--help --json`. Gates recorded by `outcome gate`, in order (`GATE_NAMES` in `contracts.mjs`): `static`, `fidelity`, `layout`, `visual`, `proofread` (`fidelity` is `fact verify` on every reader-facing edition; HTML, or Markdown with its fill map). Exit codes: `0` pass, `1` semantic failure (blockers, a failed verify, a blocked repair), `2` usage or IO. JSON on stdout, one summary line on stderr. Flags per command: `references/deliverable-phase.md` section 10.

## CONVENTIONS

- Zero runtime dependencies: plain ESM `.mjs`, `node:` builtins and sibling imports only, no `Bun.*`. Runs under Node >= 20 or Bun 1.4 (global `fetch` for `format-extract --from-url`).
- Each `.mjs` stays at or under 250 lines; split by responsibility, never by size.
- The CLI never launches a browser. The layout probe is printed as one async expression; the orchestrator evaluates it through the browser skill's owned headless engine and passes the result back with `check --layout`.
- Time is always injected (`--now`, `startedAt`); no module reads the clock inside a decision.
- Tests are co-located given/when/then bun tests; CLI tests spawn `node` with bun removed from `PATH`. Fixtures live only under `scripts/tests/fixtures/`, are depersonalized, and never ship.
- The senpi source dir (`packages/omo-senpi/skills/ulw-research/`) intentionally has no `scripts/` and fewer `references/` (its own three: `latex-report.md`, `research-collection.md`, `research-delivery.md`): the senpi sync overlays `scripts/` plus seven references from here byte-for-byte (report-gates, deliverable-phase, report-visualization, fact-ledger, fact-ledger-grammar, fidelity-claim-kinds, unit-rules) (`sharedAssets` in `packages/omo-senpi/plugin/scripts/native-skill-sources.mjs`), and both editions' sync suites assert the shipped copies equal these sources.
- References and scripts carry no Hangul (Korean match strings are `\u` escapes) and never three consecutive newlines (the senpi sync normalizes those and would break byte equality).

## ANTI-PATTERNS

- NEVER hand-edit a shipped copy under `packages/omo-senpi/plugin/skills/` or the packaged plugin tree; edit here and re-sync.
- NEVER add a severity override or a caller-supplied severity. The only sanctioned exception is `unsourced_number` following the design spec's `Lineage:` mode.
- Never let a gate or the repair loop silently skip: an unrunnable gate is recorded `not_run`, and an integrity code blocks delivery.
- Never name a browser library in the references or `SKILL.md`; route through the browser skill.
- Never test prose: pin machine-consumed values (CLI JSON, exit codes, extractor output, tracker decisions, defect-code vocabulary) and shipped-copy equality only.

## COMMANDS

```bash
# from the repository root
bun test --timeout 20000 packages/shared-skills/skills/ulw-research/scripts
node packages/shared-skills/skills/ulw-research/scripts/report-tools.mjs --help --json
node packages/omo-senpi/plugin/scripts/sync-skills.mjs && bun test packages/omo-senpi/src/skills-sync.test.ts
```

- Parent: [`packages/shared-skills/AGENTS.md`](../../AGENTS.md).
