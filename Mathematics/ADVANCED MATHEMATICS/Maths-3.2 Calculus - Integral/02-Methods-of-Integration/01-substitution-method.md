# Chapter 1: Substitution Method (u-Substitution)

[← Previous: Properties of Integrals](../01-Introduction-to-Integration/04-properties-of-integrals.md) | [Back to README](../README.md) | [Next: Integration by Parts →](02-integration-by-parts.md)

---

## 1.1 Overview

The **substitution method** (also called **u-substitution**) is the integration counterpart of the **chain rule** for differentiation. It is the most frequently used integration technique and works by transforming a complicated integral into a simpler one through a change of variable.

---

## 1.2 Theoretical Foundation — The Chain Rule in Reverse

Recall the **chain rule**:

$$\frac{d}{dx}[F(g(x))] = F'(g(x)) \cdot g'(x) = f(g(x)) \cdot g'(x)$$

Reading this from right to left gives us:

$$\int f(g(x)) \cdot g'(x)\, dx = F(g(x)) + C$$

### The Substitution Idea

Let **u = g(x)**, then **du = g'(x) dx**. The integral becomes:

$$\int f(u)\, du = F(u) + C$$

After integration, substitute back u = g(x).

---

## 1.3 Step-by-Step Procedure

```
  Step 1: Identify the "inner function" g(x) → set u = g(x)
     │
     ▼
  Step 2: Compute du = g'(x) dx
     │
     ▼
  Step 3: Express the ENTIRE integral in terms of u and du
     │    (no x or dx should remain)
     ▼
  Step 4: Integrate with respect to u
     │
     ▼
  Step 5: Substitute back u = g(x)
     │
     ▼
  Step 6: Add + C and verify by differentiation
```

---

## 1.4 How to Choose u — Guidelines

| Situation | Choose u as... | Why? |
|---|---|---|
| ∫ f(g(x)) · g'(x) dx | u = g(x) | Its derivative g'(x) appears as a factor |
| ∫ xⁿ · f(xⁿ⁺¹) dx | u = xⁿ⁺¹ | du = (n+1)xⁿ dx matches the factor |
| ∫ f(ax+b) dx | u = ax+b | Simplest linear substitution |
| ∫ f(√x) / √x dx | u = √x | du = dx/(2√x) matches |
| ∫ f(ln x) / x dx | u = ln x | du = dx/x matches |
| ∫ f(eˣ) · eˣ dx | u = eˣ | du = eˣ dx matches |

> **Golden Rule:** Look for a function whose **derivative** (or a constant multiple of it) also appears in the integrand.

---

## 1.5 Worked Examples

### Example 1: Linear Substitution

$$\int (3x + 2)^7\, dx$$

**Let** u = 3x + 2, then du = 3 dx → dx = du/3

$$= \int u^7 \cdot \frac{du}{3} = \frac{1}{3} \cdot \frac{u^8}{8} + C = \frac{(3x+2)^8}{24} + C$$

---

### Example 2: Power with Matching Derivative

$$\int x^2 \sqrt{x^3 + 1}\, dx$$

**Let** u = x³ + 1, then du = 3x² dx → x² dx = du/3

$$= \int \sqrt{u} \cdot \frac{du}{3} = \frac{1}{3} \cdot \frac{u^{3/2}}{3/2} + C = \frac{2}{9}(x^3 + 1)^{3/2} + C$$

---

### Example 3: Trigonometric Substitution

$$\int \sin^3 x \cos x\, dx$$

**Let** u = sin x, then du = cos x dx

$$= \int u^3\, du = \frac{u^4}{4} + C = \frac{\sin^4 x}{4} + C$$

---

### Example 4: Exponential Argument

$$\int e^{\sin x} \cos x\, dx$$

**Let** u = sin x, then du = cos x dx

$$= \int e^u\, du = e^u + C = e^{\sin x} + C$$

---

### Example 5: Logarithmic

$$\int \frac{\ln x}{x}\, dx$$

**Let** u = ln x, then du = dx/x

$$= \int u\, du = \frac{u^2}{2} + C = \frac{(\ln x)^2}{2} + C$$

---

### Example 6: Tangent

$$\int \tan x\, dx = \int \frac{\sin x}{\cos x}\, dx$$

**Let** u = cos x, then du = −sin x dx → sin x dx = −du

