---
name: agents-directory
description: Developer reference for all 11 Oh My OpenAgent agent definitions, factory patterns, tool restrictions, and model routing.
---

# src/agents/ — 11 Agent Definitions

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

11 built-in agents. Type enum: [`src/config/schema/agent-names.ts`](../config/schema/agent-names.ts) `BuiltinAgentNameSchema`. 10 of them register via [`builtin-agents.ts`](builtin-agents.ts) `agentSources` record (factory functions). **Prometheus is special-cased** — it has no `createPrometheusAgent` factory; instead [`prometheus-agent-config-builder.ts`](../plugin-handlers/prometheus-agent-config-builder.ts) constructs its config directly during `agent-config-handler` Phase 3.

Factories carry a static `mode` property (`AgentFactory` type in [`src/agents/types.ts`](types.ts)) and are composed via `buildAgent()` in `agent-builder.ts`. Most take `(model) -> AgentConfig`; exceptions: Atlas takes an `OrchestratorContext`, Sisyphus routes through `sisyphus-agent-factory.ts`, and Sisyphus-Junior has the override-aware `createSisyphusJuniorAgentWithOverrides()`.

## AGENT INVENTORY

Modes verified from each agent file's `const MODE: AgentMode = ...` and (for Prometheus) [`prometheus-agent-config-builder.ts:100`](../plugin-handlers/prometheus-agent-config-builder.ts#L100). Chains live in `packages/model-core/src/agent-model-requirements.ts`, re-exported by [`src/shared/model-requirements.ts`](../shared/model-requirements.ts).

| Agent | Default Model | Temp | Mode | Fallback (after default) | Purpose |
|-------|---------------|------|------|--------------------------|---------|
| **Sisyphus** | claude-opus-5-5 max | (model default) | primary | kimi-k3 -> gpt-5.6-sol medium -> glm-5.2 -> big-pickle | Main orchestrator, plans + delegates; `thinking: { type: "enabled", budgetTokens: 32000 }` |
| **Hephaestus** | gpt-6-sol medium | (model default) | primary | gpt-5.6-sol medium only (`requiresProvider`: openai \| chatgpt-subscription \| github-copilot \| opencode) | Autonomous deep worker |
| **Oracle** | gpt-5.6-sol xhigh (high on Copilot) | 0.1 | subagent | gemini-4-argon high -> gemini-3.1-pro high -> claude-opus-5-5 max -> glm-5.2 | Read-only consultation |
| **Librarian** | kimi-for-coding-highspeed (off) | 0.1 | subagent | gpt-6-luna-fast low -> claude-haiku-5-5 medium -> deepseek-flash max -> qwen3.7-plus -> minimax-m2.7 | External docs/code search |
| **Explore** | kimi-for-coding-highspeed (off) | 0.1 | subagent | same chain as Librarian | Contextual grep |
| **Multimodal-Looker** | gpt-5.6-sol low | 0.1 | subagent | kimi-k3 -> glm-4.6v -> gpt-5-nano | PDF/image analysis |
| **Metis** | claude-fable-5-1 max | **0.3** | subagent | claude-opus-5-5 max -> kimi-k3 max | Pre-planning consultant |
| **Momus** | gpt-6-astra xhigh (high on Copilot) | 0.1 | subagent | claude-opus-5-5 max -> gemini-4-argon high -> gemini-3.1-pro high -> glm-5.2 | Plan reviewer (`momus-gpt-5-6.ts` GPT prompt) |
| **Atlas** | claude-sonnet-5 | 0.1 | primary | kimi-k3 -> gpt-5.6-sol medium -> minimax-m3 -> MiniMax-M3 -> minimax-m2.7 | Todo-list orchestrator |
| **Prometheus** | claude-fable-5-1 xhigh | (override-only) | primary | claude-opus-5-5 max -> kimi-k3 max | Strategic planner (interview); built via `buildPrometheusAgentConfig` (not in `agentSources`) |
| **Sisyphus-Junior** | claude-sonnet-5 | 0.1 (`SISYPHUS_JUNIOR_DEFAULTS`) | subagent | kimi-k3 -> gpt-5.6-sol medium -> minimax-m3 -> MiniMax-M3 -> minimax-m2.7 -> big-pickle | Category-spawned executor |

## TOOL RESTRICTIONS

Static denylists: [`src/shared/agent-tool-restrictions.ts`](../shared/agent-tool-restrictions.ts). Permission overlays: `plugin-handlers/tool-config-handler.ts`.

| Agent | Denied Tools |
|-------|-------------|
| Oracle / Librarian / Explore | write, edit, task, call_omo_agent |
| Metis / Momus | write, edit |
| Multimodal-Looker | `read: true` allowlist; overlay denies task, look_at |
| Sisyphus-Junior | task |
| Sisyphus / Hephaestus / Atlas / Prometheus | overlay: call_omo_agent + todo tools denied, task allowed, `teammate: "allow"` |
| Prometheus | also bash, interactive_bash; `.md`-only writes via `prometheus-md-only` hook (path-based) |

