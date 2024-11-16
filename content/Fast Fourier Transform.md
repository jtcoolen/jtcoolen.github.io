---
title: Fast Fourier Transform
draft: false
tags:
  - math
  - analysis
  - algebra
---


## Discrete Fourier Transform (DFT)

Let $\mathbb{F}$ be a field and $\omega\in\mathbb{F}^\times$ a [primitive $n$-th root of unity](https://en.wikipedia.org/wiki/Root_of_unity#General_definition) (so by definition the natural number $n$ divides the order of the multiplicative group $\mathbb{F}^\times$).
The DFT matrix is defined by:
$$
F_\omega=\begin{bmatrix} 
    1 & 1      & 1        & \dots& 1\\
    1 & \omega & \omega^2 & \dots & \omega^{n-1}\\
    1 & \omega^2 & \omega^4 & \dots & \omega^{2(n-1)}\\
    \vdots &\vdots &\vdots & \ddots & \vdots\\
    1 & \omega^{n-1} & \omega^{2(n-1)} & \dots  & \omega^{(n-1)^2} 
    \end{bmatrix}
$$

and the inverse DFT matrix is given by $F_{\omega^{-1}}$.

The discrete Fourier transform of length $n$ is the $\mathbb{F}$-linear map:
$$
\begin{align*}
    \text{DFT}_{\omega}: \mathbb{F}^n &\to \mathbb{F}^n\\ \boldsymbol{x} &\mapsto \boldsymbol{y}= F_{\omega} \boldsymbol{x}= \big(\sum_{i=0}^{n-1} x_i \cdot\omega^{i j}\big)_{j\in ⟦0,n-1⟧}
\end{align*}
$$
which evaluates a polynomial from $\mathbb{F}[x]$ of degree strictly less than $n$ at the distinct points $1, \omega, \dots, \omega^{n-1}$. The group elements $1, \omega, \dots, \omega^{n-1}$ being distinct for the following reason: let $x\in\mathbb{F}^\times$ a group element of order $n$; for any $0<m < n, x^m \neq 1$, and $x^n=1$. If $x^j = x^k$ for $1 \leq j < k \leq n$, then $x^{k - j}= 1$ with $0 < k - j < n$, a contradiction.

For any $0\leq i,j<n$:
$$
\begin{align*}
    (F_\omega F_{\omega^{-1}})_{i,j} &= \sum_{k=0}^{n-1} \omega^{jk} (\omega^{-1})^{ik}\\
    &=  \sum_{k=0}^{n-1} \omega^{(j-i)k}\\
    &= 0 \text{ if } i\neq j \text{ or } n \text{ if } i=j.
\end{align*}
$$
As in any field, $d\in\mathbb{N}\setminus \{ 0\}$, for a primitive $n$-th root of unity $\omega$, $1 + \omega^d +\dots+ \omega^{d(n-1)} = (1-(\omega^n)^d)/(1-\omega^d) = 0$ since $\omega^n=1$, and $\omega^{-1}$ is also a primitive $n$-th root of unity.

Thus $F_\omega F_{\omega^{-1}}=n\times \text{Id}_n$, which in turn implies $(\text{DFT}_\omega)^{-1} = n^{-1} \text{DFT}_{\omega^{-1}}$.
The inverse discrete Fourier transform is thus defined if $n$ is invertible in $\mathbb{F}$ (the characteristic of $\mathbb{F}$ is strictly greater than $n$).

When $\mathbb{F}$ is a finite field this transform is also called Number Theoretic Transform (NTT).

Note that we can consider a variant of the DFT where the vector holds points of an elliptic curve group defined over a finite field $E(\mathbb{F}_p)$, which we call EC-DFT. The addition is then the elliptic curve point addition and the multiplication with roots of unity is the elliptic curve scalar multiplication (the roots of unity being defined over a prime field can be mapped to scalars).

## Convolution product with the DFT

One can check that the $n$-DFT is an homomorphism from $(\mathbb{F}^n,*)$ to $(\mathbb{F}^n,\odot)$ where $*$ is the convolution product [^1]
and $\odot$ is the element-wise multiplication:
$$\text{DFT}_{\omega}(A*B) = \text{DFT}_\omega(A)\odot\text{DFT}_\omega(B)$$
from which
$$A*B=\text{DFT}^{-1}_{\omega}(\text{DFT}_\omega(A)\odot\text{DFT}_\omega(B))$$ follows.



[^1]: $(f\times g)(x) = \sum_k h_k$ where $h_k=∑_j a_{k-j} b_j$.


Aside:

One way to see it is to remark that the DFT for field elements is a ring isomorphism (with inverse DFT<sup>-1</sup>) given by the Chinese Remainder Theorem (CRT)

$$\mathbb{F}^n\cong \mathbb{F}[x]/\langle x^n-1\rangle \cong \Pi_{0\leq i < n}\mathbb{F}[x]/\langle x-\omega^i\rangle.$$

The first isomorphism corresponds to the fact that $\mathbb{F}[x]/\langle x^n-1\rangle$ is a ring (of polynomials in $\mathbb{F}[x]$ taken modulo $x^n-1$) and an $\mathbb{F}$-vector space of dimension $n$. The second one is the CRT factorization of $\mathbb{F}[x]/\langle x^n-1\rangle$ as the ideals $\langle x-\omega^i\rangle$ are pairwise coprime so their intersection is their product $\langle x^n-1\rangle$.
Indeed, the DFT maps $f\in\mathbb{F}[x]_{< n}$ to $(f(\omega^i)=f(x) \mod (x-\omega^i))_{i\in ⟦0,n-1⟧}$. The inverse map comes down to the [Lagrange interpolation](https://en.wikipedia.org/wiki/Chinese_remainder_theorem#Lagrange_interpolation) (based on partial fraction decomposition) or [Bézout](https://en.wikipedia.org/wiki/B%C3%A9zout%27s_identity#For_polynomials) (based on Bézout's identity, whose coefficients can be computed with the extended Euclidean algorithm).


## The Fast Fourier Transform (FFT)

The FFT from Cooley and Tukey (and Gauss) splits the computation of a DFT of size $n=n_1 n_2$ into the computations of $n_2$ DFTs of size $n_1$ (inner sum) and $n_1$ DFTs of size $n_2$ (outer sum), for $0\leq k < n_1$ and $0\leq l < n_2$:

$$
\begin{equation*}
    P(\omega^{k+n_1 l}) = \sum_{i=0}^{n_2-1}\Bigg(\sum_{j=0}^{n_1-1} P_{i+n_2 j} (\omega^{n_2})^{jk}\Bigg) (\omega^{n_1})^{i l} \omega^{i k}.
\end{equation*}
$$


What follows is a reproduction of the derivation of the above FFT expression from the DFT from the section 2.3 of [An Approach to Low-power, High-performance, Fast Fourier Transform Processor Design, Bevan M. Baas, 1999](https://redirect.cs.umbc.edu/~tinoosh/cmpe691/docs/phd-thesis-FFT.pdf). The input vector $\boldsymbol{x}$ and output vector $\boldsymbol{y}$ are reshaped to form $n_1\times n_2$ matrices $X$ and $Y$ as follows: for integers $A,B,C,D$, $0\leq i,k < n_1$, $0\leq j,l < n_2$, $X_{i,j}=x_{A i + B j \mod n}$ and $Y_{k,l}=y_{C k + D l \mod n}=P(\omega^{C k + D l})$.




Plugging this reindexing into the DFT equation we obtain (where $\omega_k=\omega^{n/k}$, and is a primitive $k$-th root of unity):
$$
y_j = \sum_{i=0}^{n-1} x_i\ \omega_n^{ij}
\implies
Y_{k,l} = \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} X_{i,j}\ \omega_n^{(Ai+Bj)(Ck+Dl)}.
$$

With a column-wise reshape of $\boldsymbol{x}$ (ie. $A=n_2, B=1$) and row-wise reshape of $\boldsymbol{y}$ (ie. $C=1, D=n_1$), we find the Cooley-Tuckey FFT:

$$
\begin{align*}
Y_{k,l} &= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} X_{i,j}\ \omega_n^{(n_2 i+j)(k+n_1 l)}\\
&= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} X_{i,j}\ \omega_n^{n_2 i k +  jk +j n_1 l}\\
&= \sum_{i=0}^{n_1-1} \Bigg( \Bigg [\sum_{j=0}^{n_2-1} X_{i,j} \  \omega_{n_2}^{jl} \Bigg] \omega_n^{jk}\Bigg)\omega_{n_1}^{i k}\quad (*)\\
&= \sum_{j=0}^{n_2-1} \Bigg (\sum_{i=0}^{n_1-1} X_{j,i} \  \omega_{n_1}^{ik} \Bigg) \omega_{n_2}^{jl}\omega_n^{j k}.
\end{align*}
$$

