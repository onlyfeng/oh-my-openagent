# side-panel/popups - Terminal Text Popups

**Generated:** 2026-11-18 | c2691818a9

Modal text viewer/notice popups for the Senpi side panel. Distinct domain: the one place the side panel claims wheel input and budgets terminal rows/columns for overlay content.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `text-popup.ts` | `createTextPopup`, `PopupComponent` / `PopupOptions` - render + dismiss lifecycle. |
| `viewport.ts` | `popupBudget`, `clampScroll` - visible-row math against the panel width/height. |
| `wheel.ts` | `parsePanelWheelEvent` - decode SGR mouse-wheel reports; viewer listeners claim the event. |
| `open.ts` | `openPanelViewer` - the public open entry. |
| `*.test.ts` | bun test coverage (open/text-popup/viewport/wheel). |

## CONVENTIONS

- Wheel events are SGR mouse reports (`\x1b[<...M/m`); `parsePanelWheelEvent` is the only decoder.
- Popup text is sanitized/truncated for terminal rendering; never trust upstream content to be single-width.
- Close keys: `esc`/`q`/Enter (`\r`/`\n`) plus Kitty-protocol `CSI 27 u` (modifiers allowed); scroll: arrows or `j`/`k` (±1), PageUp/PageDown (±10). A viewer's `onKey` runs first and claims the key.
- `popupBudget`: `POPUP_CHROME_ROWS = 4`, default ratio 0.72 (clamped ≤1), terminal-rows fallback 24. `render` returns exactly `budget.total` lines - the host slices overflow off the END, so one extra row silently eats the closing border.
- `openPanelViewer` falls back to `ui.notify` (sanitized rows joined by newlines) when the host lacks `ui.custom`; wheel notch = `WHEEL_ROWS` (3 rows).

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/side-panel/popups
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

Senpi-wired; `senpi-qa` skill + evidence under `.omo/evidence/omo-senpi-adapter/<slug>/`.
