# Chapter 4: Leibniz Rule (Differentiation Under the Integral Sign)

[← Previous: Periodic Function Integrals](03-periodic-function-integrals.md) | [Back to README](../README.md) | [Next: Area Under a Curve →](../06-Applications-Area/01-area-under-curve.md)

---

## 4.1 Overview

The **Leibniz Integral Rule** tells us how to differentiate an integral whose integrand and/or limits depend on a parameter. It bridges differentiation and integration in a powerful way, enabling evaluation of integrals that resist all standard methods.

---

## 4.2 The Full Leibniz Rule

> **Theorem:** If f(x, α), ∂f/∂α, a(α), and b(α) are continuous, then:
>
> $$\frac{d}{d\alpha}\int_{a(\alpha)}^{b(\alpha)} f(x, \alpha)\, dx = f(b(\alpha), \alpha) \cdot b'(\alpha) - f(a(\alpha), \alpha) \cdot a'(\alpha) + \int_{a(\alpha)}^{b(\alpha)} \frac{\partial f}{\partial \alpha}(x, \alpha)\, dx$$

### Decomposition Diagram

```
  d/dα ∫_{a(α)}^{b(α)} f(x, α) dx  =  THREE TERMS
  ═══════════════════════════════════════════════════
  
  Term 1: UPPER BOUNDARY
  ┌─────────────────────────────────┐
  │  + f(b(α), α) · b'(α)          │
  │  "Value at top × rate top moves"│
  └─────────────────────────────────┘
  
  Term 2: LOWER BOUNDARY
  ┌─────────────────────────────────┐
  │  − f(a(α), α) · a'(α)          │
  │  "Value at bottom × rate        │
  │   bottom moves" (minus sign!)   │
  └─────────────────────────────────┘
  
  Term 3: INTERIOR CHANGE
  ┌─────────────────────────────────┐
  │  ∫_{a(α)}^{b(α)} ∂f/∂α dx     │
  │  "How integrand itself changes" │
  └─────────────────────────────────┘
```

---

## 4.3 Special Cases

### Case 1: Fixed Limits (a, b are constants)

The boundary terms vanish since a'(α) = b'(α) = 0:

$$\frac{d}{d\alpha}\int_a^b f(x, \alpha)\, dx = \int_a^b \frac{\partial f}{\partial \alpha}(x, \alpha)\, dx$$

> You can "pass the derivative inside the integral" when limits are constant.

---

### Case 2: Variable Upper Limit, No Parameter in f

This is the **generalized FTC**:

$$\frac{d}{dx}\int_a^{g(x)} f(t)\, dt = f(g(x)) \cdot g'(x)$$

---

### Case 3: Both Limits Variable, No Parameter in f

$$\frac{d}{dx}\int_{h(x)}^{g(x)} f(t)\, dt = f(g(x)) \cdot g'(x) - f(h(x)) \cdot h'(x)$$

---

### Summary of Cases

```
  ┌────────────────────────────────────────────┐
  │        WHICH TERMS ARE PRESENT?            │
  ├──────────────────┬─────┬───────┬───────────┤
  │  Situation       │ T1  │  T2   │    T3     │
  ├──────────────────┼─────┼───────┼───────────┤
  │ Fixed limits,    │  ✗  │   ✗   │    ✓      │
  │ α in f           │     │       │           │
  ├──────────────────┼─────┼───────┼───────────┤
  │ Variable limits, │  ✓  │   ✓   │    ✗      │
  │ α not in f       │     │       │           │
  ├──────────────────┼─────┼───────┼───────────┤
  │ General          │  ✓  │   ✓   │    ✓      │
  │ (everything)     │     │       │           │
  └──────────────────┴─────┴───────┴───────────┘
  
  T1 = upper boundary term
  T2 = lower boundary term  
  T3 = interior term (∫ ∂f/∂α dx)
```

---

## 4.4 Worked Examples

