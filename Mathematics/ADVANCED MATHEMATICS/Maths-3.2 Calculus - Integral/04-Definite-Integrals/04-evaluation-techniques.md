# Chapter 4: Evaluation Techniques for Definite Integrals

[← Previous: Properties of Definite Integrals](03-properties-definite-integrals.md) | [Back to README](../README.md) | [Next: Symmetry Properties →](../05-Properties-of-Definite-Integrals/01-symmetry-properties.md)

---

## 4.1 Overview

This chapter covers techniques specific to evaluating **definite** integrals — including substitution with changed limits, integration by parts with limits, and using symmetry/special properties.

---

## 4.2 Substitution in Definite Integrals

When using u-substitution in definite integrals, **change the limits** along with the variable:

$$\int_a^b f(g(x)) \cdot g'(x)\, dx = \int_{g(a)}^{g(b)} f(u)\, du$$

> **Key advantage:** No need to back-substitute!

### Example 1

$$\int_0^1 2x\, e^{x^2}\, dx$$

Let u = x², du = 2x dx. When x = 0 → u = 0; when x = 1 → u = 1.

$$= \int_0^1 e^u\, du = \Big[e^u\Big]_0^1 = e - 1$$

### Example 2

$$\int_0^{\pi/2} \sin^3 x \cos x\, dx$$

Let u = sin x, du = cos x dx. When x = 0 → u = 0; when x = π/2 → u = 1.

$$= \int_0^1 u^3\, du = \Big[\frac{u^4}{4}\Big]_0^1 = \frac{1}{4}$$

---

## 4.3 Integration by Parts for Definite Integrals

$$\int_a^b u\, dv = \Big[uv\Big]_a^b - \int_a^b v\, du$$

### Example

$$\int_0^1 x\, e^x\, dx$$

| u = x, dv = eˣ dx | du = dx, v = eˣ |

$$= \Big[xe^x\Big]_0^1 - \int_0^1 e^x\, dx = (1 \cdot e - 0) - \Big[e^x\Big]_0^1 = e - (e - 1) = 1$$

---

## 4.4 Using Even/Odd Symmetry

For integrals over **symmetric intervals** [−a, a]:

```
  If f(x) is EVEN: f(−x) = f(x)       If f(x) is ODD: f(−x) = −f(x)
  
  ∫₋ₐᵃ f(x) dx = 2∫₀ᵃ f(x) dx        ∫₋ₐᵃ f(x) dx = 0
  
     y                                    y
     │    ╱╲                               │     ╱
     │   ╱  ╲                              │    ╱
     │──╱    ╲──                           │   ╱
     │ ╱      ╲                            ├──╱──────── x
   ──┤╱    ╱╲  ╲                           │ ╲
     │    ╱  ╲                             │  ╲
     │   ╱    ╲                            │   ╲
     └──┴──────┴── x                       │
     -a    0    a                         -a    0    a
     Mirror image!                        Point symmetry!
```

### Example

$$\int_{-2}^{2} (x^3 + x^4)\, dx$$

x³ is odd, x⁴ is even:

$$= \int_{-2}^{2} x^3\, dx + \int_{-2}^{2} x^4\, dx = 0 + 2\int_0^2 x^4\, dx = 2 \cdot \frac{x^5}{5}\Big|_0^2 = 2 \cdot \frac{32}{5} = \frac{64}{5}$$

---

## 4.5 Special Evaluations

### ∫₀^{π/2} sinⁿx dx = ∫₀^{π/2} cosⁿx dx (Wallis)

These are equal due to the substitution x → π/2 − x:

| n | Value |
|---|---|
| 1 | 1 |
| 2 | π/4 |
| 3 | 2/3 |
| 4 | 3π/16 |
| 5 | 8/15 |
| 6 | 5π/32 |

### Gaussian-Type Integral

$$\int_0^{\infty} e^{-x^2}\, dx = \frac{\sqrt{\pi}}{2}$$

This famous result cannot be computed by finding an antiderivative; it requires special techniques (polar coordinates in double integrals).

---

## 4.6 Numerical Methods (Brief Introduction)

When exact evaluation is impossible, use numerical approximations:

| Method | Formula | Error Order |
|---|---|---|
| **Trapezoidal Rule** | (Δx/2)[f(x₀) + 2f(x₁) + ··· + 2f(xₙ₋₁) + f(xₙ)] | O(Δx²) |
| **Simpson's Rule** | (Δx/3)[f(x₀) + 4f(x₁) + 2f(x₂) + 4f(x₃) + ··· + f(xₙ)] | O(Δx⁴) |

```
  Trapezoidal:                Simpson's (parabolic arcs):
  
  │    /\                     │    ╱╲
  │   / \                     │   ╱  ╲
  │  /   \                    │  ╱    ╲
  │ / ╱╲  \                   │ ╱  ⌢   ╲     ← parabola through
  │/╱    ╲ \                  │╱   ⌣    ╲      3 consecutive points
  └──────────                 └──────────
  Straight lines              Parabolic arcs
  between points              through point triples
```

---

## 4.7 Strategy Flowchart

```
  ∫ₐᵇ f(x) dx
     │
     ├── Can you find an antiderivative?
     │       │
     │      Yes ──► FTC: F(b) − F(a)
     │       │
     │      No ──► Is there symmetry?
     │              │
     │             Yes ──► Use even/odd properties
     │              │
     │             No ──► Try substitution (change limits)
     │                    or parts
     │                    │
     │                    ├── Still can't? → Numerical methods
     │                    │ 
     │                    └── Or special known results (Wallis, etc.)
```

---

## Summary Table

| Technique | Key Point |
|---|---|
| FTC | ∫ₐᵇ f dx = F(b) − F(a) |
| Substitution | Change limits: u(a) to u(b); no back-sub needed |
| Parts | [uv]ₐᵇ − ∫ₐᵇ v du |
| Even function | ∫₋ₐᵃ f = 2∫₀ᵃ f |
| Odd function | ∫₋ₐᵃ f = 0 |
| Wallis | ∫₀^{π/2} sinⁿx dx — use reduction formula pattern |
| Numerical | Trapezoidal O(h²), Simpson's O(h⁴) |

---

## Quick Revision Questions

1. Evaluate ∫₁⁴ (1/√x) dx using the FTC.

2. Use substitution with changed limits to evaluate ∫₀^{π/4} tan x sec²x dx.

3. Evaluate ∫₋₃³ (x⁵ + x²) dx using symmetry properties.

4. What is the average value of f(x) = x² on [−1, 1]?

5. Set up Simpson's rule with n = 4 to approximate ∫₀² eˣ dx (do not need to compute).

6. Evaluate ∫₀¹ x ln x dx using integration by parts with limits.

---

[← Previous: Properties of Definite Integrals](03-properties-definite-integrals.md) | [Back to README](../README.md) | [Next: Symmetry Properties →](../05-Properties-of-Definite-Integrals/01-symmetry-properties.md)
