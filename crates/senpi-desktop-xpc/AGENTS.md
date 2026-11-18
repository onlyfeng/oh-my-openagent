# senpi-desktop-xpc

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

macOS peer-identity foundation and the opt-in O2 XPC transport - the workspace's most security-sensitive crate (identity verification, peer admission, capability/sequence framing). Eligibility is NOT a user operation grant. Its engine consumer is `crates/senpi-desktop-engine/src/xpc.rs`.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/lib.rs` | Public surface: `AgentTransport`, `Incoming`, `forward_frames`, `TransportError`, `RejectReason` / `SecurityOperation` / `VerifiedPeer`, `pub mod identity`, and `pub mod roles` (the only NOT-macos-gated module). Everything else is private + `#[cfg(target_os = "macos")]`. |
| `src/policy.rs`, `src/security.rs`, `src/identity.rs` | Peer policy, the sealed Security.framework verifier, audit-token identity. Resolved policy inputs and verified-peer constructors are PRIVATE; production callers obtain peers only through the sealed verifier. |
| `src/agent.rs`, `src/connection.rs`, `src/stream.rs`, `src/listener.rs` | Agent transport, connection lifecycle, frame forwarding, the macOS 13+ opt-in listener. |
| `src/capability.rs`, `src/peers.rs`, `src/native.rs`, `src/dictionary.rs`, `src/signing_info.rs`, `src/audit.rs` | Capability/sequence framing, peer admission, native XPC ownership, dictionary codec, signing info, audit. |
| `TRANSPORT.md` | The wire contract: handshake, framing, bounds, approval - the authoritative spec. |
| `scripts/`, `examples/`, `tests/` | Signing/fixture scripts; usage examples; macOS tests incl. the real-child SIGKILL regression. |

## CONVENTIONS

- `--xpc-agent <mach-service>` must be the FIRST flag; it installs no job, is selected by no existing client, and cannot combine with another engine mode. Legacy launches still pass the responsibility gate. The only eligible requester is O1's native-controller role; Electron main stays ineligible.
- Handshake: `version: uint64(1)` + fresh CSPRNG `challenge: data(32)`; reply must echo the challenge and carry `epoch: data("xpc-1")` + `endpoint`. Bounds: `max_request_bytes` 1 MiB, `max_response_bytes` 128 MiB (sixteen inline capture budgets after base64 + headroom), `max_approval_frames` 64.
- Approval binds up to 64 SHA-256 hashes of the EXACT upcoming NDJSON frames plus `ttl_ms` in 1..60000, as an authenticated controller assertion; O3 supplies the native UI.

## ANTI-PATTERNS

- NEVER treat a proposal-source `approved` boolean as trusted approval.
- Any invalid frame, identity refusal, stale capability, wrong sequence, expiration, or interruption CLOSES the live connection - there is no reconnect/replay path.
- Only the test configuration may inject identity; production peer constructors and policy inputs stay sealed/private.

## COMMANDS

```bash
cargo test -p senpi-desktop-xpc --locked        # on macOS
cargo clippy -p senpi-desktop-xpc --all-targets --locked -- -D warnings
```

Parent: [`crates/AGENTS.md`](../AGENTS.md).
