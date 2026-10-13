---
name: sisyphus-junior-agent
description: Developer reference for the Sisyphus-Junior category-spawned executor agent -- model variants and discipline.
---

# src/agents/sisyphus-junior/ -- Category-Spawned Executor

**Generated:** 2026-05-18 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

12 files. Sisyphus-Junior is a focused task executor spawned by `delegate-task` when category routing requires it. Runs in subagent mode with its own fallback chain. Does not delegate further; executes directly.

## FILES

| File | Purpose |
|------|---------|
| `agent.ts` | `createSisyphusJuniorAgentWithOverrides()` factory, `getSisyphusJuniorPromptSource()` routing, `SISYPHUS_JUNIOR_DEFAULTS` |
| `index.ts` | Barrel exports |
| `default.ts` | Base/Claude prompt: todo discipline, verification, termination rules |
| `gemini.ts` | Gemini-optimized prompt variant |
| `gpt.ts` | Base GPT prompt variant |
| `gpt-5-4.ts` | GPT-5.4-native prompt variant |
| `gpt-5-5.ts` | Shared GPT-5.5/GPT-5.6 model-aware prompt variant |
| `kimi-k2-6.ts` | Kimi K2.6 prompt variant |
| `kimi-k2-7.ts` | Kimi K2.7-native prompt variant |
| `kimi-k3.ts` | Kimi K3-native prompt variant (reasoning depth with built-in stop conditions) |
| `glm-5-2.ts` | GLM-5.2-native prompt variant |
| `index.test.ts` | Unit tests |

## VARIANT SELECTION

`agent.ts` `getSisyphusJuniorPromptSource()` checks in this order:
- Kimi K3 (`isKimiK3Model`) -> `kimi-k3.ts`
- Kimi K2 code model (`isKimiK2CodeModel`) -> `kimi-k2-7.ts`
- Other Kimi K2 -> `kimi-k2-6.ts` (source id `kimi-k2`)
- GPT-5.5, GPT-5.6 or GPT-6 -> shared `gpt-5-5.ts` prompt family with model-aware identity
- Contains "gpt-5.4" -> `gpt-5-4.ts`
- Contains "gpt" -> `gpt.ts`
- Contains "gemini" -> `gemini.ts`
- Contains "glm" -> `glm-5-2.ts`
- Default -> `default.ts` (Claude, etc.)

## KEY BEHAVIORS

- Mode: `subagent` (uses own fallback chain, ignores UI selection)
- Default model: `anthropic/claude-sonnet-5` (override model, then system default, win over it)
- Default temperature: `0.1` (`SISYPHUS_JUNIOR_DEFAULTS`)
- Fallback chain (`packages/model-core/src/agent-model-requirements.ts`): claude-sonnet-5 -> kimi-k3 -> gpt-5.6-sol medium -> minimax-m3 -> MiniMax-M3 -> minimax-m2.7 -> big-pickle
- Blocked tools: `task` only (`BLOCKED_TOOLS`)
- `call_omo_agent` explicitly allowed so subagents can spawn explore/librarian
- Max tokens: 64000; color `#20B2AA`
- GPT models: `reasoningEffort` from override, default "medium"; GLM models: no thinking block
- Every other model gets `buildClaudeThinkingConfig(model)` merged in
