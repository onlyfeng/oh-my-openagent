# senpi-desktop-engine

Locates, acquires, verifies, and handshakes the Rust `senpi-desktop-engine` binary for the `computer` tool. Implements no desktop behavior. Contract, candidate order, prebuild and crate layering: `README.md` (read it first). Own file: distinct domain (binary trust boundary), 25 files, exports >10, score 10.

## WHERE TO LOOK

| Path | Purpose |
|------|---------|
| `src/locator.ts` | `locateDesktopEngine`: ordered candidates, libc detection (`linux-<arch>-musl` host names), macOS quarantine check; `quarantined` outranks `native-unavailable` in diagnostics. |
| `src/acquire.ts` | `acquireDesktopEngine` / `launchDesktopEngine` / `findCachedDesktopEngine`: release download, checksum validation, verified cache generations. Async; leaves the sync locator and `enginePath` override untouched. |
| `src/stable-launch.ts` | `launchStableEngine` + `engineSha256`: the spawn callback must be synchronous (posix_spawn commits the image before the hash can change). |
| `src/release-signature.ts` / `release-assets.ts` / `checksums.ts` | `isDesktopEngineRelease`; release asset names and hosts shared with the publisher; one checksum grammar for upstream and as-shipped digests. |
| `src/handshake.ts` | `helloDesktopEngine`: `engine.hello` over `--stdio`; ABI/protocol mismatch throws `DesktopEngineAbiMismatchError`, no reply `DesktopEngineHandshakeError`. |
| `bunshin/` | `desktop.capability.json` (largest file, operation/schema/effect snapshot) + descriptor; pinned by `test/bunshin-descriptor.test.ts`. |

## CONVENTIONS

- Vitest, not bun test: `bun run --cwd packages/senpi-desktop-engine test` (`vitest --run test`); typecheck via tsgo. Root `bun run test:desktop` runs all five desktop packages after the cargo pin check.
- Only workspace dependency: `@oh-my-opencode/senpi-desktop-protocol` (README import-direction rules; the README's `scripts/desktop-package-boundaries.test.mjs` enforcer is not in this checkout, so review edges by hand).
- Consumers: `senpi-desktop-service`, omo-senpi `computer-use`, omo-native `computer-use-doctor-runtime.ts`.

## ANTI-PATTERNS

- Never clear the `com.apple.quarantine` attribute; only the checksum-verified compiled sidecar may bypass the refusal.
- Never let a local or ad-hoc build replace the permission-bearing release executable (`release-signature.ts`).
- Never hash after spawning asynchronously; `stable-launch.ts` exists to close that window.

## COMMANDS

```bash
bun run --cwd packages/senpi-desktop-engine test        # vitest suite
bun run --cwd packages/senpi-desktop-engine typecheck   # tsgo
bun run test:desktop                                    # cargo pin check + all five desktop packages
```

The README's `bun run check:prebuild` and `scripts/build-desktop-engine-local.mjs` are not defined in this checkout; a dev engine comes from `cargo build --release -p senpi-desktop-engine --locked` into `target/release/`.
