---
title: Threshold Encryption
draft: false
tags:
  - math
  - cryptography
  - pairing-based cryptography
---


Notes on a threshold encryption scheme: [Simple and Eﬃcient Threshold Cryptosystem from
the Gap Diffie-Hellman Group from Baek and Zheng](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=f613e2a76843153d19adcd7c59f2766334f799bf).
### Lagrange interpolation



A polynomial $f$ of degree $t-1$ can be recovered with a set of $t$ of its evaluations at distinct points $S=(x_i)_i$ with the following linear combination (Lagrange interpolation polynomial),

using the Lagrange coefficient $L_{S,i}(x)=\prod_{j\in S, j\neq i} \dfrac{x-x_j}{x_i-x_j}$:

$$f(x)=\sum_{i\in S} f(x_i) L_{S, i}(x).$$

Here we're taking $x_i=i$.

### Notations

- $[n]P$ for scalar $n\in F_r$ and point $P$ in elliptic curve: scalar point multiplication
- $x\stackrel{﹩}{\gets} R$: sample $x$ uniformly at random from set $R$

### Setup

BLS curve with pairing $e(\cdot,\cdot): G_1 \times G_2 \to G_t$ where $G_1,G_2$ are subgroups (elliptic curves) of the same order $r$,  generators $g_1, g_2, g_t\in G_1\times G_2 \times G_t$. The scalar field $F_r$ from BLS. A hash function $\mathsf{G} : \{0,1\}^* \to \{0,1\}^l$, hash to curve function $\mathsf{G}: \{0, 1\}^* \to G_2$, message space $\{0,1\}^l$.

A number of nodes $n>0$ indexed from 1 to $n$ (inclusive).

A threshold $0<t<n$.


A polynomial $f\in F_r[x]$ of degree $t-1$ whose coefficients $a_0, \dots, a_{t-1}\in F_r$ are not known and sampled uniformly at random.


An (unknown) secret master key $sk=f(0)\in F_r$, and corresponding public master key $Y=[sk] g_2\in G_2$.

Each node possesses a share of the secret key (called secret key share) $(sk_i)_i=(f(i))_i\in F_r^{n}$, with corresponding public verification keys for each node $(Y_i)_i=([sk_i]g_2)_i$.




#### Deriving public key shares and public key in $G_1$

Each node $i$ can compute a verification key share in $G_1$ by computing $Y_i'=[sk_i]g_1$. And after posting them on-chain, the master public key on $G_1$ can be recovered like so:

$$ Y'= \sum_{i\in S} [L_{S, i}(0)] Y_i= \sum_{i\in S} [L_{S, i}(0) f(i)] g_1 = [f(0)]g_1.$$ 

One can check that the verification key share in $G_1$ matches the one in $G_2$ with the pairing check for $$\alpha \stackrel{﹩}{\gets}  F_r^{\times}$$:

$$e(Y'+\sum_i \alpha^i Y_i', g_2)\stackrel{?}{=} e(g_1, Y+\sum_i \alpha^i Y_i).$$

### Encrypt

#### If $Y,(Y_i)_i\in G_2$

Input: $m\in\{0,1\}^{l}$, a valid setup (provided by a DKG in our case).
- Sample  $r \stackrel{﹩}{\gets}  F_r^{\times}$
- $U\gets [r] g_1\in G_1$
- Compute $V$:
    - if $Y,(Y_i)_i\in G_1$: $V\gets \mathsf{G}(serialize([r] Y)) \oplus m \in \{0,1\}^l$
    - if $Y,(Y_i)_i\in G_2$: $V\gets \mathsf{G}(serialize(e(g_1, [r] Y))) \oplus m\in \{0,1\}^l$
- $W\gets [r]\mathsf{H}(U,V)\in G_2$
- Ouput $C=(U,V,W)$


### Verify ciphertext
Input: ciphertext $C=(U,V,W)$

- Compute $H=\mathsf{H}(U,V)\in G_2$
- Return $e(g_1,W)\stackrel{?}{=}e(U, H)$


### Create decryption key share

- If ciphertext verification succeeds, $U_i\gets [f(i)] U\in G_1$, output $D_i=(i, U_i)$ 

### Verify decryption key share

#### if $Y,(Y_i)_i\in G_1$:
$e(Y_i, W)\stackrel{?}{=} e(U_i, H)$
and
$e(g_1, W)\stackrel{?}{=} e(U, H)$

can be verified with two pairings:

$$e(g_1+\alpha Y_i, W)\stackrel{?}{=}e(U+\alpha U_i, H)$$


#### Scaling with the number of ciphertexts

This works with public key shares $Y_i$ in $G_2$, which is doable as the keyholder can easily map their key shares from $G_1$ to $G_2$ with the mapping $(sk_i, g_1^{sk_i}) \mapsto g_2^{sk_i}$ which preserves the properties of the public keys, and the key shares in $G_2$ can be checked against the key shares in $G_1$ with a pairing after the DKG phase: $e(g_1, g_2^{sk_i}) = e(g_1^{sk_i}, g_2)$.

