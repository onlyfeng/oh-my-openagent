# session-history — Onboarding Session Scan (omo-senpi component)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 14 (33 files, two barrels, distinct domain: bounded multi-agent transcript scanning for onboarding). Exposed as `@oh-my-opencode/omo-senpi/onboarding/session-history`; `runtime.ts` is the runtime-extension surface.

## PIPELINE

`scanSessionHistory` (async iteration; `scan-session-history.ts` + `createSessionHistoryScanner`) walks three source kinds (`claudeAgent`, `codex`, `omo` — `types.ts`):

1. Agent homes (`agent-transcripts.ts` `defaultAgentHomes`) → `transcript-discovery.ts` → bounded reads (`transcript-reader.ts`, prefix + metadata reads) → `transcript-parser.ts` / `transcript-records.ts` / `codex-canonical-turns.ts`, with `transcript-cwd.ts` for project attribution.
2. omo session dirs (`omo-session-discovery.ts`, header reads only; a startup-only session is recognized by the 256 KiB size heuristic).
3. Project candidates (`project-candidates.ts`, `project-paths.ts`, `git-identity.ts`) → newest/recent windows (`newest-sessions.ts`, `recent-sessions.ts`).

Digests (`session-digest.ts`, `digest-store.ts` → `createSessionDigestStore` / `createMemorySessionDigestStore`) persist one bounded digest per session.

## BUDGETS (`constants.ts` — hard caps; keep them)

Transcripts 5000/source (discovery ops 4x), metadata 64 MiB/source (100k records / 1k per transcript), prefix read 32 KiB, digest 4 KiB, recent window 30 d. Import path: 100 transcripts, 200 messages, 4 MiB per record. omo discovery: 20 000 ops / 5 000 headers.

## INJECTION

Filesystem access goes through `SessionHistoryFs` (`session-history-fs.ts`); `nodeSessionHistoryFs` is the node adapter. Tests use `test-support.ts` fakes — no test touches a real agent home.

## ANTI-PATTERNS

- Do not add an unbounded read: every scan/import path must carry one of the constants above.
- Do not parse a transcript format ad hoc; extend `transcript-parser.ts` and pin it in `transcript-parser*.test.ts`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/onboarding/session-history
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