## TEAM-MODE ELIGIBILITY

Authoritative registry: `AGENT_ELIGIBILITY_REGISTRY` in `packages/team-core/src/types.ts`. Three verdict tiers:

| Verdict | Agents |
|---------|--------|
| `eligible` | sisyphus, atlas, sisyphus-junior |
| `conditional` | hephaestus (registry message still cites missing `teammate: "allow"` / D-36, although `tool-config-handler.ts` now grants it; `subagent_type: "sisyphus"` always works) |
| `hard-reject` | oracle, librarian, explore, multimodal-looker, metis, momus, prometheus (each with a specific rejection message) |

Read-only agents are rejected at TeamSpec parse time. For those, the lead delegates via `task` (delegate-task) instead. See [`team-mode/AGENTS.md`](../features/team-mode/AGENTS.md).

## STRUCTURE

```
agents/
|-- sisyphus.ts, sisyphus-agent-factory.ts      # resolveSisyphusPromptFamily() + per-family config builders
|-- sisyphus-agent-config.ts                    # Sisyphus config assembly
|-- sisyphus-dynamic-prompt*.ts (7 files)       # builder, role, exploration, execution, style, sections
|-- sisyphus-gemini-fallback-overrides.ts       # Gemini overlays on the fallback prompt
|-- sisyphus-runtime-prompt-reconciler.ts       # re-resolves prompt family when the runtime model changes
|-- sisyphus/  hephaestus/  sisyphus-junior/  atlas/  prometheus/   # per-agent dirs, each with AGENTS.md
|-- oracle.ts, librarian.ts, explore.ts, multimodal-looker.ts, metis.ts
|-- momus.ts, momus-gpt-5-6.ts                  # plan reviewer + GPT prompt
|-- builtin-agents.ts                           # createBuiltinAgents(); agentSources registry (10 entries)
|-- builtin-agents/                             # maybeCreateXXXConfig conditional factories
|-- agent-builder.ts, agent-skill-resolution.ts # buildAgent(), resolveAgentSkills()
|-- frontier-tool-schema-guard.ts, gpt-apply-patch-guard.ts, kimi-tool-loop-guard.ts  # model-specific guards
|-- gpt-prompt-identity.ts                      # shared GPT identity text
|-- env-context.ts                              # environment context for prompts
|-- dynamic-agent-*.ts (6 files)                # prompt builder, core/policy sections, tool categorization, category-skills guide, types
`-- types.ts, index.ts                          # BuiltinAgentName, AgentMode, AgentFactory; barrel
```

Largest tests: `utils.test.ts` (1,649 LOC), `sisyphus-junior/index.test.ts`, `hephaestus/agent.test.ts`.

## FACTORY PATTERN

```typescript
const createXXXAgent: AgentFactory = (model: string) => ({
  instructions: "...",
  model,
  // Set temperature only when this agent pins a role-specific default.
  // ...config
})
createXXXAgent.mode = "subagent" // or "primary" or "all"
```

Temperature is opt-in per agent. Most consultant and helper agents pin
`0.1`, Metis pins `0.3`, and coordinator / role agents intentionally
leave temperature unset when they should use the selected model's
default. Sisyphus, Hephaestus, and Prometheus leave temperature unset
unless an override or category config supplies one.

Model resolution: 4-step pipeline → override → category-default → provider-fallback → system-default. Defined in [`shared/model-resolution-pipeline.ts`](../shared/model-resolution-pipeline.ts).

## MODES

Definition (from [`src/agents/types.ts`](types.ts)):

- **`primary`** — respects user's UI-selected model. Used by: sisyphus, hephaestus, atlas, prometheus.
- **`subagent`** — uses own fallback chain, ignores UI selection. Used by: oracle, librarian, explore, multimodal-looker, metis, momus, sisyphus-junior.
- **`all`** — declared in the type for OpenCode compatibility but no built-in agent currently uses it.

## CANONICAL ORDER

`Sisyphus → Hephaestus → Prometheus → Atlas` (primary core agents) then alphabetical for the rest. Enforced by [`installAgentSortShim()`](../shared/agent-sort-shim.ts) — patches `Array.prototype.{toSorted,sort}` narrowly when ≥2 canonical core agents are in the array. See [`src/plugin-handlers/AGENTS.md`](../plugin-handlers/AGENTS.md) for the full history.

## DYNAMIC PROMPT BUILDER

`dynamic-agent-prompt-builder.ts` composes per-agent system prompts at runtime by stitching:
- Core sections (identity, mode, restrictions)
- Policy sections (citation, verification, anti-patterns)
- Tool categorization (per-domain tool guidance)
- Category-skills guide (which skills load with which categories)

This is what the Sisyphus prompt's "AGENTS / CATEGORY + SKILLS" tables come from.
