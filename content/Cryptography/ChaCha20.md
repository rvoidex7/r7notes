---
title: ChaCha20
description: ARX stream cipher — keys, nonces, keystreams and nonce reuse
tags:
  - cryptography
  - stream-cipher
---

# ChaCha20

ChaCha20 is a stream cipher: it expands a key and a nonce into a keystream, and the ciphertext is `plaintext XOR keystream`.

## Parameters (RFC 8439)

| Item | Size |
|---|---|
| Key | 256 bits |
| Nonce | 96 bits |
| Block counter | 32 bits |
| Block | 512 bits (16 words) |

The core is ARX — addition, rotation, XOR — repeated for 20 rounds (10 double rounds). No S-boxes and no table lookups, which makes it fast and naturally constant-time on general-purpose CPUs.

## Why nonce reuse is fatal

The keystream depends only on `(key, nonce)`. Encrypt two messages with the same pair and:

$$C_1 \oplus C_2 = (P_1 \oplus K) \oplus (P_2 \oplus K) = P_1 \oplus P_2$$

The key cancels out. An attacker who knows or guesses one plaintext recovers the other. This is the **two-time pad** failure, and it is why a nonce must never repeat under a key.

## Key derivation from a passphrase

ChaCha20 needs a 256-bit key, not a password. r7crypto derives one with a single SHA-256 pass over the passphrase. That is enough to make the demo work, but it is **not** a password KDF: it is fast, and fast is exactly what an offline attacker wants. A real system should use Argon2id, scrypt or PBKDF2 with a work factor.

> [!warning] Demo, not production
> The passphrase handling in r7crypto is pedagogical. Do not protect real data with it.

## Related

- [[Avalanche-Effect]] — why XOR alone does not diffuse bits
- [[Random-Number-Generators]] — where the key and nonce must come from
- [[Projects/r7crypto/Interface|Interface in r7crypto]]
