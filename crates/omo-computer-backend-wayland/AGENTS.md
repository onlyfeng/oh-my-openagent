# omo-computer-backend-wayland

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

v3 Wayland backend twin - currently a stub plus a measurement suite: `src/lib.rs` is a two-line doc comment ("Wayland libei input, ScreenCast capture, compositor probes and the stop ladder"); path deps on `omo-computer-core`, `omo-computer-safety`, `omo-computer-backend-atspi`. The live implementation is still `crates/senpi-desktop-backend-wayland`. The crate's real content is `probes/` (spike SP-W), which earned this file as a distinct measurement-governance domain.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `probes/probe-sp-w.ts` | The SP-W probe (Bun, Linux host with Docker): measures the Wayland portal/compositor facts the backend will rely on, on GNOME 46, KDE Plasma 6, and sway. |
| `probes/wayland-portal.tsv` | The ledger: one row per compositor per question; one row per condition when a question is measured under several conditions. Stable citation keys `SPW-<compositor>-<question id>`. |
| `probes/containers/` | One Dockerfile per compositor (gnome / kde / sway) plus the in-container helpers: session start/stop, portal client (`portal.py`), AT-SPI client, libei client (`ei-probe.c`), KWin window-management client; `run-plan.ts`/`host-runner.ts` drive them. |
| `probes/containers/common/verdicts.ts`, `questions*.ts` | The verdict state machine and per-surface question definitions; unit-tested via `--self-test`. |
| `knip.workspace.jsonc` | Directory-owned Knip config composed by root `knip.ts`; the probe entry is run by path. |

## CONVENTIONS

- Verdicts are exactly `CONFIRMS`, `CONTRADICTS`, `UNMEASURED`, or `INVALID`, and always answer the question in column 3 - never a design expectation.
- Every reader failure (crashed/timed-out client, missing key, failed call, empty listing) is `UNMEASURED`, never a negative. An absent portal interface is established ONLY by a successful introspection result omitting it; a failed D-Bus call is `UNMEASURED`, not "absent".
- If a requested row is missing when a container exits, write `UNMEASURED` with the exit code; never preserve the previous ledger value.
- Each question runs a fresh session and a fresh `$HOME`; portal token files are mode 0600 and opened `O_NOFOLLOW`.
- Later constants that depend on a ledger row cite it: `cites crates/omo-computer-backend-wayland/probes/wayland-portal.tsv:<row id>`.

## ANTI-PATTERNS

- Restore-leg measurements must NOT run the first-grant dialog approver; token-only restoration is the condition being measured.
- Spikes ship no engine code - never import from or add engine source to `probes/`.

## COMMANDS

```bash
bun crates/omo-computer-backend-wayland/probes/probe-sp-w.ts --compositor sway [--question release-on-close]
bun crates/omo-computer-backend-wayland/probes/probe-sp-w.ts --write crates/omo-computer-backend-wayland/probes/wayland-portal.tsv --compositor kde
bun crates/omo-computer-backend-wayland/probes/probe-sp-w.ts --verify crates/omo-computer-backend-wayland/probes/wayland-portal.tsv
bun crates/omo-computer-backend-wayland/probes/probe-sp-w.ts --self-test        # in bun run test:computer
python3 -m unittest crates/omo-computer-backend-wayland/probes/containers/common/test_iface_probe.py
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
