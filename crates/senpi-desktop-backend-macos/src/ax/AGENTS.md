# ax - AXUIElement accessibility bridge

**Generated:** 2026-11-18 / c2691818a9

## OVERVIEW

The macOS `AxBackend` implementation (32 files, ~4.6k LOC): handles wrap `CFRetained<AXUIElementRef>` in core's session-thread `AxHandle::Native`; refs and generations stay in core's `AxRegistry` - this tree only bridges native calls.

## WHERE TO LOOK

| Module | Role |
|------|------|
| `mod.rs` | `MacAx` (holds a `crate::overlay::Overlay` for target highlighting; `show_target` no-ops when the overlay is unavailable). Declares the module split and the visibility rules below. |
| `element.rs`, `retained.rs`, `props.rs`, `tree.rs` | Element handle resolution, `CFRetained` ownership, attribute props, tree walking. `element` is `pub(crate)`. |
| `batch_attrs.rs` (~360 LOC) | Batched native attribute reads - the AX hot path. |
| `actions.rs`, `text_input.rs`, `point_owner.rs` | AX actions; `insert_into_element`; `ensure_points_owned`. |
| `foreground.rs` | `make_frontmost`, `prepare_foreground_input` (both `pub(crate)`). |
| `menus/`, `popup/`, `date/` | Menu trees, popups, date pickers (submodule trees). |
| `*_tests.rs`, `tests.rs`, `live_tests.rs` | Colocated unit tests; live fixtures run under the parent crate's ignored-exact-named convention. |

## CONVENTIONS

- Visibility is graded: only `is_trusted` is `pub` (re-exported at the crate root); `make_frontmost`, `prepare_foreground_input`, `ensure_points_owned`, `insert_into_element`, `menus`, `element` are `pub(crate)`; everything else is private to this tree. Keep new surface at the lowest grade that works.
- Never send Accessibility IPC to the front app for capture, and never restore focus over an app the user switched to after the action (crate-level focus rules that `foreground.rs` implements).

## ANTI-PATTERNS

- Secure text fields (role or subrole `AXSecureTextField`, the rule `text_input.rs` applies) never report their value - `batch_attrs.rs` maps them to `Read::Unknown`; do not add a read path.
- Do not cache AX handles across generations: ownership lives in `AxRegistry`; a stale `AxHandle` must resolve through it, not through a retained pointer kept here.

## COMMANDS

```bash
cargo test -p senpi-desktop-backend-macos --locked          # on macOS
cargo clippy -p senpi-desktop-backend-macos --all-targets --locked -- -D warnings
```

Parent: [`crates/senpi-desktop-backend-macos/AGENTS.md`](../../AGENTS.md).
