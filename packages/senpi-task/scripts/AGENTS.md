# senpi-task/scripts/ - Manual QA Probes

**Score ~8** (5 files; distinct domain - hermetic manual QA drivers).

## OVERVIEW

Executable manual-QA entrypoints (not library exports) for the senpi-task engine. Each asserts behavior and prints its result for manual inspection; see CONVENTIONS for the two invocation modes. `manual-qa.ts` imports the public surface (`createTaskRecord`, `createTaskRecordStore`, `messageability`, `resolveStateDir`, `transitionTaskRecord`) directly from `../src/index`.

## WHERE TO LOOK

| File | Focus |
|------|-------|
| `manual-qa.ts` | Core record/store/transition/messageability probe (~174 LOC) |
| `manual-category-qa.ts` | Category resolution incl. gated/deprecated/throwing-registry cases (~347 LOC - largest) |
| `manual-agents-qa.ts` | Agent loading: frontmatter `models`/`tools`, broken + symlinked roots, escaped external paths |
| `manual-output-qa.ts` | `task_output` tail / lost-record / cross-session fail-closed views |
| `manual-routing-policy-qa.ts` | Fallback-chain model selection across fake registries |

## CONVENTIONS

- Two invocation modes: `manual-qa.ts` / `manual-agents-qa.ts` require `<evidence-dir>` (argv[2]; a missing arg throws with the usage string) and build `<evidence>/manual-*-fixture`; `manual-category-qa.ts` / `manual-routing-policy-qa.ts` are zero-arg in-memory probes over a fake `SenpiModelRegistryPort` (no filesystem); `manual-output-qa.ts` is zero-arg and mkdtemps its store under `os.tmpdir()`.
- Import boundary: `manual-qa.ts` / `manual-agents-qa.ts` use the public barrel (`../src/index`); the others import deeper paths (`../src/category`, `../src/store`, `../src/tools/output`).
- Isolated fixture roots under the supplied evidence dir; deterministic timestamps/PIDs; JSON summary printed (`manual-output-qa.ts` prints labeled `=== SECTION` console output instead); fixtures cleaned up.

## ANTI-PATTERNS

- Must NOT use real user state, leak sentinel secrets, accept corrupt/malformed task records, apply illegal lifecycle transitions, or permit traversal outside the state directory.

## COMMANDS

- `bun packages/senpi-task/scripts/manual-qa.ts <evidence-dir>` · `manual-agents-qa.ts` takes the same arg · the other three are zero-arg
