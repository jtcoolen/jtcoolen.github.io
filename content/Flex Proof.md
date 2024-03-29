---
title: Flex Proof
draft: false
tags:
  - math
  - analysis
---

In December 2020, as part of an Information Theory class, we encountered a question in a homework assignment prompting us to demonstrate that $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx = \pi /2$.
Our teacher refrained from grading it upon realizing its impossibility to solve within the recommended time. However, I gained insight into a solution by utilizing Wolfram Alpha. Although I cannot recall the exact method employed to gain the insight (probably through the iterative examination of function graphs), the proof unfolded as follows:

The value of the integral $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx$ can be obtained by finding a function whose Fourier transform matches, up to a multiplicative constant, the function $x\mapsto \frac{1-\cos x}{x}$. The key insight lies in considering a piecewise-defined function $f: \mathbb{R}\to\mathbb{C}$, where $f=1$ over  $(0, \frac{1}{2\pi}]$, $f=-1$ over $[-\frac{1}{2\pi}, 0)$, and $f=0$ elsewhere. Indeed:
$\mathcal{F}f(y)=1/(2\pi)\int_{-1/(2\pi)}^{0} -e^{-i y x}dx+1/(2\pi)\int_{0}^{1/(2\pi)} e^{-i y x}dx=-i(1-\cos y)/(2\pi y) -i(1-\cos y)/(2\pi y)=-i/\pi (1-\cos y)/y$. (Skipping how the integral of $x\mapsto e^{-i y x}$ is computed for brevity here).

From this, we derive:

$$
\begin{align*}
\forall x\in\mathbb{R}, \int_{-\infty}^{\infty}\Big(\frac{1-\cos x}{x}\Big)^2 dx &= \int_{-\infty}^{\infty} \Big|-\mathcal{F}f(x)\frac{\pi}{i}\Big|^2 dx \\
&= \pi^2 \int_{-\infty}^{\infty} |\mathcal{F}f(x)|^2 dx \\
&= \pi^2 \int_{-\infty}^{\infty} |f(x)|^2 dx \text{ (by Plancherel)} \\
&= \pi^2 \int_{-1/(2\pi)}^{1/(2\pi)} 1 dx \\
&= \pi.
\end{align*}
$$

Therefore, by exploiting the evenness of the cosine function, we conclude that: $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx = \pi /2$. 💪