### Radix-2 decimation in time vs. decimation in frequency


The terminology comes from the Digital Signal Processing community where the DFT maps a signal (DFT input) to its frequencies (DFT output).

The radices are the prime factors from the decomposition of the DFT domain size $n$. Typically $n=2^k$ so the FFT is radix-2. We consider $n=2^k$.

#### Decimation in **time** (DIT)
From $(*)$, choosing $n_1=n/2$, $n_2=2$: the **input sequence of the DFT $\boldsymbol{x}$ is decomposed** into the even- and odd-indexed subsequences $X_{i,0}=x_{2i}$ and $X_{i,1}=x_{2i+1}$:
$$
Y_{k,l}=P(\omega^{k+(n/2)l})=\sum_{i=0}^{n/2-1} x_{2i}\omega_{n/2}^{ik} + \omega_n^{(n/2)l+k}\sum_{i=0}^{n/2-1} x_{2i+1}\omega_{n/2}^{ik}
$$
which amounts to the "textbook FFT formula", with a change of variable, and for $0\leq k<n$:
$$
P(\omega^{k})=\sum_{i=0}^{n/2-1} x_{2i}\omega_{n/2}^{ik} + \omega_n^{k}\sum_{i=0}^{n/2-1} x_{2i+1}\omega_{n/2}^{ik}.
$$
Thus we obtain a recursive definition of the DFT: $\text{DFT}_{\omega_n}(\{x_k\})=\text{DFT}_{\omega_{n/2}}(\{x_{2i}\})+\omega_n^k\text{DFT}_{\omega_{n/2}}(\{x_{2i+1}\})$.

