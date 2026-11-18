# ast-grep/ - `sg` Binary Provisioning (utils)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 10 (14 files, own barrel, distinct domain: binary resolution + provisioning). Parent [`../../AGENTS.md`](../../AGENTS.md) keeps one row; this file maps the modules.

## STRUCTURE

| File | Role |
|------|------|
| `index.ts` | Barrel re-exporting all seven modules |
| `sg-manifest.ts` | The pinned sg version/platform manifest |
| `sg-resolver.ts` | Resolution + cache paths; platform/arch normalized explicitly; suppresses a known deprecated-command warning (`sg-resolver.test.ts` pins the suppression) |
| `sg-provisioner.ts` | `provisionSgBinary()` - plan, download, verify, extract into the cache |
| `install-script.ts` | Generates the standalone install script consumers run |
| `sg-candidates.ts` | Candidate planning (which binary fits the host) |
| `sg-install-hints.ts` | User-facing install hints |
| `types.ts` | Shared types + errors |
| `sg-resolution.test.ts` (498 lines), `sg-probe-contract.test.ts` (271) | The two contract suites: the resolution matrix, and the probe contract the opencode `astGrepSgProvision` hook and `ast-grep-mcp` rely on |

## CONVENTIONS

- Named environment-key constants; no inline env strings.
- Platform/architecture pairs are normalized through the manifest tables, never stringly compared at call sites.
- The deprecated-command suppression in `sg-resolver.ts` is intentional and test-pinned - do not "clean it up".

## ANTI-PATTERNS

- Do not download or extract outside the provisioner's cache layout.
- Do not widen the manifest's platform table without a matching `sg-resolution.test.ts` case.

## COMMANDS

```bash
bun test packages/utils/src/ast-grep
```
