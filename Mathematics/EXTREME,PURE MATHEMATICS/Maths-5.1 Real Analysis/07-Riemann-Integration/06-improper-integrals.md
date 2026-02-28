# Chapter 6: Improper Integrals

[← Previous: Fundamental Theorem of Calculus](05-fundamental-theorem-of-calculus.md) | [Back to README](../README.md) | [Next: Unit 8 — Pointwise Convergence →](../08-Sequences-and-Series-of-Functions/01-pointwise-convergence.md)

---

## Overview

The Riemann integral requires a **bounded** function on a **finite** interval. **Improper integrals** extend this to unbounded intervals (Type I) and unbounded functions (Type II) by taking limits of proper integrals.

---

## 6.1 Type I: Infinite Intervals

**Definition 6.1.** If $f \in \mathcal{R}[a, R]$ for every $R > a$:

$$\int_a^\infty f(x)\,dx = \lim_{R \to \infty} \int_a^R f(x)\,dx$$

if the limit exists (**convergent**), otherwise **divergent**. Similarly for $\int_{-\infty}^b$ and $\int_{-\infty}^\infty$.

```
  Type I: Unbounded interval

  f(x)
   │╲
   │  ╲
   │    ╲───────────── → 0
   │      ╲
   └──────────────────────── x
   a        R → ∞

   ∫ₐ^∞ f = lim_{R→∞} ∫ₐ^R f
```

**Example.** $\int_1^\infty \frac{1}{x^p}\,dx = \lim_{R\to\infty}\frac{x^{1-p}}{1-p}\Big|_1^R$:

$$= \begin{cases} \frac{1}{p-1} & p > 1 \quad (\text{convergent}) \\ \infty & p \leq 1 \quad (\text{divergent}) \end{cases}$$

---

## 6.2 Type II: Unbounded Functions

**Definition 6.2.** If $f$ is unbounded near $b$ but $f \in \mathcal{R}[a, b-\varepsilon]$ for every $\varepsilon > 0$:

$$\int_a^b f(x)\,dx = \lim_{\varepsilon \to 0^+} \int_a^{b-\varepsilon} f(x)\,dx$$

```
  Type II: Unbounded function

  f(x)
   │            │
   │         ╱  │
   │       ╱    │  f → ∞ near b
   │     ╱      │
   │───╱        │
   └──────────────── x
   a      b-ε   b

   ∫ₐᵇ f = lim_{ε→0⁺} ∫ₐ^{b-ε} f
```

**Example.** $\int_0^1 \frac{1}{x^p}\,dx = \lim_{\varepsilon \to 0^+}\frac{x^{1-p}}{1-p}\Big|_\varepsilon^1$:

$$= \begin{cases} \frac{1}{1-p} & p < 1 \quad (\text{convergent}) \\ \infty & p \geq 1 \quad (\text{divergent}) \end{cases}$$

---

## 6.3 p-Integral Summary

| Integral | Converges when | Value |
|----------|---------------|-------|
| $\int_1^\infty x^{-p}\,dx$ | $p > 1$ | $\frac{1}{p-1}$ |
| $\int_0^1 x^{-p}\,dx$ | $p < 1$ | $\frac{1}{1-p}$ |

**Observation.** $p = 1$ is critical for both — $\int_1^\infty \frac{dx}{x}$ and $\int_0^1 \frac{dx}{x}$ both diverge.

---

## 6.4 Absolute Convergence

**Definition 6.4.** $\int_a^\infty f$ **converges absolutely** if $\int_a^\infty |f|$ converges.

**Theorem 6.4.** Absolute convergence $\Rightarrow$ convergence.

**Proof.** Use $0 \leq f^+ \leq |f|$ and $0 \leq f^- \leq |f|$ where $f = f^+ - f^-$. Comparison gives $\int f^+$ and $\int f^-$ converge, so $\int f = \int f^+ - \int f^-$ converges. $\blacksquare$

**Conditional convergence:** $\int_a^\infty f$ converges but $\int_a^\infty |f|$ diverges.

