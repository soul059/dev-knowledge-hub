# Chapter 1: Antiderivatives

[← Back to README](../README.md) | [Next: Indefinite Integrals →](02-indefinite-integrals.md)

---

## 1.1 Overview

An **antiderivative** (also called a **primitive**) of a function f(x) is any function F(x) whose derivative equals f(x):

$$F'(x) = f(x)$$

Integration is fundamentally the **reverse of differentiation**. While differentiation asks "given a function, what is its rate of change?", antidifferentiation asks "given a rate of change, what was the original function?"

### Conceptual Diagram — Differentiation vs Antidifferentiation

```
              differentiate
   F(x)  ─────────────────────►  f(x) = F'(x)
    ▲                                │
    │                                │
    │     antidifferentiate          │
    └────────────────────────────────┘
```

---

## 1.2 Definition

> **Definition:** A function F(x) is called an **antiderivative** of f(x) on an interval I if
> F'(x) = f(x) for every x in I.

### Key Insight — Non-Uniqueness

If F(x) is an antiderivative of f(x), then so is **F(x) + C** for any constant C, because:

$$\frac{d}{dx}[F(x) + C] = F'(x) + 0 = f(x)$$

This means antiderivatives are **not unique** — they form a **family of curves** differing only by a vertical shift.

### Visual Representation — Family of Antiderivatives

```
   y
   │        ╱  F(x) + 2
   │      ╱
   │    ╱───── F(x) + 1
   │  ╱
   │╱──────── F(x)          ← All are antiderivatives of f(x)
   │
   │╲──────── F(x) − 1
   │  ╲
   │    ╲──── F(x) − 2
   │      ╲
   └──────────────────── x
```

All curves have the **same slope** at each x-value — they are vertical translates of one another.

---

## 1.3 The Constant of Integration

> **Theorem:** If F(x) is any antiderivative of f(x) on an interval I, then the **most general antiderivative** of f(x) on I is:
>
> $$F(x) + C$$
>
> where C is an arbitrary constant called the **constant of integration**.

### Why the Constant Matters

Without the constant C, we would only find **one particular** antiderivative. The constant captures all possible initial conditions.

**Example:** Find the antiderivative of f(x) = 2x.

We need F(x) such that F'(x) = 2x.

$$F(x) = x^2 + C$$

Verification: d/dx(x² + C) = 2x ✓

---

## 1.4 Finding Antiderivatives — Reverse Rules

Since antidifferentiation reverses differentiation, we reverse the derivative rules:

| Derivative Rule | Antiderivative Rule |
|---|---|
| d/dx(xⁿ⁺¹/(n+1)) = xⁿ | Antiderivative of xⁿ is xⁿ⁺¹/(n+1), n ≠ −1 |
| d/dx(sin x) = cos x | Antiderivative of cos x is sin x |
| d/dx(−cos x) = sin x | Antiderivative of sin x is −cos x |
| d/dx(eˣ) = eˣ | Antiderivative of eˣ is eˣ |
| d/dx(ln\|x\|) = 1/x | Antiderivative of 1/x is ln\|x\| |

### Process Diagram

```
  Given: f(x)
     │
     ▼
  Ask: "What function has derivative f(x)?"
     │
     ▼
  Recall derivative rules in reverse
     │
     ▼
  Write F(x) + C
     │
     ▼
  Verify: d/dx [F(x) + C] = f(x) ✓
```

---

## 1.5 Worked Examples

### Example 1: Power Functions

Find the antiderivative of f(x) = x⁴.

**Solution:**

$$F(x) = \frac{x^{4+1}}{4+1} + C = \frac{x^5}{5} + C$$

**Check:** d/dx(x⁵/5 + C) = 5x⁴/5 = x⁴ ✓

---

### Example 2: Negative Exponents

Find the antiderivative of f(x) = 1/x³ = x⁻³.

**Solution:**

$$F(x) = \frac{x^{-3+1}}{-3+1} + C = \frac{x^{-2}}{-2} + C = -\frac{1}{2x^2} + C$$

