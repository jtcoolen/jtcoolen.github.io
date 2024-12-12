---
title: Notes on ZKP Optimization Folding, Aggregation, and Accumulation Techniques
draft: false
tags:
  - math
  - cryptography
  - zero-knowledge proofs
---

Different types of methods:

+ __Folding__: Aggregates multiple witnesses into a single, relaxed one. Produces only one proof with all the witnesses. (e.g. Nova, Supernova, Hypernova, Sangria, Protostar, Protogalaxy...)
+ __Aggregation__: Aggregates several proofs into one proof. (e.g. [SnarkPack](https://eprint.iacr.org/2021/529), [aPlonK](https://eprint.iacr.org/2022/1352)...)
+ __Accumulation__: Proof-Carrying Data (PCD) or IVC; Given a proof, a state, and a transformation, computes a new proof.

- Folding combines witnesses whereas aPlonK combines commitments.
- Folding better than aPlonK or accumulation in general.
- If a verification function is too long or a proof is too large, one try to make a proof of a proof. For example, prove a STARK (with a non-constant proof and verification function) using Groth16 (3 group elements, verification function of 2 pairings).
