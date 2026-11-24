---
title: NIST Tests
description: What SP 800-22 and SP 800-90B actually measure
tags:
  - cryptography
  - statistics
---

# NIST Tests

Two different documents, often confused:

| Document | Question it answers |
|---|---|
| [SP 800-22 Rev. 1a](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-22r1a.pdf) | Does this bit sequence *look* random under a battery of statistical tests? |
| [SP 800-90B](https://csrc.nist.gov/publications/detail/sp/800-90b/final) | How much entropy does this *source* actually contain? |

Passing 800-22 does not prove a generator is secure. A counter can be engineered to pass several tests; a good CSPRNG will occasionally fail one by chance. Statistical testing is a filter, not a proof.

## SP 800-22 tests used in r7crypto

| Test | Section | Catches |
|---|---|---|
| Frequency (monobit) | 2.1 | Global bias toward 0 or 1 |
| Block frequency | 2.2 | Local bias inside blocks |
| Runs | 2.3 | Too many or too few transitions |
| Longest run of ones | 2.4 | Clustering / periodicity |
| Cumulative sums | 2.13 | Drift |
| Serial | 2.11 | Pattern dependence in m-bit words |

## Reading a P-value

A P-value is the probability of seeing a result at least this extreme **if the sequence were random**. With significance level α = 0.01:

- `P ≥ 0.01` → no evidence against randomness
- `P < 0.01` → fail
- `P ≈ 1.0` → suspiciously perfect, not "very random"

That last case matters: a deterministic counter produces `χ² = 0` and therefore `P = 1.0`. An implementation that treats "high P" as "better" gets the counter exactly backwards.

## Golden vectors

The official examples in the standard are the only way to know an implementation is right:

| Test | Input | Expected P-value |
|---|---|---|
| Frequency | 100-bit π sequence | 0.109599 |
| Runs | 100-bit π sequence | 0.500798 |
| Longest run | 128-bit sequence | 0.180609 |

If an implementation disagrees with these, the implementation is wrong — not the standard.

## Related

- [[Entropy]]
- [[Random-Number-Generators]]
- [[Projects/r7crypto/Architecture|Architecture in r7crypto]]
