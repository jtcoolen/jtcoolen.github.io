---
title: Castagnos-Laguillaumie (CL) lineary homomorphic encryption scheme

draft: false
tags:
  - math
  - cryptography
  - ideal class groups
  - linearly homomorphic encryption
---
# Ideal Class Group Homomorphic Encryption Faster than Pailler's

See [Ideal Class Groups](Ideal-class-groups).


- class groups have been used to construct additive homomorphic encryption that's similar to ElGamal encryption
- allows encryption of large 256-bit messages and efficient decryption by using composite order of group of unknown order with underlying subgroup of known order where the dlog is easy

 
### Castagnos-Laguillaumie (CL) lineary homomorphic encryption scheme


- M integer
- setup
- computations in $F$ are efficient!
- exponentiations in $F$
- discrete logarithm in $F$
```rust
impl<Z, BQF> ClHSMqInstance<Z, BQF>
where
    Z: crate::z::Z + Debug + Clone + std::cmp::PartialEq,
    BQF: BinaryQuadraticForm<Z> + Clone,
{
    pub fn dlog_solve_f(&self, fm: &BQF) -> Z {
        let mut m = Z::from(0u64);
        if !fm.equals(&fm.identity()) {
            m = fm.b().divide_exact(&self.q).invert_mod(&self.q).unwrap();
        }
        m
    }
}
```

