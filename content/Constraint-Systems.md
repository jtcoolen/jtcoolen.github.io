---
title: Constraint systems
draft: false
tags:
  - math
  - cryptography
  - zero-knowledge proofs
  - arithmetization
---


Arithmetization is the process by which a computer program is transformed into a set of polynomial equations for use in a zero-knowledge proof system. There are four main methods:

- R1CS: quadratic rank-1 constraint system, systems of equations, at most quadratic in each variable, of the form, for $\boldsymbol{A},\boldsymbol{B},\boldsymbol{C}\in (F^n)^3,\boldsymbol{z}\in F^n$, $\cdot$ the scalar dot product: $(\boldsymbol{A} \cdot \boldsymbol{z}) \times (\boldsymbol{B} \cdot \boldsymbol{z}) - (\boldsymbol{C} \cdot \boldsymbol{z}) = \boldsymbol{0}$.
- PlonK: system of equations of the form $q_l a + q_r b + q_o c + q_m a b + q_c=0$ (models arithmetic gates for addition, multiplication, addition by a constant)
    - $q_l,q_r,q_o,q_m,q_c$ are called selectors, for the left input, the right input, the output, the multiplication, the constant term in an arithmetic gate
    - One can also extend the equations (custom gates): by adding a cubic term, using more than three wires (two inputs and one output), more than one constraint...
- [AIR (section 5.1)](https://eprint.iacr.org/2021/582.pdf): A matrix representing the evolution of variables over time (_execution trace_), where the $w$ columns correspond to registers and the rows represent the state of these registers at a given step $i$ (as field elements). There are relations linking the state at step $i$ to the state at step $i+1$, expressed through _polynomial constraints_: polynomials $\{f_j\}_j$ of a predefined degree $d$ and in $2w$ variables. The execution trace is valid if for any polynomial constraint $f_j$, and any consecutive rows $\boldsymbol{x_i}$ and $\boldsymbol{x_{i+1}}$, $f_j(\boldsymbol{x_i}, \boldsymbol{x_{i+1}})=0$.
- CCS: generalizes and captures R1CS, PlonK and AIR, allowing for efficient conversions between these constraint systems
