# .github/ - CI/CD Workflows, Release Automation, Repo Templates

## OVERVIEW

GitHub Actions automation for the whole monorepo: 30 workflows, 5 helper scripts, issue/PR templates, actionlint config. Earned its file at score 10: distinct domain (CI gates, publishing, deploys), 46 text files / 6.8k LOC, no other AGENTS.md covers the workflow layer.

## STRUCTURE

```
.github/
├── workflows/            # 30 *.yml; entrypoints are workflow files, not code modules
├── scripts/              # cla-first-signature-replay.mjs, macos-sign-and-notarize.sh (+ omo-bun-executable.entitlements),
│                         # windows-ci-telemetry.ps1, write-job-summary.sh (shared job-summary emitter, used by 20 workflows)
├── ISSUE_TEMPLATE/       # bug_report, feature_request, general, lazycodex_bug_report (.yml) + config.yml
├── actionlint.yaml       # registers the ubuntu-26.04 runner label for actionlint
├── pull_request_template.md, RELEASE-ANNOUNCEMENTS.md, FUNDING.yml
└── assets/               # README/marketing images (binary + SVG)
```

## WHERE TO LOOK

| Concern | Workflow(s) | Notes |
|---------|-------------|-------|
| Main CI gate | `ci.yml` (~1k LOC) | push/PR/merge_group/workflow_call; `script/ci-fast-path.mjs` mode, test shards, typecheck, codex/senpi compatibility, build, payload check; `script/ci-leg-tests-guard.mjs` ends every leg |
| Optional full matrix | `ci-full-matrix-label.yml` | `ci:full-matrix` label; calls `ci.yml`, non-gating |
| Publish | `publish.yml` (~1.8k LOC), `publish-platform.yml` (~900 LOC) | manual dispatch; npm + platform launchers + lazycodex sync |
| Release follow-ups | `release-announce.yml`, `npm-dist-tag-rollback.yml`, `npm-deprecate-version.yml`, `installer-mirror.yml` (R2) | Discord announce rules in `RELEASE-ANNOUNCEMENTS.md` |
| Desktop engine (Rust) | `desktop-engine.yml`, `desktop-linux-qa.yml`, `desktop-windows-qa.yml`, `macos-signing-canary.yml` | Trigger on `crates/**`, `packages/senpi-desktop-*/**`, `Cargo.toml`, `Cargo.lock`; `cargo test/clippy --workspace --locked` |
| Isolation backends | `isolation-linux-fs.yml` | Linux filesystem matrix for `packages/isolation-core` |
| Compiled session worker | `compiled-worker.yml` | PR + dispatch |
| get.omo.dev | `get-worker-ci.yml`, `get-worker-deploy.yml` | `packages/get-worker` Cloudflare Worker |
| Web | `web-ci.yml`, `web-deploy.yml`, `cutover-target-deploy.yml` | `packages/web/**` + `docs/**` |
| Review / PR hygiene | `review-claims.yml`, `changelog-gate.yml`, `pr-automation-signal.yml`, `bot-merge.yml`, `package-labels.yml`, `cla.yml` | Claim labels gate merge (root AGENTS.md) |
| Agents + housekeeping | `sisyphus-agent.yml`, `refresh-model-capabilities.yml`, `stats.yml`, `windows-flake-soak.yml`, `lint-workflows.yml` | |

## CONVENTIONS

- Declare least-privilege `permissions` and a `concurrency` group in new workflows (25/30 set top-level `permissions`, 14/30 top-level `concurrency` today); required-check job names are stable, so renaming a job breaks branch protection.
- Workflows call root Bun scripts (`script/*`) or `.github/scripts/*`; keep logic in scripts, not inline YAML, when it needs tests.
- Platform helpers match the runner: PowerShell for Windows telemetry, Bash for macOS signing, Bun ESM for the CLA replay (`bun .github/scripts/cla-first-signature-replay.mjs <action checkout>`).
- Lint every workflow edit with `actionlint` (shellcheck disabled in `lint-workflows.yml`).

## ANTI-PATTERNS

- Never let a skipped test leg report a plain pass; `ci-leg-tests-guard.mjs` exists for that.
- Never add `continue-on-error`, retry masking, or platform exclusion to turn a red required check green.
- Never hand-publish or hand-tag; `publish.yml` is the only release path.
- `sisyphus-agent.yml` prompt rules: never push directly to the target branch, use heredocs for comments containing backticks or code.
