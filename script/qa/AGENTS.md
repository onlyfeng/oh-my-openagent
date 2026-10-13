# script/qa/ -- Live QA Drivers

## OVERVIEW

Purpose-built QA harnesses that drive real binaries (senpi/omo-native, Codex marketplace, desktop engine, xterm). No package manifest: invoke each file with its runtime (`bun`, `node`, `bash`) per its shebang/docs. Earned its own file: ~120 files across direct drivers + `desktop/` + `dependency-audit/`, own `tsconfig.json`, distinct isolation rules.

## WHERE TO LOOK

| Area | Files | Notes |
|------|-------|-------|
| Codex marketplace | `codex-marketplace-e2e.sh` | 743-line single install/upgrade scenario, marked `SIZE_OK` |
| Terminal evidence | `web-terminal-visual-qa.mjs`, `xterm-live-terminal.mjs`, `web-terminal-redaction.mjs`, `strip-ansi.mjs` | Renders TUI evidence through real xterm.js + node-pty in a browser, true color; NEVER tmux capture-pane |
| Telemetry privacy | `omo-native-telemetry-qa.mjs` + `omo-native-telemetry-{provider,drive,capture,assertions,evidence}.mjs` | `--evidence-dir <dir> --senpi-bin <path>`; see TELEMETRY below |
| Eval smoke | `omo-native-eval-smoke*.mjs` | Host/provider/sandbox/run/stop split; stop path has its own test |
| Parity smoke | `omo-native-parity-{smoke,compare,provider}.mjs` | Scripted provider vs compare |
| Pool/kernel probes | `omp-item2*.{ts,mjs}`, `omp-item6*.ts`, `omp-item9.ts` | Admission, denials, stale pools, revive, scope producer |
| Member boot | `member-boot-profile.ts`, `member-boot-diff.ts` | Team member boot profile + diff |
| Dependency diet | `dependency-audit-capture.ts`, `dependency-audit-compare.ts`, `dependency-audit/` | `contracts.ts` (Zod schemas, gates), `runtime.ts` (sandbox, normalize, hash), one module per case; `README.md` is the runbook |
| Desktop | `desktop/{linux,macos,windows}.ts` + per-OS dirs | Linux `engine.ts` (`Engine`, ABI handshake, fake-eis Rust helper), macOS `agent.ts` (`AgentSession`, sandbox), Windows PowerShell probes |
| Types | `types/senpi-pi-tui.d.ts` | Re-exports `sanitizeTerminalLabel` from `@code-yeongyu/senpi` |

## TELEMETRY DRIVE

Adjacent modules own the scripted provider, isolated RPC lifecycle, capture server, privacy checks, and evidence output. Prompt progression waits for `agent_settled`; failure termination reserves a one-second grace inside the 120-second total drive budget, then escalates to SIGKILL without losing the original error. Evidence includes each session JSONL, redacted payloads, opt-out checks, and cleanup receipts.

## COMMANDS

```bash
bunx --no-install tsgo --noEmit -p script/qa/tsconfig.json
bun script/qa/dependency-audit-capture.ts --phase baseline|post --binary <bin> --out <dir> [--case <name>]
bun script/qa/dependency-audit-compare.ts --baseline <dir> --post <dir> --gate p0|p1
bun script/qa/desktop/linux.ts --all --json [--provision]   # Linux host only
SENPI_BIN=<path> bun script/qa/desktop/macos.ts --all --json # granted, unlocked Aqua session
```

Dependency audit needs Bun 1.4.2, Node >=24, hyperfine; acceptance runs belong on the designated remote test host. Capture exit 0 means recording finished, not that probes passed; compare is the gate.

## CONVENTIONS

- Dependency-audit cases isolate HOME/USERPROFILE/XDG/agent dirs, allowlist process env, use fake credentials, bind loopback ephemeral ports, and remove the sandbox in `finally`; desktop drivers also tear down in `finally`.
- Desktop drivers emit JSON lines with `--json`; scenarios are host-specific.
- Evidence goes to the gitignored `.omo/evidence/` tree, never beside these sources.

## ANTI-PATTERNS

- Never use tmux for live terminal color capture (`web-terminal-visual-qa.mjs`); node-pty + xterm.js only.
- A macOS child agent must never inherit the agent package, session, or auth directory (`desktop/macos/agent.ts`).
- Refusal/stale/restored paths in `omp-item6-*` must not grant, dispatch, spawn, create tasks, or pass denied host calls; keep those assertions intact.
- PowerShell: never name a variable `$windows` in `desktop/windows/pointer-probe.ps1` (case-insensitive collision with `-Windows`).
- Do not run dependency-audit acceptance on the orchestration host.
