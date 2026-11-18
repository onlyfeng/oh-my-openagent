# script/qa/computer/ -- Computer-Use QA

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

QA for the computer-use engine (`crates/omo-computer-*`): consumer-replay parity, cross-platform GUI conformance, and hosted/owned live pools.

## WHERE TO LOOK

| Piece | Role |
|---|---|
| `replay.ts` | Replays corpus cases against an engine binary the way a consumer drives it: `--engine <bin> --case <file>... [--dir <dir>] [--assert-sink-empty] [--deltas <tsv>]`. Replies are deep-compared modulo the case's `variable` JSON pointers and `ORACLE-DELTAS.tsv` rows (default: `crates/omo-computer-core/parity/`); prints `REPLAY_OK <case> steps=<n>`. A reply mismatch exits 1; every input/setup error prints `REPLAY_ERROR <why>` and exits 2. A case without `transport` replays over stdio only when `ORACLE-PROVENANCE.tsv` lists it as an imported oracle case, else refused (`transport_missing`). A case's `supersedes` names must all exist (`supersedes_missing`, nothing replayed); superseded cases print `SKIPPED`. Each case runs in a fresh temp runtime dir (`OMO_COMPUTER_RUNTIME_DIR`, `${RUNTIME_DIR}` env substitution, `runtime.plant` may pre-seed files/dirs with modes), removed afterwards |
| `replay.test.ts` | Drives `replay.ts` as a subprocess; resolves the engine's Cargo target dir via `CARGO_TARGET_DIR`, Cargo metadata, or the repo `target/` |
| `probes/` | `hosted-windows-live.ps1 -Leg <facts\|disconnect\|lock\|elevated\|secure-desktop\|mixed-dpi> -Out <file>` emits ONE JSON observation per leg (C# helpers in `hosted-windows-live/`). The probe only observes: an exception is an error observation, never a verdict. Legs can lock/disconnect a session or install a display driver -- disposable hosted runner only, never a person's session |
| `sentinel/sentinel.ts` | Driver target used by `conformance/run.ts` (`engine` \| `legacy`) |

## SUBDIRS

- `conformance/` -- catalog-driven GUI conformance matrix (own AGENTS.md).
- `pool/` -- hosted/owned live tiers: verdict ledgers and VM orchestration (own AGENTS.md).

## CONVENTIONS

- Exit-code discipline: a reply mismatch is the ONLY exit-1 condition; every input/setup error (arguments, unreadable/malformed case or delta file, unspawnable engine) prints `REPLAY_ERROR <why>` and exits 2. Never conflate the two.
- `--assert-sink-empty` fails the run when the engine left per-process sink files (`fake-sink-*`) behind in the runtime dir.
- Replay waits are bounded (`MESSAGE_TIMEOUT_MS` 10 s per reply, `EXIT_TIMEOUT_MS` 5 s for engine exit) -- no unbounded polling.
- The corpus contract is cross-tree: cases plus `ORACLE-DELTAS.tsv` and `ORACLE-PROVENANCE.tsv` live in `crates/omo-computer-core/parity/`; changing engine reply shapes means updating those, not weakening the comparison.

## ANTI-PATTERNS

- Verdicts live in `pool/` ledgers, never in probes or replay: observations record, ledgers decide.
- Do not infer compatibility silently: unproven missing transport, missing superseded cases, and malformed inputs are refused before anything runs.
