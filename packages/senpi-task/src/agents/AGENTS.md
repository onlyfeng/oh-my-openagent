# src/agents/ - Agent Resolution Layer

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 24 (46 files; barrel re-exported from the package barrel and consumed by manager/runners). The builtin roster and plan-gating story live in [`../../AGENTS.md`](../../AGENTS.md) - not repeated here.

## OVERVIEW

`resolveAgent` and its parts: builtin registry assembly, `omo.json` agent mapping, persona/model resolution, interaction + tool policies.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: `loadAgents`, `mapOmoConfigAgents`, `resolveAgent`, `defineAgent`, `registerAgent`, `resolveToolRule`, `agentToolPolicy`, `AGENT_INTERACTION_POLICIES` |
| `resolve-agent.ts` | Persona + model resolution with fallback-chain selection |
| `registry.ts` / `schema.ts` | `registerAgent`; `defineAgent` - the typed `AgentDefinition` entry |
| `loader.ts` | `loadAgents` - agent sources to definitions |
| `omo-config-agents.ts` / `omo-overlay.ts` | `mapOmoConfigAgents` - `omo.json` `agents` mapping and overlay |
| `agent-model-entry.ts` / `agent-model-registry.ts` | `agentModelCandidates`, `findExactAgentModel`, `parseAvailableAgentModels` |
| `agent-persona.ts` | `agentPersona` - persona text resolution |
| `category-shadow.ts` | `categoryShadowedAgentNames` / `categoryShadowMessage` |
| `interaction-policy.ts` | `AGENT_INTERACTION_POLICIES`, `ONE_SHOT_AGENT_NAMES`, `interactionPolicyForAgent` |
| `invocation-guard.ts` | ulw-plan gating - plan agents spawnable only after an explicit USER `ulw-plan` request |
| `tools.ts` | `resolveToolRule` behind `agentToolPolicy` |
| `builtin/` | `index.ts` assembles `BUILTIN_AGENTS`, `CURATED_READONLY_*` / `ULW_REVIEWER_*` defaults and name sets; `fallback-chains.ts`; `names.ts`; `legacy-agent-names.ts`; the 7 builtin definitions (`explore`, `librarian`, `plan-consultant`, `plan-reviewer`, `code-reviewer`, `qa-executor`, `gate-reviewer`) |
| `markdown.ts` / `paths.ts` / `task-agents.ts` | Prompt markdown loading, agent paths, task defaults |

## CONVENTIONS

- Agent names are canonical kebab-case ids; definitions are typed `AgentDefinition`.
- Prompt prose for builtins lives in the definition modules (Markdown prompt content); Senpi builtin definitions adapt OpenCode/Codex contracts to Senpi capabilities (e.g. librarian research uses the curated shell-free gh/curl broker).

## ANTI-PATTERNS

- `fallback-chains.test.ts` must NEVER import chain data from `@oh-my-opencode/model-core` - the local transcription + pins guard against drift; the two curated keys are renamed `plan-consultant` / `plan-reviewer` while provider/model/variant/order rungs stay mirrored.
- Legacy builtin identity spelling is load-bearing for the ULW quality gate - do not rename externally addressed agents casually.
- Preserve the role-specific NEVER/ALWAYS rules inside builtin prompt text when editing those definitions.

## COMMANDS

```bash
bun test packages/senpi-task/src/agents
tsgo --noEmit -p packages/senpi-task/tsconfig.json
```
