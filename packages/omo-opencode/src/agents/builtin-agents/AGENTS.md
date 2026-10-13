---
name: builtin-agents-factory-layer
description: Conditional factory wrappers that apply overrides, model resolution, skill filtering, and provider gating to the 11 agent definitions.
---

# src/agents/builtin-agents/ -- Conditional Factory Layer

**Generated:** 2026-05-18 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

Conditional factory layer beneath the 10 raw factories in the `agentSources` registry of `src/agents/builtin-agents.ts` (Prometheus is built separately by `plugin-handlers/prometheus-agent-config-builder.ts`). Each `maybeCreateXXXConfig` wrapper decides whether an agent registers, resolves its model, applies user overrides, filters skills, and returns the final `AgentConfig` -- or `undefined` if the agent is disabled or requirements are not met. `createBuiltinAgents()` collects them into a `Record<string, AgentConfig>` consumed by `plugin-handlers/agent-config-handler.ts`.

## FILE CATALOG

| File | Purpose |
|------|---------|
| `agent-overrides.ts` | `applyCategoryOverride`, `mergeAgentConfig`, `applyOverrides`: category expansion, `deepMerge` of model/temp/prompt/permissions, `file://` prompt resolution; `reasoning` wins over deprecated `variant`, which is kept only when `reasoning` is absent |
| `agent-overrides.test.ts` | Override merge and reasoning/variant precedence tests |
| `model-resolution.ts` | `applyModelResolution` (override -> category -> provider fallback -> system default) + `getFirstFallbackModel` |
| `resolve-file-uri.ts` | Converts `file://` paths to absolute paths, bounds them to project root, reads content for prompt append |
| `resolve-file-uri.test.ts` | Tests for URI decoding, path expansion, project-root bounds, missing-file handling |
| `environment-context.ts` | Appends `createEnvContext()` block to agent prompt unless `disableOmoEnv` is set |
| `available-skills.ts` | `buildAvailableSkills` -- merges builtin skills with discovered user skills, filters disabled |
| `available-skills.test.ts` | Tests for builtin + discovered skill merging and disabled filtering |
| `sisyphus-agent.ts` | `maybeCreateSisyphusConfig` -- checks disabled list, model requirements, applies overrides + frontier tool schema guard + GPT patch guard |
| `sisyphus-agent.test.ts` | Tests for disabled-agent filtering, model resolution, override application, first-run fallback behavior |
| `hephaestus-agent.ts` | `maybeCreateHephaestusConfig` -- provider gating (`requiresProvider`), category override support, variant defaulting to medium |
| `atlas-agent.ts` | `maybeCreateAtlasConfig` -- UI-selected model respect, variant resolution |
| `general-agents.ts` | `collectPendingBuiltinAgents` -- handles all non-special-cased agents (skips sisyphus/hephaestus/atlas/sisyphus-junior), bulk model resolution and override application |

## PIPELINE FIT

Phase 3 of the config pipeline (`plugin-handlers/agent-config-handler.ts`) calls `createBuiltinAgents()`, which invokes these factories. It reads model and provider state from caches and `fetchAvailableModels`; NEVER call OpenCode client APIs here, because it runs inside the config handler and a client call deadlocks plugin init (#1301). Returns `undefined` per agent for disabled agents or unmet model requirements.
