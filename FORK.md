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

### 1. Windows: backport of upstream's `WM_DPICHANGED` fix for Windows 11

A backport of upstream commit [`488c036`](https://github.com/rust-windowing/winit/commit/488c036a05d418e13bbdcdc349e2db2f6b7f58e2) ("win32: disable DPI re-adjustments on Windows 11", fixing [rust-windowing/winit#4041](https://github.com/rust-windowing/winit/issues/4041)), which is on upstream `master` but not in any 0.30.x release. The code is upstream's, adapted only to the 0.30.13 file layout and names.

When a window crosses onto a monitor with a different DPI, Windows sends `WM_DPICHANGED` with a suggested rect ([WM_DPICHANGED](https://learn.microsoft.com/en-us/windows/win32/hidpi/wm-dpichanged)). winit 0.30.13 always replaces it with its own calculation (keep the logical size, keep the cursor's horizontal position while dragging, nudge the window onto the new monitor a pixel at a time), a workaround needed on early Windows 10 builds. On Windows 11 it makes a dragged window jump away from the cursor, flip between monitors and possibly end up off-screen. With the backport the handler applies the suggested rect as is, except on Windows 10 builds before 22000, where the old workaround still runs.

- `src/platform_impl/windows/util.rs`: the `WIN10_BUILD_VERSION` detection (`RtlGetVersion`) moves here from `dark_mode.rs`.
- `src/platform_impl/windows/dark_mode.rs`: uses `util::WIN10_BUILD_VERSION`.
- `src/platform_impl/windows/event_loop.rs`: the `WM_DPICHANGED` handler uses `suggested_rect` unless `WIN10_BUILD_VERSION` is below 22000; the old calculation moves unchanged into `apply_win10_dpi_adjustment`.

When featherbit terminal moves to a winit release that contains this commit, this change, and with it the fork, can be dropped.

## Updating

To move to a newer winit release: fetch upstream tags (`git remote add upstream https://github.com/rust-windowing/winit.git`, `git fetch upstream tag <tag>`), branch `fbterm/<version>` from the tag, cherry-pick the commits of the current `fbterm/*` branch, check this file still describes them, and bump the `rev` in featherbit terminal.
