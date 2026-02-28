# Chapter 7.3: Runge-Kutta Methods

[← Previous: Modified Euler](02-modified-euler.md) | [Next: Adams-Bashforth →](04-adams-bashforth.md)

---

## Overview

**Runge-Kutta (RK) methods** achieve high-order accuracy by evaluating $f(x,y)$ at several strategically chosen points within each step, without requiring higher-order derivatives. The **4th-order RK (RK4)** is the most widely used single-step ODE solver.

---

## 1. General Idea

An $s$-stage Runge-Kutta method:

$$y_{n+1} = y_n + h \sum_{i=1}^{s} b_i k_i$$

where each $k_i$ is a slope estimate at a different point within the step.

---

## 2. Second-Order RK (RK2)

$$k_1 = f(x_n, y_n)$$
$$k_2 = f(x_n + h, y_n + h k_1)$$
$$y_{n+1} = y_n + \frac{h}{2}(k_1 + k_2)$$

This is the Modified Euler / Heun's method (order 2).

---

## 3. Classical Fourth-Order RK (RK4) — THE Method

$$\boxed{\begin{aligned} k_1 &= f(x_n, y_n) \\ k_2 &= f\left(x_n + \frac{h}{2}, y_n + \frac{h}{2}k_1\right) \\ k_3 &= f\left(x_n + \frac{h}{2}, y_n + \frac{h}{2}k_2\right) \\ k_4 &= f(x_n + h, y_n + h k_3) \\ y_{n+1} &= y_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4) \end{aligned}}$$

```
  Slope evaluations within one step [xₙ, xₙ + h]:

  │ k₁ (start)
  │    │
  │    │ k₂ (midpoint, using k₁)
  │    │    │
  │    │    │ k₃ (midpoint, using k₂)
  │    │    │    │
  │    │    │    │ k₄ (endpoint, using k₃)
  │    │    │    │
  ┼────┼────┼────┼────
  xₙ   xₙ+h/2  xₙ+h

  Final slope = weighted average: (k₁ + 2k₂ + 2k₃ + k₄)/6
                                   ↑    ↑     ↑     ↑
                               Simpson's 1/3 weights!
```

---

## 4. Worked Example

Solve $y' = y - x^2 + 1$, $y(0) = 0.5$, find $y(0.2)$ with $h = 0.2$.

**Step**: $x_0 = 0, y_0 = 0.5, h = 0.2$

$k_1 = f(0, 0.5) = 0.5 - 0 + 1 = 1.5$

$k_2 = f(0.1, 0.5 + 0.1 \times 1.5) = f(0.1, 0.65) = 0.65 - 0.01 + 1 = 1.64$

$k_3 = f(0.1, 0.5 + 0.1 \times 1.64) = f(0.1, 0.664) = 0.664 - 0.01 + 1 = 1.654$

$k_4 = f(0.2, 0.5 + 0.2 \times 1.654) = f(0.2, 0.8308) = 0.8308 - 0.04 + 1 = 1.7908$

$$y_1 = 0.5 + \frac{0.2}{6}(1.5 + 2(1.64) + 2(1.654) + 1.7908)$$

$$= 0.5 + \frac{0.2}{6}(1.5 + 3.28 + 3.308 + 1.7908)$$

$$= 0.5 + \frac{0.2}{6}(9.8788) = 0.5 + 0.32929 = 0.82929$$

**Exact:** $y(0.2) = (1+0.2)^2 + 0.5e^{0.2} - 1 = 0.82930$ ← Matches to 5 significant figures!

---

## 5. Error Analysis

| Method | Order | Local Error | Global Error | $f$-evals/step |
|--------|-------|-------------|--------------|----------------|
| RK1 (Euler) | 1 | $O(h^2)$ | $O(h)$ | 1 |
| RK2 | 2 | $O(h^3)$ | $O(h^2)$ | 2 |
| RK3 | 3 | $O(h^4)$ | $O(h^3)$ | 3 |
| **RK4** | **4** | **$O(h^5)$** | **$O(h^4)$** | **4** |
| RK5 (Fehlberg) | 5 | $O(h^6)$ | $O(h^5)$ | 6 |

> **RK4 is optimal**: for methods up to order 4, an order-$p$ method needs $p$ function evaluations. Beyond order 4, more evaluations are needed per order gained. This is why RK4 is the "sweet spot."

---

## 6. Adaptive Step Size (RK45 / Runge-Kutta-Fehlberg)

Compute both a 4th and 5th order estimate at each step. The difference estimates the local error:

$$\text{error} \approx |y_5 - y_4|$$

- If error > tolerance: **reject** and reduce $h$
- If error < tolerance: **accept** and try larger $h$

$$h_{\text{new}} = h \cdot \left(\frac{\text{tol}}{\text{error}}\right)^{1/5}$$

This is the basis of modern ODE solvers (e.g., MATLAB's `ode45`).

---

## 7. Systems of ODEs

For a system $\mathbf{y}' = \mathbf{f}(x, \mathbf{y})$, apply RK4 component-wise:

$$\mathbf{k}_1 = \mathbf{f}(x_n, \mathbf{y}_n)$$

$$\mathbf{k}_2 = \mathbf{f}(x_n + h/2, \mathbf{y}_n + (h/2)\mathbf{k}_1)$$

etc. Higher-order ODEs are converted to systems: $y'' = g(x,y,y')$ becomes $y_1' = y_2, \; y_2' = g(x,y_1,y_2)$.

---

## Summary Table

| Property | RK4 Value |
|----------|-----------|
| **Formula** | $y_{n+1} = y_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)$ |
| **Order** | 4 |
| **Local error** | $O(h^5)$ |
| **Global error** | $O(h^4)$ |
| **$f$-evals** | 4 per step |
| **Self-starting** | Yes (single-step method) |
| **Adaptive** | RK45 (Fehlberg) for automatic step control |

---

## Quick Revision Questions

1. **Write the complete RK4 algorithm with all four $k$-values.**

2. **Solve $y' = x + y$, $y(0) = 1$ for one step of $h = 0.1$ using RK4.**

3. **Why is RK4 considered the "optimal" Runge-Kutta method?**

4. **How does the Runge-Kutta-Fehlberg (RK45) method achieve adaptive step sizing?**

5. **Convert $y'' + 3y' + 2y = 0$ into a system suitable for RK4.**

6. **Compare the efficiency of RK2 and RK4 for achieving a given accuracy.**

---

[← Previous: Modified Euler](02-modified-euler.md) | [Next: Adams-Bashforth →](04-adams-bashforth.md)
