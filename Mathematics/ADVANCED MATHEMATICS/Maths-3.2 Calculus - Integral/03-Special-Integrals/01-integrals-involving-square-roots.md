# Chapter 1: Integrals Involving Square Roots

[← Previous: Weierstrass Substitution](../02-Methods-of-Integration/05-weierstrass-substitution.md) | [Back to README](../README.md) | [Next: Integrals of Type ∫dx/(ax²+bx+c) →](02-integrals-rational-quadratic.md)

---

## 1.1 Overview

This chapter covers the systematic evaluation of integrals involving the square root expressions **√(a²−x²)**, **√(x²+a²)**, and **√(x²−a²)**. These appear frequently in geometry, physics, and engineering problems.

---

## 1.2 The Three Fundamental Integrals

### Formula 1: ∫ √(a² − x²) dx

$$\int \sqrt{a^2 - x^2}\, dx = \frac{x\sqrt{a^2-x^2}}{2} + \frac{a^2}{2}\sin^{-1}\left(\frac{x}{a}\right) + C$$

### Formula 2: ∫ √(x² + a²) dx

$$\int \sqrt{x^2 + a^2}\, dx = \frac{x\sqrt{x^2+a^2}}{2} + \frac{a^2}{2}\ln\left|x + \sqrt{x^2+a^2}\right| + C$$

### Formula 3: ∫ √(x² − a²) dx

$$\int \sqrt{x^2 - a^2}\, dx = \frac{x\sqrt{x^2-a^2}}{2} - \frac{a^2}{2}\ln\left|x + \sqrt{x^2-a^2}\right| + C$$

### Pattern Recognition

```
  ┌───────────────────────────────────────────────────┐
  │  All three formulas share a similar structure:     │
  │                                                    │
  │  ∫√(___) dx = (x · √(___))/2                     │
  │             ± (a²/2) · [sin⁻¹ or ln] + C         │
  │                                                    │
  │  First term:   always  x·√(expression) / 2        │
  │  Second term:  a²/2 times  sin⁻¹  or  ln          │
  └───────────────────────────────────────────────────┘
```

---

## 1.3 Derivation of Formula 1: ∫ √(a² − x²) dx

**Method:** Trigonometric substitution x = a sin θ

Let x = a sin θ, so dx = a cos θ dθ and √(a²−x²) = a cos θ.

$$\int a\cos\theta \cdot a\cos\theta\, d\theta = a^2\int \cos^2\theta\, d\theta$$

Using cos²θ = (1 + cos 2θ)/2:

$$= a^2 \cdot \frac{1}{2}(\theta + \sin\theta\cos\theta) + C$$

From the reference triangle:

```
  sin θ = x/a
  cos θ = √(a²−x²)/a
  θ = sin⁻¹(x/a)
```

$$= \frac{a^2}{2}\sin^{-1}\left(\frac{x}{a}\right) + \frac{a^2}{2} \cdot \frac{x}{a} \cdot \frac{\sqrt{a^2-x^2}}{a} + C$$

$$= \frac{x\sqrt{a^2-x^2}}{2} + \frac{a^2}{2}\sin^{-1}\left(\frac{x}{a}\right) + C$$

---

## 1.4 Related Standard Forms

### Reciprocal Square Root Integrals

| Integral | Result |
|---|---|
| ∫ dx/√(a²−x²) | sin⁻¹(x/a) + C |
| ∫ dx/√(x²+a²) | ln\|x + √(x²+a²)\| + C |
| ∫ dx/√(x²−a²) | ln\|x + √(x²−a²)\| + C |

### With x in the Numerator

| Integral | Result |
|---|---|
| ∫ x dx/√(a²−x²) | −√(a²−x²) + C |
| ∫ x dx/√(x²+a²) | √(x²+a²) + C |
| ∫ x dx/√(x²−a²) | √(x²−a²) + C |

> **Note:** The second group uses simple substitution u = (expression under root).

---

## 1.5 Worked Examples

### Example 1: Direct Application

$$\int \sqrt{9 - x^2}\, dx$$

Here a = 3. Apply Formula 1:

$$= \frac{x\sqrt{9-x^2}}{2} + \frac{9}{2}\sin^{-1}\left(\frac{x}{3}\right) + C$$

---

### Example 2: With a Linear Term

$$\int \sqrt{4 - (x-1)^2}\, dx$$

Let u = x − 1, du = dx. This becomes ∫√(4 − u²) du with a = 2:

