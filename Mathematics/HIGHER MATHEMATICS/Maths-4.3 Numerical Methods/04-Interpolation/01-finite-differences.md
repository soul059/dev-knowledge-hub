# Chapter 4.1: Finite Differences

[← Previous: Relaxation Methods](../03-System-of-Equations/06-relaxation-methods.md) | [Next: Newton's Forward Difference →](02-newton-forward.md)

---

## Overview

**Finite differences** are discrete analogs of derivatives. They form the foundation for interpolation formulas, numerical differentiation, and numerical integration. Given a table of equally spaced data points, finite differences organize the data into a powerful triangular table from which polynomials can be constructed.

---

## 1. Notation and Setup

Given data points at **equally spaced** arguments:

$$x_0, \; x_1 = x_0 + h, \; x_2 = x_0 + 2h, \; \ldots, \; x_n = x_0 + nh$$

with corresponding function values $f_0, f_1, f_2, \ldots, f_n$ (where $f_i = f(x_i)$).

The spacing $h$ is called the **step size** or **interval**.

---

## 2. Types of Finite Differences

### Forward Differences ($\Delta$)

$$\Delta f_i = f_{i+1} - f_i \quad \text{(first forward difference)}$$

$$\Delta^2 f_i = \Delta f_{i+1} - \Delta f_i \quad \text{(second forward difference)}$$

$$\Delta^n f_i = \Delta^{n-1} f_{i+1} - \Delta^{n-1} f_i$$

### Backward Differences ($\nabla$)

$$\nabla f_i = f_i - f_{i-1} \quad \text{(first backward difference)}$$

$$\nabla^2 f_i = \nabla f_i - \nabla f_{i-1}$$

### Central Differences ($\delta$)

$$\delta f_i = f_{i+1/2} - f_{i-1/2}$$

$$\delta^2 f_i = f_{i+1} - 2f_i + f_{i-1}$$

### Shift Operator ($E$)

$$E f_i = f_{i+1}, \quad E^n f_i = f_{i+n}$$

---

## 3. Operator Relations

| Relation | Formula |
|----------|---------|
| $E = 1 + \Delta$ | $Ef_i = f_i + \Delta f_i = f_{i+1}$ |
| $\Delta = E - 1$ | |
| $\nabla = 1 - E^{-1}$ | |
| $\delta = E^{1/2} - E^{-1/2}$ | |
| $E = e^{hD}$ | where $D = d/dx$ (Taylor's) |
| $\Delta = hD + \frac{h^2D^2}{2!} + \cdots$ | |

---

## 4. Forward Difference Table

```
  x     f       Δf       Δ²f      Δ³f      Δ⁴f
  ─── ─────── ──────── ──────── ──────── ────────
  x₀    f₀
              Δf₀
  x₁    f₁          Δ²f₀
              Δf₁          Δ³f₀
  x₂    f₂          Δ²f₁          Δ⁴f₀
              Δf₂          Δ³f₁
  x₃    f₃          Δ²f₂
              Δf₃
  x₄    f₄
```

### Worked Example

Given: $f(0) = 1, f(1) = 3, f(2) = 7, f(3) = 13, f(4) = 21$

| $x$ | $f$ | $\Delta f$ | $\Delta^2 f$ | $\Delta^3 f$ | $\Delta^4 f$ |
|:---:|:---:|:----------:|:-------------:|:-------------:|:-------------:|
| 0 | 1 | | | | |
| | | 2 | | | |
| 1 | 3 | | 2 | | |
| | | 4 | | 0 | |
| 2 | 7 | | 2 | | 0 |
| | | 6 | | 0 | |
| 3 | 13 | | 2 | | |
| | | 8 | | | |
| 4 | 21 | | | | |

Since $\Delta^2 f$ is constant (= 2), the data fits a **quadratic** polynomial exactly.

---

## 5. Key Properties

### Property 1: nth Difference of a Polynomial of Degree $n$

If $f(x)$ is a polynomial of degree $n$, then $\Delta^n f$ is **constant** and $\Delta^{n+1} f = 0$.

### Property 2: Explicit Formula

$$\Delta^n f_0 = \sum_{k=0}^{n} (-1)^{n-k} \binom{n}{k} f_k$$

### Property 3: Linear Operator

$$\Delta(af + bg) = a\Delta f + b\Delta g$$

### Property 4: Product Rule (Leibniz-like)

$$\Delta(f_i \cdot g_i) = f_i \Delta g_i + g_{i+1} \Delta f_i$$

---

## 6. Backward Difference Table

```
  x     f       ∇f       ∇²f      ∇³f      ∇⁴f
  ─── ─────── ──────── ──────── ──────── ────────
  x₀    f₀
              ∇f₁
  x₁    f₁          ∇²f₂
              ∇f₂          ∇³f₃
  x₂    f₂          ∇²f₃          ∇⁴f₄
              ∇f₃          ∇³f₄
  x₃    f₃          ∇²f₄
              ∇f₄
  x₄    f₄

  Note: ∇fₙ = Δfₙ₋₁ (same values, different labeling)
```

---

## 7. Relationship Between Differences

| Forward | Backward | Central |
|---------|----------|---------|
| $\Delta f_0 = f_1 - f_0$ | $\nabla f_1 = f_1 - f_0$ | $\delta f_{1/2} = f_1 - f_0$ |
| $\Delta^2 f_0 = f_2 - 2f_1 + f_0$ | $\nabla^2 f_2 = f_2 - 2f_1 + f_0$ | $\delta^2 f_1 = f_2 - 2f_1 + f_0$ |

All three are different **notations** for the same underlying differences!

---

## 8. Error Detection Using Differences

If the data comes from a smooth function, the differences should decrease smoothly. An **irregular bump** in the difference table indicates a data error.

```
  Δ²f:   2   2   2   9   2   2
                      ↑
                   DATA ERROR!
                   (should be ≈ 2)
```

---

## Summary Table

| Concept | Symbol | Formula |
|---------|:------:|---------|
| Forward difference | $\Delta$ | $\Delta f_i = f_{i+1} - f_i$ |
| Backward difference | $\nabla$ | $\nabla f_i = f_i - f_{i-1}$ |
| Central difference | $\delta$ | $\delta f_i = f_{i+1/2} - f_{i-1/2}$ |
| Shift operator | $E$ | $Ef_i = f_{i+1}$ |
| Relation | — | $\Delta = E - 1$, $\nabla = 1 - E^{-1}$ |
| Degree $n$ polynomial | — | $\Delta^{n+1} f = 0$ |

---

## Quick Revision Questions

1. **Construct the forward difference table for $f(x) = x^3$ at $x = 0, 1, 2, 3, 4$.**

2. **Show that $\Delta = E - 1$ and $\nabla = 1 - E^{-1}$.**

3. **Prove that the $n$th forward difference of a polynomial of degree $n$ is constant.**

4. **How can a difference table be used to detect errors in tabulated data?**

5. **Express $\Delta^3 f_0$ in terms of $f_0, f_1, f_2, f_3$.**

6. **What is the relationship between $\Delta f_i$ and $\nabla f_{i+1}$?**

---

[← Previous: Relaxation Methods](../03-System-of-Equations/06-relaxation-methods.md) | [Next: Newton's Forward Difference →](02-newton-forward.md)
