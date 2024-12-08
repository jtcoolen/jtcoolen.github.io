---
title: Distributed Key Generation
draft: false
tags:
  - math
  - cryptography
  - elliptic curves
  - ideal class groups
  - distributed key generation
  - threshold cryptography
  - threshold encryption
  - threshold signatures
---

# Purpose of a Distributed Key Generation

Distributed Key Generation allows to generate a set of keys in a distributed manner for use in threshold cryptographic schemes, such as threshold signatures, [threshold encryption](Threshold-Encryption), and blockchain consensus.

# Current approches

A popular DKG as of today is the Joint-Feldman DKG due to Pedersen.

+ Implementation: https://github.com/celo-org/celo-threshold-bls-rs/tree/master/crates/dkg-core.

+ Paper: https://link.springer.com/chapter/10.1007/3-540-46416-6_47.

It consists of a state machine with 4 phases

0. Each key holder acts as a dealer and sends a secret share to each party through a secret channel
1. Each key holder verifies the shares he received, and if the check fails for a certain party it broadcasts a complaint against that party
2. Each party challenged with a complaint reveals to the participants the shares it sent secretly to each complaining party
3. Each party checks the revealed shares, disqualifies dishonest parties, and derives a common public key and secret key shares of that public key using the contributions from the parties that have not been disqualified

Drawbacks:
- A node can verify whether it has received correct shares or not, but it cannot check if the other parties have received valid shares or not, for this it relies on honest parties to file complaints or confirm the correctness of their share, making the process interactive
- If a node doesn’t complain or confirm that it has received a correct share, it could be that either it is disconnected, has crashed, or will eventually confirm or complain in the future, so nodes are in doubt and waiting for potential complaints or confirmations, leading to deadlock-like situations (nodes can be stuck in a phase without a mechanism to break the cycle)
- Implementation-wise: the logic is complex due to the interactive nature of the protocol, making it error-prone and subject to edge cases (especially around timeouts)
- Deployment-wise: the current design relies on key holders to cooperate off-chain to run their DKG (with manual intervention on their part) with no way for users to verify the DKG ceremony

# Publicly-Verifiable and Non-Interactive Distributed Key Generation: why is that useful?

By contrast, a publicly verifiable, non-interactive DKG solves these issues and brings stronger and more appealing guarantees:
- _Public verifiability_: anyone (not only the participants to the DKG) can check that all exchanged shares are correct. Meaning that in a decentralized setup, users external to the subnet, can check that the subnet they are using is using valid and secure keys. This is especially important in a blockchain environment where trust is based on transparent and accessible evidence of its correctness and integrity. Each DKG contribution can be verified for validity by the smart-contract, allowing to publicly identify misbehaving keyholders even if more than a threshold (or possibly all) are corrupt.


- _Non-interactiveness_: the DKG ceremony happens without back-and-forth communication between the key holders, that is each key holder posts a single message to the L2 for anyone to see and check. A dedicated smart-contract orchestrates the DKG and its logic is simple: gather a single message from at least a threshold of key holders (after authenticating them), and key holders derive the keys using the first (>= threshold) valid messages so that they agree on the same transcript (ordered set of exchanged messages). The non-interactive nature of the latter resolves the issue of requiring keyholders to run the DKG at a specific time, allowing for a more robust setup against disconnections, while also reducing overall bandwidth consumption and network delays by getting rid of complex communication patterns.




# General framework

