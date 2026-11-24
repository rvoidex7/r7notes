---
title: Roadmap
description: Current priority order, shipped work, and where contributions fit
tags:
  - project
  - roadmap
---

# Roadmap

No dates. Order matters more than timing. The full internal version lives in the repo's gitignored `docs/05`; this page is the public view. The current platform and test state is in [[Projects/r7crypto/index|Status]].

## Shipped — 0.4.0

- **Rename:** repo, crate and binary are now `r7crypto` / **R7 Crypto Lab**. Earlier release notes keep the old name; nothing after this point uses it.
- **Web/WASM is live** at <https://rvoidex7.github.io/r7crypto/> from the same commit as the native build. The job path behind both targets is in [[Projects/r7crypto/Architecture|Architecture]].
- The CI gate, local check script and release artifacts are described in [[Projects/r7crypto/Build-and-Release|Build and Release]].

## Shipped — 0.2.0

- ChaCha20 encrypt/decrypt with SHA-256 passphrase derivation
- Six NIST SP 800-22 tests with the standard's golden vectors
- SP 800-90B MCV min-entropy, Markov-1 min-entropy, lag-1 correlation
- SHA-256 avalanche analyzer
- Score cap that catches the uniform-but-predictable counter
- `Learn` tab with three step-by-step visualizers

## Shipped — 0.3.0

- **RNG Arena** — header source selector plus six new generators: seeded ChaCha20, LCG (glibc and RANDU presets), 16-bit LFSR, biased coin with an adjustable `P(1)`, and a deterministic counter. Same tests, different sources. See [[Cryptography/Random-Number-Generators|Random Number Generators]].
- Real application icon in the window, header and README; Windows builds embed it in the executable and no longer open a console window
- Linux archives ship a `.desktop` entry and a hicolor icon
- Automatic Linux and Windows binaries attached to each release tag — first exercised in 0.3.0
- Dependency cleanup and `statrs` 0.19

## In progress — unreleased

1. **Avalanche screen — rework.** A `🔀 Avalanche` tab exists with a 16×16 grid, but that is only one example. The real version shows a **distribution** over many inputs/bit positions, contrasts SHA-256 (~50%) with a weak XOR checksum (~1 bit), draws a `NEAREST`-filtered heatmap (no emoji, no blurred neighbours), and explains cell semantics (output bit index, MSB-first, 8 bits = 1 byte). Runs are deterministic; re-clicking repeats the same experiment. See [[Cryptography/Avalanche-Effect|Avalanche Effect]].
2. **Benchmark — rework.** Today it measures only the selected source's speed. That is the RNG, not ChaCha20/SHA-256/NIST. The UI will separate **source speed** from **operation speed** and label what each number does and does not mean.
3. **Two-Time Pad experiment** — encrypt two messages with one keystream and show `C₁ ⊕ C₂ = P₁ ⊕ P₂`. See [[Cryptography/ChaCha20|ChaCha20]].
4. **In-app links to these notes** — reader → note → live experiment → PR. The web URL is already live; the links inside the app are not.

## Next

- **Multi-mode `Use` tab** — HMAC, HKDF, Ed25519, Argon2 after Two-Time Pad. Each mode gets golden vectors and a headless UI test. Crates are added only when the mode is written.
- **2D entropy map** — 65,536 bytes rendered as a 256×256 `NEAREST` texture, plus a frequency histogram. LCG lattices become visible here. `egui_plot` is added with this module, not before.
- **Mobile layout** — bottom tab bar below 480 px, larger touch targets, tap-to-open info boxes instead of hover, copy/paste/sample buttons. The 360 px overflow check already exists; this is about layout, not detection.

## Web — shipped

- `wasm32-unknown-unknown` build with `trunk`, published to GitHub Pages at <https://rvoidex7.github.io/r7crypto/>; `dist/` is generated and not committed.
- `std::thread::spawn` is gone from the job path: native threads, wasm incremental frames.
- In the browser, the **System** source means the Web Crypto API (`crypto.getRandomValues`), not the OS CSPRNG — see [[Projects/r7crypto/Dependencies|Dependencies]].
- A headless Chromium smoke test checks console errors, uncaught exceptions, canvas/WebGL and the frame loop; its own detector is verified with `--self-test`.

Web came before the Play Store because a reader should reach a working experiment in one click. Installing an app breaks that path.

## Later — Android / Play Store

Reference material is kept in the repo's gitignored `docs/03`. NDK + Gradle + Play signing is real work but deliberately behind the one-click web path. The engineering goal is one commit feeding desktop, web and Play Store builds.

## Later still — system and post-quantum modules

- **Blockchain as a system layer:** a hash-chain / Merkle-tree experiment in the same repo. Blockchain is *not* a subfield of cryptology — it consumes hashes and signatures. Product code stays in its own repo; these notes would link both. See [NIST IR 8202](https://csrc.nist.gov/pubs/ir/8202/final).
- **Post-quantum reading module:** a Learn view explaining what breaks (RSA/ECC via Shor) and what holds (AES/SHA; Grover halves bits), hybrid X25519 + ML-KEM, and the NIST 2030/2035 migration timeline. No quantum emulation and no new crate — see [[Projects/r7crypto/Dependencies|Dependencies]]. Sources: [NIST PQC](https://csrc.nist.gov/Projects/post-quantum-cryptography/), [IR 8547](https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf), [Cloudflare's hybrid TLS post](https://blog.cloudflare.com/post-quantum-for-all/).

## Deferred

| Target | Why |
|---|---|
| Android `.aab` | Not scheduled; NDK + Gradle + Play signing with no current demand |
| macOS `.app` | Unsigned only; Gatekeeper warning on every first launch |
| linux-aarch64 | CI-only work, not scheduled |
| AppImage / Flatpak | Packaging maintenance; the tarball is enough for now |

## Known gaps

| Gap | Difficulty |
|---|---|
| Avalanche shows one sample, not a distribution | Medium — analyzer + visual rework |
| Benchmark measures source speed only | Small — two labelled sections, use `benchmark_avg` |
| No `egui_plot` histogram | Small — one dependency, one widget, added with the entropy map |
| wasm host-only API guard fails on `Instant::now` in the benchmark and job poll | Small — clock fix is deliberately held until approved |
| No mobile tab bar | Medium — layout only; 360 px overflow is already covered |
| No in-app link to r7notes | Small-to-medium — the "Try" benefit depends on it |

Anything marked *Small* is a reasonable first contribution: it touches one module and has an existing test to extend.

## Testing rule for every item

Each new feature gets its own test file: input, button, output format, error state, determinism. Run `./scripts/check.sh` locally; CI is authoritative — the commands and what they cover are in [[Projects/r7crypto/Build-and-Release|Build and Release]]. Human UX check is mandatory after any UI change — I do not let the assistant approve its own visuals.

## Related

- [[Projects/r7crypto/Interface|Interface]] — what exists today
- [[Cryptography/index|Cryptography]] — background for each item
