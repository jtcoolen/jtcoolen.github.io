In December 2020, as part of an Information Theory class, we encountered a question in a homework assignment prompting us to demonstrate that $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx = \pi /2$.
Our teacher refrained from grading it upon realizing its impossibility to solve within the recommended time. However, I gained insight into a solution by utilizing Wolfram Alpha. Although I cannot recall the exact method employed (probably through the iterative examination of function graphs), the proof unfolded as follows:

The value of the integral  $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx$ can be obtained by finding a function whose Fourier transform closely matches, up to a multiplicative constant, the function $x\mapsto \frac{1-\cos x}{x}$. This function is defined piecewise as $\tilde{f}$, where $\tilde{f}=1$ over  $(0, \frac{1}{2\pi}]$, $\tilde{f}=-1$ over $[-\frac{1}{2\pi}, 0)$, and $\tilde{f}=0$ elsewhere (that's the insight!). From this, we derive:

$$
\begin{align}
\forall x\in\mathbb{R}, \int_{-\infty}^{\infty}\Big(\frac{1-\cos x}{x}\Big)^2 dx &= \int_{-\infty}^{\infty} \Big|-\mathcal{F}\tilde{f}(x)\frac{\pi}{i}\Big|^2 dx \\
&= \pi^2 \int_{-\infty}^{\infty} |\mathcal{F}\tilde{f}(x)|^2 dx \\
&= \pi^2 \int_{-\infty}^{\infty} |\tilde{f}(x)|^2 dx \text{ (by Plancherel)} \\
&= \pi^2 \int_{-1/(2\pi)}^{1/(2\pi)} 1 dx \\
&= \pi.
\end{align}
$$

Therefore, by exploiting the evenness of the cosine function, we conclude that: $\int_0^{\infty}(\frac{1-\cos x}{x})^2 dx = \pi /2$. 💪