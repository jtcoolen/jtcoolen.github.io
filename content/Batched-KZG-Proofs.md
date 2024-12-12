---
title: Batched KZG Proofs
draft: false
tags:
  - math
  - cryptography
  - elliptic curves
  - pairing-based cryptography
---

Follow-up of [KZG Polynomial Commitments](KZG-Polynomial-Commitments): details efficient ways to verify multiple KZG proofs at once, and to compute multiple KZG proofs.

## Batched KZG

Prove returns the proof of opening of $f$ on $x_1, \cdots, x_n$. Note that we usually give the evaluations $f(x_1), \cdots f(x_n)$ with the proof. 

> $\textsf{Prove}(f(X), x_1, \cdots, x_n):$
> 1. $Z \gets \Pi_{i=0}^n (X -x_i)$
> 2. $Q \gets f/Z$ (*we compute the euclidian division without the remainder*)
> 3. $\textbf{return }\textsf{Commit}(Q)$

Verifies that a pre-image of c evaluates to $y_i$ on $x_i$. 

> $\textsf{Verify}(C, \pi, x_1, \cdots ,x_n, y_1 \cdots, y_n):$
> 1. $R  \gets \textsf{interpolate}(x_1, \cdots ,x_n, y_1 \cdots, y_n)$
> 2. $[R] \gets \textsf{Commit}(R)$
> 3. $Z \gets \Pi_{i=0}^n (X -x_i)$
> 4. $[Z] \gets \textsf{Commit}(Z)$
> 5. $\textbf{return} \hspace{0.5em} e(C - [R], g_2) \overset{?}{=} e(\pi, [Z])$

### Roots of unity
Note that we usually use this algorithms when $\{x_1, \cdots x_n\}$ is a root of unity group.
This means that $Z$ has a sparse representation $X^n - x_1^n$ where $x_1$ is a generator.


### Correcteness
To prove correctness we need to show that $f \mod Z = \textsf{interpolate}(x_1, \cdots ,x_n, y_1 \cdots, y_n))$.
Notice that both polynomials are of degree less than $n$ and have the same evaluations on $n$ points $x_1, \cdots , x_n$.

### Security
The security is similar to KZG. We work in the algebraic group model. We try to reduce to the discrete log. Assumes that the adversary gives us a commitment, a proof, and some $y_i$ which are incorrect but passes the verification. We will solve a discrete log.  The adversary returns us $\pi$  and $C$ computed from the srs. It means that $\pi$ can be identified with the commitment to a polynomial $\Pi$ and $f$.
If the verification passes then $e(C + [\bar{R}], g_2)= e(\pi,[Z])$. This means that $f = \Pi*Z + \bar{R}$, where $\bar{R}$ is the interpolation of the given evaluations, or the adversary manages to find a non zero linear combination of srs elements that cancels (which reduces to discrete log).
To see that this reduces to the discrete log, notice that if the adversary produces $a_i$ not all zeroes, then $\sum_i a_i*srs.i = 0$ implies that the trapdoor of the srs is one of the root of the polynomial $P =\sum_i a_i X^I$ whose coefficients are $a_i$. So we can solve the discrete log of srs.1 with probability $\frac{1}{deg(P)}$. Assumes that the adversary did not produce such linear combination.
So the equation is correct on the polynomial, and we have $R = \bar{R} \mod Z$ 
However, the adversary if crafting a proof for a false statement we have $\bar{R}$ which is not the remainder of $f/Z$ (by the same reasoning as in the correctness). By construction we also have $deg(\bar{R}) = n-1$. This is a contradiction. So the adversary managed to solved a discrete log.


---
More formally, to prove the evaluation binding, we need to reduce the problem to the $t$-bilinear strong Diffie-Hellman assumption which slightly extends the $t$-SDH assumption:

### $t$-bilinear strong Diffie-Hellman assumption
> Let $\mathbb{G}$ be an abelian group and $g\in\mathbb{G}$, for any adversary $\mathcal{A}$:
>
> $$P \{(c,e(g,g)^{1/(\alpha+c)})\gets\mathcal{A}(g,g^\alpha,\dots, g^{\alpha^t})\in\mathbb{G}^{t+1}\}=\epsilon(\lambda)$$,
> for $c\in \mathbb{Z}_p \setminus \{ \alpha \}$.

