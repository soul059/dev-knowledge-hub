# Chapter 2: King Property

[← Previous: Symmetry Properties](01-symmetry-properties.md) | [Back to README](../README.md) | [Next: Periodic Function Integrals →](03-periodic-function-integrals.md)

---

## 2.1 Overview

The **King Property** (also called the **reflection property** or **Queen Rule**) is one of the most elegant and powerful properties of definite integrals. It states:

$$\boxed{\int_a^b f(x)\, dx = \int_a^b f(a + b - x)\, dx}$$

This property is a favorite in competitive exams and appears in many tricky definite integral problems.

---

## 2.2 Proof

Let $t = a + b - x$. Then $dt = -dx$.

When x = a → t = b; when x = b → t = a.

$$\int_a^b f(x)\, dx = \int_b^a f(a+b-t)(-dt) = \int_a^b f(a+b-t)\, dt$$

Since t is a dummy variable, we can rename it x:

$$= \int_a^b f(a+b-x)\, dx \quad \blacksquare$$

---

## 2.3 Geometric Interpretation

The substitution x → (a + b − x) **reflects** the integration about the midpoint (a+b)/2:

```
  x:     a ──────────── b
         │              │
         │   midpoint   │
         │   (a+b)/2    │
         │              │
  a+b-x: b ──────────── a
  
  The function is "flipped" about the midpoint,
  but the integral value remains unchanged.
```

---

## 2.4 Special Cases

### Case 1: [0, a] — Most Common Form

$$\int_0^a f(x)\, dx = \int_0^a f(a - x)\, dx$$

### Case 2: [0, π/2]

$$\int_0^{\pi/2} f(\sin x)\, dx = \int_0^{\pi/2} f(\cos x)\, dx$$

Because sin(π/2 − x) = cos x.

### Case 3: [0, π]

$$\int_0^{\pi} x\, f(\sin x)\, dx = \frac{\pi}{2}\int_0^{\pi} f(\sin x)\, dx$$

**Proof:** Apply King Property with a = 0, b = π:

$$I = \int_0^{\pi} x\, f(\sin x)\, dx = \int_0^{\pi}(\pi - x) f(\sin(\pi-x))\, dx = \int_0^{\pi}(\pi - x) f(\sin x)\, dx$$

Adding: $2I = \pi \int_0^{\pi} f(\sin x)\, dx$, so $I = \frac{\pi}{2}\int_0^{\pi} f(\sin x)\, dx$.

---

## 2.5 The Power of King Property — Strategy

```
  Given: I = ∫ₐᵇ f(x) dx    (looks hard to compute directly)
     │
     ▼
  Apply King: I = ∫ₐᵇ f(a+b−x) dx
     │
     ▼
  Add both expressions: 2I = ∫ₐᵇ [f(x) + f(a+b−x)] dx
     │
     ▼
  Often the sum f(x) + f(a+b−x) SIMPLIFIES dramatically!
     │
     ▼
  Solve for I = (1/2) ∫ₐᵇ [simplified expression] dx
```

---

## 2.6 Worked Examples

### Example 1: Classic Application

$$I = \int_0^{\pi/2} \frac{\sin x}{\sin x + \cos x}\, dx$$

Apply King Property (a + b − x = π/2 − x):

$$I = \int_0^{\pi/2} \frac{\sin(\pi/2-x)}{\sin(\pi/2-x) + \cos(\pi/2-x)}\, dx = \int_0^{\pi/2} \frac{\cos x}{\cos x + \sin x}\, dx$$

Add both:

$$2I = \int_0^{\pi/2} \frac{\sin x + \cos x}{\sin x + \cos x}\, dx = \int_0^{\pi/2} 1\, dx = \frac{\pi}{2}$$

$$\boxed{I = \frac{\pi}{4}}$$

---

### Example 2: Logarithmic Trick

$$I = \int_0^{\pi/2} \ln(\tan x)\, dx$$

Apply King Property:

$$I = \int_0^{\pi/2} \ln(\tan(\pi/2 - x))\, dx = \int_0^{\pi/2} \ln(\cot x)\, dx = \int_0^{\pi/2} \ln\left(\frac{1}{\tan x}\right) dx$$

$$= \int_0^{\pi/2} (-\ln(\tan x))\, dx = -I$$

