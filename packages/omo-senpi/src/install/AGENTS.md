# src/install/ — Senpi Installer

**Score ~12** (12 files, ~1.7k LOC; install/launch orchestration domain).

## OVERVIEW

The omo-senpi local installer: orchestrates installing/uninstalling the Senpi plugin payload, launching a local CLI, verifying plugin artifacts, and writing settings atomically. Barrel `index.ts` re-exports `runSenpiInstaller` / `runSenpiUninstaller`; `cli-local.ts` consumes the installer API.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel: `runSenpiInstaller`, `runSenpiUninstaller` |
| `install-senpi.ts` | Install orchestration |
| `cli-local.ts` | Local CLI entry consuming the installer |
| `local-launcher.ts` | Local launcher |
| `plugin-artifacts.ts` | Plugin artifact verification (~212 LOC) |
| `senpi-settings.ts` | Atomic settings read/write helpers |
| `install-test-fixture.ts` | Test fixture |

## CONVENTIONS

- Logic is split across orchestration, local launcher, plugin artifacts, and settings helpers.
- Settings writes are atomic (temp + rename).

## ANTI-PATTERNS

- Current install is local-path only — do NOT document npm/git/marketplace distribution for the Senpi adapter skill surface.
- Keep settings writes atomic; never write settings in place.

## COMMANDS

- `bun test packages/omo-senpi` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json` · `bun run test:senpi`
