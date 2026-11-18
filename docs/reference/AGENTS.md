# docs/reference/ — Reference Pages

**Generated:** 2026-11-18 / c2691818a9 · Earned: score 23 (34 pages > 20; distinct genres; consumed by web CI). Per-page index lives here, not in `docs/AGENTS.md`.

## OVERVIEW

34 API / config / CLI / protocol / threat-model pages; the internal-engineering subset (threat models, RFCs, inventories) is design and ops documentation, not user docs.

## WHERE TO LOOK

| Page | Group | Role |
|------|-------|------|
| [configuration.md](configuration.md) | Config | Field reference (1,626 lines); layering: user layer lowest, nearest project file wins |
| [omo-json.md](omo-json.md) | Config | Harness-neutral `omo.json`/`omo.jsonc`; strict typed blocks, precedence, `[opencode]`/`[native]`/`[codex]` blocks (`[senpi]` = legacy spelling for `[native]`) |
| [opencode-config.md](opencode-config.md) | Config | LEGACY: `[opencode]` keys only the OpenCode edition reads (verbatim moves from configuration.md/features.md) |
| [cli.md](cli.md) | CLI | CLI command reference |
| [omo-daemon.md](omo-daemon.md) | CLI | Engine hosts per session, `omo daemon`, migration and rollback |
| [omob-dev-binary.md](omob-dev-binary.md) | CLI | `omob`: single-file dev binary built from latest mainline commits (`bun run omob`) |
| [omo-thread.md](omo-thread.md) | Threads | `omo thread` CLI + thread SDK; JSON shapes, exit codes, outbox contract |
| [remote-sessions.md](remote-sessions.md) | Threads | `omo remote`: verbs, states, command outcomes, device service install |
| [computer.md](computer.md) | Security | Native `computer` tool contract |
| [computer-stop-authority.md](computer-stop-authority.md) | Security | Protected computer-use stop authority design (#10238, revision 2, local admission slice) |
| [computer-run-sandbox-threat-model.md](computer-run-sandbox-threat-model.md) | Security | `computer.run` QuickJS sandbox threat model + compatibility matrix |
| [computer-use-eval.md](computer-use-eval.md) | Security | 7-task / 3-arm eval + latency bench baselines (fake engine, never a real model) |
| [desktop-attach-threat-model.md](desktop-attach-threat-model.md) | Security | Desktop engine attach and `--serve` multi-connection threat model |
| [keychain-helper-custody-threat-model.md](keychain-helper-custody-threat-model.md) | Security | macOS keychain helper custody (where the helper runs, what the ACL trusts) |
| [engine-trust-user-namespaces.md](engine-trust-user-namespaces.md) | Security | Engine trust under user namespaces: bwrap sandbox + inode attestation — design, under review |
| [features.md](features.md) | Product | Feature-by-feature reference (1,212 lines) |
| [monitor.md](monitor.md) | Product | Monitor tool reference |
| [mass-ulw-protocol.md](mass-ulw-protocol.md) | Product | mass-ULW dag protocol: events, catch-up, overflow |
| [senpi-telemetry.md](senpi-telemetry.md) | Telemetry | Senpi telemetry internals |
| [codex-telemetry.md](codex-telemetry.md) | Telemetry | Codex telemetry internals |
| [release-process.md](release-process.md) | Ops | Release process (`bun run typecheck` gate) |
| [operator-built-darwin-release.md](operator-built-darwin-release.md) | Ops | Operator-built darwin artifacts replacing hosted macOS legs; verification, provenance |
| [changelog-fragments.md](changelog-fragments.md) | Ops | Per-PR `changes/unreleased/<n>.md` fragments, migration, batches |
| [omo-ai-publishing.md](omo-ai-publishing.md) | Ops | npm publishing via GitHub Actions + npm OIDC ONLY |
| [lazycodex-npm-reservation.md](lazycodex-npm-reservation.md) | Ops | Claiming the lazycodex npm name through `publish.yml` |
| [known-issues.md](known-issues.md) | Ops | Known issues & workarounds |
| [prompt-async-gate-rfc.md](prompt-async-gate-rfc.md) | Internal | `prompt_async_gate` deep-dive |
| [opencode-prompt-composition.md](opencode-prompt-composition.md) | Internal | Engine-owned prompt presets; the adapter never selects a local model body |
| [rpc-relay-parity.md](rpc-relay-parity.md) | Internal | GENERATED engine RPC local-vs-relay parity table — regenerate, never hand-edit |
| [re-export-shim-inventory.md](re-export-shim-inventory.md) | Internal | Snapshot inventory of adapter re-export shims (`@oh-my-opencode/*`) |
| [shared-core-multi-pr.md](shared-core-multi-pr.md) | Internal | Shared-core multi-PR extraction QA |
| [rules-injection-cross-module-comparison.md](rules-injection-cross-module-comparison.md) | Internal | Rules-injector cross-module comparison |
| [web-terminal-visual-qa.md](web-terminal-visual-qa.md) | Internal | Real-pty + xterm.js visual QA helper (`script/qa/web-terminal-visual-qa.mjs`) |
| [github-attachment-upload.md](github-attachment-upload.md) | Internal | GitHub PR evidence attachments |

## CONVENTIONS

- Threat models state their scope and separate observed behavior / assumptions / decisions / required tests.
- Design docs carry their status ("design, under review") — proposals, not shipped behavior.
- Generated pages open with a GENERATED comment naming the generator and its regen command.

## ANTI-PATTERNS

- Never hand-edit [rpc-relay-parity.md](rpc-relay-parity.md): record a relay verdict or pair test in `script/rpc-relay-parity.rows.json`, then run `bun script/rpc-relay-parity.ts` — `script/rpc-relay-parity.test.ts` fails until the table is regenerated.
- Never present `opencode-config.md` keys as cross-edition: omo-senpi ignores every one of them.
- Publishing is GitHub Actions + npm OIDC only; never add a local publish path or a manual `NPM_AUTH_TOKEN` publish ([omo-ai-publishing.md](omo-ai-publishing.md), [lazycodex-npm-reservation.md](lazycodex-npm-reservation.md)).
- PRs never edit root or package `changes.md` — one unreleased changelog fragment per shipping PR ([changelog-fragments.md](changelog-fragments.md)).
- Compatibility re-export shims stay thin; remove one from the inventory only after in-repo consumers import the extracted package path directly ([re-export-shim-inventory.md](re-export-shim-inventory.md)).
- Config: `reasoning` is canonical; `reasoningEffort` and `variant` are deprecated compatibility inputs (precedence reasoning > reasoningEffort > variant). `mcp_env_allowlist` and `browser_automation_engine.playwright_mcp_args` are user-layer-only — project layers cannot extend them.
- `omo thread` connectors: never run `outbox --ack` before posting rows (a crash loses the page); delivery is at-least-once, so deduplicate by `(binding_id, cursor)`; the SQLite WAL is not a supported wake signal.
