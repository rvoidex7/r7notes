---
title: Avalanche Effect
description: The strict avalanche criterion, and the XOR misconception
tags:
  - cryptography
  - hash-functions
---

# Avalanche Effect

The **strict avalanche criterion** (Webster & Tavares, 1985) says: flipping one input bit must flip each output bit with probability 1/2.

For a 256-bit output, one input bit flip should change about 128 output bits. "About" is doing work there — the useful statement is statistical, not exact.

## The misconception

XOR-based stream ciphers do **not** have a plaintext avalanche effect. With a fixed keystream:

```
plaintext:  0101...
keystream:  1100...
ciphertext: 1001...

flip one plaintext bit  ->  exactly one ciphertext bit flips
```

That is not a bug in the cipher; it is what XOR does. It is called **malleability**, and it is why stream ciphers need authentication (Poly1305 in ChaCha20-Poly1305, or an AEAD construction) before they can be trusted with data an attacker can modify.

Avalanche belongs to **confusion layers** — S-boxes, hash compression functions, block cipher rounds — not to the XOR step itself.

## What to measure instead

- **SHA-256** — flip one input bit, count changed output bits over many trials. The mean should sit near 128 of 256.
- **ChaCha20** — flip one **key or nonce** bit and compare keystreams. Flipping a plaintext bit proves nothing.

## Related

- [[ChaCha20]]
- [[Projects/r7crypto/Architecture|Architecture in r7crypto]]
