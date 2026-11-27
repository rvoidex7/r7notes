---
title: Architecture
description: How r7crypto is structured and why
tags:
  - project
  - architecture
---

# Architecture

`r7crypto` is one crate with two layers: a **library** that knows nothing about UI, and an optional **GUI** on top of it. The rename is complete; only older release notes still use `entropy-forge`.

```
r7crypto
├── entropy/     EntropySource trait + implementations
├── crypto/      StreamCipher, KeyedStreamCipher, AvalancheAnalyzer
├── quality/     bit reader, metrics, NIST SP 800-22 tests
├── bench/       source throughput / latency (operation benchmarks planned)
├── time.rs      shared monotonic clock (Instant native, performance.now() on wasm)
├── learn/       step-by-step teaching models (no UI)
├── viz/         egui app + job abstraction  (feature = "gui")
├── web.rs       WASM bootstrap (Trunk entry point)
├── index.html   canvas page for the web build
├── Trunk.toml   web build configuration
├── examples/    runnable API examples
├── scripts/     local gate and browser smoke
└── tests/       integration and headless UI tests
```

`viz` sits behind `feature = "gui"` (on by default). The library builds and tests without a display server.

## Why the layers exist

Five things stay separate on purpose: pluggable sources, quality verification, source-speed measurement, visual feedback and step-by-step teaching. A new source should immediately feed the metrics, tests and benchmark without touching the UI; a teaching view should not smuggle in a new dependency. The header selector and the `Learn` tab are different kinds of code for that reason.

## The one abstraction

Everything hangs off a single trait:

```rust
pub trait EntropySource {
    fn fill_bytes(&mut self, dest: &mut [u8]);
    fn name(&self) -> &str;
}
```

`next_byte`, `next_u32`, `next_u64` and `reset` have default implementations, so a source only has to fill a buffer. That is what makes the app extensible: a new generator is a small struct and immediately works with the quality metrics, the NIST tests and the source-speed benchmark.

Seven sources ship today:

| Source | Kind | What it demonstrates |
|---|---|---|
| `SystemEntropy` | OS CSPRNG (`getrandom`) | The reference: should pass everything |
| `ChaCha20Source` | Seeded CSPRNG | Deterministic yet statistically sound |
| `LcgSource` (glibc) | Linear congruential | Uniform-looking, structurally broken |
| `LcgSource` (RANDU) | Linear congruential | The 15-planes-in-3D classic |
| `Lfsr16Source` | Linear feedback shift register | Linear, 65,535-byte period |
| `BiasedSource` | Independent biased bits | Adjustable `P(1)`; long samples fail frequency, small ones can pass |
| `CounterSource` | `0,1,2,…,255` | Perfect histogram, zero predictability |

`MockEntropy` also exists as a deterministic helper for tests and doc examples.

### The RNG Arena

The header selector is driven by `SourceKind`, a small `Copy` enum with one variant per source. Because it carries no generator state, it can be handed to a job, where `kind.build(seed, bias)` constructs the actual `EntropySource`. Changing the source or the bias rebuilds the live source, clears stale Test/Benchmark results, and cancels any running job, so a result is never attributed to the wrong generator.

**Scope correction:** the selector drives **Test and Benchmark only**. `Use` uses an explicit key/nonce, `Learn` uses fixed examples, and `Avalanche` uses a deterministic SHA-256 input. Earlier notes said "all tabs use the selected source"; that was wrong and is fixed here and in the UI.

## Data flow

```
EntropySource::fill_bytes
        │
        ├─► QualityMetrics     (Shannon, min-entropy, Markov-1, χ²)
        ├─► NistTests          (6 SP 800-22 families + χ² → 9 P-values)
        └─► PerformanceBench   (SOURCE speed only today)

KeyedStreamCipher (ChaCha20, explicit key + nonce) ─► Use tab
AvalancheAnalyzer (deterministic SHA-256 input)   ─► Avalanche tab
```

Operation benchmarks (ChaCha20 / SHA-256 / NIST suite over the same data) are a planned addition to `bench/`.

Heavy paths go through the job abstraction in `viz/job.rs`: a worker thread on native, bounded per-frame polling on wasm. `std::thread::spawn` compiles for wasm but panics at runtime, so the web path never uses it. Both paths call the same analysis code, so NIST values and quality metrics match; benchmark timings are wall-clock measurements and are not comparable between native and web.