**Check:** d/dx(−1/(2x²)) = d/dx(−x⁻²/2) = (−1/2)(−2)x⁻³ = x⁻³ = 1/x³ ✓

---

### Example 3: Fractional Exponents

Find the antiderivative of f(x) = √x = x^(1/2).

**Solution:**

$$F(x) = \frac{x^{1/2+1}}{1/2+1} + C = \frac{x^{3/2}}{3/2} + C = \frac{2}{3}x^{3/2} + C$$

---

### Example 4: Trigonometric Function

Find the antiderivative of f(x) = sec²x.

**Solution:**

We know d/dx(tan x) = sec²x, therefore:

$$F(x) = \tan x + C$$

---

### Example 5: Initial Value Problem

Find F(x) if F'(x) = 3x² − 4x + 1 and F(0) = 5.

**Step 1:** Find the general antiderivative.

$$F(x) = x^3 - 2x^2 + x + C$$

**Step 2:** Apply the initial condition F(0) = 5.

$$F(0) = 0 - 0 + 0 + C = C = 5$$

**Result:**

$$F(x) = x^3 - 2x^2 + x + 5$$

---

## 1.6 Existence of Antiderivatives

Not every function has an antiderivative in terms of elementary functions. For example:

- $e^{-x^2}$ — its antiderivative involves the **error function** erf(x)
- $\frac{\sin x}{x}$ — leads to the **sine integral** Si(x)
- $\frac{1}{\ln x}$ — leads to the **logarithmic integral** Li(x)

However, a key existence theorem states:

> **Theorem:** If f(x) is **continuous** on an interval [a, b], then f(x) has an antiderivative on [a, b].

---

## 1.7 Real-World Applications

| Application | Given (rate) | Find (total) |
|---|---|---|
| **Physics** | Velocity v(t) | Position s(t) = ∫v(t)dt |
| **Physics** | Acceleration a(t) | Velocity v(t) = ∫a(t)dt |
| **Economics** | Marginal cost MC(x) | Total cost C(x) = ∫MC(x)dx |
| **Biology** | Growth rate dP/dt | Population P(t) = ∫(dP/dt)dt |

### Example — Free Fall

A ball is dropped from rest. Acceleration due to gravity is a(t) = −9.8 m/s².

**Find velocity:**
$$v(t) = \int (-9.8)\, dt = -9.8t + C_1$$
At t = 0, v(0) = 0 → C₁ = 0, so v(t) = −9.8t

**Find position:**
$$s(t) = \int (-9.8t)\, dt = -4.9t^2 + C_2$$
At t = 0, s(0) = h₀ → C₂ = h₀, so s(t) = −4.9t² + h₀

---

## Summary Table

| Concept | Key Point |
|---|---|
| Antiderivative | F(x) such that F'(x) = f(x) |
| Constant of integration | Antiderivatives differ by a constant C |
| General antiderivative | F(x) + C (family of curves) |
| Power rule (reverse) | Antiderivative of xⁿ = xⁿ⁺¹/(n+1) + C, n ≠ −1 |
| Initial value problem | Use given condition to find specific C |
| Existence | Continuous functions always have antiderivatives |
| Verification | Always check by differentiating your answer |

---

## Quick Revision Questions

1. **Define** an antiderivative of f(x). Why is it not unique?

2. If F(x) = x³ + 7 and G(x) = x³ − 3 are both antiderivatives of f(x) = 3x², explain why this does not contradict the theory.

3. Find the most general antiderivative of f(x) = 6x² − 4x + 7.

4. Find F(x) given that F'(x) = cos x and F(π/2) = 0.

5. Why does the function f(x) = 1/x NOT have an antiderivative of the form xⁿ⁺¹/(n+1)? What is its antiderivative instead?

6. A car's velocity is v(t) = 3t² + 2t (m/s). If its position at t = 0 is 10 m, find the position function s(t).

---

[← Back to README](../README.md) | [Next: Indefinite Integrals →](02-indefinite-integrals.md)
