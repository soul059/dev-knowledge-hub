# Chapter 1: Definition — Riemann Sums

[← Previous: Reduction Formulas](../03-Special-Integrals/04-reduction-formulas.md) | [Back to README](../README.md) | [Next: Fundamental Theorem of Calculus →](02-fundamental-theorem.md)

---

## 1.1 Overview

The **definite integral** is defined as the **limit of Riemann sums** — a process of approximating the area under a curve by summing the areas of many thin rectangles. This gives integration a rigorous foundation independent of antiderivatives.

---

## 1.2 The Area Problem

**Goal:** Find the exact area under a curve y = f(x) from x = a to x = b, where f(x) ≥ 0.

```
  y
  │
  │     ╱──╲
  │    ╱    ╲        f(x)
  │   ╱      ╲
  │  ╱ Area=? ╲
  │ ╱          ╲
  │╱            ╲
  └──┬──────────┬──── x
     a          b
```

**Idea:** Divide [a, b] into n thin strips and approximate each strip with a rectangle.

---

## 1.3 Constructing Riemann Sums

### Step 1: Partition the Interval

Divide [a, b] into n subintervals of width:

$$\Delta x = \frac{b - a}{n}$$

The partition points are:

$$x_0 = a, \quad x_1 = a + \Delta x, \quad x_2 = a + 2\Delta x, \quad \ldots, \quad x_n = b$$

### Step 2: Choose Sample Points

In each subinterval [xᵢ₋₁, xᵢ], choose a sample point xᵢ* (left endpoint, right endpoint, midpoint, or any point).

### Step 3: Form the Sum

$$S_n = \sum_{i=1}^{n} f(x_i^*) \cdot \Delta x$$

### Visual Representation (n = 4, right endpoints)

```
  y
  │    ┌──┐
  │    │  │┌──┐
  │ ┌──┤  ││  │
  │ │  │  ││  │ ┌──┐
  │ │  │  ││  │ │  │
  │ │  │  ││  │ │  │  f(x)
  ──┴──┴──┴┴──┴─┴──┴──── x
  a  x₁  x₂  x₃  x₄=b
  
  Area ≈ f(x₁)Δx + f(x₂)Δx + f(x₃)Δx + f(x₄)Δx
```

---

## 1.4 Types of Riemann Sums

| Type | Sample Point xᵢ* | Tendency |
|---|---|---|
| **Left Riemann Sum** | xᵢ* = xᵢ₋₁ (left endpoint) | Underestimate if f is increasing |
| **Right Riemann Sum** | xᵢ* = xᵢ (right endpoint) | Overestimate if f is increasing |
| **Midpoint Sum** | xᵢ* = (xᵢ₋₁ + xᵢ)/2 | Generally more accurate |

### Comparison Diagram (for increasing f)

```
  Left endpoints:         Right endpoints:        Midpoint:
  
  │   ╱                   │   ╱                    │   ╱
  │ ┌╱┐                   │  ╱┌─┐                  │ ╱┌─┐
  │─┤╱ ├─┐                │ ╱ │ ├─┐                │╱ │·├─┐
  │ │  │ ├─╱              │╱  │ │ ├──              │  │ │·├──
  ├─┤  │ │╱               ├─┐ │ │ │               ├──┤ │ │
  └─┴──┴─┴──              └─┴─┴─┴─┴──             └──┴─┴─┴──
  Underestimate           Overestimate             Better approximation
```

---

## 1.5 The Definite Integral as a Limit

> **Definition:** The **definite integral** of f from a to b is:
>
> $$\int_a^b f(x)\, dx = \lim_{n \to \infty} \sum_{i=1}^{n} f(x_i^*) \cdot \Delta x$$
>
> provided this limit exists (in which case f is called **integrable** on [a, b]).

### Key Facts

- If f is **continuous** on [a, b], then f is integrable.
- If f is **bounded** with finitely many discontinuities on [a, b], then f is integrable.
- The choice of sample points does not affect the limit (for integrable functions).

