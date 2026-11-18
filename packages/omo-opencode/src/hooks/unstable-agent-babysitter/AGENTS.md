# src/hooks/unstable-agent-babysitter/ - Unstable Task Agent Babysitter

## OVERVIEW

4 files, ~980 LOC. Continuation-tier hook that watches sessions spawned for
unstable tasks and nudges them when the agent stalls or wanders: analyzes
task messages, detects instability, and injects targeted reminders. Wired in
`create-continuation-hooks.ts`. Parent: [`src/hooks/AGENTS.md`](../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel exporting `createUnstableAgentBabysitterHook` |
| `unstable-agent-babysitter-hook.ts` | Factory + session.idle handling |
| `task-message-analyzer.ts` | `extractMessages`, `isUnstableTask`, `buildReminder`, message info helpers |
| `index.test.ts` | Colocated suite |

## CONVENTIONS

- Detection is pure (`task-message-analyzer.ts`); the hook composes it with
  session state only.
- Reminders are injected only when the session is a recognized unstable-task
  session; other session types pass through untouched.

## ANTI-PATTERNS

- Do not widen `isUnstableTask` heuristics without colocated coverage -
  false positives spam sessions.
- Do not make blocking calls from the idle handler.

## QA

```bash
bun test packages/omo-opencode/src/hooks/unstable-agent-babysitter
bun run --cwd packages/omo-opencode typecheck
```

Wired into OpenCode: real-harness QA required for changes (root AGENTS.md).
