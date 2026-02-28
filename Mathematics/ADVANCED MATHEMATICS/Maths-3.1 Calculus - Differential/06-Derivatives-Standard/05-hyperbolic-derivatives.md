# Unit 6 — Derivatives of Standard Functions: Hyperbolic Function Derivatives

[← Previous: Exponential & Logarithmic Derivatives](04-exponential-log-derivatives.md) · [Next: Unit 7 — Higher-Order Derivatives →](../07-Higher-Order/01-nth-derivatives.md)

---

## Chapter Overview

Hyperbolic functions are defined using exponentials and have derivatives remarkably similar to (but not identical with) trigonometric functions. This chapter covers all six hyperbolic functions, their inverses, and their derivatives.

---

## 6.5.1 Definitions of Hyperbolic Functions

| Function | Definition |
|----------|-----------|
| $\sinh x$ | $\dfrac{e^x - e^{-x}}{2}$ |
| $\cosh x$ | $\dfrac{e^x + e^{-x}}{2}$ |
| $\tanh x$ | $\dfrac{\sinh x}{\cosh x} = \dfrac{e^x - e^{-x}}{e^x + e^{-x}}$ |
| $\text{coth}\, x$ | $\dfrac{\cosh x}{\sinh x}$ |
| $\text{sech}\, x$ | $\dfrac{1}{\cosh x}$ |
| $\text{csch}\, x$ | $\dfrac{1}{\sinh x}$ |

---

## 6.5.2 Key Hyperbolic Identities

| Identity | Compare with Trig |
|----------|-------------------|
| $\cosh^2 x - \sinh^2 x = 1$ | $\cos^2 x + \sin^2 x = 1$ |
| $1 - \tanh^2 x = \text{sech}^2\, x$ | $1 + \tan^2 x = \sec^2 x$ |
| $\coth^2 x - 1 = \text{csch}^2\, x$ | $\cot^2 x + 1 = \csc^2 x$ |
| $\sinh(2x) = 2\sinh x \cosh x$ | $\sin(2x) = 2\sin x \cos x$ |
| $\cosh(2x) = \cosh^2 x + \sinh^2 x$ | $\cos(2x) = \cos^2 x - \sin^2 x$ |

