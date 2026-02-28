# Chapter 5: Higher-Order Derivatives and Convexity

[← Previous: Taylor's Theorem](04-taylors-theorem.md) | [Back to README](../README.md) | [Next: Unit 7 — Riemann Sums →](../07-Riemann-Integration/01-riemann-sums.md)

---

## Overview

Higher-order derivatives refine our understanding of a function's geometry: the second derivative detects concavity and inflection points, while the $n$-th derivative test classifies extrema. We also explore smooth function spaces and convexity.

---

## 5.1 Higher-Order Derivatives

**Definition 5.1.** The $n$-th derivative $f^{(n)}$ is defined inductively:

$$f^{(0)} = f, \quad f^{(n)} = (f^{(n-1)})'$$

$f$ is **$n$-times differentiable** at $c$ if $f^{(n)}(c)$ exists; **$C^n$** on $(a,b)$ if $f^{(n)}$ exists and is continuous; **$C^\infty$** (smooth) if differentiable infinitely many times.

```
  Smoothness classes:

  C⁰ ⊃ C¹ ⊃ C² ⊃ ... ⊃ C^∞ ⊃ Analytic

  C⁰ = continuous
  C¹ = continuously differentiable
  C^∞ = smooth (all derivatives exist and are continuous)
  Analytic = equals its Taylor series locally
```

---

## 5.2 Second Derivative Test

**Theorem 5.2.** If $f'(c) = 0$ and $f''(c)$ exists:
- $f''(c) > 0 \Rightarrow$ $f$ has a **local minimum** at $c$
- $f''(c) < 0 \Rightarrow$ $f$ has a **local maximum** at $c$
- $f''(c) = 0 \Rightarrow$ test is inconclusive

**Proof (min case).** $f''(c) > 0$ means $f'$ is increasing at $c$. Since $f'(c) = 0$: $f'(x) < 0$ for $x$ slightly left of $c$ and $f'(x) > 0$ slightly right. So $f$ decreases then increases — local min. $\blacksquare$

---

## 5.3 General $n$-th Derivative Test

**Theorem 5.3.** Suppose $f'(c) = f''(c) = \cdots = f^{(n-1)}(c) = 0$ and $f^{(n)}(c) \neq 0$.

- If $n$ is **even** and $f^{(n)}(c) > 0$: **local minimum**
- If $n$ is **even** and $f^{(n)}(c) < 0$: **local maximum**
- If $n$ is **odd**: **inflection point** (no extremum)

**Proof idea.** Taylor expansion: $f(x) - f(c) \approx \frac{f^{(n)}(c)}{n!}(x-c)^n$. Sign behavior depends on parity of $n$.

---

## 5.4 Convexity

**Definition 5.4.** $f: (a,b) \to \mathbb{R}$ is **convex** if for all $x, y \in (a,b)$ and $t \in [0,1]$:

$$f(tx + (1-t)y) \leq tf(x) + (1-t)f(y)$$

Geometrically: the graph lies below every chord.

```
  Convex function:

  f(x) ▲
       │         ● f(y)
       │       ╱ │
       │     ╱   │ ← chord
       │   ╱  ●  │
       │ ╱  ╱╲   │ ← graph is BELOW chord
       ●╱  ╱  ╲  │
       │ ╱    ╲ │
       └──┼──────┼──► x
          x      y

  f(tx + (1-t)y) ≤ tf(x) + (1-t)f(y)
```

**Theorem 5.5 (Differentiable characterization).** For twice-differentiable $f$:

$$f \text{ convex on } (a,b) \quad\iff\quad f''(x) \geq 0 \;\;\forall\, x \in (a,b)$$

| $f''$ on $(a,b)$ | $f$ is |
|:-:|---------|
| $f'' > 0$ | Strictly convex |
| $f'' \geq 0$ | Convex |
| $f'' < 0$ | Strictly concave |
| $f'' \leq 0$ | Concave |

---

## 5.5 Jensen's Inequality

**Theorem 5.6 (Jensen).** If $f$ is convex and $t_1 + \cdots + t_n = 1$ with $t_i \geq 0$:

$$f(t_1 x_1 + \cdots + t_n x_n) \leq t_1 f(x_1) + \cdots + t_n f(x_n)$$

**Applications:**
- AM-GM inequality (from convexity of $-\ln x$)
- Power mean inequalities
- Information theory (entropy bounds)

---

## 5.6 Inflection Points

**Definition 5.6.** An **inflection point** of $f$ at $c$ is where the concavity changes: $f''$ changes sign at $c$.

Necessary condition: $f''(c) = 0$ (if $f''$ exists). But $f''(c) = 0$ is NOT sufficient (e.g., $f(x) = x^4$).

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| $C^n$ functions | $f^{(n)}$ exists and is continuous |
| Second derivative test | $f'(c)=0$, $f''(c)>0$ → local min; $f''(c)<0$ → local max |
| $n$-th derivative test | Even $n$, $f^{(n)}(c) \neq 0$ → extremum; odd $n$ → inflection |
| Convexity | $f(tx+(1-t)y) \leq tf(x) + (1-t)f(y)$; iff $f'' \geq 0$ |
| Jensen's inequality | Convexity extends to weighted averages |
| Inflection point | Change of concavity; $f''$ changes sign |

---

## Quick Revision Questions

1. **State** the second derivative test for local extrema with proof.

2. **What** does $f'(c) = f''(c) = 0$ but $f'''(c) \neq 0$ tell you about $c$?

3. **Define** convexity and prove: $f$ convex $\iff f'' \geq 0$ (for $C^2$ functions).

4. **State** Jensen's Inequality and derive AM-GM from it.

5. **Give** an example where $f''(c) = 0$ but $c$ is NOT an inflection point.

6. **Explain** the relationship between $C^0 \supset C^1 \supset C^2 \supset \cdots \supset C^\infty \supset$ Analytic.

---

[← Previous: Taylor's Theorem](04-taylors-theorem.md) | [Back to README](../README.md) | [Next: Unit 7 — Riemann Sums →](../07-Riemann-Integration/01-riemann-sums.md)