**Example.** $\int_1^\infty \frac{\sin x}{x}\,dx$ converges conditionally (by Dirichlet's test) but not absolutely.

---

## 6.5 Comparison Tests

**Theorem 6.5 (Direct Comparison).** If $0 \leq f(x) \leq g(x)$ for $x \geq a$:
- $\int_a^\infty g$ converges $\Rightarrow$ $\int_a^\infty f$ converges.
- $\int_a^\infty f$ diverges $\Rightarrow$ $\int_a^\infty g$ diverges.

**Theorem 6.5' (Limit Comparison).** If $f, g > 0$ and $\lim_{x\to\infty} f(x)/g(x) = L$:
- $0 < L < \infty$: both converge or both diverge.
- $L = 0$ and $\int g$ converges $\Rightarrow$ $\int f$ converges.
- $L = \infty$ and $\int g$ diverges $\Rightarrow$ $\int f$ diverges.

```
  Comparison Test Decision Tree:

  Given ∫ₐ^∞ f(x)dx with f ≥ 0:

       Find comparison g(x)
              │
       ┌──────┴──────┐
       │  f ≤ g ?    │  f ≥ g ?
       │             │
    ∫g conv.      ∫g div.
    ⟹ ∫f conv.   ⟹ ∫f div.
```

---

## 6.6 Dirichlet and Abel Tests

**Theorem 6.6 (Dirichlet).** $\int_a^\infty f \cdot g$ converges if:
1. $F(R) = \int_a^R f$ is **bounded**
2. $g$ is **monotone decreasing** to $0$

**Theorem 6.6' (Abel).** $\int_a^\infty f \cdot g$ converges if:
1. $\int_a^\infty f$ **converges**
2. $g$ is **bounded and monotone**

**Application.** $\int_1^\infty \frac{\sin x}{x^p}\,dx$ converges for $p > 0$ (Dirichlet with $f = \sin x$, $g = x^{-p}$).

---

## 6.7 Worked Examples

**Example 1.** $\int_0^\infty e^{-x}\,dx = \lim_{R\to\infty}[-e^{-x}]_0^R = 0 - (-1) = 1$.

**Example 2.** $\int_0^\infty \frac{dx}{1 + x^2} = \lim_{R\to\infty}[\arctan x]_0^R = \frac{\pi}{2}$.

**Example 3.** Does $\int_1^\infty \frac{x}{x^3 + 1}\,dx$ converge?

Limit comparison with $1/x^2$: $\frac{x/(x^3+1)}{1/x^2} = \frac{x^3}{x^3+1} \to 1$. Since $\int 1/x^2$ converges ($p = 2 > 1$), so does $\int \frac{x}{x^3+1}$. $\checkmark$

**Example 4.** $\int_0^1 \frac{\sin x}{\sqrt{x}}\,dx$. Near $0$: $\frac{\sin x}{\sqrt{x}} \approx \frac{x}{\sqrt{x}} = \sqrt{x}$, which is integrable. Converges.

**Example 5 (Gamma function).** $\Gamma(n) = \int_0^\infty x^{n-1}e^{-x}\,dx$ converges for $n > 0$.
- Near $0$: Type II, $x^{n-1}$ integrable iff $n - 1 > -1$, i.e., $n > 0$. $\checkmark$
- Near $\infty$: $x^{n-1}e^{-x}$ decays faster than any $x^{-2}$ for large $x$. $\checkmark$

---

## 6.8 Cauchy Principal Value

When $\int_{-\infty}^\infty f$ diverges, the **Cauchy principal value** may still exist:

$$\text{P.V.}\int_{-\infty}^\infty f(x)\,dx = \lim_{R \to \infty}\int_{-R}^R f(x)\,dx$$

**Example.** $\int_{-\infty}^\infty x\,dx$ diverges, but $\text{P.V.}\int_{-\infty}^\infty x\,dx = \lim_{R\to\infty}\frac{x^2}{2}\Big|_{-R}^R = 0$.

**Warning:** The principal value is NOT the same as the improper integral. Convergence of the improper integral requires independent limits.

---

## Summary Table

| Type | Setup | Limit |
|------|-------|-------|
| Type I | Infinite interval | $\lim_{R\to\infty} \int_a^R f$ |
| Type II | Unbounded $f$ | $\lim_{\varepsilon\to 0^+}\int_a^{b-\varepsilon}f$ |
| $p$-test ($\infty$) | $\int_1^\infty x^{-p}$ | Conv. $\iff p > 1$ |
| $p$-test ($0$) | $\int_0^1 x^{-p}$ | Conv. $\iff p < 1$ |
| Comparison | $0 \leq f \leq g$ | $\int g$ conv. $\Rightarrow \int f$ conv. |
| Absolute conv. | $\int|f|$ conv. | $\Rightarrow \int f$ conv. |
| Dirichlet | $\int f$ bdd, $g \downarrow 0$ | $\int fg$ conv. |

---

## Quick Revision Questions

1. **Define** Type I and Type II improper integrals. Give an example of each.

2. **Prove** the $p$-test: $\int_1^\infty x^{-p}\,dx$ converges iff $p > 1$.

3. **Determine** convergence of $\int_0^\infty \frac{dx}{\sqrt{x}(1 + x)}$ (split at $x = 1$, use $p$-tests).

4. **Show** $\int_1^\infty \frac{\sin x}{x}\,dx$ converges using Dirichlet's test.

5. **Explain** the difference between convergence and the Cauchy principal value.

6. **State** the limit comparison test for improper integrals and apply it to $\int_1^\infty \frac{\ln x}{x^2}\,dx$.

---

[← Previous: Fundamental Theorem of Calculus](05-fundamental-theorem-of-calculus.md) | [Back to README](../README.md) | [Next: Unit 8 — Pointwise Convergence →](../08-Sequences-and-Series-of-Functions/01-pointwise-convergence.md)
