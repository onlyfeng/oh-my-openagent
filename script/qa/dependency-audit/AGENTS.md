# script/qa/dependency-audit/ -- Dependency-Diet Receipts

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Capture/compare receipt subsystem behind `../dependency-audit-{capture,compare}.ts`: measures the native release binary's dependency diet (bytes, graph, startup, rpc, extension, webfetch, photon, changelog, providers, skills) into normalized receipts, then validates the receipts against gates.

## STRUCTURE

| Module | Role |
|---|---|
| `contracts.ts` | Case registry (`CASES`), receipt schemas, gate definitions -- the subsystem contract |
| `runtime.ts` / `inventory.ts` / `graph.ts` / `providers.ts` | Case capture machinery: sandbox provisioning, binary copy+hash+inventory, Bun `--metafile` graph capture, provider probes |
| `rpc.ts` / `webfetch.ts` / `startup.ts` / `changelog.ts` / `session-cases.ts` / `model-server.ts` | Per-case capture logic |
| `comparison.ts` | Receipt validation and baseline-vs-post case behavior checks; exits nonzero on a failed gate or malformed/incomplete receipts |
| `README.md` | Full operating manual: host requirements, gates, baseline provenance |

## COMMANDS

```sh
bun script/qa/dependency-audit-capture.ts --phase baseline|post --binary <bin> --out <dir>
bun script/qa/dependency-audit-compare.ts --baseline <dir> --post <dir> --gate p0
bunx --no-install tsgo --noEmit -p script/qa/tsconfig.json
```

Acceptance runs on the designated remote test host (Bun 1.4.2, Node >= 24, hyperfine, repo deps installed, `gh` for release inventory) -- never on the orchestration host.

## CONVENTIONS

- Capture exits zero when RECORDING completes -- including recorded probe failures. Check each row's `pass`/`exitCode`, or run compare; a partial capture can never satisfy a full compare.
- Each case copies + hashes the binary first, provisions a fresh HOME/USERPROFILE/XDG/agent sandbox, and removes it in `finally`; env is allowlisted with only fake credentials entering the audited process; telemetry offline/opt-out set; fixtures bind loopback ephemeral ports; GitHub release-asset downloads are never executed.
- Receipts normalize ephemeral paths and loopback ports; graph capture reuses the current release compile-arg array and FAILS on an unrecognized expression rather than silently drifting.
- Gates (see README for all): darwin-arm64 P0 <= 104,857,600 B / P1 <= 94,371,840 B; other targets <= 80% of own baseline and <= 157,286,400 B; graph >= 1,000 modules; startup = 30 hyperfine runs after 3 warmups with limit baseline mean x 1.10 (version) / x 1.15 (one-shot) + `3*sqrt(baseline.var/30 + post.var/30)`; skills = exact sorted path/size/SHA256 manifest equality (an intentional change needs an explicitly reviewed reference manifest; never auto-accepted).

## ANTI-PATTERNS

- `../fixtures/dependency-audit/baseline-76e54b0-806f8e0/` records the exact audited dev artifact (130,850,802 B, copied -- never rebuilt); the published release asset is NOT byte-identical to it.
- Known provider RED: all audited provider modules fail to resolve in both isolates, and the audited dev launcher omits `brand.changelog.version` (`whatsNew: false`, `pass: false`) -- comparison deliberately fails these; no baseline exemption or hidden patch.
