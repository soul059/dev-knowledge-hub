# Chapter 7.5: Milne's Predictor-Corrector Method

[← Previous: Adams-Bashforth](04-adams-bashforth.md) | [Next: Boundary Value Problems →](06-boundary-value-problems.md)

---

## Overview

**Milne's method** is a popular predictor-corrector method for solving ODEs. It uses 4 previous values in the predictor and 3 in the corrector. While efficient, it can suffer from **instability** for certain problems.

---

## 1. The Formulas

### Predictor (Milne's Predictor)

$$\boxed{y^*_{n+1} = y_{n-3} + \frac{4h}{3}(2f_{n} - f_{n-1} + 2f_{n-2})}$$

### Corrector (Simpson's Corrector)

$$\boxed{y_{n+1} = y_{n-1} + \frac{h}{3}(f^*_{n+1} + 4f_n + f_{n-1})}$$

where $f^*_{n+1} = f(x_{n+1}, y^*_{n+1})$.

---

## 2. Algorithm

```
  Given: y₀, y₁, y₂, y₃ (from RK4 or exact solution)

  For each step:
  ┌──────────────────────────────────────────────────┐
  │ 1. PREDICT: y*ₙ₊₁ = yₙ₋₃ + (4h/3)(2fₙ - fₙ₋₁ + 2fₙ₋₂) │
  │ 2. EVALUATE: f*ₙ₊₁ = f(xₙ₊₁, y*ₙ₊₁)          │
  │ 3. CORRECT: yₙ₊₁ = yₙ₋₁ + (h/3)(f*ₙ₊₁ + 4fₙ + fₙ₋₁) │
  │ 4. (Optional) Re-evaluate and re-correct        │
  └──────────────────────────────────────────────────┘
```

---

## 3. Error Analysis

| Component | Error |
|-----------|-------|
| Predictor | $\frac{14h^5}{45}y^{(5)}(\xi)$ |
| Corrector | $-\frac{h^5}{90}y^{(5)}(\xi)$ |

The **corrector** is about 28 times more accurate than the predictor.

### Error Estimate (Milne's Device)

$$E \approx \frac{1}{29}(y_C - y_P)$$

If $|y_C - y_P|$ is small, the result is reliable. If large, reduce $h$.

---

## 4. Worked Example

Solve $y' = x + y$, $y(0) = 1$, with $h = 0.1$.

Starting values (from RK4 or exact $y = 2e^x - x - 1$):

| $i$ | $x_i$ | $y_i$ | $f_i = x_i + y_i$ |
|-----|--------|--------|-------------------|
| 0 | 0.0 | 1.0000 | 1.0000 |
| 1 | 0.1 | 1.1103 | 1.2103 |
| 2 | 0.2 | 1.2428 | 1.4428 |
| 3 | 0.3 | 1.3997 | 1.6997 |

**Find $y_4$ at $x = 0.4$:**

**Predict:**
$$y^*_4 = y_0 + \frac{4(0.1)}{3}(2f_3 - f_2 + 2f_1)$$
$$= 1.0000 + \frac{0.4}{3}(2(1.6997) - 1.4428 + 2(1.2103))$$
$$= 1.0000 + 0.13333(3.3994 - 1.4428 + 2.4206)$$
$$= 1.0000 + 0.13333 \times 4.3772 = 1.0000 + 0.5836 = 1.5836$$

**Evaluate:** $f^*_4 = 0.4 + 1.5836 = 1.9836$

**Correct:**
$$y_4 = y_2 + \frac{0.1}{3}(f^*_4 + 4f_3 + f_2)$$
$$= 1.2428 + 0.03333(1.9836 + 4(1.6997) + 1.4428)$$
$$= 1.2428 + 0.03333(1.9836 + 6.7988 + 1.4428)$$
$$= 1.2428 + 0.03333 \times 10.2252 = 1.2428 + 0.3408 = 1.5836$$

**Exact:** $y(0.4) = 2e^{0.4} - 0.4 - 1 = 1.5836$ ✓

---

## 5. Stability Issue

Milne's method can exhibit **weak instability**: parasitic solutions can grow even when the true solution is decaying.

For $y' = \lambda y$ ($\lambda < 0$), the characteristic equation of the predictor has a root outside the unit circle for some $h\lambda$ values, causing the solution to blow up eventually.

```
  Milne's method:
    │  Solution oscillates and grows!
    │    ╱╲    ╱╲
    │   ╱  ╲  ╱  ╲   ╱
    │──╱────╲╱────╲─╱────── True solution (decaying)
    │ ╱      ╲     ╱
    │╱        ╲   ╱
    └──────────────────── x

  Adams-Moulton does NOT have this problem.
```

> **Recommendation**: For problems where $y \to 0$, prefer **Adams-Bashforth-Moulton** over Milne.

---

## 6. Milne vs Adams

| Property | Milne | Adams-Bashforth-Moulton |
|----------|-------|------------------------|
| Order | 4 | 4 |
| $f$-evals per step | 2 | 2 |
| Stability | **Weakly unstable** | Stable |
| Predictor | Open NC (uses $y_{n-3}$) | Uses $y_n$ only |
| Corrector | Simpson (uses $y_{n-1}$) | Uses $y_n$ |
| Error estimate | $\frac{1}{29}(y_C - y_P)$ | $\frac{1}{270}(y_C - y_P)$ |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Predictor** | $y_{n-3} + \frac{4h}{3}(2f_n - f_{n-1} + 2f_{n-2})$ |
| **Corrector** | $y_{n-1} + \frac{h}{3}(f^*_{n+1} + 4f_n + f_{n-1})$ |
| **Order** | 4 |
| **Starting values** | 4 needed $(y_0, y_1, y_2, y_3)$ |
| **Error estimate** | $\frac{1}{29}(y_C - y_P)$ |
| **Weakness** | Susceptible to weak instability |

---

## Quick Revision Questions

1. **State the Milne predictor and corrector formulas.**

2. **How many starting values are needed and how are they obtained?**

3. **Apply Milne's method to find $y(0.4)$ given: $y(0)=0, y(0.1)=0.01, y(0.2)=0.0410, y(0.3)=0.0941$ for $y'=x+y^2$.**

4. **What is Milne's error estimation device?**

5. **Explain the instability problem of Milne's method.**

6. **Why might Adams-Bashforth-Moulton be preferred over Milne's method?**

---

[← Previous: Adams-Bashforth](04-adams-bashforth.md) | [Next: Boundary Value Problems →](06-boundary-value-problems.md)
