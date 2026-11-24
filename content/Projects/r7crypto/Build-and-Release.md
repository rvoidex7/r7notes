---
title: Build and Release
description: Local build, CI workflows, and how a release is cut
tags:
  - project
  - rust
  - ci
---

# Build and Release

This page is the engineering view: how to build, check, package and publish. The public starting points are the [repository README](https://github.com/rvoidex7/r7crypto#readme) and the [web app](https://rvoidex7.github.io/r7crypto/).

## Requirements

- Rust stable (edition 2021)
- Linux: X11/Wayland and OpenGL development packages (`libxkbcommon`, `libx11`, `libwayland`, `libGL`) for the GUI build
- Web build: `trunk` (`cargo install trunk`) and the `wasm32-unknown-unknown` target
- Browser smoke test: Node 22+ and a Chromium binary (`CHROME_BIN` if it is not in a standard location)

The library half builds without any GUI dependency:

```bash
cargo build --no-default-features
```

## Feature layout

| Feature | Effect |
|---|---|
| `gui` (default) | `egui` + `eframe` + `image`, and the library's UI layer |
| `desktop` (default) | The native binary target only |
| none | Library only: entropy, crypto, quality, bench, learn |

`desktop` exists so Trunk can build the web library with `--no-default-features --features gui`. Otherwise the binary and the `cdylib` both write `r7crypto.wasm` and the web build fails with "more than one target artifact".

## Local commands

```bash
cargo run --release                 # launch the native app
./scripts/check.sh                  # the full local gate
trunk serve                         # web dev server
trunk build --release               # static site in dist/ (gitignored)
```

`./scripts/check.sh` is the one-command local mirror of CI: formatting, Clippy, the test suite, the feature matrix, wasm type-checks, a headless Chromium smoke test of the built web app plus its `--self-test`, and a wasm-only Clippy rule. That last rule rejects host-only APIs such as `std::time::Instant::now`: they compile for `wasm32` but panic in the browser, so native tests alone cannot catch them. CI is authoritative; the script is a convenience, not a replacement.

## CI

| Workflow | Trigger | Does |
|---|---|---|
| `ci.yml` | push, PR, manual | Linux: fmt, Clippy, tests, feature matrix, wasm type-checks, browser smoke. Windows: `cargo check --all-targets --all-features --locked` |
| `ci.yml` → `deploy` | `main`, after `test`, `lint`, `wasm`, `browser-smoke`, `windows` | Builds with Trunk and deploys to Pages (`dist/` is never committed) |
| `audit.yml` | weekly + manual | `cargo audit`; files an issue per advisory |
| `release-notes.yml` | tag `v*`, manual, release | Creates/updates the GitHub release body from `CHANGELOG.md` |
| `build-assets.yml` | tag `v*`, manual | Builds Linux and Windows artifacts and attaches them |

Pages must be configured with **Source: GitHub Actions**; publishing from a branch is not used. If any gating job fails, the deploy does not run and Pages keeps serving the last successful version.

The Windows check is not decoration: Cargo's `unused_dependencies` lint fired there and not on Linux, because Linux pulls `thiserror` in transitively through Wayland. The same lint is denied in `Cargo.toml`; it is Cargo's own lint, not part of Clippy's `-D warnings`.

## Release process

1. Move the `[Unreleased]` entries in `CHANGELOG.md` into a new `## [x.y.z]` section.
2. Commit.
3. Tag locally: `git tag vx.y.z`. **Pushing and tagging are my call** — the assistant commits locally but does not push, tag, release or manage labels.
4. The tag triggers both workflows: release notes come from the CHANGELOG section, binaries are built and attached.

Release notes are **always** derived from the CHANGELOG version section, never from commit messages. Every `v*` tag produces a release with Linux and Windows binaries attached.

## Artifacts

| Platform | Contents |
|---|---|
| Linux | `tar.gz` with the binary, `r7crypto.desktop`, a 256×256 hicolor icon, and `LICENSE` |
| Windows | `zip` with `r7crypto.exe` and `LICENSE`; the icon is embedded in the `.exe` |

Release builds do not open a console window on Windows. Linux declares the app id (`with_app_id(...)`) so Wayland panels can match the window to the installed `.desktop` entry.

To install a downloaded Linux archive:

```bash
tar -xzf r7crypto-<version>-linux-x86_64.tar.gz
cd r7crypto-<version>-linux-x86_64
install -Dm755 r7crypto ~/.local/bin/r7crypto
install -Dm644 r7crypto.desktop \
  ~/.local/share/applications/r7crypto.desktop
install -Dm644 icons/hicolor/256x256/apps/r7crypto.png \
  ~/.local/share/icons/hicolor/256x256/apps/r7crypto.png
```

`~/.local/bin` must be in `PATH` for the menu entry to launch the app. On Windows, unzip and run `r7crypto.exe`; no Rust toolchain is needed for either platform.

## Web

- Build: `trunk build --release`; served from `https://rvoidex7.github.io/r7crypto/`
- The job path is shared with native; see [[Projects/r7crypto/Architecture|Architecture]]
- The browser smoke test catches wasm-bindgen init, WebGL and frame-loop boot failures that a type-check cannot see
- The wasm-only host-only-API lint is part of the gate; its current status is tracked in [[Projects/r7crypto/Roadmap|Roadmap]]

## Not built

macOS, linux-aarch64 and the Android/Play pipeline are not part of the current release flow. Reasons are in [[Projects/r7crypto/Roadmap|Roadmap]]; Android reference notes are kept in the repo's deferred `docs/03`.

## Related

- [[Projects/r7crypto/Dependencies|Dependencies]]
- [[Projects/r7crypto/Roadmap|Roadmap]]
