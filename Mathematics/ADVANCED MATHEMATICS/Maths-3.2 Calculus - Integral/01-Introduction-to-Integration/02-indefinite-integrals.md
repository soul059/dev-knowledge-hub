# Chapter 2: Indefinite Integrals

[← Previous: Antiderivatives](01-antiderivatives.md) | [Back to README](../README.md) | [Next: Basic Integration Formulas →](03-basic-integration-formulas.md)

---

## 2.1 Overview

The **indefinite integral** is the formal notation for the collection of all antiderivatives of a function. It introduces the integral sign ∫ and establishes the language and notation used throughout integral calculus.

---

## 2.2 Definition and Notation

> **Definition:** The **indefinite integral** of f(x) with respect to x is written as:
>
> $$\int f(x)\, dx = F(x) + C$$
>
> where F'(x) = f(x) and C is the constant of integration.

### Anatomy of the Integral Symbol

```
    ┌─── Integral sign (elongated S, from "summa")
    │
    │    ┌─── Integrand (the function being integrated)
    │    │
    ∫  f(x)  dx    ◄─── Differential (indicates variable of integration)
                         Also acts as a "closing bracket"
```

### Historical Note

The integral sign ∫ was introduced by **Leibniz** in 1675. It is a stylized letter "S" standing for "summa" (Latin for sum), reflecting the integral's connection to summation.

---

## 2.3 Terminology

| Term | Meaning |
|---|---|
| **∫** | Integral sign |
| **f(x)** | Integrand — the function being integrated |
| **dx** | Differential of x — indicates the variable of integration |
| **F(x)** | Antiderivative / Primitive |
| **C** | Constant of integration |
| **∫ f(x) dx** | The entire expression: "the integral of f(x) with respect to x" |

---

## 2.4 The Operation of Integration

Integration is an **operator** that acts on functions, similar to differentiation:

```
  Differentiation operator:    d/dx
  ┌──────────────────────────────────────┐
  │   d/dx : F(x)  ──►  f(x)            │
  └──────────────────────────────────────┘

  Integration operator:        ∫ ... dx
  ┌──────────────────────────────────────┐
  │   ∫ dx : f(x)  ──►  F(x) + C       │
  └──────────────────────────────────────┘
```

### Inverse Relationship

Differentiation and integration are **inverse operations** (up to a constant):

$$\frac{d}{dx}\left[\int f(x)\, dx\right] = f(x)$$

$$\int F'(x)\, dx = F(x) + C$$

> The first says: **differentiating an integral gives back the integrand.**
>
> The second says: **integrating a derivative gives back the function (up to a constant).**

### Flow Diagram

```
           d/dx
  F(x)+C ──────► f(x)
     ▲              │
     │    ∫ dx      │
     └──────────────┘

  These operations UNDO each other
  (except integration introduces +C)
```

---

## 2.5 Indefinite vs Definite Integrals — Preview

| Feature | Indefinite Integral | Definite Integral |
|---|---|---|
| **Notation** | ∫ f(x) dx | ∫ₐᵇ f(x) dx |
| **Result** | A function + C | A number |
| **Meaning** | Family of antiderivatives | Net signed area under curve |
| **Constant C** | Present | Not present |

```
  Indefinite:               Definite:

  ∫ f(x) dx = F(x) + C     ∫ₐᵇ f(x) dx = F(b) − F(a)

  Output: function           Output: number
```

We will study definite integrals in detail in Unit 4.

---

## 2.6 Worked Examples

### Example 1: Polynomial

$$\int (3x^2 + 2x - 5)\, dx$$

Integrate term by term:

$$= 3 \cdot \frac{x^3}{3} + 2 \cdot \frac{x^2}{2} - 5x + C = x^3 + x^2 - 5x + C$$

---

### Example 2: Radical and Rational Expression

$$\int \left(\sqrt{x} + \frac{1}{x^2}\right) dx = \int \left(x^{1/2} + x^{-2}\right) dx$$

