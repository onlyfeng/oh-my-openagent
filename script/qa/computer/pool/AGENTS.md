# script/qa/computer/pool/ -- Live QA Pools

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

Hosted/owned live tiers for computer-use: turn hosted-probe facts into verdict ledgers, and orchestrate owned macOS guests and X.org uinput legs. Verdict policy lives HERE, never in probes.

## WHERE TO LOOK

| Module | Role |
|---|---|
| `windows-full.ts` | Ledger CLI: `--ledger <facts.jsonl>... --out <tsv> --host <label> [--measured-at <iso>]`, `--validate <tsv>`, `--plan <tsv> --guest on\|off [--out <plan.json>]`, `--compare <committed.tsv> <fresh.tsv>` (exit 1 naming each leg whose verdict moved). Writes `crates/omo-computer-backend-win32/probes/hosted-windows-live.tsv` |
| `windows-full-verdicts.ts` | The ONLY place an observation becomes `CONFIRMS` / `CONTRADICTS` / `UNMEASURED`. A finding needs its positive control in the same run; a failed read or an errored leg is never a negative |
| `windows-full-ledger.ts` | TSV ledger rows/serialization, `compareLedgers`, `decideTier`, `LEGS` |
| `live-set.ts` | Scenario-set protocol: `SCENARIO <name> route=<synthetic-user\|virtual-hid\|none> PASS\|FAIL\|BLOCKED` lines, then `SET <os> pass=<n> fail=<n> blocked=<n>`. Exit 1 on any failure or if no scenario ran at all; exit 3 when nothing passed, so a job never shows green while nothing ran |
| `macos-vm.ts` | Owned-guest pool: `--up <15\|26> --run-id <id>` clones the recorded base image into `omo-cu-macos<major>-<id>`, the guest self-registers as an ephemeral runner (`computer-live-macos`, `macos-<major>`), waits up to 120 s for running; `--down` always runs after the job and deletes the guest; `--sweep` deletes only guests this pool minted (`omo-cu-*`), keeps running/uncountable ones, never touches base images |
| `macos-vm-{plan,ops}.ts` / `macos-vm-up.ts` | VM planning and lifecycle; the `--up` hold process covers the clone-exits to run-start gap |
| `xorg-uinput-{run,ledger,probe}.ts`, `xorg-{session,dummy-conf}.ts`, `uinput-keyboard.ts` | X.org dummy-server + uinput virtual-keyboard legs and their ledger |
| `teardown-check.ts` | Teardown receipt enforcement |

## CONVENTIONS

- Teardown receipts are measured lines: `procs <n>` (`procs UNREAD` when unreadable), `guest DELETED <name>` / `guest NOT DELETED <name> (<reason>)`; anything but 0 / UNREAD / NOT-DELETED fails teardown.
- The macOS live tier runs on owned macOS guests (hosted pid-routed clicks measured unreachable on macOS 15/26, so `HOSTED_MACOS_LIVE=no`); nothing is rented.
- Physical stop-chord legs use the hypervisor's virtual keyboard (`route=virtual-hid`); a hosted runner's second-process keystroke is never the physical chord.
- Pool-host variables and the run sequence are documented in `README.md`.

## ANTI-PATTERNS

- Never decide a verdict in a probe or scenario: probes observe, ledgers decide.
- `BLOCKED reason=backend_not_built` is the honest starting state; do not paper over it.
