# senpi-desktop-engine

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The standalone desktop engine binary: JSON-RPC 2.0 over NDJSON on stdio (default) or a local socket, plus `--selftest` and `--schema` locator/drift gates. A leaf binary - every module is private, there is no library surface.

## STRUCTURE

| Path | Role |
|------|------|
| `src/main.rs` | CLI dispatch. `--xpc-agent` is matched before Clap (it must own the main thread), then the #10207 launch gate runs before EVERY mode. |
| `src/engine.rs` | `Engine`: engine-wide state shared by every connection - the one desktop session, supervisor + (possibly fake) clock, stop paths, resume token, notification outbox, request permits. `start` / `hello` / `capabilities` / `begin` / `shutdown`. |
| `src/connection.rs` | Connection fanout (largest file, ~640 lines): the multi-client hot path. |
| `src/rpc.rs`, `src/route.rs`, `src/settle.rs` | Method dispatch, request routing, settle mapping. |
| `src/serve.rs`, `src/daemon.rs`, `src/peer.rs` (unix) | Local-socket serve mode and daemon lifecycle. |
| `src/client.rs` | `exchange_or_start`: client mode that attaches to a running engine or starts one. |
| `src/oneshot.rs` | One-shot mode: `desktop.`-prefixed calls translated and run once. |
| `src/mcp.rs` | MCP `Server` / `Catalog`. |
| `src/selftest.rs` | Selftest mode; `--schema` prints the frozen schema for drift checks. |
| `src/stop_path.rs` | `StopPaths`: engine-side stop-path wiring. |
| `src/outbox.rs`, `src/trace.rs`, `src/config.rs`, `src/cli.rs`, `src/fake_listener.rs` | Notification outbox, tracing, config, CLI parsing, fake stop listener for tests. |
| `src/xpc.rs`, `src/stop_latch.rs`, `src/sessions/` (macOS) | XPC agent transport, stop latch, session bookkeeping; `xpc_test_harness` under test. |

## CONVENTIONS

- Platform-only modules are `cfg`-gated (`peer` on unix; `xpc` / `stop_latch` / `sessions` on macOS); tests are colocated.
- The #10207 launch gate runs before every mode, the private overlay helper included - never add an entry path around it.
- `--xpc-agent` is the only transport using message authentication instead of inherited responsibility; it must own the main thread before Tokio or a session starts.

## COMMANDS

```bash
cargo test -p senpi-desktop-engine --locked
cargo clippy -p senpi-desktop-engine --all-targets --locked -- -D warnings
cargo build --release -p senpi-desktop-engine --locked
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
