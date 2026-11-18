---
name: sisyphus-junior-agent
description: Sisyphus-Junior factory, shared Native model core, and OpenCode role composition.
---

# Sisyphus-Junior

**Generated:** 2026-11-18 / c2691818a9

The category-spawned executor renders its model core through
`native-model-prompt.ts` and appends its OpenCode role through
`default.ts`. There are no local Native model-core copies or routing tables.
The public Senpi preset contract owns both selection and content.

## SURFACE

- `index.ts` exports `SISYPHUS_JUNIOR_DEFAULTS`, `buildSisyphusJuniorPrompt`,
  `createSisyphusJuniorAgentWithOverrides` and the prompt-source type.
- `agent.ts` owns the factory, defaults, permission merging and user appends.
- `index.test.ts` covers configuration and permission behavior.
- The parent `per-agent-prompt-parity.test.ts` compares rendered content over the catalog.

## DEFAULTS

Mode is subagent, the default model is anthropic/claude-sonnet-5, temperature is
0.1 and maxTokens is 64000; override order is `override.model` → `systemDefaultModel`
→ defaults. The task tool is denied for every model;
call_omo_agent remains allowed for research. GPT uses medium reasoning by
default; GLM has no injected thinking, and the existing Claude thinking
configuration applies elsewhere. User file-backed prompt appends still
resolve through the existing URI boundary and remain last.
