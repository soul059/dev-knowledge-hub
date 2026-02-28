# Chapter 2: Integrals of Type ∫ dx/(ax² + bx + c)

[← Previous: Integrals Involving Square Roots](01-integrals-involving-square-roots.md) | [Back to README](../README.md) | [Next: Integrals of Type ∫dx/√(ax²+bx+c) →](03-integrals-irrational-quadratic.md)

---

## 2.1 Overview

Integrals of the form ∫ dx/(ax² + bx + c) appear frequently in physics and engineering. The strategy is to **complete the square** in the denominator to reduce the integral to one of the standard forms involving tan⁻¹ or logarithms.

---

## 2.2 The Method: Complete the Square

Every quadratic ax² + bx + c can be written as:

$$ax^2 + bx + c = a\left[\left(x + \frac{b}{2a}\right)^2 + \frac{4ac - b^2}{4a^2}\right]$$

Let $u = x + \frac{b}{2a}$ and $k^2 = \left|\frac{4ac - b^2}{4a^2}\right|$.

The integral reduces to one of two standard forms depending on the **discriminant** Δ = b² − 4ac.

---

## 2.3 Three Cases Based on the Discriminant

```
  Δ = b² − 4ac
     │
     ├── Δ < 0 (no real roots): denominator = a[(u)² + k²]
     │   ──► Result involves tan⁻¹
     │
     ├── Δ > 0 (two real roots): denominator factors into linear terms
     │   ──► Use partial fractions → Result involves ln
     │
     └── Δ = 0 (repeated root): denominator = a(x − r)²
         ──► Result involves 1/(x − r)
```

---

### Case 1: Δ < 0 → Inverse Tangent Result

$$\int \frac{dx}{ax^2 + bx + c} = \frac{1}{a}\int \frac{du}{u^2 + k^2} = \frac{1}{a} \cdot \frac{1}{k}\tan^{-1}\left(\frac{u}{k}\right) + C$$

where $u = x + \frac{b}{2a}$ and $k^2 = \frac{4ac - b^2}{4a^2}$.

**Final formula:**

$$= \frac{2}{\sqrt{4ac-b^2}}\tan^{-1}\left(\frac{2ax+b}{\sqrt{4ac-b^2}}\right) + C$$

---

### Case 2: Δ > 0 → Logarithmic Result

The quadratic factors as a(x − r₁)(x − r₂), use partial fractions:

$$\int \frac{dx}{a(x-r_1)(x-r_2)} = \frac{1}{a(r_1-r_2)}\ln\left|\frac{x-r_1}{x-r_2}\right| + C$$

Or equivalently, using the standard form:

$$\int \frac{du}{u^2 - k^2} = \frac{1}{2k}\ln\left|\frac{u-k}{u+k}\right| + C$$

---

### Case 3: Δ = 0 → Power Result

$$\int \frac{dx}{a(x-r)^2} = -\frac{1}{a(x-r)} + C$$

---

## 2.4 Worked Examples

### Example 1: Δ < 0 — ∫ dx/(x² + 4x + 13)

**Step 1:** Complete the square:

$x^2 + 4x + 13 = (x+2)^2 + 9 = (x+2)^2 + 3^2$

**Step 2:** Let u = x + 2:

$$\int \frac{du}{u^2 + 3^2} = \frac{1}{3}\tan^{-1}\left(\frac{u}{3}\right) + C = \frac{1}{3}\tan^{-1}\left(\frac{x+2}{3}\right) + C$$

---

### Example 2: Δ > 0 — ∫ dx/(x² − 5x + 6)

**Step 1:** Factor: $x^2 - 5x + 6 = (x-2)(x-3)$

**Step 2:** Partial fractions:

$$\frac{1}{(x-2)(x-3)} = \frac{A}{x-2} + \frac{B}{x-3}$$

- x = 2: 1 = A(−1) → A = −1
- x = 3: 1 = B(1) → B = 1

$$\int \left(\frac{-1}{x-2} + \frac{1}{x-3}\right) dx = -\ln|x-2| + \ln|x-3| + C = \ln\left|\frac{x-3}{x-2}\right| + C$$