<!-- An interesting observation due to Bernstein is that the above expression is a transformation of the form $\boldsymbol{F}[x]/\langle x^n - 1 \rangle \to \boldsymbol{F}[x]/\langle x^{n/2} - 1 \rangle\times \boldsymbol{F}[x]/\langle x^{n/2} + 1 \rangle\cong \boldsymbol{F}[x]/\langle x^{n/2} - 1 \rangle\times \boldsymbol{F}[x]/\langle \tilde{x}^{n/2} - 1 \rangle$ with $\tilde{x}=\omega_n x$.-->


#### Decimation in **frequency** (DIF)
From $(*)$, choosing $n_1=2$, $n_2=n/2$: the **output sequence of the DFT $\boldsymbol{y}$ is decomposed** into the even- and odd-indexed subsequences $Y_{0,l}=P(\omega^{2l})$ and $Y_{1,l}=P(\omega^{2l+1})$:
$$
\begin{align*}
Y_{k,l}=P(\omega^{k+2l})&=\sum_{i=0}^{n/2-1} x_{i}\omega_{n}^{i(2l+k)} + \sum_{i=0}^{n/2-1} x_{n/2+i}\omega_{n}^{(n/2+i)(2l+k)}\\
&=\sum_{i=0}^{n/2-1} (x_{i} +(-1)^k x_{n/2+i})\omega_{n}^{i(2l+k)}\\
&=\sum_{i=0}^{n/2-1} (x_{i} +(-1)^k x_{n/2+i})\omega_{n/2}^{il}\omega_{n}^{ik}.\end{align*}
$$
Hence the recursive definition of the DFT: $\{ y_{2i} \}=\text{DFT}_{\omega_{n/2}}(\{x_i+x_{n/2+i}\}_{i\in ⟦0,n/2-1⟧})$ and $\{ y_{2i+1} \}=\text{DFT}_{\omega_{n/2}}(\{(x_i-x_{n/2+i})\omega_n^i \}_{i\in ⟦0,n/2-1⟧})$.