> Notice: the identities look like trig but with **sign changes** (Osborn's rule: replace products of two sines with minus sign).

---

## 6.5.3 Derivatives of Hyperbolic Functions

| $f(x)$ | $f'(x)$ | Compare with Trig |
|---------|----------|--------------------|
| $\sinh x$ | $\cosh x$ | $\sin' = \cos$ ✓ Same pattern |
| $\cosh x$ | $\sinh x$ | $\cos' = -\sin$ ✗ **No minus!** |
| $\tanh x$ | $\text{sech}^2\, x$ | $\tan' = \sec^2$ ✓ Same |
| $\coth x$ | $-\text{csch}^2\, x$ | $\cot' = -\csc^2$ ✓ Same |
| $\text{sech}\, x$ | $-\text{sech}\, x\,\tanh x$ | $\sec' = \sec\tan$ ✗ **Extra minus** |
| $\text{csch}\, x$ | $-\text{csch}\, x\,\coth x$ | $\csc' = -\csc\cot$ ✓ Same |

```
  ┌────────────────────────────────────────────┐
  │  Key difference from trig:                 │
  │  cosh'(x) = sinh(x)  (NO minus sign)      │
  │  sech'(x) = -sech(x)tanh(x) (HAS minus)   │
  └────────────────────────────────────────────┘
```

---

## 6.5.4 Proofs

### $\frac{d}{dx}(\sinh x) = \cosh x$

$$\frac{d}{dx}\left(\frac{e^x - e^{-x}}{2}\right) = \frac{e^x + e^{-x}}{2} = \cosh x \quad \blacksquare$$

### $\frac{d}{dx}(\cosh x) = \sinh x$

$$\frac{d}{dx}\left(\frac{e^x + e^{-x}}{2}\right) = \frac{e^x - e^{-x}}{2} = \sinh x \quad \blacksquare$$

### $\frac{d}{dx}(\tanh x) = \text{sech}^2 x$

Using quotient rule on $\frac{\sinh x}{\cosh x}$:

$$\frac{\cosh x \cdot \cosh x - \sinh x \cdot \sinh x}{\cosh^2 x} = \frac{\cosh^2 x - \sinh^2 x}{\cosh^2 x} = \frac{1}{\cosh^2 x} = \text{sech}^2\, x \quad \blacksquare$$

---

## 6.5.5 Inverse Hyperbolic Function Derivatives

| $f(x)$ | $f'(x)$ | Domain |
|---------|----------|--------|
| $\sinh^{-1} x$ | $\dfrac{1}{\sqrt{x^2 + 1}}$ | $\mathbb{R}$ |
| $\cosh^{-1} x$ | $\dfrac{1}{\sqrt{x^2 - 1}}$ | $x > 1$ |
| $\tanh^{-1} x$ | $\dfrac{1}{1 - x^2}$ | $\|x\| < 1$ |
| $\coth^{-1} x$ | $\dfrac{1}{1 - x^2}$ | $\|x\| > 1$ |

### Logarithmic Forms

$$\sinh^{-1} x = \ln(x + \sqrt{x^2 + 1})$$

$$\cosh^{-1} x = \ln(x + \sqrt{x^2 - 1})$$

$$\tanh^{-1} x = \frac{1}{2}\ln\frac{1+x}{1-x}$$

### Proof: $\frac{d}{dx}(\sinh^{-1} x)$

Let $y = \sinh^{-1} x$, so $\sinh y = x$.

$$\cosh y \cdot \frac{dy}{dx} = 1$$

$$\frac{dy}{dx} = \frac{1}{\cosh y} = \frac{1}{\sqrt{1 + \sinh^2 y}} = \frac{1}{\sqrt{1 + x^2}} \quad \blacksquare$$

---

## 6.5.6 Comparison: Trig vs. Hyperbolic Derivatives

| Trig | Hyperbolic | Difference |
|------|-----------|------------|
| $(\sin x)' = \cos x$ | $(\sinh x)' = \cosh x$ | Same pattern |
| $(\cos x)' = -\sin x$ | $(\cosh x)' = \sinh x$ | **No minus** |
| $(\sin^{-1} x)' = \frac{1}{\sqrt{1-x^2}}$ | $(\sinh^{-1} x)' = \frac{1}{\sqrt{1+x^2}}$ | Sign in radical |
| $(\tan^{-1} x)' = \frac{1}{1+x^2}$ | $(\tanh^{-1} x)' = \frac{1}{1-x^2}$ | Sign in denominator |

---

## 6.5.7 Worked Examples

### Example 1: $y = \cosh(3x)$

$$\frac{dy}{dx} = \sinh(3x) \cdot 3 = 3\sinh(3x)$$

### Example 2: $y = \tanh^2 x$

$$\frac{dy}{dx} = 2\tanh x \cdot \text{sech}^2\,x$$

### Example 3: $y = \ln(\cosh x)$

$$\frac{dy}{dx} = \frac{\sinh x}{\cosh x} = \tanh x$$

### Example 4: $y = \sinh^{-1}\left(\frac{x}{3}\right)$

$$\frac{dy}{dx} = \frac{1/3}{\sqrt{x^2/9 + 1}} = \frac{1/3}{\sqrt{(x^2 + 9)/9}} = \frac{1}{\sqrt{x^2 + 9}}$$

**General result:** $\frac{d}{dx}\left[\sinh^{-1}\frac{x}{a}\right] = \frac{1}{\sqrt{x^2 + a^2}}$

### Example 5: $y = x\tanh^{-1} x + \frac{1}{2}\ln(1 - x^2)$

$$\frac{dy}{dx} = \tanh^{-1}x + \frac{x}{1-x^2} + \frac{1}{2}\cdot\frac{-2x}{1-x^2}$$

$$= \tanh^{-1}x + \frac{x}{1-x^2} - \frac{x}{1-x^2} = \tanh^{-1}x$$

---

## 6.5.8 Graphs

```
   y                              y
   |               sinh x         |         cosh x
   |           *                  |     *       *
   |         *                    |      *     *
   |       *                      |       *   *
   |     *                        |        * *
   |   *                         1+         *
   | *                            |
  -*                              |
   *                              +---+---+---→ x
   +---+---+---→ x

   Both sinh and cosh have the same derivative structure
   but cosh ≥ 1 (always positive, like a "hanging chain")
```

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| $(\sinh x)' = \cosh x$ | Analogous to trig, same sign |
| $(\cosh x)' = \sinh x$ | **No minus** (unlike cos) |
| $(\tanh x)' = \text{sech}^2 x$ | Same pattern as $(\tan x)' = \sec^2 x$ |
| $\cosh^2 - \sinh^2 = 1$ | Fundamental hyperbolic identity |
| $(\sinh^{-1}x)' = 1/\sqrt{1+x^2}$ | Contrast: $(\sin^{-1}x)' = 1/\sqrt{1-x^2}$ |
| Logarithmic forms | $\sinh^{-1}x = \ln(x+\sqrt{x^2+1})$ etc. |

---

## Quick Revision Questions

**Q1.** Verify that $\frac{d}{dx}(\coth x) = -\text{csch}^2\, x$ using the quotient rule.

**Q2.** Differentiate $y = \sinh(x^2 + 1)$.

**Q3.** Show that $\frac{d}{dx}[\cosh^{-1} x] = \frac{1}{\sqrt{x^2 - 1}}$ using implicit differentiation.

**Q4.** Find $\frac{dy}{dx}$ for $y = e^x \cosh x$. Simplify using exponential definitions.

**Q5.** Differentiate $y = \tanh^{-1}(\sin x)$.

**Q6.** Why does $(\cosh x)' = \sinh x$ (no minus sign) while $(\cos x)' = -\sin x$? Explain using the exponential definitions.

---

[← Previous: Exponential & Logarithmic Derivatives](04-exponential-log-derivatives.md) · [Next: Unit 7 — Higher-Order Derivatives →](../07-Higher-Order/01-nth-derivatives.md)
