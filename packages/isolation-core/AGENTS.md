# isolation-core

Harness-neutral filesystem isolation PAL with patch and branch merge-back.
Task-runner integration is supplied by callers.

Provenance: ported from oh-my-pi `crates/pi-iso` + `task/isolation-*`, MIT.
Candidate ordering, unavailable-only fallback and retention follow that upstream.
The owner protocol adds process-instance identity, daemon session ownership and
atomic publication. No Rust/napi crate or `.node` sidecar is required.
ProjFS is excluded: its driver-callback provider API requires a native callback
host, not a Bun-callable clone operation.

## Contract

- Backends write only inside the supplied context base directory. `start` targets
  its `m` child; `stop` must be safe after a partially failed start.
- `IsolationUnavailableError` permits fallback. Other errors propagate. A failed
  teardown preserves the tree rather than recursively removing an active mount.
- `chooseBaseDir(repoRoot, homeDir, id)` selects a same-device location (mkdir/remove probe; optional I/O seam for deterministic device tests); a cross-device home fallback excludes tree-cloning backends.
- `ensureIsolation` claims a `.creating-<pid>` sibling before starting, then publishes via the backend's optional `relocate(fromBase, toBase)` hook (or rename for non-mounted backends); retention uses the same hook. Published/in-flight directories are never overwritten - reclaim via stale sweeping.
- Ownership is checked before age. Foreign, retained, live and unknown owners
  are never automatically deleted. Creating/malformed markers get a ten-minute
  grace period. A live creating owner survives regardless of age.
- Process identity reuses memory-core's lazily loaded native functions without
  its subprocess fallback. Node returns null and uses PID-only liveness.
- `sweepStaleIsolations(roots, { backends, probe })` stops the recorded backend
  before removal. Without a matching backend implementation it reports a skip;
  without a daemon probe it keeps host-session children as unknown.
- `retainIsolation(handle, reason)` transfers the tree to a retained path and
  invalidates the old handle paths. Manual removal requires stopping its recorded
  backend at `<retainedPath>/m` before recursively removing the retained directory.

- `mergeIsolatedChanges` writes root/nested patches and an isolation summary before
  merging. `apply: false` retains artifacts without mutating the parent. Callers
  own teardown; on artifact-write failure keep the handle and call
  `retainIsolation(handle, reason)` rather than cleaning it up.
- Patch apply is atomic per repository and never automatically uses `--3way`.
  Root failure leaves nested repositories untouched; nested failure after root
  success is reported as partial. Nested changes are committed separately.
- Branch replay preserves child commits on clean baselines and filters inherited
  WIP on dirty baselines. Unresolvable replay throws `IsolationCommitReplayError`
  naming the commit and retained branch (the facade reports branch-merge-failed).
  It does not fall back to committing the user's overlapping WIP.
- Both modes serialize through memory-core's identity-bearing lock in Git's
  common directory, including linked-worktree parents. Branch merges stash WIP
  and restore with `--index`; failed pops preserve the stash and report warnings.
- Low-level `applyDeltaPatch` takes `{ id, artifactsDir }`; `commitToBranch`
  returns `{ branchName, baseSha, nestedPatches }` for `mergeTaskBranch`.

## Verification

Run focused files with `bun test packages/isolation-core/src/<name>.test.ts`.
The package suite is `bun test packages/isolation-core`; typecheck uses
`tsgo --noEmit -p packages/isolation-core/tsconfig.json`.
Tests use fake native/CLI boundaries against actual directories, plus injected
device/liveness probes. `started_at` is an ISO timestamp recorded after successful
start; old `{ backend }` markers remain readable by the sweeper. ZFS markers also
record dataset/snapshot identity, and overlay markers retain the lower path.

The Linux filesystem workflow creates real loopback btrfs and (when the kernel module loads) ZFS, exercising publication, COW and teardown; required check `Isolation Linux filesystems / linux-fs` for isolation-core and isolated-child PRs (branch protection is not managed here - the PR must request it). Missing filesystem env vars produce explicit local skip reasons; the workflow also mutation-tests EOPNOTSUPP rejection on the runner's ext4 /tmp. Reflink's Node-hosted fallback uses argv-only `cp -a --reflink=always`, never a silent plain copy; FFI walks and rcopy plain copies account bytes during traversal, reserve 10% headroom, and default to a 2 GiB ceiling (no separate prewalk).

## ReFS manual verification (D2)

Hosted Windows runners do not provide ReFS; the default is native-contract tests only - never describe those as a live ReFS proof. On a disposable, provisioned ReFS volume (e.g. `R:`), from the repo root in PowerShell:

```powershell
fsutil fsinfo volumeinfo R:
$env:ISOLATION_TEST_REFS_ROOT = 'R:\isolation-tests'
New-Item -ItemType Directory -Force $env:ISOLATION_TEST_REFS_ROOT
bun test packages/isolation-core/src/backends/fs-integration.test.ts -t ReFS
Remove-Item Env:ISOLATION_TEST_REFS_ROOT
```

Covers publication, aligned extents, unaligned tails, tiny/empty files, source immutability, teardown (long-path prefixes; cluster-rounded EOF allocation; clone the aligned prefix, copy the tail; original logical size kept); no volume is formatted. Record the output with the Windows build + volume cluster size for a live validation.
