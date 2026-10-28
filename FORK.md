# featherbitplatform/winit

This is a fork of [winit](https://github.com/rust-windowing/winit), maintained for [featherbit terminal](https://github.com/featherbitplatform/terminal), which uses winit through iced. It is not affiliated with or endorsed by the winit project; please report problems with these changes here, not upstream. All credit for winit goes to its authors.

featherbit terminal uses this fork as a Cargo patch pinned to a commit:

```toml
[patch.crates-io]
winit = { git = "https://github.com/featherbitplatform/winit", rev = "<full commit sha>" }
```

## Branches

- `master` follows upstream `master` (GitHub fork).
- `fbterm/v0.30.13` starts at upstream's `v0.30.13` release tag (commit `e9809ef54b18499bb4f2cac945719ecc2a61061b`, the version iced 0.14 uses) and adds the changes below as separate commits.

Files changed on a `fbterm/*` branch start with the line `// Modified by the featherbit authors for featherbitplatform/winit; see FORK.md for the list of changes.`

## Changes from winit 0.30.13

### 1. Windows: dragging a window onto a monitor with a different scale

`src/platform_impl/windows/event_loop.rs`, the `WM_DPICHANGED` handler.

When a window crosses onto a monitor with a different DPI while the user drags it, Windows sends `WM_DPICHANGED` with a suggested rect, computed to keep the window under the cursor at the new DPI ([WM_DPICHANGED](https://learn.microsoft.com/en-us/windows/win32/hidpi/wm-dpichanged)). winit 0.30.13 ignores that rect: it keeps the old logical size, keeps only the cursor's horizontal position, and, when the result still lies mostly on the old monitor, nudges it a pixel at a time towards the new one. With monitors of different scale, above all stacked vertically or with offset edges, the window then jumps away from the cursor, flips between the monitors on every mouse move, and can end up off-screen.

While the window is being dragged (`MARKER_IN_SIZE_MOVE`), the handler now applies the suggested rect as is:

```rust
// before
let new_outer_rect: RECT;
{
    let suggested_ul = ...
// after
let new_outer_rect: RECT;
if dragging_window {
    new_outer_rect = suggested_rect;
} else {
    let suggested_ul = ...
```

Outside a drag (a monitor's scale changed, a window moved by code) nothing changes. During a drag, a size the application sets through `ScaleFactorChanged`'s `InnerSizeWriter` is not applied; the window gets the size Windows suggests and reports it with the usual `Resized` event.

## Updating

To move to a newer winit release: fetch upstream tags (`git remote add upstream https://github.com/rust-windowing/winit.git`, `git fetch upstream tag <tag>`), branch `fbterm/<version>` from the tag, cherry-pick the commits of the current `fbterm/*` branch, check this file still describes them, and bump the `rev` in featherbit terminal.
