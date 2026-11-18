# feedback — `/feedback` + `/bug` Commands (omo-senpi component)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 10 (19 files, own barrel, distinct domain: redacted user feedback collection). Registers two commands (`feedback`, `bug`) and two RPC methods (`omo.feedback.prepare` / `omo.feedback.submit`, the `FEEDBACK_*_RPC_METHOD` constants) against `https://api.omo.dev`.

## STRUCTURE

| File | Role |
|------|------|
| `index.ts` | `createFeedbackComponent` — command + RPC registration, host-redactor wiring (`pi.redactForUpload`), credential-store injection, `FeedbackComponentOptions` |
| `collect.ts` + `collect-host.ts` | Draft collection flow |
| `draft.ts` | `FeedbackDraft` / `FeedbackKind`, `UploadRedactor` |
| `submit.ts` | `ReportTransport`, `SubmitResult` |
| `runtime.ts` + `runtime-loader.ts` | The lazily loaded feedback runtime (`FeedbackCore`, `FeedbackCommandUi`, `FeedbackSessionLike`); load failures surface as `feedbackRuntimeLoadFailure` |
| `fixtures/service-contract/`, `report-file.test-support.ts` | Pinned service contract + report-file harness |

## CONTRACT

Submit requires a redactor: without `pi.redactForUpload` the component answers `{ kind: "refused", reason: "unavailable" }` and the commands notify `FEEDBACK_UNAVAILABLE`. Every upload passes through the redactor; the reported count is the sum of `redactForUpload`'s per-kind counts. Runtime loads lazily per command/RPC call — a runtime load failure becomes a UI error, never a broken registration.

## ANTI-PATTERNS

- Do not upload unredacted text; the redactor seam is mandatory, not best-effort.
- Do not inline the runtime — load it through `runtime-loader.ts` (keeps the entry bundle flat).
- Do not read credentials directly; `CredentialStore` is injected from `@oh-my-opencode/omo-opencode/service-auth`.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/feedback
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```