One can check for a threshold of keyholders indexed by $i\in V$ ($V$ is keyholder set):

$$\{e(D_{i,c}, g_2)\stackrel{?}{=} e(U_c, Y_i)\}_{i,c}\}=\{\text{true}_{i,c}\}_{i,c}$$



the above predicate is equivalent to with high probability (for nonzero random scalar $\alpha$):

$$ \{e(\sum_{c=0}^n [\alpha^c] D_{c,i}, g_2) \stackrel{?}{=} e(\sum_{c=0}^n [\alpha^c] U_{c}, Y_i)\}_i  = \{\text{true}_i\}_i$$

Alpha can be sampled solely from ALL the arguments to the pairings (Fiat-Shamir transformation).

For kernel execution, the random scalar $\alpha$ can be chosen as the hash of all the arguments to the pairings

So that we go (for $B$ ciphertexts) from $2\times V\times B$ pairings down to $2\times V$ ($V$ pairing checks instead of $V\times B$ pairing checks).


##### Why is the above batching secure?
Let's consider the simpler problem:

> Let A be an adversary who outputs pairs of group    
> elements ${(C_i, D_i)}_i$ and wins iff 
> $(\sum_i r^i C_i) = (\sum_i r^i D_i)$ where $r := Hash({(C_i, D_i)}_i)$.
> Prove that either $C_i = D_i$ 
> forall $i=0,...,n$ or A cannot win except with negligible probability.

We obtain
$(\sum_i r^i C_i) - (\sum_i r^i D_i) = 0$
thus
$(\sum_i x^i (C_i - D_i))(r) = 0$ which is a polynomial of degree $n$ (so has at most $n$ roots in the base finite field $F$) that vanishes at $r$. Assuming $C_i \neq D_i$ forall $i=0,...,n$ (otherwise the equality holds for any choice of $r$), by the Schwartz-Zippel lemma, the probability that a uniformly random $r$ vanishes the polynomial is $\leq n/|F|$ which is negligible when working in a big finite field $F$. We can take $r=Hash({(C_i, D_i)}_i)$ by the Fiat-Shamir transform as the hash function uniformly distributes outputs, regardless of the inputs, so that the probability of success of the adversary would remain $\leq n/|F|$.

One can reduce the above predicate to this simpler problem with the following chain of equivalences:

$$ \{e(\sum_c [\alpha^c] D_{c,i}, g_2) = e(\sum_c [\alpha^c] U_{c}, Y_i)\}_i  $$

$$ \{e(\sum_c [\alpha^c d_{c,i}] g_1, g_2) = e(\sum_c [\alpha^c u_c] g_1, Y_i)\}_i $$

$$ \{e(\sum_c [\alpha^c d_{c,i}] g_1, g_2) = e(\sum_c [\alpha^c u_c] g_1, y_i g_2)\}_i $$

$$ \{\sum_c [\alpha^c d_{c,i}] e( g_1, g_2) =\sum_c (\alpha^c u_c) y_i e( g_1, g_2)\}_i $$

$$ \{\sum_c \alpha^c d_{c,i}  =\sum_c (\alpha^c u_c) y_i \}_i$$

$$ \{(\sum_c (d_{c,i} - u_c y_i)x^c )(\alpha) =0 \}_i$$

### Combine decryption key shares


Input: $t$ decryption key shares $(D_i)_{i\in S}$ from a set $S$ of $t$ distinct nodes
- Verify ciphertext
- Verify decryption key shares $(D_i)_i$
- Recover plaintext $m$:
    - if $Y,(Y_i)_i\in G_1$: $m\gets \mathsf{G}(serialize(\sum_{i\in S} [L_{S, i}(0)] U_i))\oplus V$
    - if $Y,(Y_i)_i\in G_2$: $m\gets \mathsf{G}(serialize(e(\sum_{i\in S} [L_{S, i}(0)] U_i, g_2)))\oplus V$


Correctness:

$$
\begin{align}
\sum_{i\in S} [L_{S, i}(0)]U_i &= \sum_{i\in S} [L_{S, i}(0)f(i) r] g_1\\
&= [r\sum_{i\in S} L_{S, i}(0)f(i)] g_1\\
&= [rf(0)]g_1\\
&= [r]([f(0)]g_1)\\
&= [r]([sk]g_1)
\end{align}
$$

For $Y,(Y_i)_i\in G_2$: by bilinearity of the pairing (and $e(g_1, g_2)=g_t$):

$e(g_1,[r]Y)=e(g_1, [r][sk]g_2)=e(g_1, g_2)^{r\times sk}=g_t^{r\times sk}=e([r\times sk]g_1, g_2).$

