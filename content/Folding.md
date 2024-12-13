---
title: Notes on ZKP Optimization Folding, Aggregation, and Recursion Techniques
draft: false
tags:
  - math
  - cryptography
  - zero-knowledge proofs
---

Different types of methods:

+ __Folding__: Aggregates multiple witnesses into a single, relaxed one (relaxed R1CS) outside of the circuit. Produces only one proof with all the witnesses, and usually a proof for the folding. (e.g. [Nova](https://github.com/microsoft/Nova), Supernova, Hypernova, Sangria, [Protostar](https://eprint.iacr.org/2023/620), Protogalaxy...)
+ __Aggregation__: Aggregates several proofs into one proof. (e.g. [SnarkPack](https://eprint.iacr.org/2021/529), [aPlonK](https://eprint.iacr.org/2022/1352)...)
+ __Recursion__: Proof-Carrying Data (PCD) or [IVC](Recursive_Hash_Chain_Verification); Given a proof, a state, and a transformation, computes a new proof.

- Folding combines witnesses whereas aPlonK combines commitments.
- Folding better than aPlonK (whose verifier requires $\log n$ operation on $G_t$) or accumulation in general.
- Folding interesting when proving several instances of the same statement at the same time.
- If a verification function is too long or a proof is too large, one try to make a proof of a proof. For example, prove a STARK (with a non-constant proof and verification function) using Groth16 (3 group elements, verification function of 2 pairings).
