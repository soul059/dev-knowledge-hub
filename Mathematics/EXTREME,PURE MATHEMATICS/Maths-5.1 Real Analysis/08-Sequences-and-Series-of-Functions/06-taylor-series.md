# Chapter 6: Taylor Series

[← Previous: Power Series](05-power-series.md) | [Back to README](../README.md) | [Next: Unit 9 — Metric Spaces →](../09-Metric-Spaces/01-definition-and-examples.md)

---

## Overview

The **Taylor series** of a function is the power series formed from its derivatives. The central question: **when does a function equal its Taylor series?** Not always — smoothness alone is not enough. We need control over the **remainder**.

---

## 6.1 Taylor Series Definition

**Definition 6.1.** If $f$ is infinitely differentiable at $c$, its **Taylor series** at $c$ is:

$$\sum_{n=0}^\infty \frac{f^{(n)}(c)}{n!}(x - c)^n$$

When $c = 0$, this is the **Maclaurin series**.

A function is **analytic** at $c$ if it equals its Taylor series in some neighborhood of $c$.

---

## 6.2 Taylor Remainder Analysis

From Taylor's theorem (Unit 6): $f(x) = T_N(x) + R_N(x)$ where:

$$T_N(x) = \sum_{n=0}^N \frac{f^{(n)}(c)}{n!}(x-c)^n, \qquad R_N(x) = \frac{f^{(N+1)}(\xi)}{(N+1)!}(x-c)^{N+1}$$

**$f$ equals its Taylor series $\iff$ $R_N(x) \to 0$ as $N \to \infty$.**

```
  Taylor series convergence:

  f(x) = T_N(x) + R_N(x)

  If R_N(x) → 0:
      f(x) = lim T_N(x) = Σ f⁽ⁿ⁾(c)/n! · (x-c)ⁿ  ✓ analytic

  If R_N(x) ↛ 0:
      Taylor series ≠ f(x)  ✗ smooth but not analytic
```

---

## 6.3 Standard Taylor Series

| Function | Maclaurin Series | Radius |
|----------|-----------------|--------|
| $e^x$ | $\sum_{n=0}^\infty \frac{x^n}{n!}$ | $\infty$ |
| $\sin x$ | $\sum_{n=0}^\infty \frac{(-1)^n x^{2n+1}}{(2n+1)!}$ | $\infty$ |
| $\cos x$ | $\sum_{n=0}^\infty \frac{(-1)^n x^{2n}}{(2n)!}$ | $\infty$ |
| $\frac{1}{1-x}$ | $\sum_{n=0}^\infty x^n$ | $1$ |
| $\ln(1+x)$ | $\sum_{n=1}^\infty \frac{(-1)^{n-1}x^n}{n}$ | $1$ (conv. at $x=1$) |
| $(1+x)^\alpha$ | $\sum_{n=0}^\infty \binom{\alpha}{n}x^n$ | $1$ (binomial) |
| $\arctan x$ | $\sum_{n=0}^\infty \frac{(-1)^n x^{2n+1}}{2n+1}$ | $1$ (conv. at $x=\pm 1$) |

---

## 6.4 Verifying $R_N \to 0$

**Example 1: $e^x$ is analytic on $\mathbb{R}$.**

$$|R_N(x)| = \frac{|e^\xi|}{(N+1)!}|x|^{N+1} \leq \frac{e^{|x|}}{(N+1)!}|x|^{N+1} \to 0$$

since $|x|^{N+1}/(N+1)! \to 0$ for any fixed $x$ (factorial beats exponential).

**Example 2: $\sin x$ and $\cos x$ are analytic on $\mathbb{R}$.**

$$|R_N(x)| \leq \frac{|x|^{N+1}}{(N+1)!} \to 0$$

since all derivatives are bounded by $1$.

---

## 6.5 Smooth but Not Analytic

