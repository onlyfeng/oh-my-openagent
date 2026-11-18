# Atlas

**Generated:** 2026-11-18 / c2691818a9

Primary orchestration agent (`MODE: "primary"`, temperature 0.1, color `#10B981`): orchestrates work via `task()` to complete ALL tasks in a todo list until fully done. `agent.ts` builds the agent (`createAtlasAgent(ctx)` + `atlasPromptMetadata`); `index.ts` exports only those two plus `AtlasPromptSource` / `OrchestratorContext` types.

## PROMPT ROUTING

The model core comes from `buildNativeAgentPrompt(model, ...)`; `getAtlasPromptSource = getNativePromptSource`. `getAtlasRoleSource(model)` resolves the OpenCode role variant through prompts-core (`atlasPromptVariants`, `resolveVariant`), evaluated in this order:

1. Claude Opus 4.7 → `opus-4-7.md` (literal-following + explicit fan-out push)
2. GPT family → `gpt.md`; Gemini family → `gemini.md`
3. Kimi K3 → `kimi-k3.md` (checked first among Kimi); Kimi K2.7 → `kimi-k2-7.md`; generic Kimi K2.x → `kimi.md`
4. GLM family → `glm.md`; default (Claude 4.6 family) → `default.md`

Unknown resolved variant throws `AtlasPromptVariantError`. Role variants in prompts-core are NOT copies of Native model cores; there is no local Native core copy here.

## DYNAMIC PROMPT BUILD (`buildDynamicOrchestratorPrompt`)

Merges user categories (`mergeCategories`), then composes: category section, agent selection section, skills section, decision matrix, category-skills delegation guide (`prompt-section-builder.ts` + `../dynamic-agent-prompt-builder`), agent identity (`buildAgentIdentitySection`). `loadPromptSync` injects them at placeholders `{CATEGORY_SECTION}`, `{AGENT_SECTION}`, `{DECISION_MATRIX}`, `{SKILLS_SECTION}`, `{{CATEGORY_SKILLS_DELEGATION_GUIDE}}`; the final prompt is identity + role body wrapped in `withRefusalRule`.

## TESTS

- `prompt-routing.test.ts` pins model → variant routing.
- `prompt-runtime-injection.test.ts` protects live context data; the parent catalog `per-agent-prompt-parity.test.ts` compares rendered Native content over the catalog.

## ANTI-PATTERNS

- Category and agent are mutually exclusive in delegation; never provide both.
- Don't add local role/prompt copies: edit prompts-core variants or the dynamic builders.
