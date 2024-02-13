Let's fix a prime field $\mathbb{F}_p$ for prime $p$. Knowing a generator $g$ of $\mathbb{F}_p^\times$, $g^{(p-1)/k}$ is a primitive $k$-th root of unity in the field. But how to find a primitive root without a generator of $\mathbb{F}_p^\times$?

1. For $n$ and $m$ relatively prime, the product of a primitive $n$-th root of unity $\zeta_n$ and a primitive $m$-th root of unity $\zeta_m$ is a primitive $nm$-th root of unity.
    
    _Proof_: First $\zeta_n\zeta_m$ is a $nm$-th root of unity since $(\zeta_n\zeta_m)^{nm}=1$. Suppose there exists $k$ for which $(\zeta_n\zeta_m)^k=1$. Then $\zeta_m^{nk}=(\zeta_n\zeta_m)^{nk}=1$. Thus $m|nk$, but since $n$ and $m$ are coprime, $m|k$. In the same manner $n|k$. Thus $k|nm$ as $n$ and $m$ don't share any common factors. So $\zeta_n \zeta_m$ is a primitive $nm$-th root of unity.
    
2. For $n|p-1$, $\zeta\in\mathbb{F}_p^\times$ is a primitive $n$-th root of unity if and only if $\zeta^n=1$ and for all $m<n, m|n$, $\zeta^m\neq 1$.
	
	_Proof_: If $\zeta$ is a primitive $n$-th root of unity then by definition $\zeta^n=1$ and for all $m<n$, $\zeta^m\neq 1$ so in particular for all $m|n$, $\zeta^m\neq 1$. For the other implication: we know that $\zeta^n=1$ and for all $m|n$, $\zeta^m\neq 1$. In particular, $\zeta$ is a $n$-th root of unity, so its order $k$ divides $n$ by Lagrange. Since $k<n$ then $k|n/p'$ for some prime factor $p'$ of $n$. Let's call $m=n/p'|n$. Hence there exists $l$ such $kl=m$, and so $\zeta^m=(\zeta^k)^l=1$. Contradiction! Thus $\zeta$ is primitive $n$-th root of unity, ie. $\zeta$ has order $n$.
	
3. For a prime $q$ and the greatest $\alpha$ such that $q^\alpha|p-1$, sample a primitive $q^\alpha$-th root of unity: sample $x\in \mathbb{F}_p^{\times}$, if $x^{(p-1)/q}\neq 1$ then $\zeta=x^{(p-1)/q^\alpha}$ is a primitive $q^\alpha$-th root of unity in $\mathbb{F}_p$.
    
    _Proof_: We notice that $\zeta^{q^\alpha}=x^{p-1}=1$ and $\zeta^{q^{\alpha-1}}=x^{(p-1)/q}\neq 1$. Is $\zeta^{q^d}\neq 1$ for $0\leq d<\alpha-1$? Assume there exists $0\leq d<\alpha-1$ such that $\zeta^{q^d}=x^{(p-1)/q^{\alpha-d}}=1$. Then $(\zeta^{q^d })^{q^{\alpha-d-1}}=x^{(p-1)/q^{\alpha-d-(\alpha-d-1)}}=x^{(p-1)/q}=1$, a contradiction.
    
For the prime decomposition $k=\Pi_i p_i^{\alpha_i}|p-1$, a primitive $k$-th root of unity in $\mathbb{F}_p$ is obtained by multiplying each primitive $p_i^{\alpha_i}$-th root of unity found through 3. thanks to 1.