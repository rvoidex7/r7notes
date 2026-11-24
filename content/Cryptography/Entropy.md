---
title: Entropy
description: Shannon entropy vs min-entropy, and why the difference matters
tags:
  - cryptography
  - information-theory
---

# Entropy

"Entropy" is used for two different things in cryptography, and mixing them up is the most common mistake in randomness discussions.

## Shannon entropy

Measures the average information per symbol:

$$H_1(X) = -\sum_{x} p(x)\log_2 p(x)$$

For byte data the maximum is 8 bits/byte. It answers: *how surprising is the next symbol on average?*

## Min-entropy

Measures the worst case:

$$H_\infty(X) = -\log_2 \max_x p(x)$$

It answers: *how many guesses does an attacker need in the worst case?* This is the number a key generator actually cares about.

## Why they diverge

A repeating counter `0,1,2,...,255,0,1,...` has:

- Shannon entropy ≈ **8.0 bits/byte** — every value appears equally often
- MCV min-entropy ≈ **8 bits/byte** — the *marginal* histogram is uniform, so the most common value estimator sees nothing wrong
- Markov-1 conditional min-entropy ≈ **0 bits/byte** — every transition `a → a+1` has probability 1, so the next value is fully determined

The distribution is perfectly uniform; the *sequence* is perfectly predictable. Uniformity and unpredictability are not the same property.

> [!important] Takeaway
> A frequency histogram can look perfect while the generator is worthless. Any quality metric that only looks at symbol frequencies — including a plain MCV min-entropy — will pass the counter. What catches it is a metric that looks at *transitions*, like the Markov-1 conditional min-entropy below.

## Estimating min-entropy in practice

NIST SP 800-90B gives estimators for real sources. The one used in r7crypto is the **MCV (most common value)** estimator with a 99% confidence bound:

$$\hat{p}_{\max} = \frac{\max_x c_x}{L}, \quad
p_u = \min\!\left(1,\; \hat{p}_{\max} + 2.576\sqrt{\frac{\hat{p}_{\max}(1-\hat{p}_{\max})}{L-1}}\right), \quad
H_\infty = -\log_2 p_u$$

This is only valid for IID samples. For non-IID data the full 800-90B suite is needed.

A cheap complement is the **Markov-1 conditional min-entropy**, which catches periodic sequences:

$$H_{\infty,\text{Markov1}} = -\log_2 \max_{a,b} P(s_{i+1}=b \mid s_i=a)$$

For a counter, every transition has probability 1, so this collapses to 0.

## Related

- [[Random-Number-Generators]]
- [[NIST-Tests]]
- [[Projects/r7crypto/Architecture|Architecture in r7crypto]]
