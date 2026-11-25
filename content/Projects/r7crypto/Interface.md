---
title: Interface
description: What the tabs of R7 Crypto Lab do today, and what is being reworked
tags:
  - project
  - ui
---

# Interface

Five tabs, one header. Every screen is built from the same primitives: a heading, a short explanation, sections separated by a rule, and inline help where a number needs context.

> The Avalanche tab and the Benchmark section are mid-rework. This page describes what is on screen now and flags the known gaps; [[Projects/r7crypto/Roadmap|Roadmap]] has the target state.

## Header

The header carries the app identity on the left and the RNG Arena controls to its right, on the same row. When the window is too narrow the controls move below the brand, and the bias slider gets its own row.

- **RNG selector** — a combo box labelled `RNG` listing all seven sources (`System`, `ChaCha20`, `LCG`, `RANDU`, `LFSR-16`, `Biased coin`, `Counter`). **It drives the Test and Benchmark tabs only.** `Use` works from an explicit key/nonce, `Learn` uses fixed pedagogical examples, and `Avalanche` uses a deterministic SHA-256 input. The UI says this so nobody assumes the source affects every tab.
- **`P(1)` slider** — appears only for the biased source.
- **Hover text** — one sentence about the selected source and where it fails.

Switching the source (or moving the bias slider) rebuilds the generator and discards any Test/Benchmark result produced by the previous one, so numbers on screen always belong to the source named beside them.

## Use

Real ChaCha20 encryption and decryption. Currently one mode; planned to become a mode selector.

- **Passphrase** → SHA-256 → 256-bit key
- **Nonce** — 96 bits as 24 hex characters, with 🎲 *New* pulling a fresh one from the OS
- **Input** — plaintext or ciphertext
- **Encrypt / Decrypt** — exact inverses for the same passphrase and nonce
- **Ciphertext as hex** — switches between hex and lossy binary display
- **Use output as input** — round-trips the result back through the other direction
- **Learn how this works** — jumps to the XOR visualizer

The nonce hint is deliberate: reusing a nonce under one key leaks `P₁ ⊕ P₂`. The Two-Time Pad experiment will make that concrete. The SHA-256 passphrase step is pedagogical, **not** a password KDF (Argon2id would be), and says so. See [[Cryptography/ChaCha20|ChaCha20]].

## Test

Quality analysis of a live sample from the selected entropy source. This is the strongest part of the app.

- The selected source is named above the controls (`Analyzing: …`)
- Sample size slider, 1,000 to 1,000,000 bytes
- **Metrics** — Shannon entropy, MCV min-entropy, Markov-1 conditional min-entropy, lag-1 correlation, χ²
- **Overall score** — capped at 30 when the data is structurally fake
- **NIST section** — per-test P-values with pass/fail at α = 0.01

The score cap is the part worth understanding: a counter has 8.0 bits of Shannon entropy and still scores at most 30. See [[Cryptography/Entropy|Entropy]] and [[Cryptography/NIST-Tests|NIST Tests]].

## Benchmark

**Now:** throughput and latency for the selected source — MB/s, microseconds per byte, total bytes, wall time. It is source speed, not cryptography speed, and one measurement per click (numbers jitter). The source is named above the controls (`Benchmarking: …`).

**Target:** two clearly labelled sections —

1. **Source speed** — same as today, explicitly "this measures entropy generation only".
2. **Operation speed** — ChaCha20 MB/s, SHA-256 MB/s, NIST suite duration, run against the same data. This is what connects Benchmark back to Use and Test.

`benchmark_avg` already exists in the code and will replace the single measurement. See [[Projects/r7crypto/Roadmap|Roadmap]].

## Avalanche

**Now:** a `🔀 Avalanche` tab with a message field, an input-bit slider, a 16×16 grid of the 256 SHA-256 output bits (red = flipped, grey = unchanged), hover details, and mean/min/max diffusion stats.

**Target:** the grid becomes a heatmap of flip *frequency* over many inputs/bit positions, drawn with `TextureOptions::NEAREST` (no emoji, no blurred neighbours). A weak XOR checksum is shown beside SHA-256 as contrast (~1 bit vs ~128). Cell semantics are printed on screen: output bit index, MSB-first, 8 bits = 1 byte, 16×16 = 256 bits. A rerun is the same deterministic experiment, not a new random trial. See [[Cryptography/Avalanche-Effect|Avalanche Effect]].

## Learn

Three step-by-step models, each with its own visualizer:

| Mode | Shows |
|---|---|
| XOR cipher | Byte-by-byte keystream XOR, with auto-play and speed control |
| Shannon entropy | How byte frequency turns into bits/byte, and why 8.0 is not "secure" |
| NIST frequency | How the monobit test turns a count into a P-value |

The Learn tab is allowed to repeat material from these notes: it has to teach on its own, without sending the reader somewhere else. Later it will gain a short post-quantum reading view ("what breaks, what holds") — no emulation, no new crate.

## UX principles

- No fixed minimum widths. Every control sizes itself from `available_width()`.
- A headless test renders all tabs at 360×800 and fails if any widget crosses the right edge.
- Heavy work runs on a worker thread today; on web it must become incremental per-frame work (`std::thread::spawn` panics on wasm).
- Errors appear inline next to the action that caused them, never as a modal.
- Every new widget gets a test file: input, click, output format, error state, determinism. Human UX check is mandatory after any UI change.

## Related

- [[Projects/r7crypto/Roadmap|Roadmap]] — what is still missing from these screens
- [[Projects/r7crypto/Architecture|Architecture]]
