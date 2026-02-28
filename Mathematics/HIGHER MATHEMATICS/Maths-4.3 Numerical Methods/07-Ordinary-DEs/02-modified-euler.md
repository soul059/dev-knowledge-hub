# Chapter 7.2: Modified Euler's Method (Heun's Method)

[← Previous: Euler's Method](01-eulers-method.md) | [Next: Runge-Kutta Methods →](03-runge-kutta.md)

---

## Overview

The **Modified Euler method** (also called **Heun's method** or the **improved Euler method**) is a second-order predictor-corrector method. It predicts with Euler's formula, then corrects by averaging slopes at both ends of the interval.

---

## 1. The Formula

**Predictor** (Euler):
$$y^*_{n+1} = y_n + h \cdot f(x_n, y_n)$$

**Corrector** (average slope):
$$\boxed{y_{n+1} = y_n + \frac{h}{2}\left[f(x_n, y_n) + f(x_{n+1}, y^*_{n+1})\right]}$$

---

## 2. Geometric Interpretation

```
  y
  │              ● y*ₙ₊₁ (predicted)
  │             ╱│
  │    slope₁  ╱ │ slope₂ = f(xₙ₊₁, y*ₙ₊₁)
  │           ╱  │
  │          ╱   ● yₙ₊₁ (corrected — using average slope)
  │         ╱  ╱
  │   (xₙ,yₙ)●╱
  │           ╱
  └──────────┼────────┼──── x
             xₙ      xₙ₊₁

  Average slope = ½(slope₁ + slope₂)
```

---

## 3. Error Analysis

| Error Type | Value |
|-----------|-------|
| **Local truncation error** | $O(h^3)$ |
| **Global truncation error** | $O(h^2)$ |

Modified Euler is a **second-order method** — one order better than basic Euler, at the cost of 2 function evaluations per step.

---

## 4. Worked Example

Solve $y' = x + y$, $y(0) = 1$, find $y(0.2)$ with $h = 0.1$.

### Step 1: $x_0 = 0, y_0 = 1$

**Predict:** $y^* = 1 + 0.1(0 + 1) = 1.1$

**Correct:** $y_1 = 1 + \frac{0.1}{2}[(0 + 1) + (0.1 + 1.1)] = 1 + 0.05[1 + 1.2] = 1 + 0.11 = 1.1100$

### Step 2: $x_1 = 0.1, y_1 = 1.1100$

**Predict:** $y^* = 1.1100 + 0.1(0.1 + 1.1100) = 1.1100 + 0.1210 = 1.2310$

**Correct:** $y_2 = 1.1100 + \frac{0.1}{2}[(0.1 + 1.1100) + (0.2 + 1.2310)]$

$= 1.1100 + 0.05[1.2100 + 1.4310] = 1.1100 + 0.05 \times 2.6410 = 1.1100 + 0.13205 = 1.24205$

| Step | $x$ | $y$ (Modified Euler) | Exact |
|------|-----|---------------------|-------|
| 0 | 0.0 | 1.0000 | 1.0000 |
| 1 | 0.1 | 1.1100 | 1.1103 |
| 2 | 0.2 | 1.2421 | 1.2428 |

Much better than basic Euler ($y(0.2) = 1.2200$)!

---

## 5. Iterated Modified Euler

The corrector can be applied **repeatedly** until convergence:

1. **Predict**: $y^{(0)} = y_n + hf(x_n, y_n)$
2. **Correct (iterate)**:
   $$y^{(k+1)} = y_n + \frac{h}{2}[f(x_n, y_n) + f(x_{n+1}, y^{(k)})]$$
3. Stop when $|y^{(k+1)} - y^{(k)}| < \epsilon$

---

## 6. Connection to Runge-Kutta

Modified Euler is actually **RK2** (2nd order Runge-Kutta):

$$k_1 = hf(x_n, y_n)$$
$$k_2 = hf(x_n + h, y_n + k_1)$$
$$y_{n+1} = y_n + \frac{1}{2}(k_1 + k_2)$$

This is identical to the predictor-corrector form above.

---

## 7. Comparison

| Method | Order | $f$-evals/step | Formula |
|--------|-------|----------------|---------|
| Euler | 1 | 1 | $y + hf$ |
| **Modified Euler** | **2** | **2** | **$y + \frac{h}{2}[f_n + f_{n+1}^*]$** |
| Midpoint | 2 | 2 | $y + hf(x+h/2, y+hf/2)$ |
| RK4 | 4 | 4 | (see next chapter) |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Predictor** | $y^* = y_n + hf(x_n, y_n)$ |
| **Corrector** | $y_{n+1} = y_n + \frac{h}{2}[f_n + f(x_{n+1}, y^*)]$ |
| **Order** | 2 |
| **Local error** | $O(h^3)$ |
| **$f$-evals** | 2 per step |
| **Also known as** | Heun's method, Improved Euler, RK2 |

---

## Quick Revision Questions

1. **Write the predictor and corrector formulas for the Modified Euler method.**

2. **Solve $y' = -2y$, $y(0) = 1$, with $h = 0.1$ for one step using Modified Euler.**

3. **Why is Modified Euler second-order while basic Euler is first-order?**

4. **Show that Modified Euler is equivalent to RK2.**

5. **Compare the accuracy of Euler and Modified Euler for the same step size and problem.**

6. **Describe the iterated Modified Euler method.**

---

[← Previous: Euler's Method](01-eulers-method.md) | [Next: Runge-Kutta Methods →](03-runge-kutta.md)
