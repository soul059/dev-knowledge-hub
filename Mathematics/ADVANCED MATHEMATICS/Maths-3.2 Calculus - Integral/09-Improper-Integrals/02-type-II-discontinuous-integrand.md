# Chapter 2: Type II — Improper Integrals with Discontinuous Integrands

[← Previous: Type I — Infinite Limits](01-type-I-infinite-limits.md) | [Back to README](../README.md) | [Next: Convergence Tests →](03-convergence-tests.md)

---

## 2.1 Overview

**Type II improper integrals** arise when the integrand has a vertical asymptote (infinite discontinuity) at or within the interval of integration. The function "blows up" at some point, and we handle this using limits.

---

## 2.2 Definition

### Discontinuity at an endpoint

$$\int_a^b f(x)\, dx = \lim_{t \to a^+} \int_t^b f(x)\, dx \quad \text{(singularity at } x = a\text{)}$$

$$\int_a^b f(x)\, dx = \lim_{t \to b^-} \int_a^t f(x)\, dx \quad \text{(singularity at } x = b\text{)}$$

### Discontinuity in the interior

If f has a singularity at c ∈ (a, b):

$$\int_a^b f(x)\, dx = \int_a^c f(x)\, dx + \int_c^b f(x)\, dx$$

where **both** are improper integrals evaluated with limits.

```
  │           │           │
  │  ╱        │∞          │        ╲
  │ ╱         ││          │         ╲
  │╱          ││          │          ╲
  ├───────────┼┼──────────┼──────────── x
  a           c           b
  
  Singularity at x = c (interior point)
  Must split: ∫ₐᶜ + ∫ᶜᵇ, BOTH as limits
```

---

## 2.3 The p-Integral at Zero

> $$\int_0^1 \frac{1}{x^p}\, dx = \begin{cases} \frac{1}{1-p} & \text{if } p < 1 \quad \text{(CONVERGES)} \\[6pt] \infty & \text{if } p \geq 1 \quad \text{(DIVERGES)} \end{cases}$$

### Comparison with Type I

```
  ┌───────────────────────────────────────┐
  │     TYPE I           TYPE II          │
  │   ∫₁^∞ 1/xᵖ dx    ∫₀¹ 1/xᵖ dx       │
  ├───────────────────────────────────────┤
  │   p > 1: CONV      p < 1: CONV       │
  │   p ≤ 1: DIV       p ≥ 1: DIV        │
  │                                       │
  │   Threshold: p = 1 (BOTH diverge)     │
  │   p > 1: converges at ∞, diverges at 0│
  │   p < 1: diverges at ∞, converges at 0│
  └───────────────────────────────────────┘
  
  They have OPPOSITE behavior!
```

---

## 2.4 Worked Examples

### Example 1: Convergent at Left Endpoint

$$\int_0^1 \frac{1}{\sqrt{x}}\, dx = \lim_{t\to 0^+}\int_t^1 x^{-1/2}\, dx = \lim_{t\to 0^+} [2\sqrt{x}]_t^1 = 2 - 0 = 2$$

(This is p = 1/2 < 1, so it converges.)

---

### Example 2: Divergent at Left Endpoint

$$\int_0^1 \frac{1}{x}\, dx = \lim_{t\to 0^+}[\ln x]_t^1 = 0 - (-\infty) = \infty$$

(This is p = 1, diverges.)

---

### Example 3: Interior Singularity

$$\int_0^3 \frac{1}{(x-1)^{2/3}}\, dx$$

Singularity at x = 1. Split:

$$= \int_0^1 \frac{dx}{(x-1)^{2/3}} + \int_1^3 \frac{dx}{(x-1)^{2/3}}$$

First integral: let u = 1 − x:

$$\int_0^1 \frac{dx}{(1-x)^{2/3}} = \lim_{t\to 1^-}\left[-3(1-x)^{1/3}\right]_0^t = 0 - (-3) = 3$$

Second integral:

