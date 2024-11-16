---
title: Reed-Solomon Codes
draft: false
tags:
  - math
  - coding theory
---


### MDS codes

An MDS (Maximum Distance Separable) code is a linear code of dimension $k$ and length $n$ over a finite field $\mathbb{F}$ (a vector subspace of dimension $k$ of $\mathbb{F}^n$), which reaches the Singleton bound (the minimal Hamming distance between any two codewords is $d=n-k+1$), and so provides the maximum erasure-correcting capability possible (as a linear code cannot verify $d > n - k + 1$, again by the Singleton bound). So we can correct up to $d-1=n-k$ erasures by finding the closest codeword under the Hamming distance. A generating matrix $G\in\mathbb{F}^{n\times k}$ of a linear code $\mathcal{C}$ is defined by $\mathcal{C}=\text{Column-span}({G})=\{ {G}\boldsymbol{x}^T : \boldsymbol{x}\in \mathbb{F}^k \}$. The property that any $k$ coefficients $\boldsymbol{\tilde{c}}$ of a codeword  $\boldsymbol{c}$ determine it, comes from the fact that any set of $k$ rows of its generating matrix forms a full rank $k\times k$ matrix $A$ (the equation $A\boldsymbol{c}^T=\boldsymbol{\tilde{c}}^T$ where $\boldsymbol{c}$ is the unknown has a unique solution, as $A$ is invertible).

