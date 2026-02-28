# Chapter 4: Properties of Integrals

[← Previous: Basic Integration Formulas](03-basic-integration-formulas.md) | [Back to README](../README.md) | [Next: Substitution Method →](../02-Methods-of-Integration/01-substitution-method.md)

---

## 4.1 Overview

The properties of indefinite integrals allow us to break down complex integrals into simpler parts. These properties mirror (and are derived from) the corresponding rules of differentiation, making integration a **linear operation**.

---

## 4.2 Linearity of Integration

Integration is a **linear operator**. This means it satisfies two crucial properties:

### Property 1: Constant Multiple Rule

$$\int k \cdot f(x)\, dx = k \int f(x)\, dx, \quad k \text{ is a constant}$$

> *A constant factor can be pulled out of the integral.*

**Proof:** Let F'(x) = f(x). Then d/dx[kF(x)] = kF'(x) = kf(x), so kF(x) is an antiderivative of kf(x).

---

### Property 2: Sum / Difference Rule

$$\int [f(x) \pm g(x)]\, dx = \int f(x)\, dx \pm \int g(x)\, dx$$

> *The integral of a sum (or difference) equals the sum (or difference) of the integrals.*

**Proof:** Let F'(x) = f(x) and G'(x) = g(x). Then d/dx[F(x) ± G(x)] = f(x) ± g(x).

---

### Combined: General Linearity

$$\int [\alpha f(x) + \beta g(x)]\, dx = \alpha \int f(x)\, dx + \beta \int g(x)\, dx$$

### Linearity — Diagram

```
  ∫ [3f(x) + 2g(x)] dx
     │
     │  Split by linearity
     ▼
  3 ∫ f(x) dx  +  2 ∫ g(x) dx
     │                  │
     ▼                  ▼
  3 F(x)         +  2 G(x)  + C
```

> **Important:** Only ONE constant C is needed for the final answer once all terms are combined.

---

## 4.3 Properties That Do NOT Hold

Understanding what integration **cannot** do is just as important.

### ❌ No Product Rule for Integration

$$\int f(x) \cdot g(x)\, dx \neq \int f(x)\, dx \cdot \int g(x)\, dx$$

**Counterexample:** ∫ x · x dx = ∫ x² dx = x³/3 + C

But (∫ x dx)(∫ x dx) = (x²/2)(x²/2) = x⁴/4 ≠ x³/3

> For products, we need **Integration by Parts** (Unit 2, Chapter 2).

---

### ❌ No Quotient Rule for Integration

$$\int \frac{f(x)}{g(x)}\, dx \neq \frac{\int f(x)\, dx}{\int g(x)\, dx}$$

> For quotients, we often use **Partial Fractions** (Unit 2, Chapter 3) or substitution.

---

### ❌ No Chain Rule for Integration

$$\int f(g(x))\, dx \neq F(g(x)) + C \quad \text{in general}$$

> For compositions, we need the **Substitution Method** (Unit 2, Chapter 1).

---

### Comparison Table: What Works and What Doesn't

| Operation | Works for Integrals? | Technique Needed |
|---|---|---|
| Constant multiple: k·f(x) | ✅ Yes | Pull out the constant |
| Sum/Difference: f ± g | ✅ Yes | Split into separate integrals |
| Product: f · g | ❌ No | Integration by parts |
| Quotient: f / g | ❌ No | Partial fractions / substitution |
| Composition: f(g(x)) | ❌ No | Substitution (u-sub) |

---

## 4.4 Zero and Identity Properties

### Integral of Zero

$$\int 0\, dx = C$$

The antiderivative of the zero function is any constant.

### Integral of 1

$$\int 1\, dx = x + C$$

---

## 4.5 Applying Properties — Step by Step

### Strategy Flowchart

```
  Given: ∫ [complex expression] dx
     │
     ▼
  Step 1: Pull out constants
     │    ∫ k·f(x) dx = k ∫ f(x) dx
     ▼
  Step 2: Split sums and differences
     │    ∫(f ± g) dx = ∫ f dx ± ∫ g dx
     ▼
  Step 3: Simplify each term algebraically
     │    (expand, rewrite powers, use identities)
     ▼
  Step 4: Apply standard formulas to each term
     │
     ▼
  Step 5: Combine results and write + C
```

---

## 4.6 Worked Examples

### Example 1: Linearity in Action

$$\int (5x^3 - 3\sin x + 7e^x)\, dx$$

**Step 1 & 2:** Split and extract constants:

$$= 5\int x^3\, dx - 3\int \sin x\, dx + 7\int e^x\, dx$$

**Step 3:** Apply standard formulas:

$$= 5 \cdot \frac{x^4}{4} - 3(-\cos x) + 7e^x + C$$

$$= \frac{5x^4}{4} + 3\cos x + 7e^x + C$$

---

### Example 2: Algebraic Simplification First

$$\int \frac{(x+1)(x+2)}{x}\, dx$$

