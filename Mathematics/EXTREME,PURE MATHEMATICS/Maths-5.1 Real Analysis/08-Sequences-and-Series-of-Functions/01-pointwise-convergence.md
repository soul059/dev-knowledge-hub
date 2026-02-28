# Chapter 1: Pointwise Convergence

[← Previous: Unit 7 — Improper Integrals](../07-Riemann-Integration/06-improper-integrals.md) | [Back to README](../README.md) | [Next: Uniform Convergence →](02-uniform-convergence.md)

---

## Overview

A **sequence of functions** $(f_n)$ converges **pointwise** if it converges at every point in the domain. This is the most natural notion of convergence, but it has serious deficiencies: pointwise limits of continuous functions need not be continuous, and we cannot always interchange limits with integrals or derivatives.

---

## 1.1 Definition

**Definition 1.1.** Let $f_n: A \to \mathbb{R}$. We say $(f_n)$ **converges pointwise** to $f: A \to \mathbb{R}$ if:

$$\forall\, x \in A,\; \forall\, \varepsilon > 0,\; \exists\, N = N(x, \varepsilon) \in \mathbb{N}: \; n \geq N \Rightarrow |f_n(x) - f(x)| < \varepsilon$$

**Key observation.** $N$ depends on **both** $\varepsilon$ and $x$. This is the critical difference from uniform convergence.

Equivalently: for each fixed $x \in A$, the numerical sequence $(f_n(x))_{n=1}^\infty$ converges to $f(x)$.

**Notation:** $f_n \to f$ pointwise on $A$, or $\lim_{n\to\infty} f_n(x) = f(x)$ for all $x \in A$.

---

## 1.2 Examples

**Example 1.** $f_n(x) = x^n$ on $[0, 1]$.

$$f(x) = \lim_{n\to\infty} x^n = \begin{cases} 0 & 0 \leq x < 1 \\ 1 & x = 1 \end{cases}$$

```
  f_n(x) = xⁿ on [0,1]:

  1 ┤─────────────────●  f₁ = x
    │              ╱╱╱│
    │           ╱╱╱   │  f₂ = x²
    │        ╱╱╱      │
    │     ╱╱╱   ╱     │  f₅ = x⁵
    │  ╱╱╱   ╱╱       │
    │╱╱   ╱╱╱         │  f₂₀ = x²⁰
    │  ╱╱╱            │
  0 ┤═══════════════╤─┤
    0               1

  Limit: f(x) = 0 on [0,1), f(1) = 1
  Each fₙ continuous, but limit DISCONTINUOUS!
```

**Example 2.** $f_n(x) = \frac{nx}{1 + n^2x^2}$ on $\mathbb{R}$.

$f_n(x) \to 0$ for all $x$. But $f_n(1/n) = 1/2$, so the convergence is slow near $0$.

**Example 3.** $f_n(x) = n x(1 - x)^n$ on $[0, 1]$.

$f_n(x) \to 0$ pointwise. But $\int_0^1 f_n = \frac{n}{(n+1)(n+2)} \to \frac{1}{2} \neq 0 = \int_0^1 f$.

---

## 1.3 Failures of Pointwise Convergence

| Property | Preserved? | Counterexample |
|----------|-----------|----------------|
| Continuity | **NO** | $x^n$ on $[0,1]$ |
| Integrability | **NO** (can fail) | $\chi_{\mathbb{Q}_n}$ examples |
| $\lim \int = \int \lim$ | **NO** | $nx(1-x)^n$ |
| $\lim f_n' = (\lim f_n)'$ | **NO** | $\frac{\sin(nx)}{n} \to 0$ but $f_n' = \cos(nx)$ diverges |
| Boundedness | **NO** | $f_n(x) = n$ on $(0, 1/n)$ |

These failures motivate **uniform convergence**.

---

## 1.4 Pointwise Convergence of Series

**Definition 1.4.** $\sum_{n=1}^\infty g_n(x)$ converges pointwise if the partial sums $S_N(x) = \sum_{n=1}^N g_n(x)$ converge pointwise.

**Example.** Geometric series: $\sum_{n=0}^\infty x^n = \frac{1}{1-x}$ for $|x| < 1$ (pointwise).

---

## 1.5 Finding the Pointwise Limit

**Strategy:** For each fixed $x$, compute $\lim_{n\to\infty} f_n(x)$. This may give different formulas in different regions.

**Example.** $f_n(x) = \frac{x^{2n} - 1}{x^{2n} + 1}$ on $\mathbb{R}$.

$$f(x) = \begin{cases} 1 & |x| > 1 \\ 0 & |x| = 1 \\ -1 & |x| < 1 \end{cases}$$

---

## Summary Table

| Concept | Details |
|---------|---------|
| Pointwise convergence | $\forall x$, $f_n(x) \to f(x)$; $N$ depends on $x$ |
| Limit may be discontinuous | $x^n$ on $[0,1]$ |
| Cannot swap $\lim$ and $\int$ | $nx(1-x)^n$ |
| Cannot swap $\lim$ and $d/dx$ | $\sin(nx)/n$ |
| Need stronger notion | $\Rightarrow$ Uniform convergence |

---

## Quick Revision Questions

1. **Define** pointwise convergence. Why does $N$ depend on $x$?

2. **Find** the pointwise limit of $f_n(x) = x^n$ on $[0, 1]$ and show the limit is discontinuous.

3. **Show** $f_n(x) = \frac{x}{1 + nx^2} \to 0$ pointwise on $\mathbb{R}$.

4. **Give** an example where $\lim \int f_n \neq \int \lim f_n$.

5. **Find** the pointwise limit of $f_n(x) = (1 + x/n)^n$ on $\mathbb{R}$.

6. **Explain** why pointwise convergence of continuous functions does not preserve continuity.

---

[← Previous: Unit 7 — Improper Integrals](../07-Riemann-Integration/06-improper-integrals.md) | [Back to README](../README.md) | [Next: Uniform Convergence →](02-uniform-convergence.md)
