# Chapter 2.5: Fixed-Point Iteration

[← Previous: Secant Method](04-secant-method.md) | [Next: Convergence Analysis →](06-convergence-analysis.md)

---

## Overview

**Fixed-point iteration** is a general framework that unifies many iterative methods. A **fixed point** of a function $g$ is a value $r$ such that $g(r) = r$. By reformulating $f(x) = 0$ as $x = g(x)$, we can iterate $x_{n+1} = g(x_n)$ to find roots. The key question: under what conditions does this converge?

---

## 1. Fixed Points

### Definition

A value $r$ is a **fixed point** of $g$ if $g(r) = r$.

```
  y
    │          y = x (45° line)
    │         ╱      
    │        ╱   ╱ y = g(x)
    │       ╱  ╱
    │      ╱╱    ← intersection = fixed point
    │     ╳
    │   ╱╱
    │  ╱╱
    │ ╱╱
  ──┼────────────────── x
    │     r
    │
    │  At r: g(r) = r  (the curves intersect)
```

### Relationship to Root-Finding

Every equation $f(x) = 0$ can be written as $x = g(x)$ in many ways:

**Example**: $f(x) = x^2 - x - 2 = 0$

| Rearrangement | $g(x)$ |
|--------------|--------|
| $x = x^2 - 2$ | $g_1(x) = x^2 - 2$ |
| $x = \sqrt{x + 2}$ | $g_2(x) = \sqrt{x + 2}$ |
| $x = 1 + 2/x$ | $g_3(x) = 1 + 2/x$ |
| $x = (x + 2)/(x + 1) \cdot x$ | ... |

**Not all choices converge!** The choice of $g$ determines convergence.

---

## 2. The Iteration

$$x_{n+1} = g(x_n), \quad n = 0, 1, 2, \ldots$$

### Cobweb Diagram (Convergent Case)

```
  y
    │       y = x
    │      ╱    ╱ y = g(x)
    │     ╱   ╱
    │    ╱  ╱  ●──────── g(x₂) = x₃
    │   ╱ ╱  ╱ │
    │  ╱╱  ╱   │
    │ ╳  ╱─────● g(x₁) = x₂
    │╱ ╱ │   ╱
    │╱───●─╱──── g(x₀) = x₁
    │  x₀ x₁ x₂  x₃
  ──┼────────────────── x
    │     r (fixed point)
    │
    │  Spiral converging to r (|g'(r)| < 1)
```

### Cobweb Diagram (Divergent Case)

```
  y
    │       y = x
    │      ╱         ╱ y = g(x)
    │     ╱        ╱
    │    ╱       ╱     Spiral AWAY from r
    │   ╱      ╱
    │  ╱     ╱
    │ ╱    ╱
    │╱   ╱
    │╳ ╱
    │╱   (|g'(r)| > 1)
  ──┼────────────────── x
```

---

## 3. Convergence Theorem (Contraction Mapping)

**Theorem**: If $g$ is continuous on $[a, b]$ and:
1. $g(x) \in [a, b]$ for all $x \in [a, b]$ (maps the interval to itself)
2. $|g'(x)| \leq L < 1$ for all $x \in (a, b)$ (contraction condition)

Then:
- $g$ has a **unique** fixed point $r$ in $[a, b]$
- The iteration $x_{n+1} = g(x_n)$ **converges** to $r$ for **any** $x_0 \in [a, b]$
- The convergence is **linear** with rate $L$

### Error Bounds

$$|x_n - r| \leq \frac{L^n}{1 - L} |x_1 - x_0|$$

$$|x_n - r| \leq \frac{L}{1 - L} |x_n - x_{n-1}|$$

---

## 4. Effect of $g'(r)$ on Convergence

| Condition | Behavior | Type |
|-----------|----------|------|
| $\|g'(r)\| < 1$ | Converges | Stable |
| $\|g'(r)\| > 1$ | Diverges | Unstable |
| $g'(r) = 0$ | Superfast (quadratic) | Newton's method! |
| $0 < g'(r) < 1$ | Monotone convergence | Staircase pattern |
| $-1 < g'(r) < 0$ | Oscillating convergence | Spiral pattern |
| $g'(r) > 1$ | Monotone divergence | — |
| $g'(r) < -1$ | Oscillating divergence | — |

---

## 5. Worked Example

Find a root of $f(x) = e^{-x} - x = 0$ (i.e., find where $y = e^{-x}$ meets $y = x$).

