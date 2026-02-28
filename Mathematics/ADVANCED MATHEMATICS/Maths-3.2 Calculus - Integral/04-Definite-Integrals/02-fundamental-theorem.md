# Chapter 2: Fundamental Theorem of Calculus

[← Previous: Riemann Sums](01-riemann-sums.md) | [Back to README](../README.md) | [Next: Properties of Definite Integrals →](03-properties-definite-integrals.md)

---

## 2.1 Overview

The **Fundamental Theorem of Calculus (FTC)** is arguably the most important theorem in all of calculus. It provides the crucial link between **differentiation** and **integration**, showing they are inverse operations. It has two parts.

---

## 2.2 Fundamental Theorem of Calculus — Part 1 (FTC1)

> **Theorem (FTC Part 1):** If f is continuous on [a, b], define the **accumulation function**:
>
> $$F(x) = \int_a^x f(t)\, dt$$
>
> Then F is differentiable on (a, b) and:
>
> $$F'(x) = \frac{d}{dx}\int_a^x f(t)\, dt = f(x)$$

### What This Means

```
  FTC Part 1 says:
  
  "The derivative of the integral (as a function of the upper limit)
   gives back the original integrand."

          ∫ₐˣ f(t) dt
              │
   d/dx       │          ← Differentiation UNDOES integration
              ▼
           f(x)
```

### Geometric Interpretation

The accumulation function F(x) = ∫ₐˣ f(t) dt represents the **area under f from a to x**:

```
  y
  │     f(t)
  │    ╱──╲
  │   ╱    ╲
  │  ╱██████╲         ← Shaded area = F(x) = ∫ₐˣ f(t)dt
  │ ╱████████╲
  └─┬────────┬─────── t
    a        x
    
  As x increases by Δx:
  ΔF ≈ f(x)·Δx  →  F'(x) = f(x)
```

### Example

$$\frac{d}{dx}\int_1^x \sqrt{t^3 + 1}\, dt = \sqrt{x^3 + 1}$$

### When the Upper Limit is a Function of x

If the upper limit is g(x) instead of x, use the **chain rule**:

$$\frac{d}{dx}\int_a^{g(x)} f(t)\, dt = f(g(x)) \cdot g'(x)$$

**Example:**

$$\frac{d}{dx}\int_0^{x^2} e^{-t^2}\, dt = e^{-x^4} \cdot 2x$$

---

## 2.3 Fundamental Theorem of Calculus — Part 2 (FTC2)

