# components/claude-code - Claude Code Binary Acquisition

**Generated:** 2026-11-18 | c2691818a9

Senpi component that pins, downloads, and caches the Claude Code executable used for the `/claude` compatibility surfaces. Distinct domain: the only place this adapter fetches a third-party binary.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `acquire.ts` | `tarballUrl`, `cachedExecutablePath`, `offlineMessage`; pin/acquisition types. |
| `find-on-path.ts` | `findOnPath` - locate a pre-installed binary before downloading. |
| `index.ts` | Component factory + re-exports; `executableName`. |
| `acquire.test.ts`, `index.test.ts` | bun test coverage. |

## ANTI-PATTERNS

- The pin is the contract: do not float the version. The cached path is derived from the pinned version so an upgrade produces a fresh cache entry.
- Offline failures must surface `offlineMessage`; do not silently substitute a system binary when the pin is unreachable.
- `findOnPath` is the probe for an already-installed binary; do not skip it on the hot path or you will re-download on every boot.

## CONVENTIONS

- Tests are co-located `*.test.ts` under `bun test`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/claude-code
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

Senpi-wired; `senpi-qa` skill + `.omo/evidence/omo-senpi-adapter/<slug>/` per the root AGENTS.md mandate.