So $I = -I \implies 2I = 0 \implies I = 0$.

$$\boxed{I = 0}$$

---

### Example 3: ∫₀^π x sin x / (1 + cos²x) dx

$$I = \int_0^{\pi} \frac{x\sin x}{1 + \cos^2 x}\, dx$$

Apply King Property (x → π − x, sin(π−x) = sin x, cos(π−x) = −cos x):

$$I = \int_0^{\pi} \frac{(\pi - x)\sin x}{1 + \cos^2 x}\, dx$$

Add: $2I = \pi \int_0^{\pi} \frac{\sin x}{1 + \cos^2 x}\, dx$

Let u = cos x, du = −sin x dx. When x = 0 → u = 1; x = π → u = −1.

$$2I = \pi \int_1^{-1} \frac{-du}{1+u^2} = \pi \int_{-1}^{1} \frac{du}{1+u^2} = \pi \Big[\tan^{-1}u\Big]_{-1}^{1} = \pi\left(\frac{\pi}{4} + \frac{\pi}{4}\right) = \frac{\pi^2}{2}$$

$$\boxed{I = \frac{\pi^2}{4}}$$

---

### Example 4: General f(x)/[f(x) + f(a−x)] Pattern

$$\int_0^a \frac{f(x)}{f(x) + f(a-x)}\, dx = \frac{a}{2}$$

**Proof:** Let $I = \int_0^a \frac{f(x)}{f(x)+f(a-x)}\, dx$

King Property: $I = \int_0^a \frac{f(a-x)}{f(a-x)+f(x)}\, dx$

Add: $2I = \int_0^a \frac{f(x)+f(a-x)}{f(x)+f(a-x)}\, dx = \int_0^a 1\, dx = a$

$$I = \frac{a}{2}$$

This is a **universal result** — it works for ANY continuous function f!

### Applications of the Universal Result

| Integral | Value |
|---|---|
| ∫₀^{π/2} sin x/(sin x + cos x) dx | π/4 |
| ∫₀^{π/2} cos x/(sin x + cos x) dx | π/4 |
| ∫₀^{π/2} √(sin x)/(√(sin x) + √(cos x)) dx | π/4 |
| ∫₀¹ xⁿ/(xⁿ + (1−x)ⁿ) dx | 1/2 |
| ∫₀^{π/2} sinⁿx/(sinⁿx + cosⁿx) dx | π/4 |

---

## 2.7 When to Suspect King Property

Look for these clues:

1. Limits are 0 to a (or a to b with special structure)
2. The integrand has a ratio of the form f(x)/[f(x) + f(a−x)]
3. Trigonometric integrals on [0, π/2] involving sin and cos interchangeably
4. The integral looks difficult by standard methods but has a "clean" answer expected

---

## Summary Table

| Concept | Key Point |
|---|---|
| King Property | ∫ₐᵇ f(x) dx = ∫ₐᵇ f(a+b−x) dx |
| Special case | ∫₀ᵃ f(x) dx = ∫₀ᵃ f(a−x) dx |
| Strategy | Add I and its King-transformed version → simplification |
| Universal result | ∫₀ᵃ f(x)/[f(x)+f(a−x)] dx = a/2 |
| Trig special | ∫₀^{π/2} f(sin x) dx = ∫₀^{π/2} f(cos x) dx |
| [0, π] special | ∫₀^π x·g(sin x) dx = (π/2)∫₀^π g(sin x) dx |

---

## Quick Revision Questions

1. State and prove the King Property of definite integrals.

2. Evaluate ∫₀^{π/2} cos x / (sin x + cos x) dx using the King Property.

3. Show that ∫₀¹ ln(1 + x)/(2 + x²) dx requires King Property (set up the approach).

4. Evaluate ∫₀^{π/2} sin²x / (sin²x + cos²x) dx (simplify first!).

5. Prove that ∫₀^π x f(sin x) dx = (π/2) ∫₀^π f(sin x) dx for any continuous f.

6. Evaluate ∫₀^{π/2} (sinⁿx)/(sinⁿx + cosⁿx) dx for any positive integer n.

---

[← Previous: Symmetry Properties](01-symmetry-properties.md) | [Back to README](../README.md) | [Next: Periodic Function Integrals →](03-periodic-function-integrals.md)
