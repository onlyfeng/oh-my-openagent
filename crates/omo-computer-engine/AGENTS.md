# omo-computer-engine

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The v3 engine binary: NDJSON JSON-RPC on stdio today, with daemon / launcher / MCP-shim modes arriving together with the parity rows that build them. A leaf binary - every module is private; it has `knip.workspace.jsonc` composed by root `knip.ts`.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/cli.rs` | `Mode` + `parse`: `--stdio` (default) / `--schema` / `--selftest [--json]` / `--version`. Any other flag or combination exits 2 with `USAGE`; daemon flags are refused until their rows land. |
| `src/route.rs` | Backend selection from the launch environment. `OMO_COMPUTER_BACKEND=fake` selects the fake over `OMO_COMPUTER_FAKE_SCENARIO`; real OS backends register here in wave 3. |
| `src/rpc.rs`, `src/stdio.rs` | Method dispatch to the session; NDJSON framing - whitespace-only lines are skipped, invalid UTF-8/JSON gets a parse-error reply and the loop continues. |
| `src/selftest.rs`, `src/schema.rs` | Fake-backend session drive through the RPC layer; committed-schema print. |
| `probes/` | Spike ledgers SP-B (bunshin hosting gaps, `bunshin-hosting.tsv`) and SP-R (relay limits, `relay-limits.tsv`); `probes/README.md` is the authoritative verdict/citation rulebook. |
| `tests/` | `fixtures/v1-consumers/` scripted stdio sessions (10), `secure_audit_engine_process.rs`, `selftest_lock_path.rs`. |

## CONVENTIONS

- Seams are declared env-name consts: `BACKEND_SEAM` (`OMO_COMPUTER_BACKEND`), `RUNTIME_DIR_SEAM` (`OMO_COMPUTER_RUNTIME_DIR`). A fake-only seam set with a real backend is refused (`FakeOnlySeamWithRealBackend`); an unregistered value is `NoBackend`.
- The fake's recording sink is `fake-sink-<pid>.log` under the runtime dir (`SINK_FILE_PREFIX`) - one file per engine process, so a restarted engine never truncates its predecessor's lines.
- Probe ledgers: a condition this host cannot measure is `UNMEASURED`, never a negative; `--verify` compares verdict columns only, exits 1 on mismatch, and never writes; every later constant derived from a row cites it as `cites <ledger>:<row id>`; spikes ship no engine code.

## ANTI-PATTERNS

- Never accept an undocumented mode or flag combination - the CLI exits 2 rather than guessing; extend `Mode` and `USAGE` together.
- Never run SP-B's source checks against a live or remote agent; they use a fresh temporary HOME and local surfaces only. SP-R uses its local relay harness and locally minted credentials - no account, hosted relay, or model.

## COMMANDS

```bash
cargo build -p omo-computer-engine --locked
cargo test -p omo-computer-engine --locked
bun crates/omo-computer-engine/probes/probe-sp-b.ts --verify crates/omo-computer-engine/probes/bunshin-hosting.tsv
bun crates/omo-computer-engine/probes/probe-sp-r.ts --self-test   # pure rules + pair-helper self-test
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
