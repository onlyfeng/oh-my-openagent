# .opencode/ — Project-Scope Skills & Commands

**Generated:** 2026-11-18 (update mode)

## OVERVIEW

Project-scope OpenCode config: 3 skills + 5 slash commands, all git-tracked; loaded by [`opencode-skill-loader`](../packages/omo-opencode/src/features/opencode-skill-loader/) and the slash-command discovery pipeline.

## WHERE TO LOOK

| Need | Location |
|------|----------|
| Add/edit a slash command | `command/<name>.md` — frontmatter `description`, optional `argument-hint` |
| Add/edit a skill | `skills/<name>/SKILL.md` + optional `scripts/`, `references/`, `assets/` |
| GitHub fetch CLI | `skills/github-triage/scripts/gh_fetch.py` — `uv run --script gh_fetch.py {issues|prs|all} [--repo owner/repo]` (typer+rich, py>=3.11) |
| Background-task runtime state | `background-tasks.json` |

## SKILLS (3)

| Skill | Purpose |
|-------|---------|
| `work-with-pr/` | PR lifecycle in a task-owned worktree: implement with evidence-bound manual QA → atomic commits → PR → CI/Cubic verification loop → merge. |
| `github-triage/` | Read-only triage of issues + PRs; 1 item = 1 background `quick` task; every claim needs a GitHub permalink. ZERO GitHub mutations. |
| `pre-publish-review/` | 12-agent release gate: `/get-unpublished-changes` → up to 10 ultrabrain agents per change → oracle synthesis. ONLY on explicit request; `/publish` never triggers it. |

(`hyperplan` is documented in older snapshots but lives in `.agents/skills/` only — NOT here.)

## COMMANDS (5)

| Command | Purpose |
|---------|---------|
| `/get-unpublished-changes` | Diff HEAD against the latest published npm versions, by release layer. |
| `/publish` | Release via GitHub Actions `publish.yml` (`gh workflow run`). Arg: `<patch|minor|major|explicit-semver>`. Ships only — never reviews, repairs, or opens PRs. |
| `/remove-deadcode` | LSP-verified dead-code removal via parallel agents, atomic commits. |
| `/security-research` | Team Mode audit: 3 vulnerability hunters + 2 PoC engineers. |
| `/omomomo` | Easter egg. |

## MIRROR RULE (`.agents/`)

`.agents/` is the rename-transition migration target and a SUPERSET: its `skills/` holds 13 skills — the 3 here mirrored **byte-identical** plus 10 more — and its `command/` mirrors all 5 commands. Both trees load during the transition; consumers prefer `.agents/`. **Never edit one side of a mirrored pair alone.**

## CONVENTIONS

- Skill YAML frontmatter (`name` + `description`) is mandatory; the loader rejects skills without it.
- Trigger words inside `description` control when a skill loads — be specific.
- Project scope overrides user scope: `.opencode/skills/X/` beats `~/.config/opencode/skills/X/`.
- Command filenames normalize to leading-slash commands; the workflow contract lives inline in the file.
- `command/.npmignore` + `skills/.npmignore` keep `__*`/`.private`/`.draft` assets out of npm. Root `.npmignore` does NOT apply to dirs listed in `package.json#files` under Bun 1.3.x — the co-located ignores are load-bearing, enforced by `script/package-layout-exclusion.test.ts`.

## ANTI-PATTERNS

- Never duplicate a built-in skill from `packages/skills-loader-core/src/features/builtin-skills/` here.
- Never commit `background-tasks.json` containing session IDs/tokens. Open hygiene issue: the currently tracked file contains `ses_*` IDs — scrub before next release.
- Never call `session.promptAsync` from a skill — route through `dispatchInternalPrompt` (`packages/omo-opencode/src/features/background-agent/manager.ts`).
- GitHub-triage subagents never mutate GitHub: no comments, merges, closes, labels, reviews; no `gh api` POST/PUT/PATCH/DELETE — reports only.