### Evaluation binding
> By reduction to the $t$-BSDH assumption, suppose an adversary $\mathcal{A}$, given a trusted setup $\textsf{PK}$, outputs two different valid witness tuples $(c,Z,Y,\pi_{f(Z)=Y}=[Q(\alpha)]_1)$ and $(c,z', y',\pi_{f(z')=y'}=[q(\alpha)]_1)$. Let's call $r(x)$ the interpolated polynomial from the values of $Y$. By assumption, $\textsf{VerifyEvalBatch}(\textsf{PK},c,Z,Y,\pi_{f(Z)=Y})=1$, $\textsf{VerifyEval}(\textsf{PK}, c, z',y',\pi_{f(z')=y'})=1$, with $z'\in Z$ and $r(z')\neq y'$.
> Let's set $p(x)=\prod_{z\in Z} (x-z)$ and $p'(x)=p(x)/(x-z')$. By the correctness of the scheme,
> $$
>    e(c-[y']_1,g_2)=e(\pi_{f(z')=y'}, [\alpha-z']_2)
> $$
> and
> $$
> e(c-[r(\alpha)]_1,g_2)=e(\pi_{f(Z)=Y}, [p(\alpha)]_2).
> $$
> So that $f(\alpha)-y'=q(\alpha)(\alpha-z')$ and
> $f(\alpha)-r(\alpha) = Q(\alpha)p(\alpha)$, which entails $ q(\alpha)(\alpha-'z)+y' = Q(\alpha)p(\alpha)+r(\alpha)$.
> Factoring by  $(\alpha-z')$ we obtain
> $$ (\alpha - z') (q(\alpha) - Q(\alpha)p'(\alpha)) = r(\alpha) - y' $$
> We're almost there but we can't compute $r(\alpha)$, so we rewrite the expression using the Euclidean division $r(x)=r'(x)(x-z)+r(z)$:
> $$ (\alpha - z') (q(\alpha) - Q(\alpha)p'(\alpha)) = r'(\alpha)(\alpha-z')+r(z') - y'. $$
> With a bit of algebra ($r(z')\neq y'$)
> $$
>      (\alpha - z')^{-1} = \frac{q(\alpha) - Q(\alpha)p'(\alpha) -r'(\alpha)}{r(z') - y'}.
> $$
> Hence the need for a pairing to be able to multiply  $Q(\alpha)$ and $p'(\alpha)$ in $\mathbb{G}_T$.
> A solution to the $t$-BSDH instance is given by $-z'$, and for example
> $$ (\alpha- z')^{-1}e(g_1,g_2)=(r(z') - y')^{-1} e(\pi_{f(z')=y'}-[r'(\alpha)]_1 , g_2) e(-\pi_{f(Z)=Y}, [p'(\alpha)]_2).$$
---

Note that in the next proofs, we will assume that everything that is verified in the pairing equations is true for the corresponding polynomial equation. The same reduction to discrete log applies.

### Performance
The prover performances are discussed in the Batch opening part.
For the verifier, the main cost is to compute a commitment to the remainder. We propose a variation in the next part, to avoid this commimtent on the verifier side.

## Verifier efficient variations

We differ from the preceeding algorithm by adding the commitment to the remainder in the proof. We then generate a pseudo random challenge via Fiat-Shamir, and give a KZG proof for evaluation on that pseudo random challenge. 

> $\textsf{Prove}(f(X), x_1, \cdots, x_n, y_1,\cdots, y_n):$
> 1. $Z \gets \Pi_{i=0}^n (X -x_i)$
> 2. $[Q] \gets f/\textsf{Z}$ (*Same as preceeding algorithm*)
> 3. $[R] \gets \textsf{Commit}(f \mod Z)$
> 4. $y_i \gets f(x_i)$
> 5. $s = \textsf{Hash}([R], [Q], y_1, \cdots, y_n)$ (*Fiat shamir*)
> 6. $\pi_{KZG}= \textsf{Commit}(\frac{R-R(s)}{X-s})$
> 7. $\textbf{return }[Q],[R], \pi_{KZG}$


Verifies that a pre-image of C evaluates to $y_i$ on $x_i$ 
> $\textsf{Verify}(C, y_1,\cdots, y_n, [Q], [R], \pi_{KZG})$:
> 1. $s = \textsf{Hash}(C, [Q], [R], y_1, \cdots, y_n)$ (*Fiat Shamir*)
> 2. $R_s  \gets R(s)$
> 3. $[Z] \gets \textsf{Commit}(\Pi_{i=0}^n (X -x_i))$
> 4. $\alpha \gets \textsf{Hash}(C, [R], [Q], y_1, \cdots, y_n, \pi_{KZG})$ (*Get a (pseudo) random  number to batch pairings*)
> 5. $\textbf{return }  e(C - [R] + \alpha * ([R] - R_s) , g_2) \overset{?}{=} e([Q], [Z]) *e(\alpha*\pi_{KZG}, [\tau-s])$

Note that line 2 is done  using the barycentric evaluation : https://hackmd.io/@vbuterin/barycentric_evaluation

### Security
We want to reduce to the security of KZG and the batched version of it.
To do that we need to handle the two pseudo random numbers $\alpha$ and $s$.
Lets start with $\alpha$.
Assume that the adversary computes $\alpha$ using a random oracle
(otherwise his probability of cheating is negligible).
We use rewinding. Denote by $f$ the pre-image of $C$ (given by the algebraic group model) and $\Pi_{KZG}$ the pre-image of $\pi_{KZG}$.
We get for multiple $\alpha$, in the realm of polynomials (see preceeding proof): 
$f - \bar{R} + \alpha * (\bar{R} - R_s) = Q * Z + \alpha*\Pi_{KZG} *(X-s)$
Seeing that as a polynomial in $\alpha$ we get that the two terms
$f-\bar{R} - Q*Z$ and $\bar{R} -R_s - \Pi_{KZG}*(X-s)$ are zeros (otherwise the polynomial in $\alpha$ of degree one has more than one root).
By the security of batched KZG, the first term being zero means that $f \mod Z = \bar{R}$.
For the second, let's rewind again.
Again asumes that $s$ is computed by calling a random oracle
(otherwise his probability of cheating is negligible).
We get multiple $R_s$ and $s$ st.
$(\bar{R} - R_s) = \Pi_{KZG} *(X-s)$.
Rembember that since the $y_i$ are fixed, the different $R_s$ correspond to the evaluations on the different $s$ of the same polynomial, which is the  interpolation of the $y_i$ on $x_i$. We denote by $R$ that unique polynomial.
By the KZG security, this means that $\bar{R}(s)=R_s$ for multiple $s$.
Now, since the degree of $\bar{R}$ is bounded (because of the srs size) this means that $\bar{R}$ is equal to $R$ on more points than it's degree, so $\bar{R}=R$. This plus $f \mod Z = \bar{R}$ contradicts the fact that the statement of the adversary is incorrect to begin with.

## Batching verification
We now present an algorithm that checks multiple shards at once.
The main idea will be to generate a pseudo random $\alpha$, to commit only once to the remainder, and to put everything in one pairing.


 We denote by $X_i$ and $Y_i$ a set of $x$ and $y$. Verifies that a pre-image of $C$ evaluates to $Y_i$ (the shards in our context) on $X_i$ (the shifted subgroup on our context) for all $i$. $\pi_i$ are as before $f/Z_{X_i}$. Denote by $x_i$ the generator of $X_i$. Denote $\ell$ the size of a subgroup $|X_i|$. Note that $Z_{X_i}=X^{\ell} - x_i^{\ell}$. Note that to batch, instead of multipying directly $\pi_i*Z$, we expand into $\pi_i*X^{\ell} - \pi_i * x_i^{\ell}$, as done in the batched version of KZG presented in Plonk (part 3 of https://eprint.iacr.org/2019/953). The second term is denoted $W$ as in Plonk.

> $\textsf{Verify}(C, \pi_i, X_i, Y_i):$
> 1. $R_i  \gets \textsf{interpolate}(X_i,Y_i)$
> 2. $\alpha \gets H(C, Y_i, \pi_i)$
> 3. $R \gets \sum_i \alpha^i * R_i$
> 4. $[R] \gets \textsf{Commit}(R)$
> 5. $C_{batched} \gets (\sum_i \alpha^i) *C$ (*We compute the sum first to avoid doing several EC multiplications*)
> 6. $W \gets \sum_i \alpha^i * x_i^{\ell} * \pi_i$
> 7. $\textbf{return }  e(C_{batched}- [R] + W, g_2) \overset{?}{=} e(\sum_i \alpha^i * \pi_i, [\tau^{\ell}])$

### Security
Again we rewind to reduce to the security of the basic scheme.
The algebraic adversary gives us polynomial $f$, $\Pi$ which are the pre-images of respectively $C$ and $\pi$.
We get multiple $\alpha$ st. the following holds in the realm of polynomials (otherwise a d log is solved)
$f_{batched} - R = \sum_i \alpha^i (X^{\ell} - x_i^{\ell})*\Pi_i$.
Again the $R_i$ are the same because the $Y_i$ are. Same for $f$. The $X_i$ are the chosen by the verifier. Grouping the sums, we get :
$\sum_i \alpha^i (f -R_i - \Pi_i (X^{\ell} - x_i^{\ell}))=0$
We see this as a polynomial in $\alpha$, which has more roots than its degree. Therefore all his terms are zeroes. By the security of batched KZG, this is contradictory with the fact that the adversary lied.

## Batch opening

### Multi-reveals
This feature is described in the KZG extended paper under section 3.4 as batch opening https://link.springer.com/chapter/10.1007/978-3-642-17373-8_11 on arbitrary points. The paper https://github.com/khovratovich/Kate/blob/master/Kate_amortized.pdf shows how to commit and verify quickly when the points form cosets of a group of roots of unity.

For $n~|~|\mathbb{F}_r^{\times}|$, let $\omega$ be a primitive $n$-th root of unity. For $l~|~n$, let $\psi=\omega^{n/l}$ be a primitive $l$-th root of unity and $\Psi=\langle \psi \rangle$.

For $i=0,\dots, n/l - 1$, the proof of the evaluations of $f(x)$ at the $l$ points $\omega^i\Psi$ is the Kate commitment to the quotient of the euclidean division of $f(x)$ by the vanishing polynomial $x^l - \omega^{i l}$ whose only roots are $\omega^i\Psi$. In other words, given the euclidean division ${f(x)=(x^l-\omega^{il}) q_i(x)+r_{i}(x)}$, $\deg r(x) < l$, the proof is $\pi_i = [q_i(\tau)]_1$. Opening at one point corresponds to the case $l=1$ where $r_{i}(x)=f(\omega^{i})$.

To verify the proof, we gather the alleged evaluations of $f(x)$ at the points $\omega^i\Psi$. From these possibly correct evaluations, we can construct an alleged remainder $r_{i}(x)$ by computing the inverse DFT on the domain $\omega^i\Psi$, as $r_{i}(x)=f(x)$ on this domain, and as $r_{i}(x)$ is determined by its evaluations at $l$ distinct points. We then check $$ e(c-[r_i(\tau)]_1, g_2) \overset{?}{=} e(\pi, [\tau^l]_2 - [\omega^{i l}]_2). $$

### Multiple multi-reveals
We now wish to reveal not on the domain $\Omega=\langle \omega \rangle$, but on several subdomains: its $n/l>1$ cosets $\omega^i \Psi$ of $l$ elements each. The committed polynomial $f(x)$ has degree $k-1$ where $k$ corresponds to the dimension of the Reed-Solomon code. We present and slightly extend the result from https://eprint.iacr.org/2023/033.pdf, which assumes the size of the domains and of their cosets to be powers of two.

Computing the proofs for all such cosets would cost $n/l$ euclidean divisions and multi-exponentiations. Even though the euclidean division by $x^l-\omega^{il}$ is linear in the degree of the committed polynomial, as well as the multi-exponentiation thanks to the Pippenger algorithm (See https://cr.yp.to/papers/pippenger.pdf), computing all proofs leads to a complexity $\mathcal{O}(n/l \times k)$. It turns out the proofs for the cosets are related, so all proofs can be computed in time $\mathcal{O}(n/l\ \log_2 (n/l))$.

Again, for $i=0,\dots,n/l-1$, given the euclidean division $f(x)=(x^l-\omega^{il}) q\_i(x)+r\_i(x)$, $\deg r\_i(x) \leq l-1$, the proofs to be computed are $\pi_i ≔ [q_i(\tau)]_1$.


We denote $d=\deg f$, $m$ the next power of 2 of $d+1$, and set $f_m,f_{m-1},\dots,f_{d+1}=0$. For our purposes we further assume $l|m$, $l<m$.

We don't require $m$ to be a power of two. However $m$ should be of the form $2^i p$ for a small prime $p$, and $l$ should be of the form $2^j p$ for the same small prime $p$ and $j<i$. Thus $m/l$ is a power of two.

The floor designates here the truncated polynomial long division, where terms $x^i$ for $i<0$ are dropped.

Letting $\varphi≔\omega^l$ a primitive $n/l$-th root of unity:

$$
\begin{align*}
q_i(x)&=\frac{f(x)-r_i(x)}{x^l-\omega^{il}}\\
&=\Bigg\lfloor\frac{f(x)-r_i(x)}{x^l-\omega^{il}}\Bigg\rfloor\\
&=\Bigg\lfloor\frac{f(x)}{x^l-\omega^{il}}\Bigg\rfloor \text{ since } \deg r_i < l\\
&=\Bigg\lfloor\sum_{k=0}^\infty \frac{f(x)}{x^{(k+1)l}}\omega^{kil}\Bigg\rfloor\quad\text{formal power series of } 1/(x^l+c)\\
&= \sum_{k=0}^{m/l-1} \Bigg\lfloor \frac{f(x)}{x^{(k+1)l}}\Bigg\rfloor\varphi^{ik}\\
&=\begin{cases}
\sum_{k=0}^{m/l-1}  (f_m x^{m-(k+1)l} + f_{m-1}x^{m-(k+1)l-1} + \dots + f_{(k+1)l+1}x + f_{(k+1)l})\varphi^{ik} & \text{if } d\geq 2l\\[2ex]
f_m x^{m-l}+\dots+ f_{d}x^{d-l}+\dots+f_{l+1}x+f_l&\text{if } l\leq d<2l.
\end{cases}
\end{align*}
$$

There is a subtle condition which is not stated in the original paper (but is more apparent with the above derivation) which is $d\geq 2l$. Indeed, if $l\leq d<2l$, then the powers of $\varphi$ are absent of the quotient:
$$q_i(x)=f_l+f_{l+1}x+\dots+f_{d}x^{d-l}.$$

For this reason, we assume $d\geq 2l$ (thus $m>2l$).
We could support the other case ($l\leq d < 2l$) but it is a bit cumbersome, and is sort of an edge case for which there are too few shards.

Thus,

$$
    \pi_i =[q_i(x)]_1= \sum_{k=0}^{ m/l-1} (f_m[\tau^{m-(k+1)l}] + f_{m-1}[\tau^{m-(k+1)l-1}] %+ f_{m-2}[\tau^{m-kl-2}] 
    + \dots + f_{(k+1)l+1}[\tau] + f_{(k+1)l})\varphi^{ik}.
$$

Letting 
$$h_{k}≔
    \begin{cases}
            \sum_{j=kl}^{m} f_j[\tau^{j-kl}] &   \text{for } 0\leq k\leq  m/l,\\
            0 &         \text{for }  m/l<k\leq n/l
    \end{cases}
$$
we obtain $\pi_i = \sum_{k=0}^{n/l-1}  h_{k+1}\varphi^{ik}.$

So by definition $\boldsymbol{\pi}=(\pi_0,\dots, \pi_{n/l-1})$ is the $\text{EC-DFT}_{\varphi}$ of the vector $(h_1,\dots, h_{n/l})\in\mathbb{F}^{n/l}$ ($\star$).

Now, let's address the computation of the coefficients of interest $h_{k}$ for $k=1,\dots,n/l$. To this end, https://github.com/khovratovich/Kate/blob/master/Kate_amortized.pdf observe that the computation of the $h_k$'s can be decomposed into the computation of the $l$ "offset" sums:
$\forall j=0,\dots,l-1$,

$$
h_{k,j}=f_{m-j}[\tau^{m-kl-j}]+f_{m-l-j}[\tau^{m-(k+1)l-j}]%+f_{m-2l-j}[\tau^{m-(k+2)l-j}]
+\dots+f_{(m-j)\%l+kl}[\tau^{(m-j)\%l}].
$$

So the desired coefficients can then be obtained with $h_k=\sum_{j=0}^{l-1} h_{k,j}$. This decomposition of the calculation allows the $l$ vectors $(h_{1,j}, \dots, h_{\lfloor \frac{m-j}{l} \rfloor, j})$ for $j=0,\dots, l-1$ to be computed with $l$ Toeplitz matrix-vector multiplications:

$$
\begin{bmatrix}
h_{1,j} \\
h_{2,j} \\
\vdots \\
h_{\lfloor \frac{m-j}{l} \rfloor - 1, j}\\
h_{\lfloor \frac{m-j}{l}\rfloor, j}
\end{bmatrix}
=
\begin{bmatrix}
f_{m-j}     & f_{m-l-j} & f_{m-2l-j}   & \dots & f_{(m-j)\%l+l}\\
0      & f_{m-j} & f_{m-l-j}   & \dots & f_{(m-j)\%l +2l}\\
\vdots & \vdots     & \vdots & \ddots & \vdots \\
0 & 0 & 0   & \dots & f_{m-l-j} \\
0 & 0 & 0   & \dots & f_{m-j}
\end{bmatrix}
\times
\begin{bmatrix}
\tau^{m-l-j} \\
\tau^{m-2l-j} \\
\vdots \\
\tau^{(m-j)\%l+l}\\
\tau^{(m-j)\%l}
\end{bmatrix}
$$

We can extend this Toeplitz matrix to form a circulant matrix whose columns are shifted versions of the vector $\boldsymbol{c}=f_{m-j}\ \Vert\ 0^{\lfloor \frac{m-j}{l}\rfloor-1}\ \Vert\ f_{(m-j)\%l+l}\dots f_{m-j-l}$. We can then compute circulant matrix-vector multiplication with the FFT. See this presentation from Kyle Kloster, student at Purdue University: https://www.youtube.com/watch?v=w0peHpfFVpc.

The length of the following transforms is $2m/l$, which we assume is a power of two, for the reasons mentioned above. Though we could in some instances allow transforms of different size using the Prime Factor Algorithm as we currently do for FFTs operating on vectors of scalars.

 
Given the euclidean divisions $m-j = ql+r$, $0\leq r < l$ for $j=0,\dots,l-1$:
1. Compute $l$ EC-FFTs over $\mathbb{G}_1$: $\forall j=0,\dots,l-1,$
$$ \boldsymbol{s}_j=\text{EC-FFT}_{2m/l}(\tau_{m-j-l} \tau_{m-j-2l} \tau_{m-j-3l} \dots \tau_{m-j-ql=r} \ \Vert\ 0^{2m/l - \lfloor \frac{m-j}{l}\rfloor}).$$

The above calculation can be done once per trusted setup and can thus be cached.


2. Compute $l$ FFTs over $\mathbb{F}_r$: $\forall j=0, \dots, l-1$, with $f_{m}=0$:

$$\boldsymbol{f}_j = \text{FFT}_{2m/l}(f_{m-j} \ \Vert\ 0^{q +2\times pad+1 } \ \Vert\ \underbrace{f_{r+l} f_{r+2l} \dots f_{r+(q-1)l=m-j-l}}_{q-1 \ elements}  \ \Vert\ 0^{2m/l-(2q+2\times pad+1) }).$$

where $q=\lfloor \frac{m-j}{l}\rfloor$ and $pad=2^{log2up(q)}-q$.

3. Then compute $\boldsymbol{h}=(h_k)_{k\in ⟦1, n/l⟧ }$ with circulant matrix-vector multiplication via FFT:

$$
\begin{align*}
    \boldsymbol{h}&= \sum_{j=0}^{l-1} (h_{1,j} \dots h_{\lfloor \frac{m-j}{l} \rfloor, j} \ \Vert\ 0^{2m/l-\lfloor \frac{m-j}{l}\rfloor })\\
    &=\sum_{j=0}^{l-1}\text{EC-IFFT}_{2m/l}(\boldsymbol{f}_j \odot_{\mathbb{G}_1} \boldsymbol{s}_j)\\
    &=\text{EC-IFFT}_{2m/l}\Big(\sum_{j=0}^{l-1} (\boldsymbol{f}_j \odot_{\mathbb{G}_1} \boldsymbol{s}_j)\Big).
\end{align*}
$$



4. The first $n/l$ coefficients is the result of the multiplication by the Toeplitz vector (with a bit of zero padding starting from the $m/l$-th coefficient): let's call this vector $\boldsymbol{h}'$. The $n/l$ KZG proofs are given by $\boldsymbol{\pi}=\text{EC-FFT}_{n/l}(\boldsymbol{h}')$ following the observation ($\star$).

### Complexity of multiple multi-reveals

For the preprocessing part, we count $l$ EC-FFTs on $\mathbb{G}_1$, so the asymptotic complexity of the step is $O(l\times (m/l) \log (m/l))=O(m\ \log(m/l))$.

For the KZG proofs generation part, we count $l$ FFTs on $\mathbb{F}_r$ and two EC-FFTs on $\mathbb{G}_1$: the runtime complexity is $O(l\times T_{\mathbb{F}_r}(m/l) + T_{\mathbb{G}_1}(n/l)+2m\log 256)$, where $T_{\mathbb{F}_r}$ and $T_{\mathbb{G}_1}$ represent the runtime cost of the FFT and EC-FFT. Both have the same complexity, even though the latter hides a bigger constant (log of scalar size in bits, here $\log 256$) due to the elliptic curve scalar multiplication.


