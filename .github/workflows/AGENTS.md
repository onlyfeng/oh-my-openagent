# .github/workflows - CI & Release Pipelines

> Earned file (score 19, high complexity): 47 workflow YAMLs + 5 TypeScript operator-darwin verifier modules, ~11.9k lines. The repository's largest config surface and its release-critical core.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Main CI: event routing, path/matrix classification, test sharding, gates | `ci.yml` | 1,438 lines; typecheck / lint / `lint:ratchet` / knip + package shards; macOS legs fail closed via `../scripts/ci-macos-status.sh` |
| npm publishing | `publish.yml` | 2,294 lines, largest file; gates release on post-publish-verify + installer-mirror, then calls `release-announce.yml` |
| Platform binary publishing | `publish-platform.yml` | 1,499 lines; invokes the operator-darwin verifier (~line 477) |
| Darwin artifact signature verification | `operator-darwin-verify.ts` + `-macho` / `-cms` / `-services` / `-anchors` `.ts` | Linux-side Mach-O + CMS chain check before publish; `TEAM_ID 523JNR86LZ` |
| Computer-use engine QA | `computer-{engine,fixtures,live-linux,live-windows-full}.yml`, `trust-mutations.yml`, `engine-trust-helper.yml` | fixtures include sabotage/negative controls |
| Desktop QA | `desktop-{bench,engine,linux-qa,windows-qa}.yml` | explicit failure controls on Linux/Windows legs |
| PR flow automation | `pr-automation.yml`, `pr-automation-signal.yml`, `cla.yml`, `changelog-gate.yml`, `review-claims.yml`, `bot-merge.yml` | CLA state shared via `cla_state` step id |
| npm ops | `npm-deprecate-version.yml`, `npm-dist-tag-rollback.yml` | exact-semver guarded; a live release / current latest is never deprecated |
| get.omo.dev / web / mirror | `get-worker-{ci,deploy}.yml`, `web-{ci,deploy}.yml`, `installer-mirror.yml` | Cloudflare Workers + R2 |
| Sisyphus runs | `sisyphus-agent.yml` (569 lines), `cutover-target-deploy.yml` | manual deploy |
| Workflow lint | `lint-workflows.yml` | actionlint v1.7.10, `-shellcheck=""` |
| Reusable (`workflow_call`) entry points | `ci.yml`, `installer-mirror.yml`, `pr-platform-decision.yml`, `publish-platform.yml`, `release-announce.yml` | typed inputs, job outputs |

## CONVENTIONS

- Toolchain pins: Node 24 (`actions/setup-node@v7`), Bun 1.4.4 via `oven-sh/setup-bun` pinned to full commit SHA `0c5077e5…68d6` (`# v2`).
- Every workflow declares least-privilege `permissions`; concurrency groups cancel superseded PR runs but preserve push/schedule/release baselines.
- The 5 TS modules are ESM with explicit `.ts` extensions and readonly types, run directly under Bun with no build step; they live here so a release source-state branch can take them as a workflow-only commit.
- `ubuntu-26.04` labels are legitimized by `../actionlint.yaml` (actionlint 1.7.10 predates the hosted label).

## ANTI-PATTERNS

- **NEVER** trust CMS-supplied certificate intermediates in `operator-darwin-cms.ts`: trust is rooted only in the pinned Apple certs of `operator-darwin-anchors.ts` (sha256 re-checked at load); verify signer identity + signed attributes, never a signer-selected certificate.
- **DO NOT** outsource Mach-O signature verification to an external tool: `operator-darwin-macho.ts` hashes signed CodeDirectory coverage + special blobs itself; arm64e and malformed input are refused via `Refusal`.
- `operator-darwin-verify.ts` refusal contract: exactly ONE `::error::operator_darwin_artifacts refused: ...` line on stdout + exit 1; success writes summary rows to `--summary` and prints nothing else.
- CMS parser caps indefinite-length DER nesting at 32 (stack-exhaustion guard); primitive indefinite-length DER is refused.
- **NEVER** treat a skipped CI leg as passing - macOS status helpers fail closed (see parent `.github/AGENTS.md`); report-only temp-leak lists may only shrink, never grow.

## COMMANDS

```bash
# Lint all workflows (what lint-workflows.yml runs)
bash <(curl -sSL https://raw.githubusercontent.com/rhysd/actionlint/v1.7.10/scripts/download-actionlint.bash)
./actionlint -color -shellcheck=""

# Darwin artifact verifier (invoked by publish-platform.yml)
bun .github/workflows/operator-darwin-verify.ts --dir <downloaded assets> \
  --tag operator-darwin-<sha12> --manifest-sha256 <64-hex> --prepared-sha <40-hex> \
  --version <x.y.z> --repo <git dir> --actor <login> --triggering-actor <login> \
  --repository <owner/name> --summary <markdown file>
```
