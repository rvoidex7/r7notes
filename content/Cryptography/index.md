---
title: Cryptography
description: Working notes on the cryptographic concepts behind my projects
tags:
  - cryptography
  - index
---

# Cryptography

Working notes on the cryptographic ideas I use in my projects. These are **not** a reference manual — they are the mental model I build on, written down so I can check my own understanding later. Wherever depth matters, the note links to the canonical source instead of restating it.

> [!warning] Working notes
> These pages are deliberately incomplete and will be revised. Do not use them as a security reference.

## Topics

- [[Entropy]] — Shannon vs min-entropy, and why "looks random" is not enough
- [[Random-Number-Generators]] — TRNG / PRNG / CSPRNG, and what the OS actually gives you
- [[ChaCha20]] — stream ciphers, keys, nonces, and nonce reuse
- [[Avalanche-Effect]] — the strict avalanche criterion and the XOR misconception
- [[NIST-Tests]] — what SP 800-22 and SP 800-90B actually measure

## Where these are used

- [[Projects/r7crypto/index|r7crypto]] — the project notes. The notes are the knowledge layer; the app is the experiment layer.
- [Live app](https://rvoidex7.github.io/r7crypto/) — try these concepts in the browser without installing anything. Each topic below links to its exact screen and source range.

## Post-quantum reading list (later module, no code yet)

Quantum computers are not needed in my hands for this to matter — the industry is doing *crypto-agility* now. "Harvest now, decrypt later" is the motivation: traffic captured today can be decrypted when a large quantum computer exists.

- **What breaks:** RSA/ECC (Shor). **What holds:** AES/SHA — Grover roughly halves the security bits, so AES-256 → ~128 and SHA-256 stays fine.
- **Standards:** [NIST PQC project](https://csrc.nist.gov/Projects/post-quantum-cryptography/) — FIPS 203 (ML-KEM/Kyber), 204 (ML-DSA/Dilithium), 205 (SLH-DSA/SPHINCS+); [HQC selected as backup KEM in 2025](https://www.nist.gov/news-events/news/2025/03/nist-selects-hqc-fifth-algorithm-post-quantum-encryption).
- **Timeline:** [NIST IR 8547](https://nvlpubs.nist.gov/nistpubs/ir/2026/NIST.IR.8547.pdf) — quantum-vulnerable algorithms deprecated after 2030, disallowed after 2035.
- **What developers actually ship:** [Cloudflare's hybrid X25519 + Kyber TLS post](https://blog.cloudflare.com/post-quantum-for-all/) — hybrid means *the old algorithm stays alongside the new one* in case the new math turns out weak. It is an engineering/distribution problem, not "stronger encryption".
- **Blockchain is a different topic:** it consumes hashes and signatures but is not a subfield of cryptology. See [NIST IR 8202](https://csrc.nist.gov/pubs/ir/8202/final).

No quantum emulation, no new crate. When the time comes this becomes a short Learn view in [[Projects/r7crypto/Interface|r7crypto]] — "what breaks, what holds, what developers ship today".