$$= \frac{u\sqrt{4-u^2}}{2} + \frac{4}{2}\sin^{-1}\left(\frac{u}{2}\right) + C$$

$$= \frac{(x-1)\sqrt{4-(x-1)^2}}{2} + 2\sin^{-1}\left(\frac{x-1}{2}\right) + C$$

---

### Example 3: Completing the Square

$$\int \sqrt{5 + 4x - x^2}\, dx$$

Complete the square: $5 + 4x - x^2 = -(x^2 - 4x) + 5 = -(x^2 - 4x + 4) + 9 = 9 - (x-2)^2$

This is √(a² − u²) with a = 3, u = x − 2:

$$= \frac{(x-2)\sqrt{9-(x-2)^2}}{2} + \frac{9}{2}\sin^{-1}\left(\frac{x-2}{3}\right) + C$$

---

### Example 4: ∫ √(x² + 4) dx

Apply Formula 2 with a = 2:

$$= \frac{x\sqrt{x^2+4}}{2} + \frac{4}{2}\ln\left|x + \sqrt{x^2+4}\right| + C$$

$$= \frac{x\sqrt{x^2+4}}{2} + 2\ln\left|x + \sqrt{x^2+4}\right| + C$$

---

### Example 5: ∫ x² / √(a² − x²) dx

Let x = a sin θ, dx = a cos θ dθ:

$$\int \frac{a^2\sin^2\theta}{a\cos\theta} \cdot a\cos\theta\, d\theta = a^2\int \sin^2\theta\, d\theta$$

$$= \frac{a^2}{2}(\theta - \sin\theta\cos\theta) + C$$

$$= \frac{a^2}{2}\sin^{-1}\left(\frac{x}{a}\right) - \frac{x\sqrt{a^2-x^2}}{2} + C$$

---

## 1.6 Geometric Interpretation

The integral ∫₀ᵃ √(a² − x²) dx represents the area of a **quarter circle** of radius a:

```
  y
  a ┤·  ·  ·  .
    │ ·        . 
    │  ·       .   Curve: y = √(a² − x²)
    │   ·      .   (upper semicircle)
    │    ·     .
    │     ·   .    Area of quarter circle
    │      · .     = πa²/4
    │       ·.
    └────────┴──── x
    0        a
```

$$\int_0^a \sqrt{a^2-x^2}\, dx = \frac{\pi a^2}{4}$$

This can be verified using Formula 1 with limits.

---

## 1.7 Real-World Application

**Ellipse arc length:** The arc length of an ellipse x²/a² + y²/b² = 1 involves integrals of the form:

$$L = \int \sqrt{1 + \left(\frac{dy}{dx}\right)^2}\, dx$$

which leads to integrals involving √(a² − x²) type expressions. These are important in orbital mechanics, antenna design, and structural engineering.

---

## Summary Table

| Integral | Result | Method |
|---|---|---|
| ∫ √(a²−x²) dx | (x√(a²−x²))/2 + (a²/2)sin⁻¹(x/a) + C | x = a sin θ |
| ∫ √(x²+a²) dx | (x√(x²+a²))/2 + (a²/2)ln\|x+√(x²+a²)\| + C | x = a tan θ |
| ∫ √(x²−a²) dx | (x√(x²−a²))/2 − (a²/2)ln\|x+√(x²−a²)\| + C | x = a sec θ |
| ∫ dx/√(a²−x²) | sin⁻¹(x/a) + C | Direct |
| ∫ dx/√(x²±a²) | ln\|x + √(x²±a²)\| + C | Direct |
| Non-standard form | Complete the square → reduce to standard | Algebra first |

---

## Quick Revision Questions

1. State the three standard formulas for ∫√(a²−x²) dx, ∫√(x²+a²) dx, and ∫√(x²−a²) dx.

2. Evaluate ∫ √(16 − x²) dx directly using the formula.

3. What is the geometric meaning of ∫₋ₐᵃ √(a² − x²) dx?

4. Evaluate ∫ √(x² + 9) dx using the standard formula.

5. Transform ∫ √(3 + 2x − x²) dx by completing the square, and identify a and u.

6. Evaluate ∫ x / √(x² + 4) dx (hint: no standard formula needed — use simple substitution).

---

[← Previous: Weierstrass Substitution](../02-Methods-of-Integration/05-weierstrass-substitution.md) | [Back to README](../README.md) | [Next: Integrals of Type ∫dx/(ax²+bx+c) →](02-integrals-rational-quadratic.md)