**Example.** $f(x) = \begin{cases} e^{-1/x^2} & x \neq 0 \\ 0 & x = 0 \end{cases}$

- $f \in C^\infty(\mathbb{R})$ (proved in Unit 6)
- $f^{(n)}(0) = 0$ for all $n$
- Taylor series at $0$: $\sum 0 \cdot x^n = 0$
- But $f(x) > 0$ for $x \neq 0$

$$f(x) \neq 0 = \text{Taylor series} \quad \text{for } x \neq 0$$

```
  Smooth ≠ Analytic:

  analytic ⊂ C^∞ (strict inclusion)

  ┌─────────────────────────────┐
  │  C^∞  (smooth functions)    │
  │  ┌───────────────────────┐  │
  │  │ Analytic (= Taylor)   │  │
  │  │  eˣ, sin x, polynomials│ │
  │  └───────────────────────┘  │
  │   e^{-1/x²}  ← smooth,    │
  │               not analytic  │
  └─────────────────────────────┘
```

---

## 6.6 Operations on Taylor Series

**Addition/Subtraction:** Add/subtract coefficients.

**Multiplication:** Cauchy product of coefficients.

**Composition:** Substitute one series into another (when convergent).

**Differentiation/Integration:** Term by term (same radius).

**Example.** From $e^x = \sum x^n/n!$ and substituting $-x^2$:

$$e^{-x^2} = \sum_{n=0}^\infty \frac{(-1)^n x^{2n}}{n!}$$

Integrating term by term:

$$\int_0^x e^{-t^2}\,dt = \sum_{n=0}^\infty \frac{(-1)^n x^{2n+1}}{n!(2n+1)}$$

(This is related to the error function $\text{erf}(x) = \frac{2}{\sqrt{\pi}}\int_0^x e^{-t^2}\,dt$.)

---

## 6.7 Bernstein's Theorem

**Theorem 6.7 (Bernstein).** If $f \in C^\infty[0, R]$ and $f^{(n)}(x) \geq 0$ for all $n \geq 0$ and $x \in [0, R]$ (i.e., $f$ is **completely monotone**), then $f$ is analytic on $[0, R)$.

This gives a sufficient condition beyond just checking the remainder.

---

## 6.8 Uniqueness of Taylor Series

**Theorem 6.8.** If $f(x) = \sum a_n (x-c)^n$ in some neighborhood of $c$, then $a_n = f^{(n)}(c)/n!$.

Any power series representation of $f$ IS the Taylor series. By the identity theorem, the coefficients are uniquely determined.

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Taylor series | $\sum f^{(n)}(c)/n! \cdot (x-c)^n$ |
| Analytic | $f =$ Taylor series in some neighborhood |
| Convergence criterion | $R_N(x) \to 0$ |
| All standard functions | $e^x, \sin, \cos, \ln(1+x), (1+x)^\alpha$ are analytic |
| Smooth $\neq$ analytic | $e^{-1/x^2}$ is $C^\infty$ but not analytic at $0$ |
| Uniqueness | Power series rep. $=$ Taylor series |
| Operations | Series can be added, multiplied, composed, differentiated, integrated |

---

## Quick Revision Questions

1. **Define** the Taylor series. When is a function analytic?

2. **Prove** $e^x$ equals its Maclaurin series on $\mathbb{R}$ by showing $R_N \to 0$.

3. **Write** the Maclaurin series for $\cos x$ and verify by differentiating the series for $\sin x$.

4. **Give** an example of a $C^\infty$ function that is NOT analytic at $0$.

5. **Derive** the series for $\arctan x$ by integrating $\frac{1}{1+x^2} = \sum (-1)^n x^{2n}$.

6. **Explain** why the Taylor series uniquely determines the function (when it converges to it).

---

[← Previous: Power Series](05-power-series.md) | [Back to README](../README.md) | [Next: Unit 9 — Metric Spaces →](../09-Metric-Spaces/01-definition-and-examples.md)
