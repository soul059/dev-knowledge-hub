# Chapter 7.1: Euler's Method

[← Previous: Gaussian Quadrature](../06-Numerical-Integration/07-gaussian-quadrature.md) | [Next: Modified Euler →](02-modified-euler.md)

---

## Overview

**Euler's method** is the simplest numerical method for solving initial value problems (IVPs) of the form $y' = f(x, y)$, $y(x_0) = y_0$. It advances the solution using the tangent line at each step, giving $O(h)$ global accuracy.

---

## 1. The Formula

$$\boxed{y_{n+1} = y_n + h \cdot f(x_n, y_n)}$$

where $h$ is the step size and $f(x_n, y_n) = y'_n$ is the slope at $(x_n, y_n)$.

---

## 2. Geometric Interpretation

```
  y
  │         True solution
  │        ╱╱╱╱
  │       ╱   ╱  ← Euler follows tangent lines
  │      ╱  ╱
  │     ╱ ╱     ● (x₂, y₂)
  │    ╱╱      ╱
  │   ●──────●
  │  (x₀,y₀) (x₁,y₁)
  │    slope = f(x₀,y₀)
  └──────────────────── x
     x₀    x₁    x₂

  Each step: follow the tangent for distance h
```

---

## 3. Derivation

From the Taylor expansion:

$$y(x_0 + h) = y(x_0) + h y'(x_0) + \frac{h^2}{2}y''(\xi)$$

Dropping the $O(h^2)$ term:

$$y_1 \approx y_0 + h f(x_0, y_0)$$

---

## 4. Error Analysis

| Error Type | Value |
|-----------|-------|
| **Local truncation error** | $\frac{h^2}{2}y''(\xi) = O(h^2)$ |
| **Global truncation error** | $O(h)$ (errors accumulate over $n = (b-a)/h$ steps) |

Euler's method is a **first-order method**.

---

## 5. Worked Example

Solve $y' = x + y$, $y(0) = 1$, find $y(0.4)$ with $h = 0.1$.

| Step | $x_n$ | $y_n$ | $f(x_n, y_n) = x_n + y_n$ | $y_{n+1} = y_n + 0.1 f$ |
|------|--------|--------|--------------------------|--------------------------|
| 0 | 0.0 | 1.0000 | $0 + 1 = 1.0000$ | $1 + 0.1 = 1.1000$ |
| 1 | 0.1 | 1.1000 | $0.1 + 1.1 = 1.2000$ | $1.1 + 0.12 = 1.2200$ |
| 2 | 0.2 | 1.2200 | $0.2 + 1.22 = 1.4200$ | $1.22 + 0.142 = 1.3620$ |
| 3 | 0.3 | 1.3620 | $0.3 + 1.362 = 1.6620$ | $1.362 + 0.1662 = 1.5282$ |

$$\boxed{y(0.4) \approx 1.5282}$$

**Exact solution:** $y = 2e^x - x - 1$, so $y(0.4) = 2e^{0.4} - 0.4 - 1 = 1.5836$

**Error:** $|1.5836 - 1.5282| = 0.0554$ (significant for this step size)

---

## 6. Stability

For the test equation $y' = \lambda y$ ($\lambda < 0$):

$$y_{n+1} = (1 + h\lambda) y_n$$

The solution is stable if $|1 + h\lambda| \leq 1$, giving:

$$\boxed{0 < h < \frac{2}{|\lambda|}}$$

```
  │ Stability region (shaded):
  │
  │    Im(hλ)
  │      ●
  │     ╱│╲
  │    ╱ │ ╲
  ───●───┼───●─── Re(hλ)
  -2  ╲ │ ╱  0
  │    ╲│╱
  │     ●
  │
  │  Circle of radius 1, centered at (-1, 0)
```

---

## 7. Pros and Cons

| Advantages | Disadvantages |
|-----------|---------------|
| Very simple to implement | Low accuracy: $O(h)$ |
| Easy to understand | Requires small $h$ for precision |
| Foundation for better methods | Unstable for stiff problems |
| One $f$-evaluation per step | Error accumulates rapidly |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $y_{n+1} = y_n + hf(x_n, y_n)$ |
| **Order** | 1 (first-order method) |
| **Local error** | $O(h^2)$ |
| **Global error** | $O(h)$ |
| **Stability** | $h < 2/|\lambda|$ for $y' = \lambda y$ |
| **$f$-evals per step** | 1 |

---

## Quick Revision Questions

1. **State Euler's method and derive it from the Taylor series.**

2. **Solve $y' = y$, $y(0) = 1$, to find $y(0.2)$ with $h = 0.1$. Compare with the exact solution.**

3. **What is the stability condition for Euler's method applied to $y' = -10y$?**

4. **Explain the difference between local and global truncation error.**

5. **Why is Euler's method rarely used in practice despite its simplicity?**

6. **Sketch the stability region of Euler's method in the $h\lambda$-plane.**

---

[← Previous: Gaussian Quadrature](../06-Numerical-Integration/07-gaussian-quadrature.md) | [Next: Modified Euler →](02-modified-euler.md)
