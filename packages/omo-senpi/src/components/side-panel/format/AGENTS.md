# format - Terminal Text Formatting Primitives

**Score:** 9 (8 files, 722 LOC, 4 pure modules + 4 test twins, consumed across side-panel renderers). No barrel on purpose: import the module you need.

## OVERVIEW

Pure text-metrics layer for the side panel: visible-width measurement, truncation/wrapping, sanitization, bars, and compact units - all ANSI-safe.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Visible width / pad / truncate / wrap | `truncate.ts` - `visibleWidth`, `padVisible`, `truncateVisible`, `truncateVisibleStart`, `wrapVisible` |
| Control-effect removal | `sanitize.ts` - `sanitizeTerminalText` |
| Progress bars | `bars.ts` - `bar` (`BarOptions`) |
| Compact units | `units.ts` - `compactTokens`, `duration`, `compactCost` |

## CONVENTIONS

- ANSI escape sequences never count as printable cells: every width computation goes through `visibleWidth`, never `String.length`.
- Sanitize BEFORE theme coloring: sanitized text must not carry its own color; C0 controls, tabs, and line breaks are normalized first.
- Each module has a colocated `*.test.ts` twin with the same name - extend both together.

## ANTI-PATTERNS

- Do not measure strings with `.length` anywhere in the side panel - double-width runes and escapes break layouts (this is why `truncate.ts` exists).
- Do not color inside these modules; they are theme-neutral by design.
- Do not add a barrel - the point is minimal import cost per renderer.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/side-panel/format
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
