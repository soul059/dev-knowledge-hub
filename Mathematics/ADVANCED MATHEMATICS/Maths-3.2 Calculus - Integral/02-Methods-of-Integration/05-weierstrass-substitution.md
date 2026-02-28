# Chapter 5: Weierstrass Substitution

[← Previous: Trigonometric Substitutions](04-trigonometric-substitutions.md) | [Back to README](../README.md) | [Next: Integrals Involving Square Roots →](../03-Special-Integrals/01-integrals-involving-square-roots.md)

---

## 5.1 Overview

The **Weierstrass substitution** (also called the **tangent half-angle substitution** or **universal trigonometric substitution**) is a powerful technique that converts **any** rational function of sin x and cos x into a rational function of a new variable t, which can then be integrated using partial fractions.

$$t = \tan\left(\frac{x}{2}\right)$$

This substitution is called "universal" because it works for **every** integral of the form ∫ R(sin x, cos x) dx.

---

## 5.2 The Substitution Formulas

Let $t = \tan(x/2)$. Then:

$$\sin x = \frac{2t}{1 + t^2}$$

$$\cos x = \frac{1 - t^2}{1 + t^2}$$

$$dx = \frac{2\, dt}{1 + t^2}$$

### Derivation

Starting from $t = \tan(x/2)$:

**For sin x:** Using the double-angle formula:
$$\sin x = 2\sin\frac{x}{2}\cos\frac{x}{2} = 2 \cdot \frac{\tan(x/2)}{\sec(x/2)} \cdot \frac{1}{\sec(x/2)} = \frac{2\tan(x/2)}{\sec^2(x/2)} = \frac{2t}{1+t^2}$$

**For cos x:** Using the double-angle formula:
$$\cos x = \cos^2\frac{x}{2} - \sin^2\frac{x}{2} = \frac{1 - \tan^2(x/2)}{\sec^2(x/2)} = \frac{1-t^2}{1+t^2}$$

**For dx:** Since $t = \tan(x/2)$, we have $dt = \frac{1}{2}\sec^2(x/2)\, dx = \frac{1+t^2}{2}\, dx$

$$dx = \frac{2\, dt}{1+t^2}$$

---

## 5.3 Summary of Substitution

```
  ┌──────────────────────────────────────────┐
  │  t = tan(x/2)                            │
  │                                          │
  │  sin x  =  2t / (1 + t²)                │
  │  cos x  =  (1 − t²) / (1 + t²)         │
  │  tan x  =  2t / (1 − t²)               │
  │  dx     =  2 dt / (1 + t²)              │
  └──────────────────────────────────────────┘
```

---

## 5.4 Step-by-Step Procedure

```
  Given: ∫ R(sin x, cos x) dx
     │
     ▼
  Step 1: Set t = tan(x/2)
     │
     ▼
  Step 2: Replace sin x, cos x, and dx using formulas
     │
     ▼
  Step 3: Simplify to get ∫ R(t) dt   (rational in t)
     │
     ▼
  Step 4: Use partial fractions (or other algebra) to integrate
     │
     ▼
  Step 5: Back-substitute x = 2 tan⁻¹(t), i.e., t = tan(x/2)
```

---

## 5.5 Worked Examples

### Example 1: ∫ dx / (1 + sin x)

$$\int \frac{dx}{1 + \sin x}$$

Substitute:

$$= \int \frac{\frac{2\, dt}{1+t^2}}{1 + \frac{2t}{1+t^2}} = \int \frac{\frac{2\, dt}{1+t^2}}{\frac{1+t^2+2t}{1+t^2}} = \int \frac{2\, dt}{(1+t)^2}$$

Let u = 1 + t, du = dt:

$$= 2\int u^{-2}\, du = -\frac{2}{u} + C = -\frac{2}{1+t} + C = \frac{-2}{1+\tan(x/2)} + C$$

---

### Example 2: ∫ dx / (2 + cos x)

$$\int \frac{dx}{2 + \cos x}$$

Substitute:

$$= \int \frac{\frac{2\, dt}{1+t^2}}{2 + \frac{1-t^2}{1+t^2}} = \int \frac{\frac{2\, dt}{1+t^2}}{\frac{2+2t^2+1-t^2}{1+t^2}} = \int \frac{2\, dt}{t^2+3}$$

$$= \frac{2}{\sqrt{3}}\tan^{-1}\left(\frac{t}{\sqrt{3}}\right) + C = \frac{2}{\sqrt{3}}\tan^{-1}\left(\frac{\tan(x/2)}{\sqrt{3}}\right) + C$$

---

### Example 3: ∫ dx / (3 sin x + 4 cos x)

$$\int \frac{dx}{3\sin x + 4\cos x}$$

