# Hephaestus - Deep-Worker Subagent

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 14 (8 files; existing file kept in update mode and expanded). The agent roster and registry wiring live in [`../AGENTS.md`](../AGENTS.md).

## OVERVIEW

The Hephaestus deep-worker subagent: factory, model eligibility, Senpi-native model prompt, and the GPT-family role lane.

## WHERE TO LOOK

| File | Role |
|------|------|
| `agent.ts` | `createHephaestusAgent`, `getHephaestusPrompt`, `getHephaestusPromptSource`, `hephaestusPromptMetadata`, `isHephaestusSupportedModel`, `UnsupportedHephaestusModelError` |
| `model-support.ts` | The model eligibility predicate + error |
| `gpt.ts` | OpenCode deep-worker role/context (GPT lane) |
| `index.ts` | Barrel |
| Tests | `agent.test.ts` (support/rejection, registration, transport aliases), `delegation-table-contract.test.ts`, `gpt-5-6-registration.test.ts`, `openai-fast-alias.test.ts` |

## CONVENTIONS

- Model content comes only from Senpi through `../native-model-prompt.ts`; `gpt.ts` retains the OpenCode deep-worker role and context. No local Native model-core copies.
- The role delegation table permits explore, librarian, and oracle only - planning agents are excluded.

## ANTI-PATTERNS

- Registration uses the eligibility predicate; an unsupported per-request model leaves the previous prompt intact so an allowed runtime override cannot fail the turn - do not make it throw or replace a valid prompt.
- Do not duplicate Native model prompt bodies locally; keep the single source.

## COMMANDS

```bash
bun test packages/omo-opencode/src/agents/hephaestus
```
