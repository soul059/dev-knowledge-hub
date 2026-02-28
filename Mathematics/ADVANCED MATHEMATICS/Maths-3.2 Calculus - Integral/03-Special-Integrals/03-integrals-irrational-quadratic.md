# Chapter 3: Integrals of Type ∫ dx/√(ax² + bx + c)

[← Previous: Integrals of Type ∫dx/(ax²+bx+c)](02-integrals-rational-quadratic.md) | [Back to README](../README.md) | [Next: Reduction Formulas →](04-reduction-formulas.md)

---

## 3.1 Overview

Integrals of the form ∫ dx/√(ax² + bx + c) are handled by **completing the square** under the radical, then applying one of three standard forms depending on the sign of the leading coefficient and the discriminant.

---

## 3.2 Strategy

```
  ∫ dx / √(ax² + bx + c)
     │
     ▼
  Step 1: Complete the square inside the radical
     │    ax² + bx + c → a[(x + b/2a)² ± k²]
     ▼
  Step 2: Factor out 'a' from the radical
     │    √a · √[(x + b/2a)² ± k²]
     ▼
  Step 3: Let u = x + b/(2a) to get standard form
     │
     ├── √(k² − u²) form → sin⁻¹ result
     ├── √(u² + k²) form → ln (or sinh⁻¹) result
     └── √(u² − k²) form → ln (or cosh⁻¹) result
```

---

## 3.3 Standard Results

| Integral | Result |
|---|---|
| ∫ dx/√(a²−x²) | sin⁻¹(x/a) + C |
| ∫ dx/√(x²+a²) | ln\|x + √(x²+a²)\| + C = sinh⁻¹(x/a) + C |
| ∫ dx/√(x²−a²) | ln\|x + √(x²−a²)\| + C = cosh⁻¹(x/a) + C |

---

## 3.4 Worked Examples

### Example 1: ∫ dx/√(5 − 4x − x²)

**Complete the square:**

$$5 - 4x - x^2 = -(x^2 + 4x) + 5 = -(x^2 + 4x + 4) + 9 = 9 - (x+2)^2$$

So √(5 − 4x − x²) = √(9 − (x+2)²) = √(3² − u²) where u = x + 2.

$$\int \frac{du}{\sqrt{3^2 - u^2}} = \sin^{-1}\left(\frac{u}{3}\right) + C = \sin^{-1}\left(\frac{x+2}{3}\right) + C$$

---

### Example 2: ∫ dx/√(2x² + 4x + 7)

**Factor out 2:**

$$2x^2 + 4x + 7 = 2(x^2 + 2x) + 7 = 2(x^2 + 2x + 1) + 5 = 2(x+1)^2 + 5$$

$$\int \frac{dx}{\sqrt{2(x+1)^2 + 5}} = \frac{1}{\sqrt{2}}\int \frac{du}{\sqrt{u^2 + 5/2}}$$

where u = x + 1 and a² = 5/2, so a = √(5/2):

$$= \frac{1}{\sqrt{2}}\ln\left|u + \sqrt{u^2 + \frac{5}{2}}\right| + C = \frac{1}{\sqrt{2}}\ln\left|x + 1 + \sqrt{x^2 + 2x + \frac{7}{2}}\right| + C$$

---

### Example 3: ∫ dx/√(x² − 6x + 5)

**Complete the square:**

$$x^2 - 6x + 5 = (x-3)^2 - 4 = (x-3)^2 - 2^2$$

Let u = x − 3:

$$\int \frac{du}{\sqrt{u^2 - 4}} = \ln|u + \sqrt{u^2 - 4}| + C = \ln|x - 3 + \sqrt{x^2 - 6x + 5}| + C$$

---

## 3.5 Extended Type: ∫ (px + q)/√(ax² + bx + c) dx

When the numerator has a linear term, split:

$$px + q = \frac{p}{2a}(2ax + b) + \left(q - \frac{pb}{2a}\right)$$

This gives:

$$\int \frac{px+q}{\sqrt{ax^2+bx+c}}\, dx = \frac{p}{2a}\int \frac{2ax+b}{\sqrt{ax^2+bx+c}}\, dx + \left(q-\frac{pb}{2a}\right)\int \frac{dx}{\sqrt{ax^2+bx+c}}$$

The **first integral** = $(p/a)\sqrt{ax^2+bx+c}$ (by substitution t = ax²+bx+c).

The **second integral** is the type we just learned.

### Example 4: ∫ (3x + 2)/√(x² + 4x + 8) dx

Derivative of x² + 4x + 8 is 2x + 4.

Write: 3x + 2 = (3/2)(2x + 4) + (2 − 6) = (3/2)(2x + 4) − 4

$$= \frac{3}{2}\int \frac{2x+4}{\sqrt{x^2+4x+8}}\, dx - 4\int \frac{dx}{\sqrt{x^2+4x+8}}$$

**First part:** Let t = x² + 4x + 8, dt = (2x+4)dx:

$$\frac{3}{2} \cdot 2\sqrt{t} = 3\sqrt{x^2+4x+8}$$

**Second part:** Complete the square: x² + 4x + 8 = (x+2)² + 4

$$-4\int \frac{du}{\sqrt{u^2+4}} = -4\ln|u + \sqrt{u^2+4}| = -4\ln|x+2+\sqrt{x^2+4x+8}|$$

**Result:**

$$= 3\sqrt{x^2+4x+8} - 4\ln|x+2+\sqrt{x^2+4x+8}| + C$$

---

## 3.6 Decision Flowchart

```
  ∫ [something] / √(ax²+bx+c) dx
     │
     ├── Numerator = constant (or 1)?
     │       ──► Complete the square → standard form
     │
     ├── Numerator = px + q?
     │       ──► Split: A·(derivative of quadratic) + B
     │           First part → 2√(quadratic)
     │           Second part → complete the square
     │
     └── Numerator = px² + qx + r? (higher degree)
             ──► Write as: A(ax²+bx+c) + B(2ax+b) + C
                 First part → ∫√(quad) dx
                 Second part → 2√(quad)
                 Third part → standard form
```

---

## Summary Table

| Concept | Key Point |
|---|---|
| Core strategy | Complete the square, reduce to standard form |
| √(a²−u²) | → sin⁻¹(u/a) |
| √(u²+a²) | → ln\|u + √(u²+a²)\| |
| √(u²−a²) | → ln\|u + √(u²−a²)\| |
| Linear numerator | Split: px + q = A(2ax+b) + B |
| First term of split | → Substitution gives 2√(quadratic)/coefficient |
| Second term of split | → Standard integral (sin⁻¹ or ln) |

---

## Quick Revision Questions

1. Complete the square for 7 + 6x − x² and identify the standard form of ∫ dx/√(7 + 6x − x²).

2. Evaluate ∫ dx/√(x² + 2x + 5) using completing the square.

3. What standard integral does ∫ dx/√(x² − 4x) reduce to after completing the square?

4. Describe the splitting technique for ∫ (2x + 5)/√(x² + 4x + 10) dx.

5. Evaluate ∫ dx/√(3x² + 6x + 12) (factor out 3 first).

6. Why must you complete the square before applying standard formulas to ∫ dx/√(ax² + bx + c)?

---

[← Previous: Integrals of Type ∫dx/(ax²+bx+c)](02-integrals-rational-quadratic.md) | [Back to README](../README.md) | [Next: Reduction Formulas →](04-reduction-formulas.md)
