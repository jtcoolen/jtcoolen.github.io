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


---
## Security Proofs


__Correctness__

> By bilinearity of $e$ (we only rely on the property $\forall \lambda\in\mathbb{F}_p,  e(a,\lambda b)=e(\lambda a, b)$):
> $$
> \begin{align}e(\pi, [\textsf{SK} - z]_2) &= e((q(\textsf{SK}) (\textsf{SK} - z)) g_1 , g_2)\\ &= e(((q(\textsf{SK})(\textsf{SK}-z)+f(z))-f(z))g_1,g_2)\\ &=e(c-[f(z)]_1,g_2)\end{align}
> $$


__Hiding__
> Suppose that there exists an adversary $\mathcal{A}$ breaking the hiding property of a commitment $\mathcal{C}$: it can compute the underlying polynomial $f(x)$ of degree $t$ given the commitment to $f(x)$ and $t$ valid witnesses $(z,y,\pi_{f(z)=y})$. By reduction, it is possible to construct an algorithm $\mathcal{B}$ from $\mathcal{A}$ that breaks the discrete logarithm assumption.
> The algorithm $\mathcal{B}$ has to solve the following discrete log instance $([1]_1, [a]_1)$. It first computes a trusted setup from a random $\alpha\in (\mathbb{F}_r)^\times$: $\textsf{PK}=\{[1]_1, [\alpha]_1, [\alpha^2]_1, \dots, [\alpha^t]_1\}$. $\mathcal{B}$ then chooses $t$ random scalars $(b_i)_{i\in⟦0,\ t − 1⟧}$ and computes
> $$
> [a]_1 + \sum_{i=0}^{t-1}b_i [\alpha^{i+1}]_1 = \Big[a + \sum_{i=0}^{t-1} b_i\alpha^{i+1}\Big]_1,
> $$
> which is the commitment to $f(x)=a+\sum_{i=0}^{t-1} b_i x^{i+1}$, namely $[f(\alpha)]_1$.
> $\mathcal{B}$ then chooses $t$ arbitrary points $(z_i)_{i\in⟦0,\ t − 1⟧}$, and computes $t$ valid witnesses for the $t$ chosen evaluations $\pi_{f(z_i)=b_i}=(\alpha-z_i)^{-1}([f(\alpha)]_1 -[b_i]_1)$. It then inputs $\textsf{PK}$,  $[f(\alpha)]_1$ the commitment to $f(x)$, and the $t$ valid witness tuples $(z_i,b_i,\pi_{f(z_i)=b_i})$ to $\mathcal{A}$.
> Once $\mathcal{A}$ outputs $f(x)$, $\mathcal{B}$ outputs the constant term $a$ of $f(x)$, which is the solution to the given discrete logarithm instance.

__Polynomial binding property__
> By reduction to the discrete logarithm problem, suppose that an adversary breaks the polynomial binding property by outputting two distinct polynomials $f(x)\in\mathbb{F}_p[x]_{\leq d}$ and $g(x)\in\mathbb{F}_p[x]_{\leq d}$ such that their commitments are equal, ie. $[f(\textsf{SK})]_1 = [g(\textsf{SK})]_1$. This implies $[f(\textsf{SK})-g(\textsf{SK})]_1=0_{\mathbb{G}_1}$ which is slightly weaker as it requires $ |\mathbb{G}_1|\ |\ f(\textsf{SK})-g(\textsf{SK})$, but since  the evaluations live in $\mathbb{F}_r$ for which $|\mathbb{F}_r|=|\mathbb{G}_1|$, $f(\textsf{SK})-g(\textsf{SK})=0$ necessarily. Hence, the polynomial $h(x)=f(x)-g(x)$ has $\textsf{SK}$ as root, which can be found with polynomial factorization algorithms over finite fields, such as Berlekamp in $O(d^3)$ (as its runtime is dominated by a Gaussian elimination).

__Evaluation binding property__
> By reduction to the $t$-SDH assumption, suppose that an adversary $\mathcal{A}$ can break the evaluation binding property, i.e., can output a commitment $c$ to $f(x)$, and two different valid witness tuples $(z, y, \pi_{f(z)=y})$ and $(z,y',\pi_{f(z)=y'})$. 
> An algorithm $\mathcal{B}$ inputs a trusted setup $\textsf{PK}=([1]_1, [\alpha]_1, [\alpha^2]_1,\dots, [\alpha^t]_1)\in\mathbb{G}_1^{t+1}$ to $\mathcal{A}$, which returns a commitment $c$ to some polynomial $f(x)$ for which the witness tuples $(z,y,\pi_{f(z)=y}=[q(\alpha)]_1)$, $(z,y',\pi_{f'(z)=y'}=[q'(\alpha)]_1)$ are valid openings ($\textsf{VerifyEval}$ returns $\textsf{true}$ for them). By the correctness of the scheme,
> $$
>    e(c-[y]_1,g_2)=e(\pi_{f(z)=y}, [\alpha-z]_2)
> $$
> $$
>    e(c-[y']_1,g_2)=e(\pi_{f(z)=y'}, [\alpha-z]_2).
> $$
> From which we get:
> $$ (f(\alpha)-y)e(g_1,g_2)=q(\alpha)(\alpha-z)e(g_1,g_2).$$
> $$ (f(\alpha)-y')e(g_1,g_2)=q'(\alpha)(\alpha-z)e(g_1,g_2).$$
> Thus
> $$ f(\alpha) = q(\alpha)(\alpha-z)+y = q'(\alpha)(\alpha-z) + y'.$$
> With a bit of algebra,
> $$\frac{q(\alpha)-q'(\alpha)}{y'-y}=\frac{1}{\alpha-z}. $$
> Thus $\mathcal{B}$ computes 
> $$
>    \frac{\pi_{f(z)=y}-\pi_{f(z)=y'}}{y'-y}= \Big[\frac{q(\alpha)-q'(\alpha)}{y'-y}\Big]_1
>    = \Big[\frac{1}{\alpha-z}\Big]_1,
> $$
> and returns $(-z,\Big[\frac{1}{\alpha-z}\Big]_1)$ a solution to the $t$-SDH instance $\textsf{PK}$.
