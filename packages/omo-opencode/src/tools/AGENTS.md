# src/tools/ - 12-38 Native Tools Across 14 Tool Directories (+ shared utilities)

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Tools registered via [`createToolRegistry()`](../plugin/tool-registry.ts) in `src/plugin/`. Native tools are factory-based (`createXXXTool`) except `interactive_bash` (`ToolDefinition`). LSP tools are served by the Tier-1 built-in MCP `lsp`; structural search and rewrite is handled by the `ast-grep` skill.

## TOOL CATALOG

### Always On (12 native tools)

| Group | Tools |
|-------|-------|
| **Search** (2) | `grep`, `glob` |
| **Sessions** (4) | `session_list`, `session_read`, `session_search`, `session_info` |
| **Background tasks** (2) | `background_output`, `background_cancel` |
| **Delegation** (2) | `task` (delegate, full skill+category support), `call_omo_agent` (named agent only: explore, librarian) |
| **Skills/MCP** (2) | `skill` (load skill or invoke command), `skill_mcp` (call skill-embedded MCP tool/resource/prompt) |

> LSP tools are provided by the built-in `lsp` MCP (Tier-1 stdio), backed by `packages/lsp-tools-mcp/`. AST-aware code search and rewrite is available through the `ast-grep` skill using `sg`.

### Conditional (up to +26 native tools)

| Tool(s) | Gate | Source |
|---------|------|--------|
| `look_at` | not in `disabled_agents` for `multimodal-looker` | `look-at/` |
| `interactive_bash` | `isInteractiveBashEnabled(config)` (tmux config) | `interactive-bash/` |
| `task_create`, `task_get`, `task_list`, `task_update` | `experimental.task_system` | `task/` |
| `monitor_start`, `monitor_stop`, `monitor_list`, `monitor_output` | `monitor.enabled` | `monitor/` (watcher engine: `../features/monitor/`) |
| `edit` (hashline-edit) | `hashline_edit: true` | `hashline-edit/` |
| `create_goal`, `update_goal`, `get_goal` | `goal.enabled` | [`../hooks/goal/tools.ts`](../hooks/goal/tools.ts) (registered in `tool-registry-core-tools.ts`) |
| 12 `team_*` tools: `team_create`, `team_delete`, `team_shutdown_request`, `team_approve_shutdown`, `team_reject_shutdown`, `team_send_message`, `team_task_create`, `team_task_list`, `team_task_update`, `team_task_get`, `team_status`, `team_list` (team lifecycle, async messaging, shared atomic-claim tasklist) | `team_mode.enabled: true` | `../features/team-mode/tools/` - see [`team-mode/AGENTS.md`](../features/team-mode/AGENTS.md) |

## DELEGATION CATEGORIES (built-in 9)

`task` (delegate) selects model by category. Default category models live in provider-specific files under `src/tools/delegate-task/` and aggregate via `BUILTIN_CATEGORIES` in `builtin-categories.ts`. Authoritative fallback chains live in [`packages/model-core/src/category-model-requirements.ts`](../../../model-core/src/category-model-requirements.ts) `CATEGORY_MODEL_REQUIREMENTS`.

| Category | Default Model | Domain |
|----------|---------------|--------|
| `visual-engineering` | anthropic/claude-fable-5-2 (max) | Visual design, UI/UX, frontend |
| `ultrabrain` | openai/gpt-6-astra (max) | Hard logic / heavy reasoning; GPT-6 Astra-specific prompt append |
| `deep-low` | openai/gpt-6.1-sol (medium) | Default deep lane: 3D graphics, computer use, browser use, backend, logic, algorithms, CAPTCHA solving, multimodal, autonomous multi-step work settleable from evidence; gpt-6.1-sol-fast / gpt-5.6-sol / gpt-5.6-sol-fast medium fallback rungs gated on any GPT-6.1 Sol or GPT-5.6 Sol tier |
| `deep-high` | openai/gpt-6-astra (high) | Escalation deep lane: the goal's central decision cannot be settled from evidence (trade-off, cross-boundary contract, no in-repo pattern, correctness argued from invariants); single rung, gated on gpt-6-astra |
| `artistry` | anthropic/claude-fable-5-2 (max) | Creative / unconventional approaches |
| `quick` | openai/gpt-6-luna-fast (low) | Trivial single-file changes |
| `unspecified-low` | anthropic/claude-sonnet-5-5 (medium) | Moderate effort fallback |
| `unspecified-high` | anthropic/claude-opus-5-5 (medium) | High effort fallback; keeps the GPT-6 Astra prompt append for a user override onto a GPT-6 model |
| `writing` | anthropic/claude-opus-5-5 (low) | Documentation, prose |

User-defined categories declared in `categories: { ... }` config override and extend this set.

## TOOL DIR LAYOUT

`background-task/` (background_output/cancel; LLM interface, engine in `../features/background-agent`), `call-omo-agent/` (explore + librarian only), `delegate-task/` (task - categories, skills, provider category files), `glob/` (60s timeout, 100 file limit), `grep/` (60s timeout, 10MB limit), `hashline-edit/` (edit - hash-anchored LINE#ID edits), `interactive-bash/` (tmux session control), `look-at/` (image/PDF analysis), `monitor/` (monitor_*; engine in `../features/monitor`), `session-manager/` (4 session_* tools), `skill/` (load skill or run command), `skill-mcp/` (skill-embedded MCP), `slashcommand/` (`discoverCommandsSync` feeds the skill tool with the /-command list), `task/` (4 task_* Sisyphus task-system tools), `index.ts` (barrel exports).

## ADDING A NEW TOOL

1. Create `src/tools/{name}/index.ts` with factory `createXXXTool`; add `types.ts` for parameter Zod schemas and `tools.ts` (or a single index.ts) for the implementation.
2. Export the factory from `src/tools/index.ts`.
3. Register in `src/plugin/tool-registry.ts`: always-on tools spread into `allTools` directly; conditional tools build a `Record<string, ToolDefinition>` and gate-spread.
4. If the tool needs disabling, ensure it appears in the `filterDisabledTools` allow-list (its name is matched against `disabled_tools`).
