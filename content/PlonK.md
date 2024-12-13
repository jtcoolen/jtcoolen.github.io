---
title: PlonK SNARK
draft: false
tags:
  - math
  - cryptography
  - zero-knowledge proofs
---


TODO. See https://www.maya-zk.com/blog/plonk-overview.

See [arithmetization](Constraint-Systems).
See [KZG](KZG-Polynomial-Commitments) polynomial commitment.
See [Fiat-Shamir](FS).

Copy constraints (one has to assert equalities between the values from the circuit's wires to connect the arithmetic gates). We do so using a permutation argument, by building a permutation polynomial that can be verified.

Commit to selector polynomials with KZG (with blinding for ZK property), multiply blinding factors by polynomial vanishing at the points used for interpolating the selector coefficients, so that the equations check out.


Verify selector polynomials and copy constraints.