$$= \int \frac{-du}{u} = -\ln|u| + C = -\ln|\cos x| + C = \ln|\sec x| + C$$

---

### Example 7: Adjusting Constants

$$\int x e^{x^2}\, dx$$

**Let** u = x², then du = 2x dx → x dx = du/2

$$= \int e^u \cdot \frac{du}{2} = \frac{e^u}{2} + C = \frac{e^{x^2}}{2} + C$$

---

## 1.6 Common Patterns — Quick Reference

| Integral Pattern | Substitution | Result |
|---|---|---|
| ∫ [f(x)]ⁿ · f'(x) dx | u = f(x) | [f(x)]ⁿ⁺¹/(n+1) + C |
| ∫ f'(x)/f(x) dx | u = f(x) | ln\|f(x)\| + C |
| ∫ f'(x) · eᶠ⁽ˣ⁾ dx | u = f(x) | eᶠ⁽ˣ⁾ + C |
| ∫ f(ax+b) dx | u = ax+b | (1/a)F(ax+b) + C |
| ∫ sin(f(x))·f'(x) dx | u = f(x) | −cos(f(x)) + C |
| ∫ cos(f(x))·f'(x) dx | u = f(x) | sin(f(x)) + C |

---

## 1.7 When Substitution Fails

Substitution works when you can express the **entire** integral in terms of u and du. It fails when:

1. **No matching derivative:** The derivative of the proposed u does not appear in the integrand, and you cannot algebraically create it.
2. **Both x and u remain:** After substitution, x terms remain that cannot be expressed in terms of u.

```
  ∫ f(g(x)) · h(x) dx    where h(x) ≠ k · g'(x)
     │
     ▼
  Substitution FAILS → Try:
     ├── Integration by parts
     ├── Partial fractions
     ├── Trigonometric substitutions
     └── Other techniques
```

---

## 1.8 Substitution in Definite Integrals (Preview)

When using substitution with definite integrals, change the **limits** too:

$$\int_a^b f(g(x)) \cdot g'(x)\, dx = \int_{g(a)}^{g(b)} f(u)\, du$$

This avoids back-substitution entirely. (Covered in detail in Unit 4.)

---

## 1.9 Real-World Application

**Problem:** The rate of bacterial growth is modeled by dN/dt = 500e^(0.03t). Find the population function N(t) if N(0) = 10,000.

**Solution:**

$$N(t) = \int 500 e^{0.03t}\, dt$$

Let u = 0.03t, du = 0.03 dt → dt = du/0.03

$$= 500 \int e^u \cdot \frac{du}{0.03} = \frac{500}{0.03} e^u + C = \frac{50000}{3} e^{0.03t} + C$$

Apply N(0) = 10000:

$$10000 = \frac{50000}{3} + C \implies C = 10000 - \frac{50000}{3} = \frac{-20000}{3}$$

$$N(t) = \frac{50000}{3} e^{0.03t} - \frac{20000}{3}$$

---

## Summary Table

| Concept | Key Point |
|---|---|
| Core idea | Reverse of chain rule; change variable to simplify |
| Substitution | u = g(x), du = g'(x) dx |
| Key requirement | The derivative g'(x) must appear (up to a constant) in the integrand |
| Linear sub | ∫ f(ax+b) dx = (1/a)F(ax+b) + C |
| Important pattern | ∫ f'(x)/f(x) dx = ln\|f(x)\| + C |
| Verify answer | Always differentiate your result using the chain rule |
| When it fails | Move on to integration by parts, partial fractions, etc. |

---

## Quick Revision Questions

1. State the substitution theorem and explain how it relates to the chain rule.

2. Evaluate ∫ (2x + 1)⁵ dx using substitution.

3. Evaluate ∫ cos(3x) dx. What substitution do you use?

4. Why does ∫ √(x² + 1) dx (without an x factor) resist simple u-substitution?

5. Evaluate ∫ x/(x² + 4) dx using substitution.

6. If u = f(x) and du = f'(x) dx, rewrite ∫ sin(ln x)/x dx entirely in terms of u and evaluate.

---

[← Previous: Properties of Integrals](../01-Introduction-to-Integration/04-properties-of-integrals.md) | [Back to README](../README.md) | [Next: Integration by Parts →](02-integration-by-parts.md)