---

## 1.6 Computing with Riemann Sums — Formulas

For right-endpoint Riemann sums with equal subintervals:

$$\int_a^b f(x)\, dx = \lim_{n \to \infty} \sum_{i=1}^{n} f\left(a + i\cdot\frac{b-a}{n}\right) \cdot \frac{b-a}{n}$$

### Useful Summation Formulas

| Sum | Formula |
|---|---|
| $\sum_{i=1}^{n} 1$ | n |
| $\sum_{i=1}^{n} i$ | n(n+1)/2 |
| $\sum_{i=1}^{n} i^2$ | n(n+1)(2n+1)/6 |
| $\sum_{i=1}^{n} i^3$ | [n(n+1)/2]² |

---

## 1.7 Worked Example: ∫₀² x² dx from the Definition

$$\int_0^2 x^2\, dx = \lim_{n\to\infty} \sum_{i=1}^{n} f\left(\frac{2i}{n}\right) \cdot \frac{2}{n}$$

Here a = 0, b = 2, Δx = 2/n, xᵢ = 2i/n:

$$= \lim_{n\to\infty} \sum_{i=1}^{n} \left(\frac{2i}{n}\right)^2 \cdot \frac{2}{n} = \lim_{n\to\infty} \frac{8}{n^3}\sum_{i=1}^{n} i^2$$

$$= \lim_{n\to\infty} \frac{8}{n^3} \cdot \frac{n(n+1)(2n+1)}{6}$$

$$= \lim_{n\to\infty} \frac{8(n+1)(2n+1)}{6n^2} = \lim_{n\to\infty} \frac{8(2n^2 + 3n + 1)}{6n^2}$$

$$= \frac{8 \cdot 2}{6} = \frac{16}{6} = \frac{8}{3}$$

**Verification:** Using the antiderivative: [x³/3]₀² = 8/3 ✓

---

## 1.8 Signed Area

The definite integral gives the **net signed area**:

- f(x) > 0: area counts as **positive** (above x-axis)
- f(x) < 0: area counts as **negative** (below x-axis)

```
  y
  │    ╱╲      (+)
  │   ╱  ╲
  ├──╱────╲──────── x
  │        ╲  ╱
  │    (−)  ╲╱
  
  ∫ₐᵇ f(x) dx = (positive area) − (negative area)
```

---

## 1.9 Interpretation Diagram

```
  ∫ₐᵇ f(x) dx  ──►  What does it compute?
     │
     ├── If f(x) ≥ 0:  Area under curve above x-axis
     │
     ├── If f(x) ≤ 0:  Negative of area below x-axis
     │
     ├── If f changes sign:  Net signed area
     │
     └── In physics:  displacement, work, total change, etc.
```

---

## Summary Table

| Concept | Key Point |
|---|---|
| Partition | Divide [a, b] into n subintervals of width Δx = (b−a)/n |
| Riemann sum | Sₙ = Σ f(xᵢ*) · Δx |
| Definite integral | ∫ₐᵇ f(x) dx = lim(n→∞) Sₙ |
| Left/Right/Midpoint | Different choices of sample points |
| Integrability | Continuous ⟹ integrable |
| Signed area | Above x-axis (+), below x-axis (−) |
| Computing from def. | Use summation formulas for Σi, Σi², Σi³ |

---

## Quick Revision Questions

1. Define the definite integral using Riemann sums.

2. What is the width Δx if you partition [1, 5] into 100 subintervals?

3. Why does ∫₋₁¹ x dx = 0? Explain using the concept of signed area.

4. Compute ∫₀³ x dx using the definition (right Riemann sum, limit as n → ∞).

5. State the condition under which a function is guaranteed to be Riemann integrable.

6. Explain the difference between left, right, and midpoint Riemann sums with a diagram.

---

[← Previous: Reduction Formulas](../03-Special-Integrals/04-reduction-formulas.md) | [Back to README](../README.md) | [Next: Fundamental Theorem of Calculus →](02-fundamental-theorem.md)