### Reed-Solomon codes
Let $\mathbb{F}$ be a prime field of order $r$. Let $n=\alpha k$ such that $n | r - 1$. Let $\omega\in\mathbb{F}$ be a [primitive $n$-th root of unity](https://mathworld.wolfram.com/PrimitiveRootofUnity.html). Aside: [how do we find such primitive roots of unity in practice](Finding-a-primitive-root-in-a-prime-field)?

We consider the Reed-Solomon code of parameters $[n,k,d=n-k+1]$ and evaluation points, $\boldsymbol{\omega}=(1,\omega,\omega^2,\dots,\omega^{n-1})$ the subgroup of the $n$-th roots of unity. We denote it $\text{RS}(n,k,\boldsymbol{\omega})≔ \{ (f(\omega^i))_{i\in⟦ 0,n-1 ⟧} | f\in\mathbb{F}[x] \wedge \deg f<k \}$.

RS has rate $R=k/n=1/\alpha$. To a vector $\boldsymbol{f}$ we associate the polynomial $f(x)=\sum_i f_i x^i$, and reciprocally. RS is a linear code, whose generating matrix is the following Vandermonde matrix, for which every square submatrix is invertible, so RS is indeed MDS:
$$
    \begin{bmatrix} 
    1 & \omega_1 & \omega_1^2 & \dots & \omega_1^{k-1}\\
    1 & \omega_2 & \omega_2^2 & \dots & \omega_2^{k-1}\\
    \vdots & & & \ddots & \vdots\\
    1 & \omega_n & \omega_n^2 & \dots  & \omega_n^{k-1} 
    \end{bmatrix}.
$$
### Encoding
Encoding a message $\boldsymbol{m}=(m_0,\dots,m_{k-1})\in\mathbb{F}^k$ amounts to evaluate its associated polynomial $m(x)=\sum_{i=0}^{k-1} m_i x^i$ at the evaluation points $\boldsymbol{\omega}$. This can be done with an $n$-points [discrete Fourier transform](Fast-Fourier-Transform#discrete-fourier-transform-dft) supported by $\mathbb{F}$ in time $\mathcal{O}(n\ \log\ n)$:

Input: $\boldsymbol{m}=(m_0,\dots,m_{k-1})\in\mathbb{F}^k$

Output: $\boldsymbol{c}=(c_0,\dots,c_{n-1})\in \text{RS}(n,k,\boldsymbol{\omega})$

Return $\text{FFT}_{n} (\text{IFFT}_{k}(\boldsymbol{m}) \mathbin\Vert \boldsymbol{0}_{\mathbb{F}^{n-k}})$

### Decoding from erasures

As we saw earlier, we can decode a codeword $\boldsymbol{c}\in \text{RS}(n,k,\boldsymbol{\omega})$ with at most $d-1=n-k$ erasures, i.e. from at least $k$ components of $\boldsymbol{c}$.

Without loss of generality, let $\tilde{\boldsymbol{c}}=({c}_0,\dots,{c}_{k-1})$ be the received codeword with erasures, the first $k$ components of a codeword. We can retrieve the original message $\boldsymbol{m}$ with any $k$ components of $\boldsymbol{c}$ thanks to the Lagrange interpolation polynomial, where the $x_i$ are the evaluation points of $\tilde{\boldsymbol{c}}$

$$m(x)=\sum_{i=0}^{k-1} {c}_i  \prod_{\substack{j=0 \\ j\neq i}}^{k-1} \frac{x-x_j}{x_i - x_j}.$$

As detailed in https://arxiv.org/pdf/0907.1788v1.pdf, the idea is to rewrite $m(x)$ as a product of two polynomials $A(x)$ and $B(x)$ so that the convolution theorem allows us to recover $m(x)$ using [FFTs](Fast-Fourier-Transform) in ${O}(n\ \log\ n)$. (The authors consider sums to $n-1$ while we can consider sums to $k-1$ since $m(x)$ has degree $k$.)

To do so, let
$$A(x) ≔ \prod_{i=0}^{k-1} (x-x_i), \quad A_i(x) ≔\prod_{\substack{j=0\\ j\neq i}}^{k-1} (x-x_j).$$

Let $n_i ≔ \dfrac{{c}_i}{A_i(x_i)}$. 

The interpolation polynomial becomes:
$$m(x)=A(x) \sum_{i=0}^{k-1}\frac{ {c}_i }{(x - x_i) A_i(x_i)} = A(x)\sum_{i=0}^{k-1} \frac{n_i}{x-x_i}.$$

Note that $A_i(x_i)\neq 0$ by definition, so it is invertible in $\mathbb{F}$.

In order to replace the costly product $A_i(x)$ in this expression, we use the fact that the formal derivative $A'(x)$ of $A(x)$ satisfies for all $i\in⟦ 0, k-1 ⟧$: $A'(x_i)=A_i(x_i)$.
So we can compute $(A_i(x_i))_i$ by evaluating $A'(x)$ at the points $\boldsymbol{\omega}$ with an FFT.

Indeed:
$$ A'(x)= (\prod_{i=0}^{k-1} (x-x_i))' = \sum_{i=0}^{k-1} (x-x_i)' \prod_{\substack{j=0\\ j\neq i}}^{k-1} (x-x_j)=\sum_{j=0}^{k-1}  A_j(x).$$
So $A'(x_i)=\sum_{j=0}^{k-1} A_j(x_i)=A_i(x_i)$ as the other polynomials $A_j(x)$ have $x_i$ as root.

Writing the fraction $\frac{1}{x_i-x}=\sum_{j=0}^{\infty} \frac{x^j}{x_i^{j+1}}$ as a formal power series, we obtain
$$
\begin{align*}
m(x)/A(x)=\sum_{i=0}^{k-1} \frac{n_i}{x-x_i} \mod x^k &= -\sum_{i=0}^{k-1} \Big(\sum_{j=0}^{k-1} \frac{n_i}{x_i^{j+1}} x^j\Big).
\end{align*}
$$

But if we let $N(x)≔\sum_{i=0}^{k-1} \dfrac{n_i}{x_i} x^{i}$ then
$$ \sum_{i=0}^{k-1} \frac{n_i}{x-x_i} \mod x^k = - \sum_{j=0}^{k-1} N(x_i^{-j})x^j ≕ -B(x). $$

$B(x)$ is thus given by the first $k$ components of $n\times \text{IFFT}_n(\boldsymbol{N})$.


So the product is given by the convolution theorem, and by linearity of the DFT and pairwise product:
$$
\begin{align*}
\boldsymbol{m} &= \boldsymbol{A} * (-\boldsymbol{B}) =  -\text{IFFT}_{2k}(\text{FFT}_{2k}(\boldsymbol{A}) \odot \text{FFT}_{2k}(\boldsymbol{B})).
\end{align*}
$$
 The total cost is $O(k\ \log^2\ k + n\ \log\ n)$: the first term accounts for the computation of the product for $A(x)$ with a divide and conquer approach for the multiplication of its factors with FFT multiplication, and the second one for the other steps.
 
### Sharding
  

In some applications, such as [database sharding](https://aws.amazon.com/what-is/database-sharding/), it can be advantageous to split the codewords into chunks (aka shards).
For this purpose, let $s$ be the number of shards, $l=n/s$ the length of a shard, and $\omega$ a primitive $n$-th root of unity.

The domain of evaluation is then split into cosets: $\langle \omega \rangle=\bigsqcup_{i\in⟦ 0, s-1 ⟧ }  \Omega_i$, for $\Omega_0 = \{\omega^{s j}\}_{j\in⟦ 0,\ l-1 ⟧}$ and $\Omega_i = \omega^i \Omega_0$.

For a set of $k/s$ shard indices $Z\subseteq \{0, s-1\}$, we reorganize the product $A(x)=\prod_{i=0}^{k-1} (x-x_i)$ into $$A(x)=\prod_{\substack{i\in Z\\ |Z|=\frac{k}{s}}}  \underbrace{\prod_{\omega'\in\Omega_{i}} (x-\omega')}_{Z_i}.$$
We notice that $Z_0(x)=x^{|\Omega_0|}-1$ (as its roots are the elements of a group of order dividing $|\Omega_0|$) entails $Z_i(x)=x^{|\Omega_0|}-\omega^{i |\Omega_0|}$ (multiplying all terms by a constant $\omega^{i}$ in an integral domain), which is as sparse as it can be. More formally: every element of $\Omega_i$ is of the form $\omega^i \omega^{s j}$ for $j\in ⟦ 0,\ l-1 ⟧$. Thus
$$Z_i(\omega^i \omega^{s j})=(\omega^i \omega^{s j})^{|\Omega_0|}-\omega^{i|\Omega_0|}=(\omega^i)^{|\Omega_0|} (\omega^{s j})^{|\Omega_0|}-\omega^{i|\Omega_0|}=\omega^{i|\Omega_0|} 1-\omega^{i|\Omega_0|}=0.$$
So every element of $\Omega_i$ is a root of $Z_i(x)$. Moreover, $Z_i(x)$ is a degree $|\Omega_0|=l$ polynomial so has at most $l$ roots: $Z_i(x)$'s only roots are $\Omega_i$.


With this little observation, we've reduced the number of leaves of the recursion tree for the divide-and-conquer multiplication of the factors of $A(x)$ from $k$ to $k/s$.


