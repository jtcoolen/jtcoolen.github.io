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

|              | FRI                  | [KZG](KZG-Polynomial-Commitments.md)   | IPA     | DARK   |
|--------------|----------------------|-------|---------|--------|
| Building Block             | Hash Function        | Elliptic Curve Pairings | Discrete Log Group | Unknown Order Group |
| Computational Hardness Assumption | Hash Functions| DLP, t-SDH, [t-BSDH (for multi-reveal)](Batched-KZG-Proofs#t-bilinear-strong-diffie-hellman-assumption) | DLP | DLP, q-HSM,  Low Order & Strong Root Assumptions  |
| Transparent (no trusted setup) | Yes | No | Yes | No (RSA) / Yes ([Ideal Class Groups](Ideal-class-groups)) |
| Succinct proofs | Yes | Yes| No | Yes |
| Post-quantum secure | Yes | No | No | No |
| Proof size   | $O(\log² d)$           | $O(1)$  | $O(\log d )$ | $O(\log d)$ |
| Proving time   | $O(d \log²d)$         | $O(d)$ | $O(d)$     | $O(d)$   |
| Verification time  | $O(\log² d)$           | $O(1)$  | $O(\log d)$ | $O(\log d)$ |
