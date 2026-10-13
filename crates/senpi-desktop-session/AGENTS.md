# senpi-desktop-session - Single-Thread Desktop Session (Rust)

**Generated:** 2026-10-13 / dc373d26f

Earned its file at score 11 (8-15 band): distinct domain: request orchestration, audit, and screenshot budgets over the desktop backend.

## OVERVIEW

Rust `rlib` workspace crate. One `senpi-desktop-session` thread owns the backend, the AX ref registry, and the capture frame cache, and serves requests in submission order. Each request runs under `catch_unwind`; its waiter gives up at the injected `SessionTimeouts` deadline. Depends on `senpi-desktop-safety` for the gate and supervisor types.

## STRUCTURE

```
src/
├── lib.rs        # pub use re-exports only (SessionSafety, Op/Response, Session, BackendFactory, timeouts)
├── session.rs    # owning thread + request loop: Pending, Session, THREAD_NAME (224 LOC)
├── request.rs    # Op / Response wire types
├── mutate.rs     # THE mutation choke point, largest non-test file (277 LOC; + permission/stop tests)
├── grant.rs      # ControlSlot: process-wide foreground control grant, never stolen
├── selection.rs  # BackendFactory, BackendSelection, SelectionError
├── timeouts.rs   # SessionTimeouts, CLOSE_TIMEOUT, OPERATION_TIMEOUT
├── audit/        # audit writer + GC (`#[path]` tests: writer_test.rs, gc_test.rs)
├── budget/       # screenshot budget plan/encode
├── restore.rs pointer.rs  # focus/cursor restoration, pointer state
└── worker*.rs    # worker + worker_{ax,capture,clipboard,input,menus,observe,zoom}
```

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add a session operation | `src/request.rs` (`Op`/`Response`) + `src/session.rs` dispatch |
| Change mutation policy | `src/mutate.rs` (`SessionSafety`: gate, restore transaction, one audit event) |
| Audit trail / typed-text handling | `src/audit/` (writer, GC) |
| Screenshot budget rules | `src/budget/plan.rs` |
| Backend selection | `src/selection.rs` |

## CONVENTIONS

- Every mutating request passes one choke point (`mutate`): safety gate → focus/cursor restoration transaction → exactly one audit event.
- Audit records hash typed text (SHA-256 prefix) and store its length, never the text itself.
- Screenshot GC is best-effort and only removes stale files matching `senpi-computer-*.png`; unrelated files are preserved and GC IO failures never fail the caller.
- A request whose waiter deadline expired must not reach the desktop.
- Audit and GC failures go to stderr and never fail the action; another session's control grant is never stolen (callers retry).

## QA

```bash
cargo test -p senpi-desktop-session
```

Parent: repo-root `AGENTS.md` (no `crates/AGENTS.md` exists).