> **Theorem (FTC Part 2 — Evaluation Theorem):** If f is continuous on [a, b] and F is ANY antiderivative of f (i.e., F' = f), then:
>
> $$\int_a^b f(x)\, dx = F(b) - F(a)$$

This is often written as:

$$\int_a^b f(x)\, dx = \Big[F(x)\Big]_a^b = F(b) - F(a)$$

### What This Means

```
  FTC Part 2 says:
  
  "To evaluate a definite integral, find ANY antiderivative F
   and compute F(b) − F(a)."

  No need for Riemann sums!
  
  ∫ₐᵇ f(x) dx
       │
       ▼
  Find F such that F' = f
       │
       ▼
  Compute F(b) − F(a) ← This is just arithmetic!
```

### The Grand Connection

```
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  DIFFERENTIATION  ◄───── FTC ─────►  INTEGRATION │
  │                                                 │
  │  d/dx[F(x)] = f(x)  ←→  ∫ₐᵇf(x)dx = F(b)−F(a)│
  │                                                 │
  │  They are INVERSE operations                    │
  │  connected by the Fundamental Theorem           │
  └─────────────────────────────────────────────────┘
```

---

## 2.4 Worked Examples

### Example 1: Basic Evaluation

$$\int_1^3 (2x + 1)\, dx = \Big[x^2 + x\Big]_1^3 = (9 + 3) - (1 + 1) = 12 - 2 = 10$$

---

### Example 2: Trigonometric

$$\int_0^{\pi/2} \cos x\, dx = \Big[\sin x\Big]_0^{\pi/2} = \sin(\pi/2) - \sin(0) = 1 - 0 = 1$$

---

### Example 3: Exponential

$$\int_0^1 e^x\, dx = \Big[e^x\Big]_0^1 = e^1 - e^0 = e - 1$$

---

### Example 4: Logarithmic

$$\int_1^e \frac{1}{x}\, dx = \Big[\ln|x|\Big]_1^e = \ln e - \ln 1 = 1 - 0 = 1$$

---

### Example 5: FTC Part 1 with Chain Rule

$$G(x) = \int_1^{\sin x} \frac{dt}{1+t^2}$$

$$G'(x) = \frac{1}{1+\sin^2 x} \cdot \cos x = \frac{\cos x}{1+\sin^2 x}$$

---

### Example 6: Both Limits are Functions

$$H(x) = \int_{x^2}^{x^3} \ln t\, dt$$

Split: $H(x) = \int_a^{x^3} - \int_a^{x^2}$ for any constant a.

$$H'(x) = \ln(x^3) \cdot 3x^2 - \ln(x^2) \cdot 2x = 3x^2 \cdot 3\ln x - 2x \cdot 2\ln x = 9x^2\ln x - 4x\ln x$$

---

## 2.5 Why FTC is Revolutionary

Before FTC, mathematicians had to:
1. Calculate areas using **exhaustion** (Archimedes' method)
2. Compute limits of Riemann sums (tedious!)

After FTC:
1. Find an antiderivative (often straightforward with techniques from Units 1–3)
2. Plug in two numbers and subtract

```
  BEFORE FTC:                          AFTER FTC:
  
  ∫₀¹ x² dx                           ∫₀¹ x² dx
  = lim Σ(i/n)²·(1/n)                 = [x³/3]₀¹
  = lim (1/n³)·n(n+1)(2n+1)/6         = 1/3 − 0
  = lim (2n²+3n+1)/(6n²)              = 1/3
  = 1/3                               
                                       Done in 2 steps!
  Took many steps!
```

---

## 2.6 Important Consequences

### The Net Change Theorem

$$\int_a^b F'(x)\, dx = F(b) - F(a)$$

> "The integral of a rate of change gives the total change."

| Context | Rate F'(x) | Total Change |
|---|---|---|
| Position s(t) | Velocity v(t) | Displacement s(b) − s(a) |
| Velocity v(t) | Acceleration a(t) | Change in velocity |
| Population P(t) | Growth rate P'(t) | Population change |
| Cost C(x) | Marginal cost C'(x) | Total additional cost |

---

## Summary Table

| Concept | Key Point |
|---|---|
| FTC Part 1 | d/dx ∫ₐˣ f(t) dt = f(x) |
| FTC Part 1 (chain) | d/dx ∫ₐ^{g(x)} f(t) dt = f(g(x))·g'(x) |
| FTC Part 2 | ∫ₐᵇ f(x) dx = F(b) − F(a), where F' = f |
| Connection | Differentiation and integration are inverse operations |
| Net Change | ∫ₐᵇ F'(x) dx = F(b) − F(a) |
| No constant C | In definite integrals, C cancels out |
| Revolution | Converts limit-of-sums into simple arithmetic |

---

## Quick Revision Questions

1. State both parts of the Fundamental Theorem of Calculus.

2. Evaluate ∫₂⁵ (3x² − 2x + 1) dx using FTC Part 2.

3. Find d/dx ∫₀ˣ sin(t²) dt.

4. Find d/dx ∫₁^{eˣ} ln(t) dt using the chain rule version of FTC Part 1.

5. Explain in words why FTC Part 2 does NOT require the constant of integration C.

6. Using the Net Change Theorem, if a car has velocity v(t) = 3t² − 6t m/s, find the displacement from t = 0 to t = 4.

---

[← Previous: Riemann Sums](01-riemann-sums.md) | [Back to README](../README.md) | [Next: Properties of Definite Integrals →](03-properties-definite-integrals.md)