There is a nice symmetry: in the DIT, the input sequence decomposed in even- and odd- indexed subsequences, and the output sequence is decomposed in top and bottom subsequences; and vice-versa for the DIF.


### Prime factor algorithm (FFT variant)

This algorithm allows computing FFTs on domains of size $n$ whose factorization is included in the factorization of $|\mathbb{F}_r^{\times}|=r-1$. Let $\omega$ be a primitive $n$-th root of unity.

[When $n_1$ and $n_2$ are coprime then the Chinese Remainder Theorem (CRT) allows to re-index the DFT of size $n$ in such a way the inner DFTs don't have to be multiplied by the $n$ extra factors $\omega^{iu}$](https://www.researchgate.net/publication/3316018_Index_mappings_for_the_fast_Fourier_transform).

We start from the expression of the DFT for $k=0,\dots, n-1$:
$$P(\omega^k) = \sum_{l=0}^{n-1} P_l \omega^{l k}.$$
We re-index both input and output coefficients $l$ and $k$ thanks to the ring isomorphism $\mathbb{Z}_n ≅ \mathbb{Z}_{n_1} \times \mathbb{Z}_{n_2}$ given by the CRT. It turns out that certain combinations of input and output mappings allow getting rid of the extra factors $\omega^{iu}$. Let's take as input mapping the CRT forward mapping $\mathbb{Z}_n \xrightarrow{} \mathbb{Z}_{n_1} \times \mathbb{Z}_{n_2} : l \mapsto (l \mod n_1, l \mod n_2)$. Hence, by Bézout for $i=l\mod n_1$, $j=l\mod n_2$, $t_1 = (1/n_1 \mod n_2)$ and $t_2=(1/n_2 \mod n_1)$, we obtain $P_l=P_{i n_2 t_2 + j n_1 t_1}$. As output mapping, we take the Good's mapping $\mathbb{Z}_{n_1} \times \mathbb{Z}_{n_2} \xrightarrow{} \mathbb{Z}_n : (i, j)\mapsto j n_1+i n_2 \mod n$. We could also have permuted the input and output mappings. Applying the two mappings, we get

$$
\begin{align*}
  P(\omega^{ k n_1 + l n_2 }) &= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} P_{i n_2 t_2 + j n_1 t_1} \omega^{(i n_2 t_2 + j n_1 t_1)(l n_2 + k n_1)}  \\
  &= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} P_{i n_2 t_2 + j n_1 t_1} \omega^{i n_2 t_2 l n_2} \omega^{i n_2 t_2 k n_1} \omega^{j n_1 t_1 l n_2} \omega^{j n_1 t_1 k n_1} \\
  &= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} P_{i n_2 t_2 + j n_1 t_1} \omega^{i n_2 t_2 l n_2}\omega^{j n_1 t_1 k n_1}  \\
  &= \sum_{i=0}^{n_1-1} \sum_{j=0}^{n_2-1} P_{i n_2 t_2 + j n_1 t_1} \omega_{n_1}^{i t_2 l n_2}\omega_{n_2}^{j t_1 k n_1}  \\
  &= \sum_{i=0}^{n_1-1} \Big(\sum_{j=0}^{n_2-1} P_{i n_2 t_2 + j n_1 t_1}\omega_{n_2}^{jk} \Big) \omega_{n_1}^{i l}.
\end{align*}
$$


So in the same way as implied by the equation $(*)$, we compute $n_1$ FFTs of length $n_2$, transpose the $n_1\times n_2$ matrix, and compute $n_2$ FFTs of length $n_1$. The cost of the transposition is somewhat negligible compared to the FFTs since it only changes the data layout. 


Complexity: assuming $n_2=2^k$ and $n_1=p$ for small prime $p$: $O(2^k(p\times k+ p^2))$.