$$\int_1^3 \frac{dx}{(x-1)^{2/3}} = \lim_{t\to 1^+}\left[3(x-1)^{1/3}\right]_t^3 = 3(2)^{1/3} - 0 = 3\sqrt[3]{2}$$

$$\boxed{\int_0^3 \frac{dx}{(x-1)^{2/3}} = 3 + 3\sqrt[3]{2}}$$

---

### Example 4: Hidden Singularity

$$\int_0^{\pi/2} \tan x\, dx = \lim_{t\to(\pi/2)^-} [-\ln|\cos x|]_0^t = \infty$$

**Diverges!** Even though tan x looks fine on most of [0, π/2], it blows up at x = π/2.

---

### Example 5: Doubly Improper (Type I + Type II)

$$\int_0^{\infty} \frac{1}{\sqrt{x}(1+x)}\, dx$$

Split at x = 1 (Type II at x = 0, Type I at ∞):

$$= \int_0^1 \frac{dx}{\sqrt{x}(1+x)} + \int_1^{\infty} \frac{dx}{\sqrt{x}(1+x)}$$

Substituting u = √x (so x = u², dx = 2u du):

$$\int \frac{2u\, du}{u(1+u^2)} = 2\int\frac{du}{1+u^2} = 2\arctan u = 2\arctan\sqrt{x}$$

$$= [2\arctan\sqrt{x}]_0^{\infty} = 2 \cdot \frac{\pi}{2} - 0 = \pi$$

$$\boxed{\int_0^{\infty} \frac{dx}{\sqrt{x}(1+x)} = \pi}$$

---

## 2.5 Common Type II Situations

| Integrand | Singularity at | Converges? |
|---|---|---|
| 1/√x = x⁻¹/² | x = 0 | Yes (p=1/2<1) |
| 1/x | x = 0 | No (p=1) |
| 1/x² | x = 0 | No (p=2>1) |
| ln x | x = 0 | Yes (∫₀¹ ln x dx = −1) |
| tan x | x = π/2 | No |
| 1/(1−x)^{1/3} | x = 1 | Yes (p=1/3<1) |

---

## 2.6 Identifying Improper Integrals

```
  Given ∫ₐᵇ f(x) dx, check:
  ═══════════════════════════
  
  1. Are a or b infinite? → TYPE I
  
  2. Is f discontinuous at a, b, or  
     any point c in (a,b)?
     → TYPE II
     
  3. BOTH? → Split and handle each part
  
  ⚠ COMMON TRAP: Not recognizing interior
     singularities!
     
     ∫₋₁¹ 1/x² dx ≠ [-1/x]₋₁¹ = -2
     
     WRONG! There's a singularity at x = 0.
     Split into ∫₋₁⁰ + ∫₀¹ → both diverge!
```

---

## Summary Table

| Concept | Key Point |
|---|---|
| Type II definition | Integrand has ∞ discontinuity in [a,b] |
| At endpoint | Replace endpoint with limit |
| Interior singularity | MUST split into two integrals |
| p-test at 0: ∫₀¹ 1/xᵖ | Converges iff p < 1 |
| Opposite to Type I | Type I: p > 1 conv; Type II: p < 1 conv |
| Hidden singularities | Always check for division by zero |

---

## Quick Revision Questions

1. Evaluate ∫₀¹ 1/x^{1/3} dx.

2. Show that ∫₀¹ ln x dx converges and find its value.

3. Why is ∫₋₁¹ 1/x² dx not equal to [−1/x]₋₁¹ = −2?

4. Evaluate ∫₀⁴ 1/√(4−x) dx.

5. Classify and evaluate ∫₀^∞ e⁻√ˣ/√x dx. (Hint: what type of improper integral is this?)

6. State the p-test for both Type I (∫₁^∞) and Type II (∫₀¹) and explain why the convergence conditions are reversed.

---

[← Previous: Type I — Infinite Limits](01-type-I-infinite-limits.md) | [Back to README](../README.md) | [Next: Convergence Tests →](03-convergence-tests.md)
