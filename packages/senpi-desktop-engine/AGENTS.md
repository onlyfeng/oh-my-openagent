# senpi-desktop-engine — Binary Locator + ABI Handshake

**Generated:** 2026-11-18 / c2691818a9

**Score:** 25 (31 files, ~5.1k LOC). Finds the `senpi-desktop-engine` binary (built from `crates/senpi-desktop-engine`) for this host and proves it speaks the host's protocol; it implements NO desktop behavior — behavior lives behind the stdio JSON-RPC this package validates. Parent: [`packages/AGENTS.md`](../AGENTS.md) (desktop-family row).

## STRUCTURE

```
src/          # locator, handshake, acquire/launch, release assets, typed refusals
bunshin/      # sidecar descriptor (descriptor.mjs) derived from the engine protocol schema + desktop.capability.json
test/         # vitest suites (bun run test:desktop)
```

## WHERE TO LOOK

| Task | File |
|------|------|
| Binary discovery | `src/locator.ts` — `locateDesktopEngine()`, candidate paths, `DesktopEngineLibc` (Linux libc distinction), quarantine probe (`QUARANTINE_ATTRIBUTE`) |
| ABI/protocol proof | `src/handshake.ts` — `helloDesktopEngine()` spawns `--stdio`, sends `engine.hello`, resolves only when `abi === ENGINE_ABI` and `protocolVersion === PROTOCOL_VERSION` (else `DesktopEngineAbiMismatchError` naming both versions, or `DesktopEngineHandshakeError`) |
| Download + cache + launch | `src/acquire.ts` (`acquireDesktopEngine`, `findCachedDesktopEngine`, `launchDesktopEngine`), `src/release-assets.ts` + `src/checksums.ts` (SHA-256), `src/release-signature.ts` |
| Typed launch failures | `src/launch-refusal.ts` — `DesktopLaunchRefusal`, user-facing error info |
| Stable-launch path | `src/stable-launch.ts` |
| Bunshin sidecar contract | `bunshin/descriptor.mjs` (derived from the protocol schema so ops/effects/params cannot drift) + `bunshin/desktop.capability.json` |

## LOCATOR ORDER (first hit wins)

0. extracted runtime: `$OMO_PACKAGE_DIR/native/prebuilds/<platform>-<arch>/`
1. compiled sidecar beside `process.execPath` — quarantine-exempt ONLY when its canonical path stays inside `native/prebuilds/` and its SHA-256 matches the as-shipped checksums file
2. vendored package prebuild
3. dev build: `<repo>/target/release/`

## CONVENTIONS

- macOS `com.apple.quarantine` candidates are skipped and REPORTED — the attribute is never cleared; only the checksum-verified sidecar rule above is exempt.
- Checksum files: two-space SHA-256 lines, known release asset names, no duplicates; the digest is AS SHIPPED (post re-sign), not necessarily the upstream release digest.
- Engine talks stdio JSON-RPC; never launch before the handshake succeeds.
- May import `@oh-my-opencode/senpi-desktop-protocol` ONLY — no other workspace package, no `pi-agent-core`/`pi-ai`/`pi-tui` (enforced by the desktop package-boundaries test; see the protocol README).

## COMMANDS

```bash
bun run test:desktop    # senpi-desktop vitest suites (repo root)
```
