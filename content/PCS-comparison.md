---
title: Comparing Polynomial Commitments
draft: false
tags:
  - math
  - cryptography
  - commitment scheme
  - polynomial commitment
---


For a degree-$d$ polynomial:

|              | [FRI](FRI)                  | [KZG](KZG-Polynomial-Commitments)   | [IPA](IPA)     | DARK   |
|--------------|----------------------|-------|---------|--------|
| Building Block             | Hash Function        | Elliptic Curve Pairings | Discrete Log Group | Unknown Order Group |
| Computational Hardness Assumption | Hash Functions| DLP, t-SDH, [t-BSDH (for multi-reveal)](Batched-KZG-Proofs#t-bilinear-strong-diffie-hellman-assumption) | DLP | DLP, q-HSM,  Low Order & Strong Root Assumptions  |
| Transparent (no trusted setup) | Yes | No | Yes | No (RSA) / Yes ([Ideal Class Groups](Ideal-class-groups)) |
| Succinct[^1] proofs | Yes | Yes| No | Yes |
| Post-quantum secure | Yes | No | No | No |
| Proof size   | $O(\log² d)$           | $O(1)$  | $O(\log d )$ | $O(\log d)$ |
| Proving time   | $O(d \log²d)$         | $O(d)$ | $O(d)$     | $O(d)$   |
| Verification time  | $O(\log² d)$           | $O(1)$  | $O(\log d)$ | $O(\log d)$ |

In practice, for post-quantum secure polynomial commitment schemes:
- degree ~1: see TCitH-GGM (Seed trees)
- degree ~10: degree-enforcing commitment (TCitH-MT)
- degree: 1,000: Merkle Tree with Ligero-like proximity tests
- degree 10,000: FRI-based commitments


[^1]: Succinct is not really well-defined: can either mean polylog or constant.
