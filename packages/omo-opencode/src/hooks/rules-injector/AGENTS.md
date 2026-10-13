# src/hooks/rules-injector/ — Conditional Rules Injection

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

40 `.ts` files (~3.3k LOC incl. tests). The `rulesInjectorHook`: Tool Guard Tier hook that auto-injects AGENTS.md (and similar rule files) into context when a file in a directory is read, written, or edited. Proximity-based: closest rule file to the target path wins. Scanner/parser/matcher/distance primitives are re-exported from `@oh-my-opencode/rules-engine`.

## HOW IT WORKS

```
tool.execute.after (read/write/edit/multiedit)
  → Extract file path from tool output
  → Find rule files near that path (finder.ts)
  → Already injected this session? (cache.ts)
  → Inject rule content into tool output (injection-processor.ts)
```

Session delete/compact clear the session and project-root caches (`hook.ts`).

## TRACKED TOOLS

`["read", "write", "edit", "multiedit"]` — triggers only on file manipulation tools.

## KEY FILES

| File | Purpose |
|------|---------|
| `hook.ts` | `createRulesInjectorHook()` — wires cache + injector, handles tool events |
| `injection-processor.ts` | `createRuleInjectionProcessor()`: orchestrates find, cache, inject |
| `injector.ts` | Facade re-exporting the processor + parsed-rule-cache controls (equivalence pinned by `injector-facade.test.ts`) |
| `finder.ts` | Re-export barrel: `findRuleFiles()` + `calculateDistance()` |
| `rule-file-finder.ts` | Walk directory tree to find AGENTS.md / .rules files |
| `rule-file-scanner.ts` | Scan for rule files in a directory |
| `matcher.ts` | Match file paths against rule file scope |
| `rule-distance.ts` | Calculate path distance between file and rule file |
| `project-root-finder.ts` | Find project root (stops at .git, package.json) |
| `output-path.ts` | Extract file paths from tool output text |
| `cache.ts` | `createSessionCacheStore()` — per-session injection dedup |
| `storage.ts` | Persist injected paths across tool calls |
| `parser.ts` | Parse rule file content |
| `constants.ts` | Rule file names: `AGENTS.md`, `.rules`, `CLAUDE.md` |
| `parsed-rule-cache.ts` / `rule-scan-cache.ts` / `match-decision-cache.ts` | Parse, scan, and match-decision caches |
| `transcript-hydration.ts` / `injection-output.ts` / `rule-match-reason.ts` / `path-resolution.ts` | Rehydrate injected state, format output, explain match, resolve paths |
| `types.ts` / `injection-types.ts` | `RuleInfo`, `InjectedRulesData` (+ re-exported `RuleFileCandidate`, `RuleMetadata` from rules-engine) |

`finder.ts`, `parser.ts`, `matcher.ts`, `project-root-finder.ts`, `rule-distance.ts`, `rule-file-finder.ts` are thin re-export shims over `@oh-my-opencode/rules-engine`; change logic there.

## RULE FILE DISCOVERY

Priority (closest → farthest from target file):
1. Same directory as target file
2. Parent directories up to project root
3. Project root itself

Same-distance tie: all injected. Per-session dedup prevents re-injection.

## TRUNCATION

Uses `DynamicTruncator` — adapts injection size based on model context window (1M context models get full content, smaller models get truncated summaries).
