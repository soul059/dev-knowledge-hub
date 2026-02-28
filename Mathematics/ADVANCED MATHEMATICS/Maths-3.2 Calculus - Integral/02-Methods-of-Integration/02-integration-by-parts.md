# Chapter 2: Integration by Parts (ILATE Rule)

[← Previous: Substitution Method](01-substitution-method.md) | [Back to README](../README.md) | [Next: Partial Fractions →](03-partial-fractions.md)

---

## 2.1 Overview

**Integration by parts** is the integration counterpart of the **product rule** for differentiation. It is used when the integrand is a **product of two functions** that cannot be handled by simple substitution.

---

## 2.2 Derivation from the Product Rule

The product rule states:

$$\frac{d}{dx}[u \cdot v] = u \cdot \frac{dv}{dx} + v \cdot \frac{du}{dx}$$

Rearranging:

$$u \cdot \frac{dv}{dx} = \frac{d}{dx}[u \cdot v] - v \cdot \frac{du}{dx}$$

Integrating both sides:

$$\int u\, dv = uv - \int v\, du$$

This is the **integration by parts formula**.

---

## 2.3 The Formula

$$\boxed{\int u\, dv = uv - \int v\, du}$$

### What Each Part Means

```
  ∫ u · dv  =  u · v  −  ∫ v · du
  │   │        │   │      │   │
  │   │        │   │      │   └── derivative of u (times dx)
  │   │        │   │      └────── antiderivative of dv
  │   │        │   └───────────── antiderivative of dv
  │   │        └───────────────── first function (unchanged)
  │   └────────────────────────── second function (to integrate)
  └────────────────────────────── first function (to differentiate)
```

---

## 2.4 The ILATE Rule — Choosing u

The key challenge is: **which function should be u and which should be dv?**

The **ILATE** (or **LIATE**) rule provides a priority order for choosing **u**:

| Priority | Category | Examples |
|---|---|---|
| **I** | **I**nverse trigonometric | sin⁻¹x, tan⁻¹x, cos⁻¹x |
| **L** | **L**ogarithmic | ln x, log x |
| **A** | **A**lgebraic | x, x², x³, polynomials |
| **T** | **T**rigonometric | sin x, cos x, tan x |
| **E** | **E**xponential | eˣ, 2ˣ, aˣ |

> **Rule:** Choose **u** as the function that appears **earliest** in ILATE. The remaining function becomes part of **dv**.

### Decision Flowchart

```
  Given: ∫ f(x) · g(x) dx
     │
     ▼
  Classify f(x) and g(x) by ILATE
     │
     ▼
  The function HIGHER in ILATE → u
  The function LOWER in ILATE  → dv
     │
     ▼
  Compute du = u' dx  and  v = ∫ dv
     │
     ▼
  Apply: uv − ∫ v du
     │
     ▼
  Is ∫ v du simpler? ──Yes──► Evaluate
     │                          
     No ──► Apply parts again or use another technique
```

---

## 2.5 Worked Examples

### Example 1: Algebraic × Exponential → u = Algebraic

$$\int x e^x\, dx$$

| Component | Choice | Reason |
|---|---|---|
| u | x | Algebraic (A) — higher in ILATE |
| dv | eˣ dx | Exponential (E) — lower in ILATE |
| du | dx | |
| v | eˣ | |

$$= xe^x - \int e^x\, dx = xe^x - e^x + C = e^x(x - 1) + C$$

**Verify:** d/dx[eˣ(x−1)] = eˣ(x−1) + eˣ = eˣ·x − eˣ + eˣ = xeˣ ✓

---

### Example 2: Algebraic × Trigonometric → u = Algebraic

$$\int x \sin x\, dx$$

| u = x | dv = sin x dx |
|---|---|
| du = dx | v = −cos x |

$$= x(-\cos x) - \int (-\cos x)\, dx = -x\cos x + \sin x + C$$

---

### Example 3: Logarithmic × Algebraic → u = Logarithmic

$$\int x^2 \ln x\, dx$$

| u = ln x | dv = x² dx |
|---|---|
| du = dx/x | v = x³/3 |

$$= \frac{x^3}{3}\ln x - \int \frac{x^3}{3} \cdot \frac{dx}{x} = \frac{x^3}{3}\ln x - \frac{1}{3}\int x^2\, dx$$

$$= \frac{x^3}{3}\ln x - \frac{x^3}{9} + C = \frac{x^3}{9}(3\ln x - 1) + C$$

---

### Example 4: Inverse Trig × Algebraic → u = Inverse Trig

$$\int \tan^{-1}x\, dx = \int 1 \cdot \tan^{-1}x\, dx$$