$$= \frac{x^{3/2}}{3/2} + \frac{x^{-1}}{-1} + C = \frac{2}{3}x^{3/2} - \frac{1}{x} + C$$

---

### Example 3: Trigonometric

$$\int (3\sin x + 4\cos x)\, dx = -3\cos x + 4\sin x + C$$

---

### Example 4: Exponential

$$\int 5e^x\, dx = 5e^x + C$$

---

### Example 5: Combination

$$\int \left(\frac{1}{x} + e^x - \sec^2 x\right) dx = \ln|x| + e^x - \tan x + C$$

---

## 2.7 Common Misconceptions

| Misconception | Correction |
|---|---|
| "The integral of f(x)·g(x) is (∫f dx)·(∫g dx)" | **FALSE** — there is no product rule for integration |
| "The integral of f(x)/g(x) is (∫f dx)/(∫g dx)" | **FALSE** — there is no quotient rule for integration |
| "C is always zero" | **FALSE** — C is an arbitrary constant; it cannot be determined without an initial condition |
| "Forgetting dx" | dx is essential — it specifies the integration variable and closes the integral |
| "∫ f(x) dx is a single number" | **FALSE** — the indefinite integral is a **family of functions** |

---

## 2.8 Differential Equations Connection

An equation of the form:

$$\frac{dy}{dx} = f(x)$$

is a simple **differential equation**. Solving it means finding y:

$$y = \int f(x)\, dx = F(x) + C$$

The constant C is determined by an **initial condition** (a known point on the curve).

### Decision Flowchart

```
  Given: dy/dx = f(x)
     │
     ├──► Integrate both sides: y = ∫ f(x) dx = F(x) + C
     │
     ├──► Initial condition given?
     │         │
     │    Yes  │                    No
     │    ▼                         ▼
     │  Substitute to find C    General solution
     │  → Particular solution    y = F(x) + C
     │    y = F(x) + C₀
```

---

## 2.9 Real-World Applications

**Problem:** The marginal revenue for selling x units of a product is MR(x) = 100 − 2x (dollars per unit). The revenue from selling 0 units is $0. Find the revenue function R(x).

**Solution:**

$$R(x) = \int MR(x)\, dx = \int (100 - 2x)\, dx = 100x - x^2 + C$$

Apply R(0) = 0: C = 0

$$R(x) = 100x - x^2$$

---

## Summary Table

| Concept | Key Point |
|---|---|
| Indefinite integral | ∫ f(x) dx = F(x) + C, where F'(x) = f(x) |
| ∫ sign | elongated S from "summa" (sum) |
| dx | Specifies the variable of integration |
| C | Arbitrary constant, essential in indefinite integrals |
| Inverse of d/dx | d/dx [∫f(x)dx] = f(x) and ∫F'(x)dx = F(x) + C |
| Not a number | Indefinite integral yields a family of functions |
| No product/quotient rule | ∫(fg) ≠ (∫f)(∫g); need special techniques |

---

## Quick Revision Questions

1. Write the indefinite integral of f(x) = 4x³ − 6x + 1 and verify by differentiation.

2. Explain the meaning of each part of the expression ∫ f(x) dx.

3. True or False: ∫[f(x) · g(x)] dx = [∫f(x) dx] · [∫g(x) dx]. Justify.

4. If d/dx [∫ f(x) dx] = f(x), what is ∫ [d/dx(F(x))] dx?

5. The velocity of a particle is v(t) = 6t − 2 m/s and its position at t = 1 is s(1) = 4 m. Find s(t).

6. Explain why the indefinite integral ∫ 1/x dx = ln|x| + C uses the absolute value |x|.

---

[← Previous: Antiderivatives](01-antiderivatives.md) | [Back to README](../README.md) | [Next: Basic Integration Formulas →](03-basic-integration-formulas.md)
