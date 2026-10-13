# side-panel component

Opt-in right-hand TUI column (`side_panel` in `omo.json`, default off; `--omo-side-panel` forces it on for one run, and only `omo.json` can switch it back off). Earned its own file: 88 TS files, 6 subdirs, ~4.8k source LOC (~12k with tests), own lazy bundle - a self-contained subsystem, not a small factory (score 13, distinct domain).

## STRUCTURE

| Path | Purpose |
|------|---------|
| `index.ts` | Registration shell only: enablement resolution, `/side-panel-diff` forwarder, one forwarder per host event. Never imports the runtime eagerly. |
| `runtime.ts` | `createSidePanelController`, the composition root (largest file, 511 LOC): session lifecycle, background refresh, epoch invalidation, git/task/memory/goal/usage reads, viewer actions. |
| `host-surface.ts` | The only file touching host renderer/layout/URL-callback seams: zero-height anchor widget, `hstack` wrap + root restore, URL-callback park/restore, widget-block fallback when the layout seam is absent. |
| `store.ts` / `rows.ts` / `body.ts` | Panel state (`createPanelStore`), row assembly (`buildPanelRows`), theme application + terminal-safe visible-width fitting (`createPanelBody`). |
| `types.ts` | Structural host/data contracts (29 symbols). Deliberately imports no `pi-tui` types. |
| `sections/` | Ten pure row builders (session, context, usage, goal, memory, agents, tools, location, files, layout) emitting `{text, color}`; never touch host, disk, or network. |
| `data/` | Read/parse/transform seams: goal store, delegated-child output + task records, memory durable state, host facts. |
| `usage/` | The only networked part: subscription poller (`poller.ts`), account/credential resolution (`accounts.ts`, ordered provider-id lists), machine-shared cache (`cache.ts`). Constructed only when `sections.usage` is on. |
| `git/` | `git status` / diff reading and repo discovery through the host `exec` seam. |
| `format/` | sanitize / bars / units / truncate helpers for terminal-safe output. |
| `popups/` | Shared overlay viewer: open, wheel ownership while open, viewport height, text popup. |
| `commands.ts` / `links.ts` / `guards.ts` / `constants.ts` / `context.ts` / `width.ts` | `/side-panel-diff` command, OSC 8 action links (`omo-panel:` scheme), `unknown` narrowing guards, bounds/floors/backoff constants, event-context facts, `resolvePanelWidth`. |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add a section | `sections/<name>.ts` + wiring in `rows.ts`; data seam in `data/` when a new source is needed |
| Change enablement | `index.ts` (explicit flag beats config in either direction) |
| Host coupling / layout fallback / click hook | `host-surface.ts` |
| Usage quota + cache | `usage/poller.ts`, `usage/cache.ts` (`$XDG_CACHE_HOME/omo-senpi/side-panel-usage.json`, mode 0600, atomic replace, sanitized on read) |
| Refresh bounds, caps, backoff | `constants.ts` |
| Child cards (suspended/residency/runner kind) | `data/task-records.ts` |

## CONVENTIONS

- Two-bundle split is the design: the shell stays tiny; none of `runtime.ts` (shipped as `extensions/omo-side-panel.js`) loads for sessions without the panel.
- Late async reads are discarded by epoch (incremented on teardown/session change); event handlers start tracked background reads and return immediately because the host awaits them.
- Refresh frequency is bounded by explicit floors in `constants.ts`; no recursive filesystem watchers, no per-event git/memory polling.
- TUI registry seams use `Symbol.for` so duplicate bundled copies interoperate.
- Host surfaces stay structural: panel-local interfaces plus runtime guards; never assume undocumented host members; downgrade to widget/dark surfaces when layout APIs are absent.

## ANTI-PATTERNS

- Don't eagerly load the runtime for sessions where the panel is disabled.
- Don't let a late read cross a session boundary; epoch discard is the only correct sink.
- Don't block host event handlers on refresh work.
- Don't evict running/queued child rows to satisfy the child cap; only terminal children are eligible, counted once.
- Don't couple row builders/tests to host UI, disk, network, or memory-core; degrade missing optional data to empty output instead of dropping other valid rows.
- Don't pass untrusted text into terminal formatting or viewers: sanitize, bound diff display, drop unsafe controls.
- Don't overwrite the host URL callback permanently; park the original under a registry symbol and restore it on dispose.
- Don't treat API-key or `key`/`token` credential shapes as valid for subscription usage endpoints; they need OAuth `access` (`usage/accounts.ts`).
- Don't let a failed poll erase displayed usage windows, and re-read the shared cache before merging so a sibling's results survive (`usage/cache.ts`, `usage/poller.ts`).
- Don't let a missing or partial `git status` imply a clean tree (omit the section), and don't exceed `UNTRACKED_DIRECTORY_FILE_CAP` (50) per-file processes for untracked directories (`git/`).