## Using the library in code

The crate is a library first; these are the same paths the GUI calls.

```rust
use r7crypto::entropy::SystemEntropy;
use r7crypto::crypto::KeyedStreamCipher;

let key = KeyedStreamCipher::derive_key("correct horse battery staple");
let nonce = [0x24u8; 12];

let plaintext = "Merhaba dünya! 🔐".as_bytes();
let ciphertext = KeyedStreamCipher::process_chacha20(&key, &nonce, plaintext);
let decrypted = KeyedStreamCipher::process_chacha20(&key, &nonce, &ciphertext);
assert_eq!(decrypted, plaintext);
```

The educational one-time pad remains available as `StreamCipher::new(SystemEntropy::new())`. It draws a fresh keystream per call and is **not** a substitute for ChaCha20.

A new source only implements the trait:

```rust
use r7crypto::entropy::EntropySource;

struct MyRng {
    state: u64,
}

impl EntropySource for MyRng {
    fn fill_bytes(&mut self, dest: &mut [u8]) {
        for byte in dest.iter_mut() {
            self.state = self.state.wrapping_mul(1103515245).wrapping_add(12345);
            *byte = (self.state >> 24) as u8;
        }
    }

    fn name(&self) -> &str {
        "My custom RNG"
    }
}
```

Quality and NIST checks work on any source:

```rust
use r7crypto::entropy::SystemEntropy;
use r7crypto::quality::{NistTests, QualityMetrics};

let mut source = SystemEntropy::new();
let metrics = QualityMetrics::analyze(&mut source, 100_000);
println!("Shannon: {:.4}, score: {:.1}", metrics.shannon_entropy, metrics.overall_score());

let mut data = vec![0u8; 100_000];
source.fill_bytes(&mut data);
for (name, p_value) in NistTests::run_all_tests(&data) {
    println!("{}: {:.4} {}", name, p_value, if p_value >= 0.01 { "✓" } else { "✗" });
}
```

Run the shipped versions with `cargo run --example basic_usage`, `custom_source` and `quality_check`.

## Design decisions

### MSB-first bit reader

All tests consume bits through `quality::bits::bit_at_msb`. Before that existed, some modules read LSB-first and others MSB-first, which silently produced different results for the same input. One reader, one convention, zero allocation. Avalanche cell semantics follow the same rule.

### Two ciphers, on purpose

`StreamCipher` is an educational one-time pad that pulls a fresh keystream per call — it cannot decrypt what it encrypted, and that is the point. `KeyedStreamCipher` is real ChaCha20 with a stored key and nonce. Keeping both makes the difference between "XOR with random bytes" and "a cipher" concrete. See [[Cryptography/ChaCha20|ChaCha20]].

### Score cap

`QualityMetrics::overall_score()` caps at 30 when structural red flags appear (near-zero Markov min-entropy, suspiciously uniform χ²). Without the cap a counter scores 100/100. The reasoning is in [[Cryptography/Entropy|Entropy]].

### Rendering rule

The Avalanche heatmap uses `TextureOptions::NEAREST` (or `Painter::rect_filled`) so neighbouring bits stay distinct. Default LINEAR filtering blurs them and makes the picture misleading — and a misleading picture defeats the whole purpose.

### Errors

- Library side: infallible APIs. `fill_bytes` cannot fail.
- OS RNG failure: panic with `expect`. If the OS cannot give entropy, continuing silently would be worse.
- GUI: `Option<String>` messages rendered inline. No error enum, no `anyhow`.

### Why `crate-type = ["cdylib", "rlib"]`

The `cdylib` is the wasm module; the deferred Android `.so` would use it too. The `rlib` is for the library and tests.

## Testing strategy

- NIST golden vectors in `tests/crypto_nist_golden.rs` — the standard's own examples, not our own output.
- Adversarial cases: counter, repeating `0xAA`, short inputs.
- UTF-8 roundtrip through ChaCha20.
- Headless 360×800 UI render in `tests/mobile_ui_test.rs` via `egui_kittest`.
- **Every new feature gets its own test file** covering input, button, output format, error state and determinism. Human UX check is mandatory after UI changes.

## Related

- [[Projects/r7crypto/Dependencies|Dependencies]]
- [[Cryptography/NIST-Tests|NIST Tests]]