The non-interactive verifiable secret sharing scheme presented in the paper [Non-interactive VSS using Class Groups and Application to DKG](https://eprint.iacr.org/2023/451.pdf) follows the folklore technique where shares are encrypted by a single dealer for a set of recipients, accompanied by a non-interactive proof of correct sharing. This ensures that each recipient can independently verify the correctness of their share and the other shares without interaction.

To transform non-interactive Verifiable Secret Sharing (NI-VSS) into a non-interactive DKG, each participant acts as a dealer and independently runs a non-interactive VSS (NI-VSS) protocol with their own secret. Once all participants have completed this process, the final share of each party is computed as a linear combination of their own share and the shares received from others, by leveraging the homomorphic property of Shamir secret sharings.

Several notable variations of this approach include
- [Aggregatable Distributed Key Generation](https://eprint.iacr.org/2021/005.pdf), but outputs key shares consisting of group elements instead of scalar field elements, rendering it incompatible with many threshold schemes.
- [Groth](https://eprint.iacr.org/2021/339.pdf) & [Gentry](https://eprint.iacr.org/2021/1397.pdf) using range proofs (and chunking of shares for Groth). In contrast, when using class groups for the cryptographic setup, range proofs and chunking are unnecessary, because recovering the secret is fast enough with the class group based decryption. Additionally, Gentry’s construction uses lattice-based cryptography, making it post-quantum secure. However, this approach comes with trade-offs, including significantly higher bandwidth requirements and reduced performance compared to the other methods presented.

The non-interactive VSS using Class Groups paper also extends the notion of public verifiability to a stronger setting, where it holds even if all participants are corrupted (but then the secrecy of the secret sharing and non-termination are not guaranted). This concept, referred to as strong public verifiability, ensures robustness in adversarial settings.

All above protocols are based on [Shamir Secret Sharing (SSS)](Threshold-Encryption#setup).

# Protocol

## PKI setup with NIZK proof of exponent
Needed to send the encryption to the secret shares.

## Multi-receiver encryption (based on additively homomorphic encryption)
Needed to efficiently encrypt batch of secret shares forming a Shamir secret sharing to distinct receivers, and for receivers to efficiently decrypt their share of the secret.

See [Castagnos-Laguillaumie (CL) linearly homomorphic encryption scheme](Castagnos-Laguillaumie-LHE) the concrete instantiation of this scheme that's used in the protocol.

For the class group arithmetic we propose two backends: GMP and [rust crypto](https://github.com/RustCrypto/crypto-bigint). The latter supports fixed-width big integers with mostly constant-time operations: with a security level of 112 we can support up to 3 multiplications of 1348-bit (size of the discriminant) integers fitting inside 4096-bit ints


TODO evaluate https://github.com/hacl-star/hacl-star/blob/afromher_rs/dist/rs/bignum/bignum4096.rs

## NIZK proof of correct Shamir secret sharing

We're given a vector of encryptions of scalars $(s_1, \dots, s_n)$ supposedly forming Shamir secret sharing: $\vec{E}=\{ f^{s_i} h_i^r  \}_i$ along with a shared portion of the ciphertext $R=g_q^r$.

The NIZK proof is about the following statement:
1. There exists a degree $t$ polynomial $P(X)=\sum_{i=0}^{t} a_i X^i$ over $Z_q$ such that
    - For $i\in[n]$: $P(i)=s_i$ (Shamir secret sharing)
    - For all $j=0,\dots,t$: $A_i=g^{a_i}$ (commitment to the polynomial)
3. Encrypting $s_1,\dots,s_n$ with randomness $r$ using public keys $h_1,\dots,h_n$ yields the ciphertext $\vec{E}=\{ f^{s_i} h_i^r  \}_i$ and $R=g_q^r$.



### Proving

We assume that the discriminant of the class group fits in a 4096 bit integer, so that we can convert class group elements (in reduced form) to the same encoding regardless of the chosen instance of Z.
This is the purpose of the `convert_instance_to_bignum` function which converts the representation of a class group element in terms of `Z` elements to the structure `Bignum` defined as
```rust
use crypto_bigint::U4096;

/// Signed integer with range (-2^4096, 2^4096)
/// Does not check for overflows!
#[derive(Debug, Clone)]
pub struct Bignum {
    pub positive: bool,
    pub uint: U4096,
}
```
When serializing the instance to hash, we do not handle errors and instead crash as we treat it as an encoding bug

```rust
/// Generates a proof for the protocol based on the provided configuration, instance, and witness.
///
/// # Parameters
///
/// - `config`: The configuration for the protocol.
/// - `instance`: The instance containing public keys, ciphertexts, and other data.
/// - `witness`: The witness containing secret values and an integer `r`.
/// - `rng`: A random number generator.
///
/// # Returns
///
/// A `Proof` instance containing values used for verification.
#[cfg(feature = "random")]
pub(crate) fn prove<
    R: CryptoRng + RngCore,
    Z: z::Z + std::fmt::Debug + Clone + Serialize + PartialEq,
    E: EllipticCurve<S = S> + std::clone::Clone,
    S: crate::scalar::Scalar + Clone + Debug,
>(
    config: &Config<Z>,
    instance: &Instance<Z, E, S>,
    witness: &Witness<Z, S>,
    rng: &mut R,
) -> Proof<Z, E, S>
where
    Z: Randomizable,
{
    let alpha = S::random(rng);
    let rho = Z::sample_range(rng, &Z::zero(), &S::modulus_as_z());
    let w = config.generator_h.pow(&rho);

    let mut x = E::generator();
    x.mul_assign(&alpha);

    let instance_bn = convert_instance_to_bignum(instance);

    let gamma_s = S::hash(bincode::serialize(&instance_bn).unwrap());
    let gamma_z = S::to_z(&gamma_s);

    let points = compute_gamma_powers(config.n as usize, &gamma_z);

    let y = compute_y(config, instance, &points, &rho, &alpha);

    let gamma_prime_s = compute_gamma_prime_s(&gamma_z, &w, &x, &y);
    let gamma_prime_z = S::to_z(&gamma_prime_s);

    let z_r = witness.r.mul(&gamma_prime_z).add(&rho);
    let z_s = compute_z_s::<Z, S, E>(witness, &gamma_s, &gamma_prime_s, &alpha);

    Proof { w, x, y, z_r, z_s }
}
```

### Verifying


Note that we use the Pippenger algorithm for the multiexponentiations in the class group and elliptic curve. 

The class group inputs are reduced prior to being consumed in case they aren't. The element `f` is powered with the dedicated function `power_f` which doesn't use quadratic forms and is thus more efficient (todo check that exponent `z_r` is less than $M$)
```rust
/// Verifies the given proof against the provided configuration and instance.
///
/// # Parameters
///
/// - `config`: The configuration for the protocol.
/// - `instance`: The instance containing public keys, ciphertexts, and other data.
/// - `proof`: The proof to be verified.
///
/// # Returns
///
/// `true` if the proof is valid, `false` otherwise (with overwhelming probability).
pub(crate) fn verify<
    Z: crate::z::Z + std::fmt::Debug + Clone + Serialize + PartialEq,
    E: EllipticCurve<S = S> + std::clone::Clone,
    S: crate::scalar::Scalar + Clone + Debug,
>(
    config: &Config<Z>,
    instance: &Instance<Z, E, S>,
    proof: &Proof<Z, E, S>,
) -> bool {
    let instance_bn = convert_instance_to_bignum(instance);
    let gamma_s = S::hash(bincode::serialize(&instance_bn).unwrap());
    let gamma_z = S::to_z(&gamma_s);
    let gamma_prime_s = compute_gamma_prime_s(&gamma_z, &proof.w, &proof.x, &proof.y);
    let gamma_prime_z = S::to_z(&gamma_prime_s);

    // check shared secret from the multi-receiver encryption
    let lhs = proof
        .w
        .compose(&instance.ciphertexts_common.pow(&gamma_prime_z));
    let rhs = config.generator_h.pow(&proof.z_r);
    if !lhs.equals(&rhs) {
        return false;
    }

    // check commitments to polynomial coefficients from the Shamir secret sharing
    let scalars = compute_scalars(config, instance, &gamma_s);
    let mut lhs = E::multiexp(&instance.polynomial_coefficients_commitments, &scalars);
    lhs.mul_assign(&gamma_prime_s);
    lhs.add_assign(&proof.x);
    let mut rhs = E::generator();
    rhs.mul_assign(&proof.z_s);
    if !lhs.equals(&rhs) {
        return false;
    }

    // check ciphertexts
    let points = compute_gamma_powers(config.n as usize, &gamma_z);
    let lhs = BQF::multiexp(&instance.ciphertexts, &points)
        .pow(&gamma_prime_z)
        .compose(&proof.y.reduce());
    let mut rhs = BQF::multiexp(&instance.public_keys, &points);
    let f_pow = power_f(
        &config.generator_f,
        &config.discriminant,
        &config.q,
        &S::to_z(&proof.z_s),
    );
    rhs = rhs.pow(&proof.z_r).compose(&f_pow);
    lhs.equals(&rhs)
}
```


### Correctness and security

Correctness (completeness) follows by checking that the three verifying equations hold when the prover and verifier are honest.

For the security (statistical soundness), there are three distinct ways to produce invalid proofs that will verify correctly, and they bound the probability for the associated three events by $negl(\lambda_{st})$.

## Security of verifiable secret sharing
TODO.

Proofs with the [Universal Composability (UC)](https://en.wikipedia.org/wiki/Universal_composability) framework, a very elegant and general way to prove the security of cryptographic protocols.

In a nutshell, the UC framework defines an ideal functionality, that is an abstract model, and provides a concrete instantiation of that functionality. An adversarial environment tries to differentiate between the ideal world (based on the ideal functionality) and the real world (corresponding to the concrete instantiation). A simulator mimics the adversary’s behavior in the ideal world. The objective is to design a simulator such that no environment, given the ability to provide inputs and observe outputs from the parties, can distinguish between the ideal and real worlds.




## Security of distributed key generation
TODO.

- Why do we require $n\geq 2 t + 1$ for strong public verifiability (and so privacy of the shares...)?

# Performance

For 7 key holders and up to 3 (such that $7\geq 2\times 3+1$ holds) failing/malicious nodes, using the GMP backend:
| Operation                    | Running Time        |
|------------------------------|---------------------|
| Setup                        | 37 ms               |
| Generate Key                 | 20 ms               |
| Verify Key                   | 12 ms               |
| Generate Dealing              | 130 ms              |
| Verify Dealing                | 145 ms              |
| Verify All Dealings           | 1.1 s               |
| Aggregate Dealings (Public Result)     | 1.530792 ms         |

