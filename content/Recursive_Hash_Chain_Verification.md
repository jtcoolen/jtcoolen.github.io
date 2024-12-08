 ---
title: Recursive proof verification
draft: false
tags:
    - cryptography
    - incrementally verifiable computation
    - plonky2
    - zk circuit
---

We will review how to use the Plonky2 API for recursive circuits to implement a circuit that adds a new hash to an existing chain of hashes, while recursively verifying the chain. This enables the generation of a SNARK proof for a long hash chain, allowing verification of the chain faster than directly computing it. Instead of processing the entire chain, the verifier only needs to handle a compact proof that ensures the integrity of the chain.


The objective is to implement the logic for a recursive hash chain based on the following statement:

> Given $v_1, v_t$, and a number of steps $t$, $H$ the Poseidon hash function (SNARK-friendly hash function),
> check that $v_t = H(t || H(t-1 || \dots H(1 || v_1)))$.

At each step, the prover will:

- Perform one hash computation according to the structure of the hash chain
- Recursively verify a SNARK proof that ensures the correctness of all previous steps

The code is available at https://github.com/jtcoolen/recursive_hash_chain_verification/tree/master.


# Parameters

Plonky2 mixes STARK proofs which optimize for proof generation time at the expense of proof size, and SNARK proofs which optimize for proof size and verification time. See the [Plonky2 deep-dive](https://polygon.technology/blog/plonky2-a-deep-dive) and the [technical paper](https://github.com/0xPolygonZero/plonky2/blob/main/plonky2/plonky2.pdf) to learn more about Plonky2.
 
Here we set up some constants such as the degree of the extension field for the Goldilocks field, the Poseidon hash function Goldilocks field configuration and the field `F` used by the proof system:
```rust
use plonky2::plonk::config::GenericConfig;
use plonky2::plonk::config::PoseidonGoldilocksConfig;

pub const D: usize = 2;
pub type C = PoseidonGoldilocksConfig;
pub type F = <C as GenericConfig<D>>::F;
```

# Computing the hash chain

To compute the hash chain, we iteratively hash starting from an initial hash:

```rust
use anyhow::Result;

use plonky2::hash::hash_types::RichField;
use plonky2::hash::hashing::hash_n_to_hash_no_pad;
use plonky2::hash::poseidon::PoseidonPermutation;
use plonky2::plonk::proof::ProofWithPublicInputs;
use plonky2_field::types::Field;
use plonky2_field::{goldilocks_field::GoldilocksField, types::PrimeField64};
use rand::Rng;

use crate::constants::{C, D, F};
use crate::errors::HashChainError;

/// Generates a random hash consisting of four GoldilocksField elements.
pub fn generate_random_hash() -> [GoldilocksField; 4] {
    let mut rng = rand::thread_rng();
    [(); 4].map(|_| GoldilocksField::from_canonical_u64(rng.gen()))
}

/// Computes a hash chain starting from the initial state, applying the hash function n times
/// on the current recursion depth and the current hash.
pub fn hash_chain<F: RichField>(initial_state: [F; 4], n: usize) -> [F; 4] {
    // Use fold to iterate from 1 to n and accumulate the hash state
    (1..=n).fold(initial_state, |current, i| {
        hash_n_to_hash_no_pad::<F, PoseidonPermutation<F>>(
            &[F::from_canonical_u32(i as u32)]
                .iter()
                .chain(current.iter())
                .copied()
                .collect::<Vec<_>>(),
        )
        .elements
    })
}
```

We can check the final hash from the proof against the one computed from scratch:
```rust
/// Allows to check that the circuit correctly computes the hash chain
pub fn check_hash_chain(
    proof: &ProofWithPublicInputs<GoldilocksField, C, D>,
) -> Result<(), HashChainError<GoldilocksField>> {
    let counter = proof.public_inputs[0];
    let initial_hash = &proof.public_inputs[1..5];
    let hash = &proof.public_inputs[5..9];

    let initial_hash: [F; 4] = initial_hash
        .try_into()
        .map_err(|_| HashChainError::Other("Failed to convert hash slice.".to_string()))?;
    let hash: [F; 4] = hash
        .try_into()
        .map_err(|_| HashChainError::Other("Failed to convert hash slice.".to_string()))?;

    let expected_hash = hash_chain(initial_hash, counter.to_canonical_u64() as usize);

    if hash != expected_hash {
        return Err(HashChainError::HashVerificationFailed {
            expected: expected_hash.to_vec(),
            actual: hash.to_vec(),
        });
    }

    Ok(())
}
```

# Building the recursive circuit

The circuit consists in setting up the variables and proof's inputs and outputs on which we will put some constraints and relationships on.
Notably, it ensures that the output hash of the current proof is constructed as the hash of the `counter` (corresponding to the step $t$ of the hash chain)
concatenated with the input hash of the current proof.
The inputs of the proof for level $t$ correspond to the outputs of the inner proof for level $t-1$. So that there is only one proof to verify, the inner proof appears as an input of the current proof.

```rust
use anyhow::{Ok, Result};

use plonky2::field::goldilocks_field::GoldilocksField;
use plonky2::hash::hash_types::HashOutTarget;
use plonky2::hash::poseidon::PoseidonHash;
use plonky2::iop::target::{BoolTarget, Target};
use plonky2::plonk::circuit_builder::CircuitBuilder;
use plonky2::plonk::circuit_data::{
    CircuitConfig, CircuitData, CommonCircuitData, VerifierCircuitTarget,
};
use plonky2::plonk::config::GenericConfig;
use plonky2::plonk::proof::ProofWithPublicInputsTarget;

use plonky2::field::extension::Extendable;
use plonky2::gates::noop::NoopGate;
use plonky2::hash::hash_types::RichField;
use plonky2::plonk::config::AlgebraicHasher;

use crate::constants::{C, D, F};
use crate::errors::CircuitError;

pub(crate) struct CircuitSetup<F, C, const D: usize>
where
    F: RichField + Extendable<D>,
    C: GenericConfig<D, F = F>,
{
    pub circuit_data: CircuitData<F, C, D>,
    pub common_data: CommonCircuitData<F, D>,
    pub condition: BoolTarget,
    pub inner_cyclic_proof_with_pis: ProofWithPublicInputsTarget<D>,
    pub verifier_data_target: VerifierCircuitTarget,
}

/// Sets up the circuit builder and related data.
pub fn setup_circuit(depth: usize) -> Result<CircuitSetup<GoldilocksField, C, D>> {
    if depth == 0 {
        return Err(CircuitError::InvalidRecursionDepth(depth).into());
    }

    let config = CircuitConfig::standard_recursion_config();
    let mut builder = CircuitBuilder::<F, D>::new(config);

    // Tracks how many times recursion has occurred
    let counter = builder.add_virtual_public_input();

    let initial_hash_target = builder.add_virtual_hash();
    builder.register_public_inputs(&initial_hash_target.elements);

    let current_hash_in = builder.add_virtual_hash();

    // Hash the current depth and the previous hash (inner_cyclic_latest_hash or initial hash)
    let current_hash_out = builder.hash_n_to_hash_no_pad::<PoseidonHash>(
        [[counter].to_vec(), current_hash_in.elements.to_vec()].concat(),
    );
    builder.register_public_inputs(&current_hash_out.elements);

    let verifier_data_target = builder.add_verifier_data_public_inputs();

    let condition = builder.add_virtual_bool_target_safe();

    let one = builder.one();

    let mut common_data = common_data::<F, C, D>();
    common_data.num_public_inputs = builder.num_public_inputs();

    let inner_cyclic_proof_with_pis = builder.add_virtual_proof_with_pis(&common_data);

    // ensures consistency and proper transition between successive proofs
    connect_proof_hash_states(
        &mut builder,
        condition,
        initial_hash_target,
        current_hash_in,
        &inner_cyclic_proof_with_pis,
        counter,
        one,
    )?;

    // If condition is true, verifies the provided inner proof against the current state.
    // If condition is false, uses a dummy verification, allowing the circuit to gracefully
    // handle cases where recursion should not proceed.
    builder.conditionally_verify_cyclic_proof_or_dummy::<C>(
        condition,
        &inner_cyclic_proof_with_pis,
        &common_data,
    )?;

    let circuit_data = builder.build::<C>();

    Ok(CircuitSetup {
        circuit_data,
        common_data,
        condition,
        inner_cyclic_proof_with_pis,
        verifier_data_target,
    })
}
```

When verifying proofs recursively, the circuit should ensure the consistency between the public parameters of the proofs, that is check that:
- the `counter` corresponding to the current step $t$ in the hash chain is decremented when verifying the inner recursive (aka cyclic) proof
- the proofs refer to the same `initial_hash` from the start of the hash chain
- when reaching the base proof, the hash should match the `initial_hash`, otherwise the inner proof's output hash should match the outer proof input hash

```rust
/// Handles the logic of verifying the relationship between the initial
/// hash, the inner proof's latest hash, and the recursive counter. It also sets up the
/// conditions for either continuing recursion with the inner proof's state.
fn connect_proof_hash_states(
    builder: &mut CircuitBuilder<F, D>,
    condition: BoolTarget,
    initial_hash_target: HashOutTarget,
    current_hash_in: HashOutTarget,
    inner_cyclic_proof_with_pis: &ProofWithPublicInputsTarget<D>,
    counter: Target,
    one: Target,
) -> Result<()> {
    let inner_cyclic_pis = &inner_cyclic_proof_with_pis.public_inputs;

    let inner_cyclic_counter = inner_cyclic_pis[0];
    let inner_cyclic_initial_hash = HashOutTarget::try_from(&inner_cyclic_pis[1..5])
        .map_err(|e| CircuitError::ConversionError(e.to_string()))?;
    let inner_cyclic_latest_hash = HashOutTarget::try_from(&inner_cyclic_pis[5..9])
        .map_err(|e| CircuitError::ConversionError(e.to_string()))?;
    // Copy constraint for the initial_hash of the chain from the inner and outer proofs.
    // This ensures consistency between the initial state of the outer proof and the inner proof
    builder.connect_hashes(initial_hash_target, inner_cyclic_initial_hash);

    // Enables the circuit to either continue with a recursive step (when condition is true)
    // or reset to an initial state (when condition is false).
    let actual_hash_in = HashOutTarget {
        elements: core::array::from_fn(|i| {
            builder.select(
                condition,
                inner_cyclic_latest_hash.elements[i],
                initial_hash_target.elements[i],
            )
        }),
    };
    // copy constraint current_hash_in = actual_hash_in
    builder.connect_hashes(current_hash_in, actual_hash_in);

    // Update counter by adding 1 to inner_cyclic_counter if the condition is true.
    // Otherwise, the counter remains unchanged.
    let new_counter = builder.mul_add(condition.target, inner_cyclic_counter, one);
    builder.connect(counter, new_counter);

    Ok(())
}
```


# Building the recursive proofs

The first proof doesn't prove anything but takes the initial hash as a public input, and sets the condition public input for proof recursion to false, so that the circuit stops the recursive verification

```rust
use anyhow::{Ok, Result};

use crate::circuit::CircuitSetup;
use crate::constants::{C, D};
use plonky2::field::goldilocks_field::GoldilocksField;
use plonky2::iop::target::BoolTarget;
use plonky2::iop::witness::{PartialWitness, WitnessWrite};
use plonky2::plonk::circuit_data::{CircuitData, CommonCircuitData, VerifierCircuitTarget};
use plonky2::plonk::proof::{ProofWithPublicInputs, ProofWithPublicInputsTarget};
use plonky2::recursion::cyclic_recursion::check_cyclic_proof_verifier_data;
use plonky2::recursion::dummy_circuit::cyclic_base_proof;

/// Base Case
/// Sets up the base proof (for the initial hash in the chain).
pub fn setup_base_proof(
    initial_hash: [GoldilocksField; 4],
    cyclic_circuit_data: &CircuitData<GoldilocksField, C, D>,
    common_data: &CommonCircuitData<GoldilocksField, D>,
    condition: BoolTarget,
    inner_cyclic_proof_with_pis: &ProofWithPublicInputsTarget<D>,
    verifier_data_target: &VerifierCircuitTarget,
) -> Result<ProofWithPublicInputs<GoldilocksField, C, D>> {
    let mut pw = PartialWitness::new();
    let initial_hash_pis = initial_hash.into_iter().enumerate().collect();

    pw.set_bool_target(condition, false); // base proof so the condition to do the recursion is set to false
    pw.set_proof_with_pis_target::<C, D>(
        inner_cyclic_proof_with_pis,
        &cyclic_base_proof(
            common_data,
            &cyclic_circuit_data.verifier_only,
            initial_hash_pis,
        ),
    );
    pw.set_verifier_data_target(verifier_data_target, &cyclic_circuit_data.verifier_only);

    cyclic_circuit_data.prove(pw)
}
```

When extending a proof, we construct a new proof with the condition for recursively verifying the inner proof to true, set the inner proof as public input to the proof, and set verifier-related information that needs to be included in the circuit to verify recursive proofs.

```rust
/// Inductive Step
/// Extends an existing proof for a recursive circuit.
pub fn extend_proof(
    cyclic_circuit_data: &CircuitData<GoldilocksField, C, D>,
    condition: BoolTarget,
    inner_cyclic_proof_with_pis: &ProofWithPublicInputsTarget<D>,
    proof: &ProofWithPublicInputs<GoldilocksField, C, D>,
    verifier_data_target: &VerifierCircuitTarget,
) -> Result<ProofWithPublicInputs<GoldilocksField, C, D>> {
    let mut pw = PartialWitness::new();

    pw.set_bool_target(condition, true);
    pw.set_proof_with_pis_target(inner_cyclic_proof_with_pis, proof);
    pw.set_verifier_data_target(verifier_data_target, &cyclic_circuit_data.verifier_only);

    cyclic_circuit_data.prove(pw)
}

/// Builds the recursive proof for a given depth.
pub fn build_recursive_proof(
    depth: usize,
    initial_hash: [GoldilocksField; 4],
    circuit_setup: &CircuitSetup<GoldilocksField, C, D>,
) -> Result<ProofWithPublicInputs<GoldilocksField, C, D>> {
    let base_proof = setup_base_proof(
        initial_hash,
        &circuit_setup.circuit_data,
        &circuit_setup.common_data,
        circuit_setup.condition,
        &circuit_setup.inner_cyclic_proof_with_pis,
        &circuit_setup.verifier_data_target,
    )?;

    let proof = (0..depth).try_fold(base_proof, |current_proof, _| {
        extend_proof(
            &circuit_setup.circuit_data,
            circuit_setup.condition,
            &circuit_setup.inner_cyclic_proof_with_pis,
            &current_proof,
            &circuit_setup.verifier_data_target,
        )
    })?;

    check_cyclic_proof_verifier_data(
        &proof,
        &circuit_setup.circuit_data.verifier_only,
        &circuit_setup.circuit_data.common,
    )?;

    Ok(proof)
}

```


# Benchmarks

On an M2 Pro chip, `cargo bench` yields the following running times:

|  Hash Chain Length  |      Prove      |  Verify | Native Hash Chain Computation |
|----------|-------------|------|-----|
| 10 | 5.8 s  | 3 ms | 8 µs |
| 20 | 10 s   |   3 ms | 17 µs |
| 30 | 13 s   |   3 ms | 27 µs |
| 40 | 17 s   |   3 ms | 36 µs |
| 50 | 21 s   |   3 ms | 45 µs |
| 100 | 42 s   |   3 ms | 92 µs |
| 200 | 80 s   |   3 ms | 182 µs |
| 1000 | 392 s   |   3 ms | 921 µs |
| 4000 | 25 min 7 s   |   3 ms | 3.6 ms |




Note that the verifier’s run-time cost remains constant because Plonky2 generates SNARK proofs, which can be verified in constant time.

For shorter hash chains (less than ~4000 in length), computing the chain natively is faster than verifying the SNARK proof. However, from a bandwidth perspective, verifying the SNARK proof is more efficient since the intermediate data used in the hash chain doesn't need to be transmitted, and the SNARK proof is constant-sized (around 43 kB). This is especially useful in blockchain contexts, where validators would otherwise need to download the entire chain to verify it. With recursive SNARKs, validators can be convinced of the chain’s correctness by only receiving a proof for the latest state, without needing the full chain history. This approach is used by the [Mina](https://minaprotocol.com) blockchain.

# Optimization: several Poseidon calls per circuit

The previous approach is rather inefficient as it does the maximum number of recursion steps. We can instead pack several calls (let's try one thousand) to the Poseidon hash function per recursion step to amortize the cost of recursion:
```diff
diff --git a/benches/poseidon.rs b/benches/poseidon.rs
index 561ccdc..d300321 100644
--- a/benches/poseidon.rs
+++ b/benches/poseidon.rs
@@ -12,14 +12,12 @@ criterion_group! {
 criterion_main!(recursive_snark);

 fn bench_recursive_snark_prove(c: &mut Criterion) {
-    let depths = vec![10, 20];
+    let depths = vec![30];

     for d in depths {
         let mut group = c.benchmark_group(format!("Plonky2-Poseidon-num-steps-{}", d));
         group.sample_size(10);

-        let d = d - 1;
-
         group.bench_function("Prove", |b| {
             b.iter(|| {
                 let initial_hash = generate_random_hash();
@@ -36,7 +34,7 @@ fn bench_recursive_snark_prove(c: &mut Criterion) {
 }

 fn bench_recursive_snark_verify(c: &mut Criterion) {
     let depths = vec![10; 20];

     for d in depths {
         let mut group = c.benchmark_group(format!("Plonky2-Poseidon-num-steps-{}", d));
diff --git a/src/circuit.rs b/src/circuit.rs
index 378594e..ad5a1c6 100644
--- a/src/circuit.rs
+++ b/src/circuit.rs
@@ -73,20 +73,29 @@ pub fn setup_circuit(depth: usize) -> Result<CircuitSetup<GoldilocksField, C, D>
     let initial_hash_target = builder.add_virtual_hash();
     builder.register_public_inputs(&initial_hash_target.elements);

+    let one = builder.one();
+    let mut count = counter;
+    let mut n_iter = builder.zero();
+
     let current_hash_in = builder.add_virtual_hash();

     // Hash the current depth and the previous hash (inner_cyclic_latest_hash or initial hash)
-    let current_hash_out = builder.hash_n_to_hash_no_pad::<PoseidonHash>(
-        [[counter].to_vec(), current_hash_in.elements.to_vec()].concat(),
-    );
-    builder.register_public_inputs(&current_hash_out.elements);
+
+    let mut intermediate_hash = current_hash_in;
+    for _i in 0..1_000 {
+        intermediate_hash = builder.hash_n_to_hash_no_pad::<PoseidonHash>(
+            [[count].to_vec(), intermediate_hash.elements.to_vec()].concat(),
+        );
+        count = builder.add(count, one);
+        n_iter = builder.add(n_iter, one);
+    }
+
+    builder.register_public_inputs(&intermediate_hash.elements);

     let verifier_data_target = builder.add_verifier_data_public_inputs();

     let condition = builder.add_virtual_bool_target_safe();

-    let one = builder.one();
-
     let mut common_data = common_data::<F, C, D>();
     common_data.num_public_inputs = builder.num_public_inputs();

@@ -100,7 +109,7 @@ pub fn setup_circuit(depth: usize) -> Result<CircuitSetup<GoldilocksField, C, D>
         current_hash_in,
         &inner_cyclic_proof_with_pis,
         counter,
-        one,
+        n_iter,
     )?;

     // If condition is true, verifies the provided inner proof against the current state.
@@ -133,7 +142,7 @@ fn connect_proof_hash_states(
     current_hash_in: HashOutTarget,
     inner_cyclic_proof_with_pis: &ProofWithPublicInputsTarget<D>,
     counter: Target,
-    one: Target,
+    n_iter: Target,
 ) -> Result<()> {
     let inner_cyclic_pis = &inner_cyclic_proof_with_pis.public_inputs;

@@ -162,7 +171,7 @@ fn connect_proof_hash_states(

     // Update counter by adding 1 to inner_cyclic_counter if the condition is true.
     // Otherwise, the counter remains unchanged.
-    let new_counter = builder.mul_add(condition.target, inner_cyclic_counter, one);
+    let new_counter = builder.mul_add(condition.target, inner_cyclic_counter, n_iter);
     builder.connect(counter, new_counter);

     Ok(())
diff --git a/src/hash_chain.rs b/src/hash_chain.rs
index ed054cb..deafa5a 100644
--- a/src/hash_chain.rs
+++ b/src/hash_chain.rs
@@ -23,7 +23,7 @@ pub fn hash_chain<F: RichField>(initial_state: [F; 4], n: usize) -> [F; 4] {
     // Use fold to iterate from 1 to n and accumulate the hash state
     (1..=n).fold(initial_state, |current, i| {
         hash_n_to_hash_no_pad::<F, PoseidonPermutation<F>>(
-            &[F::from_canonical_u32(i as u32)]
+            &[F::from_canonical_u32(999 + i as u32)]
                 .iter()
                 .chain(current.iter())
                 .copied()
diff --git a/src/lib.rs b/src/lib.rs
index e7e3062..64b2ff4 100644
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -25,8 +25,9 @@ pub fn recursive_proof(
         ))?;
     }

+    let circuit_setup = setup_circuit(depth)?;
+
     let adjusted_depth = depth - 1; // for zero-based indexing
-    let circuit_setup = setup_circuit(adjusted_depth)?;

     let proof = build_recursive_proof(adjusted_depth, initial_hash, &circuit_setup)?;
```

With this simple change, we're proving a hash chain of length 30,000 in 14 seconds!
