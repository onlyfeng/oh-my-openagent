# omo-git-job-helper

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Windows-only sandboxed git-job helper (omo#10391, design v4.2): runs one git operation inside a Windows job object with kill-on-close, `run --op <id> [--linger-ms <n>]` / `probe` / `selftest`. Every other OS exits 2. Earned its file as its own sealed domain: kernel job-object behavior plus a portable, host-testable CLI contract.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/cli.rs` | The PORTABLE contract: `Command` (`Run { op, linger_ms }` / `Probe` / `Selftest`), `Exit` (16 observable codes), `DEFAULT_LINGER_MS` 600_000, `LINGER_KILL_CODE` 0xC0DE_0001. Unit-tested on every host. |
| `src/frames.rs`, `src/probe_lines.rs` | Portable wire frames and probe output parsing. |
| `src/win.rs`, `src/job.rs`, `src/spawn.rs`, `src/run.rs`, `src/members.rs`, `src/control.rs`, `src/security.rs`, `src/probe.rs`, `src/selftest.rs` | `#[cfg(windows)]` implementation: job create/assign/limits, spawn, process-tree membership, control channel, security descriptor, probe, selftest. |
| `tests/` | Contract integration rows across many focused `n*.rs` files. |
| `testkit/`, `examples/` | Test-only helper binaries (`token.rs`, `win.rs`); never shipped. |

## CONVENTIONS

- Portable-vs-Windows split is strict: `cli`, `frames`, `probe_lines` compile everywhere and carry the argument/exit contract; everything else is windows-gated and dead-code-allowed elsewhere.
- Exit codes are observable outcomes, not debug channels. There is deliberately NO separate "kill failed" code: `TerminateJobObject(F, LINGER_KILL_CODE)` and process rundown are indistinguishable to any observer, so both report `LINGER_KILL_CODE` (`cli.rs` doc).
- After the join, the job handle is never closed before exit (`job::JoinedJob`) - kill-on-close must stay the last resort, not an extra exit path.
- `OpId` accepts only 1-64 ASCII alphanumeric/hyphen bytes, keeping pipe and job names inside their namespaces.

## ANTI-PATTERNS

- Never relax `OpId` validation - names ride into Windows kernel object namespaces.
- Never add a Windows-only branch to the portable modules; extend `Exit`/`Command` portably and gate behavior in dispatch.
- Never ship or install the testkit/example binaries.

## COMMANDS

```bash
cargo test -p omo-git-job-helper --locked          # portable contract rows on every host; full kernel rows on Windows
cargo clippy -p omo-git-job-helper --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
