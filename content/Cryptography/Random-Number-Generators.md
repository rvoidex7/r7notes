---
title: Random Number Generators
description: TRNG, PRNG, CSPRNG and what the operating system actually provides
tags:
  - cryptography
  - rng
---

# Random Number Generators

## Three words that are not synonyms

| Kind | Source of unpredictability | Typical use |
|---|---|---|
| **TRNG** | Physical process (thermal noise, ring oscillators) | Seeding |
| **PRNG** | Algorithm + seed | Simulations, games |
| **CSPRNG** | Algorithm + secret seed, designed to resist prediction | Keys, nonces, tokens |

A PRNG can be perfectly uniform and still useless for crypto: if the internal state is recoverable from the output, the next output is known.

## The operating system CSPRNG

Every mainstream OS exposes one:

| OS | Interface |
|---|---|
| Linux | `getrandom(2)`, `/dev/urandom` |
| Windows | `BCryptGenRandom` / `ProcessPrng` |
| macOS / BSD | `getrandom(2)` / `arc4random_buf` |

In Rust, the [`getrandom`](https://docs.rs/getrandom) crate is the thin, audited wrapper over these. It hands you bytes with no algorithm choice to get wrong — which is exactly the property you want in a cryptographic application.

## Weak generators, on purpose

These exist to be studied, not used:

- **LCG** — `x' = (a·x + c) mod m`. Cheap, uniform-looking, structurally broken: consecutive outputs fall on lattices, which becomes visible in 2D.
- **RANDU** — a notorious LCG whose 3D output lies on 15 planes.
- **LFSR** — shift register with XOR feedback. Short period, linear, predictable from a small output window.
- **Biased coin** — independent bytes with `P(1) = p ≠ 0.5`. Fails anything that checks balance, yet can look "random enough" by eye.
- **Counter** — `0,1,2,...`. The trap described in [[Entropy]].

## Seeded CSPRNG

A CSPRNG with a fixed seed is deterministic: same seed, same stream. That is not a weakness — it is what makes reproducible tests possible. The weakness is reusing the seed, or deriving it from something guessable.

## Try it in r7crypto

- [RNG Arena + Test](https://rvoidex7.github.io/r7crypto/#/test) — opens quality analysis with the source selector in the header.
- [Benchmark](https://rvoidex7.github.io/r7crypto/#/benchmark) — opens source throughput and latency.
- Implementation: [`src/entropy/kind.rs#L38-L100`](https://github.com/rvoidex7/r7crypto/blob/main/src/entropy/kind.rs#L38-L100) and [`src/bench/performance.rs#L48-L117`](https://github.com/rvoidex7/r7crypto/blob/main/src/bench/performance.rs#L48-L117).

## Related

- [[Entropy]]
- [[ChaCha20]]
- [[NIST-Tests]]