| u = tan⁻¹x | dv = dx |
|---|---|
| du = dx/(1+x²) | v = x |

$$= x\tan^{-1}x - \int \frac{x}{1+x^2}\, dx$$

For the remaining integral, let t = 1+x², dt = 2x dx:

$$= x\tan^{-1}x - \frac{1}{2}\ln(1+x^2) + C$$

---

### Example 5: Repeated Application (Tabular Method)

$$\int x^2 e^x\, dx$$

Applying parts twice, we can use a **tabular method**:

```
  Derivatives (u)    Sign    Integrals (dv)
  ─────────────────────────────────────────
       x²             +         eˣ
       2x             −         eˣ
       2              +         eˣ
       0                        eˣ
```

**Read off diagonally with alternating signs:**

$$= x^2 e^x - 2x e^x + 2e^x + C = e^x(x^2 - 2x + 2) + C$$

### How the Tabular Method Works

```
  u-column:  x²  →  2x  →  2  →  0
               ╲      ╲      ╲
  dv-column:    eˣ     eˣ     eˣ
  Signs:       (+)    (−)    (+)

  Result: (+)x²·eˣ + (−)2x·eˣ + (+)2·eˣ + C
```

---

### Example 6: Cyclic — ∫ eˣ sin x dx

$$I = \int e^x \sin x\, dx$$

**First application:**

| u = sin x | dv = eˣ dx |
|---|---|
| du = cos x dx | v = eˣ |

$$I = e^x \sin x - \int e^x \cos x\, dx$$

**Second application** on ∫ eˣ cos x dx:

| u = cos x | dv = eˣ dx |
|---|---|
| du = −sin x dx | v = eˣ |

$$I = e^x \sin x - \left[e^x \cos x - \int e^x(-\sin x)\, dx\right]$$

$$I = e^x \sin x - e^x \cos x - \int e^x \sin x\, dx$$

$$I = e^x \sin x - e^x \cos x - I$$

**The original integral I reappears!** Solve for I:

$$2I = e^x(\sin x - \cos x)$$

$$I = \frac{e^x(\sin x - \cos x)}{2} + C$$

### Cyclic Pattern — Visual

```
  I = ∫eˣ sin x dx
     │
     ├── Parts → eˣ sin x − ∫eˣ cos x dx
     │                           │
     │                           ├── Parts → eˣ cos x + ∫eˣ sin x dx
     │                           │                         │
     │                           │                         └── This is I again!
     │                           │
     └── I = eˣ sin x − eˣ cos x − I
         2I = eˣ(sin x − cos x)
         I = eˣ(sin x − cos x)/2 + C
```

---

## 2.6 Special Cases

### Integrating ln x

$$\int \ln x\, dx$$

**Trick:** Write it as ∫ 1 · ln x dx and choose u = ln x, dv = dx.

| u = ln x | dv = dx |
|---|---|
| du = dx/x | v = x |

$$= x \ln x - \int x \cdot \frac{1}{x}\, dx = x\ln x - x + C$$

---

### General Rule for ∫ xⁿ eᵃˣ dx

Apply parts n times (or use the tabular method). Each application reduces the power of x by 1.

---

## 2.7 When NOT to Use Integration by Parts

| Situation | Better Technique |
|---|---|
| ∫ f(g(x))g'(x) dx | Substitution |
| ∫ P(x)/Q(x) dx (rational) | Partial fractions |
| ∫ involving √(a²−x²) | Trigonometric substitution |
| Simple power/trig/exp | Direct formula |

---

## Summary Table

| Concept | Key Point |
|---|---|
| Formula | ∫ u dv = uv − ∫ v du |
| ILATE priority | Inverse trig > Log > Algebraic > Trig > Exponential |
| Choose u | Function easiest to differentiate (goes to 0 eventually) |
| Choose dv | Function easy to integrate |
| Tabular method | Efficient for ∫ xⁿ · eᵃˣ dx or ∫ xⁿ · trig dx |
| Cyclic integrals | When I reappears, solve algebraically for I |
| Special case | ∫ ln x dx: use u = ln x, dv = dx |

---

## Quick Revision Questions

1. Derive the integration by parts formula from the product rule.

2. Use ILATE to determine u and dv for ∫ x² cos x dx, then evaluate.

3. Evaluate ∫ ln x dx using integration by parts.

4. Evaluate ∫ eˣ cos x dx and explain the cyclic technique.

5. Use the tabular method to evaluate ∫ x³ eˣ dx.

6. Why does ILATE suggest choosing u = ln x over u = x² in ∫ x² ln x dx?

---

[← Previous: Substitution Method](01-substitution-method.md) | [Back to README](../README.md) | [Next: Partial Fractions →](03-partial-fractions.md)
