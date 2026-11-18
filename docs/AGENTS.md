# docs/ — User-Facing Documentation

**Generated:** 2026-11-18 / c2691818a9 (update mode)

## OVERVIEW

61 tracked files: 4 at docs root + 6 subdirectories (guide 16, reference 34, examples 3 JSONC, legal 2, templates 1, troubleshooting 1). Categorized by audience: user-facing guides + reference, troubleshooting, legal. `packages/web/` generates omo.dev docs content from this tree — `web-ci.yml` and `web-deploy.yml` both trigger on `docs/**`.

## STRUCTURE

```
docs/
├── manifesto.md                       # The "why" — referenced from README
├── i18n-glossary.md                   # Translation glossary for READMEs + packages/web/messages
├── model-capabilities-maintenance.md  # Refreshing the model-capabilities cache
├── guide/             # 16 tutorial-style user guides (indexed below; no own AGENTS.md)
├── reference/         # 34 API/config/CLI/threat-model pages → reference/AGENTS.md
├── examples/          # 3 sample JSONC configs (default, coding-focused, planning-focused)
├── legal/             # privacy-policy.md + terms-of-service.md
├── templates/         # AGENTS.md.example — copyable project-rules template
└── troubleshooting/   # ollama.md
```

## WHERE TO LOOK

| Task | Location |
|------|----------|
| User guides ([docs/guide/](guide)) | overview · install/installation/binary-install · migrating-from-opencode · orchestration · team-mode · senpi-task · computer-use · agents · agent-model-matching · keywords · telemetry · btw · workflows · desktop-updates |
| Any reference page (config, CLI, daemon, threat models…) | [docs/reference/AGENTS.md](reference/AGENTS.md) — per-page index |
| The "why" manifesto | [docs/manifesto.md](manifesto.md) |
| Changing a translated term | [docs/i18n-glossary.md](i18n-glossary.md) — edit the glossary first, then every translation |
| Refreshing the model-capabilities cache | [docs/model-capabilities-maintenance.md](model-capabilities-maintenance.md) |
| Sample configs | [docs/examples/](examples) |
| Privacy & ToS | [docs/legal/](legal) |
| Copyable project-rules template | [docs/templates/AGENTS.md.example](templates/AGENTS.md.example) |
| Ollama troubleshooting | [docs/troubleshooting/ollama.md](troubleshooting/ollama.md) |

## CONVENTIONS

- **CI matrix label:** pull requests run Linux only; adding `ci:full-matrix` (the one opt-in label; honored only for a platform-path diff or a PR body line `Platform-reason: <why>`) launches a separate run with optional `full-matrix / ...` checks that do not gate merge. Keep the label and push a commit or reopen the PR to run the full matrix in the required checks.
- **User-facing language only in `guide/` and `reference/`.** No `OmO` internal jargon without explanation.
- **Path links** use the `file://` scheme so OpenCode renders them in TUI. Use absolute paths.
- **No HTML.** Markdown only. No `<details>` / `<summary>` (causes rendering issues in some terminals).
- **Code blocks** use language fences. Use `jsonc` for config snippets to preserve comments.
- **Visual QA evidence** must come from the real-pty + xterm.js flow ([`script/qa/web-terminal-visual-qa.mjs`](../script/qa/web-terminal-visual-qa.mjs)). NEVER use `tmux capture-pane` for color/visual/CJK evidence; tmux is boot-smoke only.
- **Every push under `docs/` re-runs web CI** ([`web-ci.yml`](../.github/workflows/web-ci.yml) triggers on `docs/**`); deploys via [`web-deploy.yml`](../.github/workflows/web-deploy.yml).

## ANTI-PATTERNS

- Never add a page to `reference/` without a `WHERE TO LOOK` entry in [docs/reference/AGENTS.md](reference/AGENTS.md); a new `guide/` page gets its entry HERE — `guide/` has no own AGENTS.md because `guide/agents.md` is a user-facing guide and a knowledge-base `AGENTS.md` would collide with it on case-insensitive filesystems.
- Never paste agent-facing system prompts here. Those live in [`packages/omo-opencode/src/agents/`](../packages/omo-opencode/src/agents/) or [`packages/skills-loader-core/src/features/builtin-skills/`](../packages/skills-loader-core/src/features/builtin-skills/).
- Never document changing config keys without also updating [`packages/omo-opencode/src/config/schema/`](../packages/omo-opencode/src/config/schema/) and re-running `bun run build:schema`.
