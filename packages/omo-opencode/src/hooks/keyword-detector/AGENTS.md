# src/hooks/keyword-detector/ -- Mode Keyword Injection

**Generated:** 2026-11-18 / c2691818a9 (update pass)

## OVERVIEW

Transform Tier hook on `messages.transform`. Scans the first user message for mode keywords and injects mode-specific system prompts. The detector and routing logic stay in `src/hooks/keyword-detector/`; prompt bodies now live in [`packages/prompts-core/prompts/`](../../../../prompts-core/prompts) so they can be shared by future harness adapters.

This matches the package layering direction in [`ROADMAP.md`](../../../../../ROADMAP.md): `packages/prompts-core` owns static prompt content, while this OpenCode hook owns keyword detection, model routing, and message injection.

## KEYWORDS

| Keyword | Pattern | Effect |
|---------|---------|--------|
| `ultrawork` / `ulw` | `ULTRAWORK_PATTERN` (from `constants.ts`; ignores `ulw-*` skill names, shared with the model override) | Full orchestration mode: parallel agents, deep exploration, relentless execution |
| Team mode | `TEAM_PATTERN` (from `team/`) | Forces orchestration via `team_*` tools when user invokes `team mode` / `team-mode` / `team_mode` / `teammode`; instructs user to enable `team_mode.enabled` if tools are absent and reminds lead to run the closure sequence once every task is terminal |
| Hyperplan mode | `HYPERPLAN_PATTERN` (from `hyperplan/`) | Loads the `hyperplan` skill and injects adversarial planning mode guidance |
| Hyperplan-ultrawork combo | `HYPERPLAN_ULTRAWORK_PATTERN` (from `constants.ts`) | Prepends the combo banner, requires the `hyperplan` skill, then appends the routed ultrawork message |

## STRUCTURE

```
index.ts / hook.ts / constants.ts / types.ts   # barrel, createKeywordDetectorHook() handler, KEYWORD_DETECTORS, types
detector.ts                                     # detectKeywordsWithType() + extractPromptText()
ultrawork/   # getUltraworkMessage() router, source-detector.ts, thin .md loaders per variant
team/         # TEAM_PATTERN + prompt constant from prompts-core
hyperplan/    # HYPERPLAN_PATTERN + prompt constant from prompts-core
```

## PROMPT CONTENT LOCATIONS

All prompt bodies are markdown in `packages/prompts-core/prompts/`: ultrawork variants at `ultrawork/{default,gpt,gemini,glm,planner}.md`, team/hyperplan at `mode/{team,hyperplan}.md`. The `team/`+`hyperplan/` `default.ts` keep the regex triggers in this hook layer and import markdown-backed constants from `@oh-my-opencode/prompts-core`; the ultrawork loaders import markdown with Bun's `.md` text loader so the exact prompt bytes are bundled into `dist/index.js`.

## ULTRAWORK VARIANT ROUTING

[`ultrawork/source-detector.ts`](ultrawork/source-detector.ts) decides the ultrawork source in priority order:

1. Planner agents (`prometheus`, `planner`, or normalized `plan`) route to `planner.md`.
2. GPT family models, as detected by `isGptModel(modelID)`, route to `gpt.md`.
3. Gemini family models, as detected by `isGeminiModel(modelID)`, route to `gemini.md`.
4. GLM family models, as detected by `isGlmModel(modelID)`, route to `glm.md`.
5. Everything else routes to `default.md`.

[`ultrawork/index.ts`](ultrawork/index.ts) exposes `getUltraworkMessage(agentName, modelID)`, switches on that source, and returns the loaded markdown body.

## DETECTION LOGIC

```
chat.message (user input)
  -> extractPromptText(parts)
  -> isSystemDirective? skip
  -> removeSystemReminders(text)  # strip <SYSTEM_REMINDER> blocks
  -> detectKeywordsWithType(cleanText, agentName, modelID, disabledKeywords)
  -> isNonOmoAgent(agentName)? filter keyword injection
  -> isPlannerAgent(agentName)? filter standalone ultrawork
  -> for each detected keyword: inject mode message into output
```

## CONFIG

```jsonc
{
  "keyword_detector": {
    // Skip injection for any keyword in this list.
    "disabled_keywords": ["team", "hyperplan"]
  }
}
```

Default: empty/missing means every detector is active. Schema lives at [`src/config/schema/keyword-detector.ts`](../../config/schema/keyword-detector.ts).

## GUARDS

- **System directive skip**: Messages tagged as system directives are not scanned (prevents infinite loops)
- **Planner agent filter**: Prometheus/plan agents do not receive `ultrawork` injection
- **Non-OMO agent filter**: OpenCode built-in Builder/Plan agents do not receive keyword injection
- **Session agent tracking**: Uses `getSessionAgent()` to get actual agent (not just input hint)
- **Model-aware messages**: `getUltraworkMessage(agentName, modelID)` adapts message to active model
- **Ultrawork source routing**: `ultrawork/ultrawork-source-routing.test.ts` pins agent/model-to-prompt-source routing for `getUltraworkSource`
