# .github — GitHub Automation & Community Surface

> Earned file (score 12, distinct domain). Covers the repo's GitHub integration layer: CI/release pipelines, issue/PR intake, actionlint + knip config. `workflows/` has its own AGENTS.md; `scripts/`, `ISSUE_TEMPLATE/`, and `assets/` scored below 8 and are covered here.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add/fix a CI or release pipeline | `workflows/` | 47 workflows + 5 TS verifier modules, ~11.9k lines — see `workflows/AGENTS.md` |
| CI helper scripts (status guards, Darwin signing, Windows telemetry) | `scripts/` | 11 scripts, 808 lines; invoked from 25+ workflows |
| Issue intake forms | `ISSUE_TEMPLATE/` | 5 YAML issue forms (bug, feature, general, lazycodex-bug) + `config.yml` |
| PR body contract | `pull_request_template.md` | Summary / Changes / QA & Evidence / Risks & Residuals; enforces the root AGENTS.md evidence policy |
| Discord release-announce policy | `RELEASE-ANNOUNCEMENTS.md` | Governs `workflows/release-announce.yml`: skips drafts/prereleases/LazyCodex-only; `dry_run` outranks probe; webhook secret step-scoped |
| actionlint runner-label shim | `actionlint.yaml` | Registers `ubuntu-26.04` for actionlint v1.7.10; does NOT change runner selection |
| knip ignores for this dir | `knip.workspace.jsonc` | Composed by root `script/knip-config.ts`; ignores only `scripts/cla-first-signature-replay.mjs` |
| Sponsorship | `FUNDING.yml` | `github: code-yeongyu` only |
| README/media imagery | `assets/` | 14 binary images (png/jpg/svg), no code |

## CONVENTIONS

- Issue intake uses YAML **issue forms**, not markdown templates; every form carries triage labels plus required English-language and duplicate-search acknowledgement checkboxes.
- `blank_issues_enabled: false`; `ISSUE_TEMPLATE/config.yml` routes free-form questions to Discord and the docs README.
- Bash scripts use `#!/usr/bin/env bash` + `set -euo pipefail`; PowerShell scripts use strict mode; job summaries go through `scripts/write-job-summary.sh` (`JOB_SUMMARY_*` env vars).

## ANTI-PATTERNS

- **NEVER** file a LazyCodex bug through the generic `bug_report.yml` path: `ISSUE_TEMPLATE/lazycodex_bug_report.yml` exists for the light edition, and source-level LazyCodex changes belong in `packages/omo-codex/` in this repo; templates also steer users to `/bug` in-app first.
- **DO NOT** remove the knip ignore on `scripts/cla-first-signature-replay.mjs`: it is a pinned-revision CLA replay test (`cla.yml` + `pr-automation.yml`) that nothing imports by design — "dead code" cleanup breaks it.
- **DO NOT** treat `macos-sign-and-notarize.sh --sign-only` as notarization: valid only for code never assessed by Gatekeeper, and only while neither the parent engine nor the app opts into quarantine (quarantine is inherited by child processes).
- `scripts/ci-macos-status.sh` fails closed: a skipped macOS leg counts as success only when the classifier agrees; missing/failed classification is an error, never a pass.
- Signing material comes from Actions secrets (`CSC_LINK`, `CSC_KEY_PASSWORD`, `APPLE_API_*`); `MACOS_SIGNING_REQUIRED=true` turns missing material into a failure, otherwise files are ad-hoc signed to keep signatures valid.