**Rearrangement**: $x = e^{-x} = g(x)$

**Check**: $g'(x) = -e^{-x}$, so $|g'(x)| = e^{-x} < 1$ for $x > 0$ ✓

Starting with $x_0 = 0.5$:

| Iter | $x_n$ | $g(x_n) = e^{-x_n}$ | $\|x_{n+1} - x_n\|$ |
|:----:|:-----:|:-------------------:|:-------------------:|
| 0 | 0.500000 | 0.606531 | 0.1065 |
| 1 | 0.606531 | 0.545239 | 0.0613 |
| 2 | 0.545239 | 0.579703 | 0.0345 |
| 3 | 0.579703 | 0.560065 | 0.0196 |
| 4 | 0.560065 | 0.571172 | 0.0111 |
| 5 | 0.571172 | 0.564863 | 0.0063 |
| ... | ... | ... | ... |
| 20 | 0.567143 | 0.567144 | $< 10^{-6}$ |

**Root**: $r \approx 0.56714$ (Lambert W function value).

Note: Convergence is **slow** because $|g'(r)| = e^{-0.567} \approx 0.567$ is not small.

---

## 6. Choosing a Good $g(x)$

### Example: $x^3 + x - 1 = 0$

| Choice | $g(x)$ | $g'(x)$ at root | Converges? |
|--------|--------|:----------------:|:----------:|
| A | $1 - x^3$ | $-3x^2 \approx -1.4$ | ✗ |
| B | $(1 - x)^{1/3}$ | $-\frac{1}{3}(1-x)^{-2/3} \approx -0.45$ | ✓ |
| C | $\frac{1}{x^2 + 1}$ | $\frac{-2x}{(x^2+1)^2} \approx -0.47$ | ✓ |

**Rule**: Choose $g$ such that $|g'(x)|$ is as small as possible near the root.

---

## 7. Connection to Newton-Raphson

Newton's method IS a fixed-point iteration with:

$$g(x) = x - \frac{f(x)}{f'(x)}$$

Check: $g'(x) = 1 - \frac{[f'(x)]^2 - f(x)f''(x)}{[f'(x)]^2} = \frac{f(x)f''(x)}{[f'(x)]^2}$

At the root $r$: $f(r) = 0$, so $g'(r) = 0$.

This is why Newton's method has **quadratic** convergence — $g'(r) = 0$ is the best possible!

---

## 8. Aitken's $\Delta^2$ Acceleration

For a linearly convergent sequence, **Aitken's method** accelerates convergence:

$$\hat{x}_n = x_n - \frac{(x_{n+1} - x_n)^2}{x_{n+2} - 2x_{n+1} + x_n}$$

This extrapolation often converges much faster than the original sequence.

---

## 9. Real-World Applications

| Application | Fixed-Point Problem |
|-------------|-------------------|
| Economics | Market equilibrium: supply = demand |
| Game theory | Nash equilibrium: best response iteration |
| Image processing | Iterative filtering/denoising |
| Network routing | Routing table convergence |
| Power systems | Load flow analysis |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $x_{n+1} = g(x_n)$ |
| **Convergence condition** | $\|g'(r)\| < 1$ |
| **Order** | Linear (if $g'(r) \neq 0$), Quadratic (if $g'(r) = 0$) |
| **Rate** | $L = \|g'(r)\|$ |
| **Error bound** | $\|x_n - r\| \leq \frac{L^n}{1-L}\|x_1 - x_0\|$ |
| **Unique fixed point** | Guaranteed by contraction mapping theorem |
| **Newton as special case** | $g(x) = x - f(x)/f'(x)$ gives $g'(r) = 0$ |

---

## Quick Revision Questions

1. **What is a fixed point? How is root-finding converted to a fixed-point problem?**

2. **State the Contraction Mapping Theorem and explain each condition.**

3. **For $f(x) = x^3 - 2x - 5 = 0$, find two different $g(x)$ rearrangements. Which converges?**

4. **Show that Newton-Raphson is a fixed-point iteration with $g'(r) = 0$ at a simple root.**

5. **Draw cobweb diagrams for (a) convergent monotone, (b) convergent oscillating, (c) divergent cases.**

6. **If $g'(r) = 0.3$ and $|x_1 - x_0| = 0.1$, estimate the error after 10 iterations.**

---

[← Previous: Secant Method](04-secant-method.md) | [Next: Convergence Analysis →](06-convergence-analysis.md)
