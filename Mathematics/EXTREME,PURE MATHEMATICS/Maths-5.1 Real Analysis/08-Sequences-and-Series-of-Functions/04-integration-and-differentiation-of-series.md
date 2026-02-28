# Chapter 4: Integration and Differentiation of Series

[← Previous: Weierstrass M-Test](03-weierstrass-m-test.md) | [Back to README](../README.md) | [Next: Power Series →](05-power-series.md)

---

## Overview

When can we integrate or differentiate a series **term by term**? Uniform convergence provides the answer: we can always integrate uniformly convergent series term by term, but differentiating requires the **derivatives** to converge uniformly.

---

## 4.1 Term-by-Term Integration

**Theorem 4.1.** If $g_n \in \mathcal{R}[a,b]$ and $\sum g_n$ converges uniformly to $S$ on $[a,b]$, then $S \in \mathcal{R}[a,b]$ and:

$$\int_a^b \sum_{n=1}^\infty g_n(x)\,dx = \sum_{n=1}^\infty \int_a^b g_n(x)\,dx$$

**Proof.** Let $S_N = \sum_{n=1}^N g_n$. By the uniform limit theorem for integrals:

$$\int_a^b S = \int_a^b \lim_{N\to\infty} S_N = \lim_{N\to\infty}\int_a^b S_N = \lim_{N\to\infty}\sum_{n=1}^N \int_a^b g_n = \sum_{n=1}^\infty \int_a^b g_n \quad \blacksquare$$

```
  Term-by-term integration:

  ∫ₐᵇ [g₁ + g₂ + g₃ + ···] dx
    = ∫ₐᵇ g₁ dx + ∫ₐᵇ g₂ dx + ∫ₐᵇ g₃ dx + ···

  Valid when: ∑ gₙ converges UNIFORMLY on [a,b]
```

**Example.** $\int_0^1 \frac{1}{1+x^2}\,dx = \int_0^1 \sum_{n=0}^\infty (-1)^n x^{2n}\,dx$

The series converges uniformly on $[0, 1]$ (by M-test with $M_n = 1$... but actually we need $|x| < 1$; convergence at $x = 1$ is by Abel's theorem). Integrating term by term:

$$= \sum_{n=0}^\infty (-1)^n \frac{1}{2n+1} = 1 - \frac{1}{3} + \frac{1}{5} - \cdots = \frac{\pi}{4}$$

This gives the **Leibniz formula** $\pi/4 = 1 - 1/3 + 1/5 - \cdots$.

---

## 4.2 Term-by-Term Differentiation

**Theorem 4.2.** Suppose $g_n$ differentiable on $[a,b]$ and:
1. $\sum g_n(x_0)$ converges for some $x_0 \in [a,b]$
2. $\sum g_n'$ converges **uniformly** on $[a,b]$

Then $\sum g_n$ converges uniformly to some $S$ on $[a,b]$, $S$ is differentiable, and:

$$\left(\sum_{n=1}^\infty g_n(x)\right)' = \sum_{n=1}^\infty g_n'(x)$$

**Key point:** The uniform convergence of the **derivatives** is required, not of the original series.

```
  Term-by-term differentiation:

  d/dx [g₁ + g₂ + g₃ + ···] = g₁' + g₂' + g₃' + ···

  Requires:  (1) ∑ gₙ' converges UNIFORMLY
             (2) ∑ gₙ(x₀) converges at one point
```

---

## 4.3 Comparison Table

| Operation | Hypothesis on $\sum g_n$ | Hypothesis on $\sum g_n'$ | Conclusion |
|-----------|------------------------|--------------------------|------------|
| Integration | Uniform convergence | — | $\int \sum = \sum \int$ |
| Differentiation | One-point convergence | **Uniform convergence** | $(\sum)' = \sum'$ |

---

## 4.4 Application: Defining Functions via Series

**The exponential:** $e^x = \sum_{n=0}^\infty \frac{x^n}{n!}$

- Converges uniformly on $[-R, R]$ by M-test.
- Differentiate term by term: $(e^x)' = \sum \frac{nx^{n-1}}{n!} = \sum \frac{x^{n-1}}{(n-1)!} = e^x$.
- Integrate term by term: $\int_0^x e^t\,dt = \sum \frac{x^{n+1}}{(n+1)!} = e^x - 1$.

**Fourier-like series:** $f(x) = \sum_{n=1}^\infty \frac{\sin(nx)}{n^2}$

- Converges uniformly on $\mathbb{R}$ by M-test ($M_n = 1/n^2$).
- Continuous (uniform limit of continuous functions).
- Integrable term by term on any $[a, b]$.
- But $\sum \frac{n\cos(nx)}{n^2} = \sum \frac{\cos(nx)}{n}$ does NOT converge uniformly (M-test fails), so we cannot blindly differentiate.

---

## 4.5 Worked Example

**Problem.** Show $f(x) = \sum_{n=1}^\infty \frac{x^n}{n \cdot 2^n}$ is differentiable on $(-2, 2)$ and find $f'(x)$.

**Solution.** On $[-r, r]$ for $0 < r < 2$:

$$|g_n'(x)| = \left|\frac{x^{n-1}}{2^n}\right| \leq \frac{r^{n-1}}{2^n} = \frac{1}{2r}\left(\frac{r}{2}\right)^n = M_n$$

$\sum M_n$ converges (geometric with ratio $r/2 < 1$). Also $g_n(0) = 0$, so $\sum g_n(0) = 0$ converges.

By Theorem 4.2:

$$f'(x) = \sum_{n=1}^\infty \frac{x^{n-1}}{2^n} = \frac{1}{2}\sum_{n=0}^\infty \left(\frac{x}{2}\right)^n = \frac{1}{2} \cdot \frac{1}{1 - x/2} = \frac{1}{2 - x}$$

(We can also verify: $f(x) = -\ln(1 - x/2)$, so $f'(x) = 1/(2-x)$. ✓)

---

## Summary Table

| Result | Condition | Formula |
|--------|-----------|---------|
| Integrate series | $\sum g_n$ uniformly conv. | $\int \sum g_n = \sum \int g_n$ |
| Differentiate series | $\sum g_n'$ uniformly conv. + point conv. | $(\sum g_n)' = \sum g_n'$ |
| Integration is easier | Only needs original series uniform | — |
| Differentiation is harder | Needs derivatives uniform | — |

---

## Quick Revision Questions

1. **State** the theorem for term-by-term integration of series.

2. **What** hypotheses are needed for term-by-term differentiation?

3. **Use** term-by-term integration to show $\ln 2 = 1 - 1/2 + 1/3 - 1/4 + \cdots$.

4. **Find** $f'(x)$ for $f(x) = \sum_{n=0}^\infty \frac{x^{2n+1}}{(2n+1)!}$ and identify $f$.

5. **Explain** why integration of series is "safer" than differentiation.

6. **Give** an example of a uniformly convergent series that cannot be differentiated term by term.

---

[← Previous: Weierstrass M-Test](03-weierstrass-m-test.md) | [Back to README](../README.md) | [Next: Power Series →](05-power-series.md)
