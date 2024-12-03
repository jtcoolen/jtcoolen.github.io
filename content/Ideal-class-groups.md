---
title: Ideal Class Groups
draft: false
tags:
  - math
  - cryptography
  - ideal class groups
---
 
## What are Ideal Class Groups?

- $K=Q(\sqrt\Delta)$ where $\Delta$ is a squarefree integer (adjoin)
- an order is an $n$-dimensional lattice in $K$: $O=Za_1+Za_2+\dots+Za_n$ where $\{a_1,\dots,a_n\}$ is a basis of $K$, the maximal order of K is $O_K$ and plays the role of integers (generalizes integers from $Q$ to $Q(\alpha)$) in $Q(\alpha)$.
- $O$ an order of $K$. $I_O$ the set of invertible fractional ideals of $O$, $P_O$ the set of fractional principal invertible ideals of $O$, then $Cl_O=I_O / P_O$. Outs the same class elements differing from a principal fractional ideal factor $(\alpha)$, $\alpha\in K$. It's a finite group by Minkowski bound
- to each discriminant $\Delta$ is attached a finite abelian group, the ideal class group denoted $Cl(\Delta)$, the quotient of the group sof (invertible fractional) ideals of $O_\Delta$ by the subgroup of principal ideals.
- order of the class group is called the class number $h(\Delta)$

## of Imaginary Quadratic Fields

- Imaginary quadratic fields are finite extensions of the field of the rationals, of degree 2 (vector space)
- bit-size of the discriminant determines the hardness of the discret elog
- $h(\Delta)$ is in general close to $\sqrt\Delta$ so that one can compute its bit size using th analytic class number formula (McCurley 89) in polynomial time
- $h(\Delta)$ can be computed from $\Delta$ in subexponential time $L_{1/2}(|\Delta|)$
- no trusted setup needed contrary to RSA when the prime factorisation of the modulus of the RSA group needs to be unknown.
- Dlog hard to compute in $Cl(\Delta)$ with complexity $L_{1/2}(|\Delta|)$
- system of representatives of the classes with notion of reduced ideals, equivalence relation on froms from the action of SL2(Z)
- form $(a,b,c)$ corresponding to $f(X,Y)=aX^2+bXY+c Y^2$ of discriminant $\Delta$
- ideals of the form $aZ+\frac{-b+\sqrt\Delta}{2}Z$ where $a,b\in N$, and smaller than $\sqrt\Delta$ when the ideal is reduced, if not, one has with a bit of algebra that $|b|\leq |a| \leq \sqrt{\Delta}/3$ and $|c|\leq |\Delta|$ where $\Delta$ is the discriminant
- explicit correspondence between ideals and forms $(a,b,c) \Longleftrightarrow aZ+\frac{-b+\sqrt{\Delta}}{2}Z$
- form is reduced if $-a<b\leq a$ and $a\leq c$ or if $a=c$ then $b\geq 0$
- see https://hackmd.io/@corneliuhoffman/Ideal_class_groups?utm_source=preview-mode&utm_medium=rec for group laws formulas using Gauss composition of forms and reduction algo for forms from Lagrange
- more efficient algos from Shanks: NUDUPL and NUCOMP https://www.ams.org/journals/mcom/2003-72-244/S0025-5718-03-01518-7/S0025-5718-03-01518-7.pdf