### Example 1: Variable Upper Limit (FTC Extension)

$$F(x) = \frac{d}{dx}\int_0^{x^3} e^{-t^2}\, dt$$

Here g(x) = x³, f(t) = e^{−t²}:

$$F(x) = e^{-(x^3)^2} \cdot 3x^2 = 3x^2 e^{-x^6}$$

---

### Example 2: Both Limits Variable

$$\frac{d}{dx}\int_{x}^{x^2} \sin(t^2)\, dt$$

$$= \sin((x^2)^2) \cdot 2x - \sin(x^2) \cdot 1 = 2x\sin(x^4) - \sin(x^2)$$

---

### Example 3: Fixed Limits with Parameter (Feynman's Trick)

Evaluate: $\displaystyle I(\alpha) = \int_0^1 \frac{x^\alpha - 1}{\ln x}\, dx$

**Step 1:** Differentiate under the integral:

$$I'(\alpha) = \int_0^1 \frac{\partial}{\partial \alpha}\left(\frac{x^\alpha - 1}{\ln x}\right) dx = \int_0^1 \frac{x^\alpha \ln x}{\ln x}\, dx = \int_0^1 x^\alpha\, dx$$

**Step 2:** Evaluate the simpler integral:

$$I'(\alpha) = \left[\frac{x^{\alpha+1}}{\alpha+1}\right]_0^1 = \frac{1}{\alpha + 1}$$

**Step 3:** Integrate with respect to α:

$$I(\alpha) = \ln(\alpha + 1) + C$$

**Step 4:** Find C using initial condition. At α = 0:

$$I(0) = \int_0^1 \frac{x^0 - 1}{\ln x}\, dx = \int_0^1 0\, dx = 0$$

$$0 = \ln(1) + C \implies C = 0$$

$$\boxed{I(\alpha) = \ln(\alpha + 1)}$$

Setting α = 1: $\displaystyle\int_0^1 \frac{x - 1}{\ln x}\, dx = \ln 2$

---

### Example 4: The Frullani Integral

$$\int_0^{\infty} \frac{e^{-ax} - e^{-bx}}{x}\, dx \quad (a, b > 0)$$

Define: $I(a) = \int_0^{\infty} \frac{e^{-ax} - e^{-bx}}{x}\, dx$

Differentiate with respect to a:

$$\frac{\partial I}{\partial a} = \int_0^{\infty} \frac{-x \cdot e^{-ax}}{x}\, dx = -\int_0^{\infty} e^{-ax}\, dx = -\frac{1}{a}$$

Integrate: $I = -\ln a + C(b)$.

Since I(a, a) = 0 (numerator becomes 0): $0 = -\ln a + C(a) \implies C(a) = \ln a$, so $C(b) = \ln b$.

$$\boxed{\int_0^{\infty} \frac{e^{-ax} - e^{-bx}}{x}\, dx = \ln\frac{b}{a}}$$

---

### Example 5: General Case (All Three Terms)

Let $\displaystyle G(\alpha) = \int_\alpha^{\alpha^2} (\alpha + x^2)\, dx$

$$G'(\alpha) = (\alpha + \alpha^4) \cdot 2\alpha - (\alpha + \alpha^2) \cdot 1 + \int_\alpha^{\alpha^2} 1\, dx$$

$$= 2\alpha^2 + 2\alpha^5 - \alpha - \alpha^2 + (\alpha^2 - \alpha)$$

$$= 2\alpha^5 + 2\alpha^2 - 2\alpha$$

Wait — let me redo carefully:

- T1 = f(α², α) · (2α) = (α + α⁴)(2α) = 2α² + 2α⁵
- T2 = f(α, α) · (1) = (α + α²)(1) = α + α²
- T3 = ∫_α^{α²} 1 dx = α² − α

$$G'(\alpha) = (2\alpha^2 + 2\alpha^5) - (\alpha + \alpha^2) + (\alpha^2 - \alpha)$$

