# src/release - Device-Service Release Acquisition

**Generated:** 2026-11-18 | c2691818a9

Release-manifest download + signature trust for the device-service binary. Distinct domain: the only place this package touches the network and cryptographic trust for the published binary.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `acquire.ts` | `acquireDeviceService`: the public acquire entry. Returns `AcquireOptions` / `AcquireResult` / `AcquireRefusal` and honors `DEVICE_SERVICE_ORIGIN` (overridable for testing). |
| `release-assets.ts` | Asset host/name helpers and the `ReleaseManifest` shape. |
| `release-signature.ts` | Checksum parsing and signature verification against `release-key.json` (the pinned public key). |
| `release-key.json` | The embedded public key the signature is verified against. |
| `version.ts` | Version constants and pinning. |
| `windows-dacl.ts` | win32 DACL handling on the installed binary directory. |

## ANTI-PATTERNS

- NEVER trust a downloaded asset without passing `release-signature.ts` verification; the embedded key in `release-key.json` is the trust root.
- `DEVICE_SERVICE_ORIGIN` is an override seam for tests; production callers rely on the default origin.
- The cached binary path is keyed by the pinned version; do not float it or share a path across versions.

## CONVENTIONS

- All five TS modules are pure except `acquire.ts` (network) and `windows-dacl.ts` (win32 FS).
- Tests are co-located `*.test.ts` under `bun test`.

## COMMANDS

```bash
bun test --timeout 20000 packages/device-service-core
bun run --cwd packages/device-service-core typecheck
```