---

### Example 3: With Leading Coefficient ≠ 1 — ∫ dx/(2x² + 3x + 5)

**Step 1:** Factor out the leading coefficient:

$$\frac{1}{2}\int \frac{dx}{x^2 + 3x/2 + 5/2}$$

**Step 2:** Complete the square: $x^2 + \frac{3}{2}x + \frac{5}{2} = \left(x + \frac{3}{4}\right)^2 + \frac{31}{16}$

Δ = 9 − 40 = −31 < 0, so:

$$= \frac{1}{2} \cdot \frac{1}{\sqrt{31}/4}\tan^{-1}\left(\frac{x+3/4}{\sqrt{31}/4}\right) + C = \frac{2}{\sqrt{31}}\tan^{-1}\left(\frac{4x+3}{\sqrt{31}}\right) + C$$

---

### Example 4: ∫ (px + q)/(ax² + bx + c) dx

When the numerator contains x, **split** the integral:

$$\int \frac{2x + 3}{x^2 + 4x + 13}\, dx$$

**Strategy:** Write the numerator as: 2x + 3 = A(derivative of denominator) + B

Derivative of x² + 4x + 13 is 2x + 4.

$$2x + 3 = 1 \cdot (2x + 4) + (-1)$$

$$= \int \frac{2x+4}{x^2+4x+13}\, dx - \int \frac{dx}{x^2+4x+13}$$

$$= \ln(x^2+4x+13) - \frac{1}{3}\tan^{-1}\left(\frac{x+2}{3}\right) + C$$

### General Strategy for ∫ (px+q)/(ax²+bx+c) dx

```
  ∫ (px + q)/(ax² + bx + c) dx
     │
     ▼
  Write: px + q = A · d/dx(ax²+bx+c) + B
     │           = A(2ax + b) + B
     ▼
  Solve: 2aA = p  →  A = p/(2a)
         Ab + B = q  →  B = q − pb/(2a)
     │
     ▼
  Split: A ∫ (2ax+b)/(ax²+bx+c) dx  +  B ∫ dx/(ax²+bx+c)
              │                              │
              ▼                              ▼
         A · ln|ax²+bx+c|          Use complete-the-square
```

---

## 2.5 Special Standard Forms Summary

| Integral | Result |
|---|---|
| ∫ dx/(x²+a²) | (1/a) tan⁻¹(x/a) + C |
| ∫ dx/(x²−a²) | (1/2a) ln\|(x−a)/(x+a)\| + C |
| ∫ dx/(a²−x²) | (1/2a) ln\|(a+x)/(a−x)\| + C |
| ∫ (2x+b)/(x²+bx+c) dx | ln\|x²+bx+c\| + C |

---

## Summary Table

| Concept | Key Point |
|---|---|
| Strategy | Complete the square in the denominator |
| Δ < 0 | No real roots → result is tan⁻¹ |
| Δ > 0 | Real roots → partial fractions → logarithms |
| Δ = 0 | Repeated root → power rule |
| Numerator has x | Split: px+q = A·(derivative of denom) + B |
| First part | ln of denominator |
| Second part | tan⁻¹ or ln (depending on Δ) |

---

## Quick Revision Questions

1. Complete the square for x² + 6x + 25 and evaluate ∫ dx/(x² + 6x + 25).

2. Evaluate ∫ dx/(x² − 4) using partial fractions.

3. Describe the strategy for ∫ (3x + 7)/(x² + 2x + 10) dx without fully evaluating.

4. What determines whether the result of ∫ dx/(ax²+bx+c) involves tan⁻¹ or ln?

5. Evaluate ∫ dx/(4x² + 1).

6. Factor x² − 3x + 2 and evaluate ∫ dx/(x² − 3x + 2).

---

[← Previous: Integrals Involving Square Roots](01-integrals-involving-square-roots.md) | [Back to README](../README.md) | [Next: Integrals of Type ∫dx/√(ax²+bx+c) →](03-integrals-irrational-quadratic.md)