$$= 2\alpha^5 + 2\alpha^2 - \alpha^2 - \alpha + \alpha^2 - \alpha = 2\alpha^5 + 2\alpha^2 - 2\alpha$$

---

## 4.5 Feynman's Trick — Strategy Guide

> *"I had learned to do integrals by various methods shown in a book that my high school physics teacher Mr. Bader had given me... the method of differentiating under the integral sign... It turned out that was not taught very much in the universities; they didn't emphasize it."* — Richard Feynman

### When to Use

```
  Is the integral hard to evaluate directly?
     │
     ├── Can you introduce a parameter α that
     │   makes the integral simpler when differentiated?
     │   │
     │   YES ──► Feynman's Trick!
     │           1. Define I(α) with parameter
     │           2. Find I'(α) easily  
     │           3. Integrate I'(α) w.r.t. α
     │           4. Use initial condition for C
     │           5. Set α to desired value
     │   
     │   NO ──► Try other methods
     │
     └── Common patterns:
         • xᵅ in numerator → d/dα(xᵅ) = xᵅ ln x  
         • e⁻ᵅˣ factor → d/dα(e⁻ᵅˣ) = −xe⁻ᵅˣ
         • ln(1 + α·g(x)) → d/dα gives rational
```

### Classic Integrals Solvable by Feynman's Trick

| Integral | Parameter to Introduce | Result |
|---|---|---|
| ∫₀^∞ (e⁻ᵅˣ − e⁻ᵇˣ)/x dx | α (differentiate) | ln(b/a) |
| ∫₀¹ (xᵅ − 1)/ln x dx | α (differentiate) | ln(α + 1) |
| ∫₀^∞ e⁻ˣ² dx | Scale x → αx | √π/2 |
| ∫₀^{π/2} ln(sin x) dx | Add α parameter | −(π/2)ln 2 |

---

## 4.6 Real-World Applications

| Domain | Application |
|---|---|
| **Physics** | Varying boundary problems in heat conduction and fluid mechanics |
| **Probability** | Moment generating functions: M(t) = E[e^{tX}], moments via M'(0), M''(0), etc. |
| **Optimization** | Sensitivity analysis — how does an objective change with parameters? |
| **Engineering** | Reynolds transport theorem (fluid mechanics generalization of Leibniz rule) |

---

## Summary Table

| Concept | Formula/Key Point |
|---|---|
| Full Leibniz Rule | d/dα ∫_{a(α)}^{b(α)} f(x,α) dx = f(b,α)·b' − f(a,α)·a' + ∫ ∂f/∂α dx |
| Fixed limits | d/dα ∫ₐᵇ f(x,α) dx = ∫ₐᵇ ∂f/∂α dx |
| Variable limits only | d/dx ∫_{h(x)}^{g(x)} f(t) dt = f(g)·g' − f(h)·h' |
| Feynman's trick | Introduce parameter, differentiate, integrate back |
| Frullani integral | ∫₀^∞ (e⁻ᵃˣ − e⁻ᵇˣ)/x dx = ln(b/a) |
| Key insight | Differentiation w.r.t. parameter can simplify integrands |

---

## Quick Revision Questions

1. State the full Leibniz integral rule with all three terms.

2. Find d/dx ∫₀^{x²} cos(t³) dt.

3. Find d/dx ∫_{2x}^{x³} e^{t²} dt using Leibniz rule.

4. Use Feynman's trick to evaluate ∫₀¹ (x² − 1)/ln x dx. (Hint: generalize to ∫₀¹ (xᵅ − 1)/ln x dx.)

5. Explain why the boundary terms in the Leibniz rule have opposite signs.

6. For the general Leibniz rule, what conditions must be satisfied for the formula to hold?

---

[← Previous: Periodic Function Integrals](03-periodic-function-integrals.md) | [Back to README](../README.md) | [Next: Area Under a Curve →](../06-Applications-Area/01-area-under-curve.md)