Substitute:

$$\text{Denominator} = 3 \cdot \frac{2t}{1+t^2} + 4 \cdot \frac{1-t^2}{1+t^2} = \frac{6t + 4 - 4t^2}{1+t^2}$$

$$= \int \frac{2\, dt}{6t + 4 - 4t^2} = \int \frac{2\, dt}{-4t^2 + 6t + 4} = \int \frac{-dt}{2t^2 - 3t - 2}$$

Factor: 2t² − 3t − 2 = (2t + 1)(t − 2)

Use partial fractions:

$$\frac{-1}{(2t+1)(t-2)} = \frac{A}{2t+1} + \frac{B}{t-2}$$

- t = 2: −1 = 5B → B = −1/5
- t = −1/2: −1 = (−5/2)A → A = 2/5

$$= \frac{2}{5}\int \frac{dt}{2t+1} - \frac{1}{5}\int \frac{dt}{t-2}$$

$$= \frac{1}{5}\ln|2t+1| - \frac{1}{5}\ln|t-2| + C = \frac{1}{5}\ln\left|\frac{2\tan(x/2)+1}{\tan(x/2)-2}\right| + C$$

---

### Example 4: ∫ dx / (5 + 4 sin x)

$$= \int \frac{2\, dt/(1+t^2)}{5 + 4 \cdot 2t/(1+t^2)} = \int \frac{2\, dt}{5+5t^2+8t} = \int \frac{2\, dt}{5t^2+8t+5}$$

Complete the square: 5t² + 8t + 5 = 5(t² + 8t/5) + 5 = 5(t + 4/5)² + 5 − 16/5 = 5(t + 4/5)² + 9/5

$$= \int \frac{2\, dt}{5(t+4/5)^2 + 9/5} = \frac{2}{5}\int \frac{dt}{(t+4/5)^2 + (3/5)^2}$$

$$= \frac{2}{5} \cdot \frac{5}{3}\tan^{-1}\left(\frac{t+4/5}{3/5}\right) + C = \frac{2}{3}\tan^{-1}\left(\frac{5\tan(x/2)+4}{3}\right) + C$$

---

## 5.6 When to Use vs When Not to Use

| Situation | Recommendation |
|---|---|
| ∫ R(sin x, cos x) dx — no simpler method | ✅ Use Weierstrass |
| ∫ dx / (a + b sin x) | ✅ Use Weierstrass |
| ∫ dx / (a + b cos x) | ✅ Use Weierstrass |
| ∫ sinⁿx cosᵐx dx | ❌ Use power-reduction or substitution |
| ∫ sin(mx) cos(nx) dx | ❌ Use product-to-sum identities |
| ∫ secⁿx dx | ❌ Use reduction formula or parts |
| Simple trig integrals | ❌ Overkill — use direct formulas |

> **Key Principle:** Weierstrass always **works**, but other methods are often **faster**. Use Weierstrass as a last resort or when the integral involves complicated rational expressions in sin and cos.

---

## 5.7 Alternative: Half-Angle Identities

Sometimes, instead of the full Weierstrass substitution, using half-angle identities directly is faster:

$$\sin^2\frac{x}{2} = \frac{1-\cos x}{2}, \quad \cos^2\frac{x}{2} = \frac{1+\cos x}{2}$$

$$\tan\frac{x}{2} = \frac{\sin x}{1+\cos x} = \frac{1-\cos x}{\sin x}$$

---

## Summary Table

| Concept | Key Point |
|---|---|
| Substitution | t = tan(x/2) |
| sin x | 2t/(1+t²) |
| cos x | (1−t²)/(1+t²) |
| dx | 2dt/(1+t²) |
| Converts to | Rational function of t → use partial fractions |
| Universal | Works for any R(sin x, cos x) |
| Trade-off | Always works, but may produce longer computations |
| Best for | ∫ dx/(a + b sin x + c cos x) type integrals |

---

## Quick Revision Questions

1. State all the Weierstrass substitution formulas for sin x, cos x, tan x, and dx in terms of t = tan(x/2).

2. Evaluate ∫ dx/(1 + cos x) using the Weierstrass substitution.

3. Why is the Weierstrass substitution called a "universal" trigonometric substitution?

4. Evaluate ∫ dx/(3 + 5 cos x) using t = tan(x/2).

5. For which types of integrals is Weierstrass substitution NOT recommended? Give one example and a better technique.

6. Derive the formula cos x = (1 − t²)/(1 + t²) from the double-angle formula.

---

[← Previous: Trigonometric Substitutions](04-trigonometric-substitutions.md) | [Back to README](../README.md) | [Next: Integrals Involving Square Roots →](../03-Special-Integrals/01-integrals-involving-square-roots.md)
