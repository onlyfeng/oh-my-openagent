---
name: agents-directory
description: Developer reference for all 11 Oh My OpenAgent agent definitions, factory patterns, tool restrictions, and model routing.
---

# src/agents/ - 11 Agent Definitions

Model-specific system cores come from `@code-yeongyu/senpi/prompt-presets` via
`native-model-prompt.ts` (PR #9856); `native-prompt-frame.ts` rebuilds the generated
core for the request model without replacing user appends. Do not add copies of
Native model bodies. Explore, librarian and multimodal-looker keep their original
prompts without a Native core.

## OVERVIEW

11 built-in agents. Type enum: [`src/config/schema/agent-names.ts`](../config/schema/agent-names.ts) `BuiltinAgentNameSchema`. 10 register via [`builtin-agents.ts`](builtin-agents.ts) `agentSources`. **Prometheus is special-cased** - no `createPrometheusAgent` factory; [`prometheus-agent-config-builder.ts`](../plugin-handlers/prometheus-agent-config-builder.ts) constructs its config directly during `agent-config-handler` Phase 3.

## AGENT INVENTORY

Modes verified from each agent file's `const MODE: AgentMode = ...` and (for Prometheus) [`prometheus-agent-config-builder.ts:100`](../plugin-handlers/prometheus-agent-config-builder.ts#L100). Chains verified from [`src/shared/model-requirements.ts`](../shared/model-requirements.ts).

| Agent | Default Model | Temp | Mode | Fallback (after default) | Purpose |
|-------|---------------|------|------|--------------------------|---------|
| **Sisyphus** | claude-opus-5-5 max | (model default) | primary | kimi-k3 → gpt-5.6-sol medium → glm-5.2 → big-pickle | Main orchestrator, plans + delegates; `thinking: { type: "enabled", budgetTokens: 32000 }` |
| **Hephaestus** | gpt-5.6-sol medium | (model default) | primary | GPT-5.6 Sol only (`requiresProvider`: openai \| chatgpt-subscription \| github-copilot \| opencode) | Autonomous deep worker |
| **Oracle** | gpt-5.6-sol xhigh (high on Copilot) | 0.1 | subagent | gemini-4-argon high → gemini-3.1-pro high → claude-opus-5-5 max → glm-5.2 | Read-only consultation |
| **Librarian** | gpt-6-luna-fast | 0.1 | subagent | claude-haiku-5-5 medium → qwen3.7-plus → minimax-m3 → minimax-m2.7 → gpt-5.4-nano | External docs/code search |
| **Explore** | gpt-6-luna-fast | 0.1 | subagent | claude-haiku-5-5 medium → qwen3.7-plus → minimax-m3 → minimax-m2.7 → gpt-5.4-nano | Contextual grep |
| **Multimodal-Looker** | gpt-5.6-sol low | 0.1 | subagent | kimi-k3 → glm-4.6v → gpt-5-nano | PDF/image analysis |
| **Metis** | claude-fable-5-2 max | **0.3** | subagent | claude-fable-5-1 max → claude-opus-5-5 max → kimi-k3 max | Pre-planning consultant |
| **Momus** | gpt-5.6-terra high | 0.1 | subagent | gpt-5.6-sol xhigh (high on Copilot) → claude-opus-5-5 max → gemini-4-argon high → gemini-3.1-pro high → glm-5.2 | Plan reviewer |
| **Atlas** | claude-sonnet-5 | 0.1 | primary | kimi-k3 → gpt-5.6-sol medium → minimax-m3 → MiniMax-M3 → minimax-m2.7 | Todo-list orchestrator |
| **Prometheus** | claude-fable-5-2 xhigh | (override-only) | primary | claude-fable-5-1 xhigh → claude-opus-5-5 max → kimi-k3.1-pro max → kimi-k3 max | Strategic planner (interview); built via `buildPrometheusAgentConfig` (not in `agentSources`) |
| **Sisyphus-Junior** | claude-sonnet-5 | 0.1 (`SISYPHUS_JUNIOR_DEFAULTS`) | subagent | kimi-k3 → gpt-5.6-sol medium → minimax-m3 → MiniMax-M3 → minimax-m2.7 → big-pickle | Category-spawned executor |

## TOOL RESTRICTIONS

Oracle/Librarian/Explore deny `write, edit, task, call_omo_agent`; Momus denies `write, edit, task`; Multimodal-Looker denies ALL except `read`; Atlas denies `task, call_omo_agent`; Prometheus enforces `.md`-only writes via the `prometheus-md-only` hook (path-based, not tool-based). Defined in [`src/shared/agent-tool-restrictions.ts`](../shared/agent-tool-restrictions.ts).

## TEAM-MODE ELIGIBILITY

Registry: [`AGENT_ELIGIBILITY_REGISTRY`](../features/team-mode/types.ts). `eligible`: sisyphus, atlas, sisyphus-junior. `conditional`: hephaestus (lacks `teammate: "allow"` by default - D-36 / `tool-config-handler.ts`; use `subagent_type: "sisyphus"`). `hard-reject`: oracle, librarian, explore, multimodal-looker, metis, momus, prometheus - read-only agents are rejected at TeamSpec parse time; the lead delegates via `task` instead. See [`team-mode/AGENTS.md`](../features/team-mode/AGENTS.md).

## STRUCTURE

```
agents/
├── {sisyphus,hephaestus,oracle,librarian,explore,multimodal-looker,metis,momus}.ts  # one file per simple agent
├── atlas/ · hephaestus/ · prometheus/ · sisyphus/ · sisyphus-junior/   # multi-file agents (own AGENTS.md)
├── types.ts · builtin-agents.ts        # BuiltinAgentName/AgentMode/AgentConfig; agentSources registry
├── builtin-agents/                     # maybeCreateXXXConfig conditional factories + general-agents + available-skills
├── agent-builder.ts · utils.ts · env-context.ts
└── dynamic-agent-*.ts                  # prompt builder: core/policy sections, tool categorization, category-skills guide
```

## FACTORY PATTERN

`createXXXAgent: AgentFactory = (model) => ({ instructions, model, ...config })` with a static `mode` property, composed via `buildAgent()`. Temperature is opt-in: most consultants pin `0.1`, Metis `0.3`; sisyphus/hephaestus/prometheus leave it unset unless an override or category config supplies one. Model resolution: override → category-default → provider-fallback → system-default ([`shared/model-resolution-pipeline.ts`](../shared/model-resolution-pipeline.ts)).

## MODES

`primary` respects the UI-selected model (sisyphus, hephaestus, atlas, prometheus); `subagent` uses its own fallback chain and ignores UI selection (oracle, librarian, explore, multimodal-looker, metis, momus, sisyphus-junior); `all` exists for OpenCode compatibility but no built-in uses it.

## CANONICAL ORDER

`Sisyphus → Hephaestus → Prometheus → Atlas`, then alphabetical. Enforced by [`installAgentSortShim()`](../shared/agent-sort-shim.ts); full history in [`src/plugin-handlers/AGENTS.md`](../plugin-handlers/AGENTS.md).

## DYNAMIC PROMPT BUILDER

`dynamic-agent-*.ts` supply the reusable role-context sections - core (identity/mode/restrictions), policy (citation/verification/anti-patterns), tool categorization, category-skills guide. The Sisyphus prompt's "AGENTS / CATEGORY + SKILLS" tables come from here.