**Expand and divide:**

$$= \int \frac{x^2 + 3x + 2}{x}\, dx = \int \left(x + 3 + \frac{2}{x}\right) dx$$

**Now integrate term by term:**

$$= \frac{x^2}{2} + 3x + 2\ln|x| + C$$

---

### Example 3: Using Trigonometric Identities

$$\int \cos^2 x\, dx$$

Use the identity: cos²x = (1 + cos 2x)/2

$$= \int \frac{1 + \cos 2x}{2}\, dx = \frac{1}{2}\int dx + \frac{1}{2}\int \cos 2x\, dx$$

$$= \frac{x}{2} + \frac{\sin 2x}{4} + C$$

---

### Example 4: Rationalizing Before Integrating

$$\int \frac{dx}{1 + \sin x}$$

Multiply numerator and denominator by (1 − sin x):

$$= \int \frac{1 - \sin x}{(1 + \sin x)(1 - \sin x)}\, dx = \int \frac{1 - \sin x}{1 - \sin^2 x}\, dx = \int \frac{1 - \sin x}{\cos^2 x}\, dx$$

$$= \int (\sec^2 x - \sec x \tan x)\, dx = \tan x - \sec x + C$$

---

### Example 5: Splitting a Fraction

$$\int \frac{x^4 + x^2 + 1}{x^2 + 1}\, dx$$

Perform polynomial long division:

```
  x⁴ + x² + 1  ÷  (x² + 1)

       x²        ← first quotient term
  x²+1 ) x⁴ + x² + 1
         x⁴ + x²
         ─────────
              0  + 1

  Result: x² + 1/(x²+1)
```

Wait — let's redo carefully:

x⁴ + x² + 1 = (x² + 1)(x²) + 1, so:

$$\frac{x^4 + x^2 + 1}{x^2 + 1} = x^2 + \frac{1}{x^2 + 1}$$

$$\int \left(x^2 + \frac{1}{x^2+1}\right) dx = \frac{x^3}{3} + \tan^{-1}x + C$$

---

## 4.7 Properties in Terms of Differentials

Using differential notation provides an elegant way to rewrite properties:

| Property | Integral Form | Differential Form |
|---|---|---|
| Constant multiple | ∫kf(x)dx = k∫f(x)dx | d(kF) = k·dF |
| Sum | ∫(f+g)dx = ∫fdx + ∫gdx | d(F+G) = dF + dG |
| Fundamental | d/dx[∫f(x)dx] = f(x) | d(∫f dx) = f dx |

---

## 4.8 Real-World Application

**Problem:** A factory's production rate is R(t) = 200 + 50t − t² units/hour. The maintenance cost rate is M(t) = 30 + 5t units/hour. Find the net production function if net production at t = 0 is 0.

**Solution:**

Net rate = R(t) − M(t) = (200 + 50t − t²) − (30 + 5t) = 170 + 45t − t²

By the difference property:

$$N(t) = \int (170 + 45t - t^2)\, dt = 170t + \frac{45t^2}{2} - \frac{t^3}{3} + C$$

N(0) = 0 → C = 0

$$N(t) = 170t + \frac{45t^2}{2} - \frac{t^3}{3}$$

---

## Summary Table

| Property | Formula | Status |
|---|---|---|
| Constant Multiple | ∫ kf(x) dx = k ∫ f(x) dx | ✅ Valid |
| Sum / Difference | ∫ (f ± g) dx = ∫ f dx ± ∫ g dx | ✅ Valid |
| Linearity | ∫ (αf + βg) dx = α∫f dx + β∫g dx | ✅ Valid |
| Product | ∫ (f·g) dx ≠ (∫f dx)(∫g dx) | ❌ Invalid |
| Quotient | ∫ (f/g) dx ≠ (∫f dx)/(∫g dx) | ❌ Invalid |
| Composition | ∫ f(g(x)) dx ≠ F(g(x)) + C | ❌ Invalid |
| Strategy | Simplify algebraically → split → use formulas | — |

---

## Quick Revision Questions

1. State the constant multiple rule and the sum rule for indefinite integrals.

2. Show with a counterexample that ∫ f(x)g(x) dx ≠ [∫ f(x)dx][∫ g(x)dx].

3. Evaluate ∫ (4sin x − 7cos x + 3/x) dx using the linearity property.

4. Simplify and evaluate: ∫ (x + 1)³ / x² dx. (Hint: expand the numerator first.)

5. Why does the integral ∫ f(g(x)) dx generally NOT equal F(g(x)) + C? Give an example.

6. A car's fuel consumption rate is C(t) = 8 + 0.5t liters/hour. Find the total fuel consumed from t = 0 to express it as a function, given initial fuel consumed = 0 liters.

---

[← Previous: Basic Integration Formulas](03-basic-integration-formulas.md) | [Back to README](../README.md) | [Next: Substitution Method →](../02-Methods-of-Integration/01-substitution-method.md)
