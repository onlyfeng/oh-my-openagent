# .devcontainer

Containerized dev/QA environment for GitHub Codespaces, VS Code Dev Containers, and plain Docker. Earned its file: score 14 (distinct infra domain; referenced by 11 external files: QA skills, `script/agent/*`, guard tests). Its invariants are operational, not code.

## WHERE TO LOOK

| File | Role |
|------|------|
| `devcontainer.json` | Container entry: builds `Dockerfile`; `postCreateCommand` runs `setup.sh` with `OMO_AGENT_FORCE_BUILD=1` (fresh container has no `dist/`); forwards `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` via `remoteEnv` |
| `Dockerfile` | Dev image (tag `omo-dev`): Node 24 bookworm base + Bun 1.4.4 + tmux; ends `USER node` |
| `qa.Dockerfile` | QA image (tag `omo-qa`): `FROM omo-dev` + sqlite3/jq/curl/rsync + released `opencode-ai` / `@openai/codex` CLIs (prebuilt binaries, no source build) |
| `qa-entrypoint.sh` | QA ENTRYPOINT: rsyncs read-only `/mnt/host/*` config mounts into writable `$HOME` copies, then `exec "$@"` |
| `README.md` | Human guide: credential injection, optional harness-config bind mounts |

## CONVENTIONS

- **Build order belongs to `script/agent/qa-docker.sh`, never to hand-run `docker build`:** it builds `omo-dev` first, then `omo-qa` (because `qa.Dockerfile` is `FROM omo-dev`); building the QA image directly hits a stale or missing base.
- **Bun pin is three-way:** `.devcontainer/Dockerfile`, `ci.yml` (`oven-sh/setup-bun`), and `setup.sh`'s drift warning must agree; `script/agent-env.test.ts` enforces it.
- **Pin QA CLIs per run with build args** `OMO_OPENCODE_VERSION` / `OMO_CODEX_VERSION` (default `latest`) when a released-version regression matters.
- **Both images run non-root** (`USER node`, home `/home/node`); `qa.Dockerfile` elevates to root only for apt/npm layers.
- **Credentials pass through, never bake in:** `remoteEnv` `${localEnv:...}` in `devcontainer.json`; no key ever appears in an image layer.
- **QA isolation is copy-in, not bind:** host config mounts are read-only (`/mnt/host/opencode-config` → `$HOME/.config/opencode`, `/mnt/host/codex` → `$HOME/.codex`); the entrypoint rsyncs with mandatory excludes (`cache/`, `.cache/`, `__pycache__/`, `.ruff_cache/`, `node_modules/`, `*.log`); container runs `docker run --rm` and is discarded.

## ANTI-PATTERNS

- NEVER bump Bun here without CI: `agent-env.test.ts` fails the mismatch.
- NEVER bind-mount live host config for QA: bind mounts are read-write. QA uses the copy-in entrypoint inside Docker, or `script/agent/qa-sandbox.sh` on the host.
- NEVER trust a green QA run blindly: `qa-entrypoint.sh` tolerates rsync failure (`|| true`), so a misconfigured mount yields default config, not an error.
- Codespaces has no host filesystem to mount: re-authenticate inside the container instead.
- Windows has no Docker QA path: `script/agent/qa-docker.sh` exits 3; run the QA skills' local scripts instead.

## GUARD TESTS

`script/agent-harness-wiring.test.ts` pins this directory as a set: `devcontainer.json` strict JSON + builds the Dockerfile + `remoteEnv` creds passthrough; Dockerfile base `mcr.microsoft.com/devcontainers/javascript-node`; `qa.Dockerfile` `FROM omo-dev`; entrypoint copy-in contract; README existence. Editing any file here without keeping these invariants fails the root suite. Image/bootstrap/config changes also follow the root `MAINTENANCE - KEEP THIS IN SYNC` directive (this README + `setup.sh` + root `AGENTS.md` + `CONTRIBUTING.md` together).
