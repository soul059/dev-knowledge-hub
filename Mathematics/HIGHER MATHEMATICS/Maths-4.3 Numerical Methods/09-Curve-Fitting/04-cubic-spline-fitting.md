# Chapter 9.4: Cubic Spline Fitting

[← Previous: Exponential & Logarithmic Fit](03-exponential-fit.md) | [Next: Classification of PDEs →](../10-Partial-DEs/01-classification.md)

---

## Overview

**Cubic spline fitting** combines curve fitting with spline interpolation. Unlike interpolation splines (which pass through every point), a **smoothing spline** balances closeness to data against smoothness, controlled by a parameter $\lambda$.

---

## 1. Interpolating Cubic Spline (Recap)

For $n$ data points $(x_1,y_1),\ldots,(x_n,y_n)$, an interpolating cubic spline $S(x)$ satisfies:

- $S(x_i) = y_i$ for all $i$ (passes through every point)
- $S$ is a cubic polynomial on each $[x_i, x_{i+1}]$
- $S$, $S'$, $S''$ are continuous across the entire interval
- Two boundary conditions (natural, clamped, etc.)

This works well for exact data but **overfits noisy data**.

---

## 2. Smoothing Spline

For noisy data, we minimise a **penalised objective**:

$$\boxed{J(\lambda) = \sum_{i=1}^{n} \left(\frac{y_i - S(x_i)}{\sigma_i}\right)^2 + \lambda \int_{x_1}^{x_n} [S''(x)]^2 \, dx}$$

| Term | Role |
|------|------|
| $\sum(y_i-S(x_i))^2/\sigma_i^2$ | **Fidelity** — stay close to data |
| $\lambda\int [S''(x)]^2 dx$ | **Penalty** — penalise curvature (wiggles) |
| $\lambda$ | **Smoothing parameter** — controls trade-off |

```
  λ → 0                   λ moderate              λ → ∞
  ┌──────────┐           ┌──────────┐           ┌──────────┐
  │  ●  ●    │           │  ● ●     │           │          │
  │ /\/ \●   │           │  / \_●   │           │  ────────│ ← straight line
  │●     \   │           │ ●    \   │           │ ●  ●  ●  │
  │  wiggly  │           │ balanced │           │  oversmooth│
  └──────────┘           └──────────┘           └──────────┘
  Interpolation          Good fit               Linear regression
```

---

## 3. Choosing $\lambda$

| Method | Description |
|--------|-------------|
| **Cross-validation (CV)** | Leave-one-out prediction error |
| **Generalised CV (GCV)** | $\text{GCV}(\lambda) = \frac{n^{-1}\|(\mathbf{I}-\mathbf{A})\mathbf{y}\|^2}{[n^{-1}\text{tr}(\mathbf{I}-\mathbf{A})]^2}$ |
| **AIC / BIC** | Information criteria with effective df |
| **Visual** | Plot spline for several $\lambda$ values |

The **effective degrees of freedom**: $\text{df} = \text{tr}(\mathbf{A}_\lambda)$ where $\mathbf{A}_\lambda$ is the smoother matrix.

---

## 4. B-Spline Regression

Instead of one cubic per interval, use a **basis of B-splines**:

$$S(x) = \sum_{j=1}^{K+4} c_j B_j(x)$$

where $B_j$ are cubic B-spline basis functions over $K$ knots.

### Fitting with Penalty (P-spline)

$$\min_{\mathbf{c}} \|\mathbf{y} - \mathbf{B}\mathbf{c}\|^2 + \lambda \|\mathbf{D}_d \mathbf{c}\|^2$$

where $\mathbf{D}_d$ is a $d$-th order difference matrix on coefficients.

Solution: $\mathbf{c} = (\mathbf{B}^T\mathbf{B} + \lambda \mathbf{D}_d^T\mathbf{D}_d)^{-1}\mathbf{B}^T\mathbf{y}$

---

## 5. Worked Example: Smoothing Spline Concept

Given noisy data from $y = \sin(x)$ with 6 points:

| $x$ | 0 | 1.26 | 2.51 | 3.77 | 5.03 | 6.28 |
|------|---|------|------|------|------|------|
| $y$ | 0.1 | 0.88 | 0.72 | −0.48 | −1.05 | 0.12 |

With **$\lambda = 0$**: Interpolating spline passes through every point (overfits noise).

With **$\lambda \to \infty$**: Best-fit line $y = -0.06 - 0.04x$ (underfits).

With **optimal $\lambda$** (found by GCV): Smooth curve closely resembling $\sin(x)$.

### The Trade-off

| $\lambda$ | df | $\sum(y-S)^2$ | $\int(S'')^2dx$ | Behavior |
|-----------|-----|--------------|-----------------|----------|
| 0 | 6 | 0 | Large | Interpolation |
| 0.01 | 4.8 | 0.02 | Moderate | Slight smoothing |
| 1.0 | 3.1 | 0.15 | Small | Good balance |
| 100 | 2.0 | 0.42 | ≈ 0 | Nearly linear |
| $\infty$ | 2 | 0.48 | 0 | Straight line |

---

## 6. Comparison: Fitting Methods

| Method | Passes through data? | Smoothness control | Handles noise? |
|--------|---------------------|-------------------|----------------|
| Interpolating spline | ✅ Yes | $C^2$ automatic | ❌ Poor |
| Polynomial regression | ❌ No | Fixed by degree | ✅ Yes |
| Smoothing spline | ❌ No (unless $\lambda=0$) | $\lambda$ parameter | ✅ Excellent |
| B-spline / P-spline | ❌ No | $\lambda$ + knots | ✅ Excellent |
| LOESS / LOWESS | ❌ No | Bandwidth | ✅ Yes |

---

## 7. Natural Smoothing Spline: Key Properties

The minimiser of $J(\lambda)$ is always a **natural cubic spline** with knots at the data points. Key properties:

1. Solution exists and is unique for all $\lambda > 0$
2. As $\lambda \to 0$: interpolating natural cubic spline
3. As $\lambda \to \infty$: linear least squares line
4. Computation: solve a banded system of size $n$ (efficient for large $n$)

---

## Summary Table

| Property | Detail |
|----------|--------|
| **Objective** | $\sum(y_i-S(x_i))^2 + \lambda\int[S''(x)]^2dx$ |
| **$\lambda = 0$** | Interpolation (passes through all points) |
| **$\lambda \to \infty$** | Straight line (maximum smoothness) |
| **Optimal $\lambda$** | Found by GCV or cross-validation |
| **Effective df** | $\text{tr}(\mathbf{A}_\lambda)$, between 2 and $n$ |
| **B-spline / P-spline** | Basis expansion with difference penalty |
| **Computation** | Banded linear system, $O(n)$ |

---

## Quick Revision Questions

1. **What is the objective function for a smoothing spline? Explain each term.**

2. **How does the smoothing parameter $\lambda$ control the bias-variance trade-off?**

3. **What happens as $\lambda \to 0$ and $\lambda \to \infty$?**

4. **What is Generalised Cross-Validation (GCV) and why is it used?**

5. **Compare interpolating splines and smoothing splines for noisy data.**

6. **What is a P-spline? How does it differ from a smoothing spline?**

---

[← Previous: Exponential & Logarithmic Fit](03-exponential-fit.md) | [Next: Classification of PDEs →](../10-Partial-DEs/01-classification.md)
