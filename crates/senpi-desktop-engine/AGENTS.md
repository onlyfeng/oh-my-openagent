# senpi-desktop-engine - Desktop engine binary (JSON-RPC / MCP)

## OVERVIEW

Binary crate `senpi-desktop-engine`: serves the `senpi-desktop-core` frozen JSON-RPC protocol over stdio/local socket/MCP, with a safety stop-path latch; CLI modes stdio, serve, oneshot, mcp, resume, selftest, schema. Earned its file at score 12: distinct domain (the runnable engine over the contract crate), 35 Rust files / 4.5k LOC.

## STRUCTURE

```
src/
├── main.rs        # entry: CLI mode selection; all modules private (no lib API)
├── cli.rs config.rs engine.rs  # arg parsing, EngineConfig, Engine over senpi-desktop-session
├── serve.rs daemon.rs oneshot.rs client.rs outbox.rs rpc.rs selftest.rs settle.rs  # mode + transport modules
├── connection.rs  # 265 LOC, concurrent request/response transport
├── stop_path.rs   # 256 LOC, safety latch + listener wiring (src/stop_path/)
├── route.rs       # method dispatch against the core METHODS table
├── mcp/           # MCP server mode
tests/             # integration: stdio_headless.rs (307, largest file), headless/, common/, fixtures/
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| CLI modes | `src/main.rs` | stdio default, `--serve`, `--oneshot`, `--mcp`, `--resume`, `--selftest`, `--schema` |
| Transport | `src/connection.rs` | Concurrent request/response handling |
| Stop path | `src/stop_path.rs` + `src/stop_path/` | Safety latch policy from core `Effect`/`Exposure` |
| Dispatch | `src/route.rs` | Routes to backends, enforces method exposure |
| Integration tests | `tests/` | Shared helpers in `tests/common`, replay via fake backend |

## CONVENTIONS

- Binary with private modules; there is no library API - consumers talk JSON-RPC, not Rust imports.
- Unit tests colocate in `src/`; integration tests live under `tests/` and drive the binary over stdio.
- Conformance scenarios from `senpi-desktop-core` are replayed through the fake backend.

## ANTI-PATTERNS

- `src/serve.rs` local-socket serve path is a SKELETON (daemon-session/oneshot/resume = todo 47); treat it as implementation status, not a finished contract.
- Never bypass the core frozen method table or its `Effect`/`Exposure` policy when routing.

## COMMANDS

```bash
cargo test -p senpi-desktop-engine
cargo run -p senpi-desktop-engine                    # stdio (default)
cargo run -p senpi-desktop-engine -- --selftest      # also: --serve <endpoint> --oneshot --mcp --resume --schema
```
