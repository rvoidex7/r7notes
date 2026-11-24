---
title: Dependencies
description: Every crate in r7crypto and why it is there
tags:
  - project
  - rust
  - dependencies
---

# Dependencies

The rule I follow: **a crate is added when the feature that needs it is written**, not before. An unused dependency is not free — it costs build time, attack surface and upgrade noise.

Exact versions are kept in the repository, not here: [Cargo.toml](https://github.com/rvoidex7/r7crypto/blob/main/Cargo.toml) declares the ranges, [Cargo.lock](https://github.com/rvoidex7/r7crypto/blob/main/Cargo.lock) pins the resolved crates. For a reproducible snapshot, use the same files at the release tag.

Reading-only topics do not get a crate — post-quantum is a Learn view, not an implementation. There is no C++/FFI layer either: [RustCrypto](https://rustcrypto.org/) covers hash, KDF, password hashing, MAC, signing and encryption in pure Rust, and [rustls](https://raw.githubusercontent.com/rustls/rustls/main/README.md) is used in production. `ring` contains C/assembly from BoringSSL, so "FFI is unavoidable" is not a reason to leave Rust.

## Runtime

| Crate | Role | Why this one |
|---|---|---|
| [`getrandom`](https://docs.rs/getrandom) | OS CSPRNG access | Thin, audited wrapper. No algorithm to pick wrong. On web it becomes `crypto.getRandomValues` via the `js` feature. |
| [`sha2`](https://docs.rs/sha2) | SHA-256 | Passphrase → key (pedagogical only), the avalanche analyzer, and the arena's ChaCha20 seed derivation. |
| [`hex`](https://docs.rs/hex) | Hex encoding | Nonce and ciphertext display. |
| [`chacha20`](https://docs.rs/chacha20) | ChaCha20 | RustCrypto implementation of RFC 8439. Powers both `KeyedStreamCipher` and the seeded `ChaCha20Source` — no second RNG crate. |
| [`statrs`](https://docs.rs/statrs) | χ², normal, `erfc` | P-values for the NIST tests. |
| [`egui`](https://docs.rs/egui) | Immediate-mode GUI | Same author ecosystem as eframe; no retained tree. |
| [`eframe`](https://docs.rs/eframe) | Window + renderer | Glow backend, Wayland/X11 on Linux. |
| [`image`](https://docs.rs/image) | PNG decode | Turns the bundled icon into a window/header texture. |

## Development

| Crate | Role |
|---|---|
| [`egui_kittest`](https://docs.rs/egui_kittest) | Headless UI tests, including the 360×800 layout check |
| [`winresource`](https://docs.rs/winresource) | Build-time only; embeds `icon.ico` into the Windows executable |

## eframe features

`eframe` is used with `default-features = false`, so every feature is explicit:

```toml
features = ["glow", "accesskit", "clipboard", "default_fonts",
            "links", "system_fonts", "wayland", "x11"]
```

`accesskit` is what gives the app screen-reader support (Orca, NVDA, Narrator, VoiceOver). It pulls a D-Bus chain on Linux, which is the source of the advisory note below.

## Known advisory

`accesskit_unix` → `atspi` → `zbus 5.x` pulls crates that RUSTSEC marks **unmaintained, not vulnerable**. Each one has its own tracking issue, linked directly so it stays findable after the issue is closed: [paste (#24)](https://github.com/rvoidex7/r7crypto/issues/24), [`zbus_names` (#25)](https://github.com/rvoidex7/r7crypto/issues/25), [`zvariant` (#26)](https://github.com/rvoidex7/r7crypto/issues/26), [`zvariant_derive` (#27)](https://github.com/rvoidex7/r7crypto/issues/27). All are the latest versions; the fix arrives when upstream moves to `zbus` 6. Disabling `accesskit` would drop screen-reader support to silence a hygiene warning — not worth it.

## Related

- [[Projects/r7crypto/Architecture|Architecture]]
- [[Projects/r7crypto/Build-and-Release|Build and Release]]
