# src/hooks/think-mode/ - Think-Keyword Model Switcher

**Score ~9** (7 files; distinct domain - keyword-triggered thinking/model switch).

## OVERVIEW

A session hook that detects a "think"/"ultrathink" keyword (plus a large multilingual keyword set) in the user's prompt and switches the session into a higher-reasoning model/variant for that turn. Wired into `createSessionHooks()` as `thinkMode`.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: re-exports detector, switcher, types + `clearThinkModeState`, `createThinkModeHook` |
| `detector.ts` | `detectThinkKeyword(text)`, `extractPromptText(parts)` - keyword matching |
| `switcher.ts` | Model/variant switch logic |
| `hook.ts` | `createThinkModeHook()` - session event handler |
| `types.ts` | Option/state types |

## CONVENTIONS

- Detection strips fenced code blocks and inline code BEFORE matching, so `think` inside code never triggers (`removeCodeBlocks`).
- English pattern is `\b(?:ultrathink|think)\b`; a combined regex ORs in a long multilingual keyword list (Korean, CJK, Indic, Arabic, European, etc.).
- `extractPromptText` only considers `type === "text"` parts.

## ANTI-PATTERNS

- Do not match the keyword inside code blocks/inline code - always strip code first.
- Keep the keyword a whole-word match; do not let it fire inside identifiers.

## COMMANDS

- `bun test packages/omo-opencode/src/hooks/think-mode`
- OpenCode-wired: run `opencode-qa` (isolated XDG + evidence) for behavior changes.
