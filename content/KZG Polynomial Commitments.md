---
title: KZG Polynomial Commitments
draft: false
tags:
  - math
  - cryptography
  - elliptic curves
  - pairing-based cryptography
---


[KZG](https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf) is a polynomial commitment scheme whose distinguishing features are its constant-sized commitments, proofs, and its constant-time verification.


It operates on subgroups of elliptic curves over finite fields $\mathbb{G}_1=\langle g_1 \rangle$, $\mathbb{G}_2=\langle g_2 \rangle$, and $\mathbb{G}_T=\langle g_T \rangle$ of prime order $r$ ($r$ depends on the chosen security level $\lambda$), for which a pairing $e:\mathbb{G}_1 \times \mathbb{G}_2 \mapsto \mathbb{G}_T$ exists. From now on, we adopt the convenient notation for elliptic curve multiplication: $[n]_1=n g_1$ and $[n]_2=n g_2$.

As a commitment scheme, it is a tuple of deterministic polynomial-time algorithms commit, proveEval and verifyEval such that
1. Commit, on input a polynomial, returns a commitment:

    > $\textsf{commit}(\textsf{CK}, f(x)):$
    > 1. $\textbf{assert } \deg(f(x)) < d$
    > 2. $\textbf{return }c\gets \prod_{i=0}^{n-1} p_i[\tau^i]_1=[f(\tau)]_1$


2. ProveEval outputs a proof $\pi$ that $f$ evaluates to $f(z)$ in $z$:

    > $\textsf{proveEval}(\textsf{CK}, f(x), z):$
    > 1. $q(x)\xleftarrow{}\frac{f(x)-f(z)}{x-z}$
    > 2. $\textbf{return } \pi\gets\textsf{commit}(\textsf{CK}, q)$

3. VerifyEval, on input a proof, outputs true if the given proof testifies that the committed polynomial $f$ evaluates to $y$ in $z$ (using the pairing $e$):

    > $\textsf{verifyEval}(\textsf{VK}, c, z, y, \pi):$
    > 1. $\textbf{return } e(c-[y]_1, g_2) \overset{?}{=} e(\pi, [\tau]_2 - [z]_2)$



The crux of KZG is that if $f(x)$ has $z$ as root, then the quotient $q(x)$ is well-defined. This comes from the euclidean division of $f(x)$ by $x-z$: $f(x)=q(x)(x-z)+r(x)$, $\deg r(x) = 0$ and $f(z)=r(z)\equiv r(x)$. Moreover, it is impractical to forge false proofs: this requires to find a polynomial $g(x)$ which has the same commitment as $f(x)$, i.e. $f((x)-g(x))(\tau)=0$, which either requires the knowledge of $\tau$ or to try to make the difference zero in as most places as possible in the hope to cover $\tau$. In the latter case, this is highly unlikely, as the probability to find a zero is already

$$\text{P}[(f(x)-g(x))(s)=0 | s \gets_R\mathbb{F}_r]\leq d/|\mathbb{F}_r|$$

thanks to the Schwartz-Zippel lemma in the univariate case. In typical use cases, the largest degree can be $2^{21}$ and the cardinal of the scalar field $|\mathbb{F}_r|\approx 2^{255}$ so the probability to find a zero is $1/2^{234}$.



See [batched KZG proofs](Batched-KZG-Proofs.md) for efficient ways to compute and verify KZG proofs.
