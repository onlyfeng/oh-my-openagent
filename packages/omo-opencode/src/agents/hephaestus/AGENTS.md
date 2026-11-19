# Hephaestus - Deep Worker

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 14 (8 files; existing file kept in update mode and expanded). The agent roster and registry wiring live in [`../AGENTS.md`](../AGENTS.md).

## OVERVIEW

Primary deep-worker agent (`const MODE: AgentMode = "primary"` in `agent.ts`): factory, model eligibility, Senpi-native model prompt, and the GPT-family role lane.

## WHERE TO LOOK

| File | Role |
|------|------|
| `agent.ts` | `createHephaestusAgent`, `getHephaestusPrompt`, `getHephaestusPromptSource`, `hephaestusPromptMetadata`; re-exports `isHephaestusSupportedModel`, `UnsupportedHephaestusModelError` |
| `model-support.ts` | Eligibility predicate, `UnsupportedHephaestusModelError`, `isStandaloneHephaestusPreset` (`STANDALONE_PRESETS` = `gpt-6.2`) |
| `gpt.ts` | OpenCode deep-worker role/context (GPT lane) |
| `index.ts` | Barrel |
| Tests | `agent.test.ts` (support/rejection, registration, transport aliases), `delegation-table-contract.test.ts`, `gpt-5-6-registration.test.ts`, `openai-fast-alias.test.ts` |

## CONVENTIONS

- Model content comes only from Senpi through `../native-model-prompt.ts`; `gpt.ts` retains the OpenCode deep-worker role and context. No local Native model-core copies.
- The role delegation table permits explore, librarian, and oracle only. Planning agents are excluded (`agent.ts` filters `availableAgents` to those three names).
- On a Native preset that is the whole deep-work prompt by itself (`gpt-6.2`, `isStandaloneHephaestusPreset` in `model-support.ts`) no role is appended: the prompt is the framed core alone (#14186). The role still rides in the frame context, so a per-request model switch drops it for GPT-6.2 and restores it for every other supported model.
- The refusal-is-final rule is stated once by the frame for every agent that carries it: by the model core when the core already states it, immediately after the core otherwise (`composeNativeAgentPrompt` in `../native-model-prompt.ts`; frame field in `../native-prompt-frame.ts`).

## ANTI-PATTERNS

- Registration uses the eligibility predicate; an unsupported per-request model leaves the previous prompt intact so an allowed runtime override cannot fail the turn. Do not make it throw or replace a valid prompt.
- Do not duplicate Native model prompt bodies locally; keep the single source.

## COMMANDS

```bash
bun test packages/omo-opencode/src/agents/hephaestus
```
