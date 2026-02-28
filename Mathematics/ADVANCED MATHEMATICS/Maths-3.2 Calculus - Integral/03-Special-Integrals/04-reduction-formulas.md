# Chapter 4: Reduction Formulas

[← Previous: Integrals of Type ∫dx/√(ax²+bx+c)](03-integrals-irrational-quadratic.md) | [Back to README](../README.md) | [Next: Riemann Sums →](../04-Definite-Integrals/01-riemann-sums.md)

---

## 4.1 Overview

A **reduction formula** expresses an integral Iₙ (depending on integer parameter n) in terms of a simpler integral Iₙ₋₁ or Iₙ₋₂, allowing us to "reduce" the power step by step until we reach a known base case. These are derived using **integration by parts** or **trigonometric identities**.

---

## 4.2 How Reduction Formulas Work

```
  Iₙ  ──reduction──►  Iₙ₋₂  ──reduction──►  Iₙ₋₄  ──···──►  I₁ or I₀
  
  We keep applying the formula until we reach:
  • I₀ or I₁ (for formulas that reduce by 1)
  • I₀ or I₁ (for formulas that reduce by 2)
  
  Then evaluate the base case directly.
```

---

## 4.3 Reduction Formula for ∫ sinⁿx dx

### The Formula

$$I_n = \int \sin^n x\, dx = -\frac{\sin^{n-1}x \cos x}{n} + \frac{n-1}{n} I_{n-2}$$

### Derivation

Write sinⁿx = sinⁿ⁻¹x · sin x and apply integration by parts:

| u = sinⁿ⁻¹x | dv = sin x dx |
|---|---|
| du = (n−1)sinⁿ⁻²x cos x dx | v = −cos x |

$$I_n = -\sin^{n-1}x\cos x + (n-1)\int \sin^{n-2}x \cos^2 x\, dx$$

Use cos²x = 1 − sin²x:

$$I_n = -\sin^{n-1}x\cos x + (n-1)\int \sin^{n-2}x\, dx - (n-1)\int \sin^n x\, dx$$

$$I_n = -\sin^{n-1}x\cos x + (n-1)I_{n-2} - (n-1)I_n$$

$$nI_n = -\sin^{n-1}x\cos x + (n-1)I_{n-2}$$

$$\boxed{I_n = -\frac{\sin^{n-1}x\cos x}{n} + \frac{n-1}{n}I_{n-2}}$$

### Base Cases

$$I_0 = \int dx = x + C$$

$$I_1 = \int \sin x\, dx = -\cos x + C$$

### Example: ∫ sin⁴x dx

$$I_4 = -\frac{\sin^3 x \cos x}{4} + \frac{3}{4}I_2$$

$$I_2 = -\frac{\sin x \cos x}{2} + \frac{1}{2}I_0 = -\frac{\sin x \cos x}{2} + \frac{x}{2}$$

$$I_4 = -\frac{\sin^3 x \cos x}{4} + \frac{3}{4}\left(-\frac{\sin x \cos x}{2} + \frac{x}{2}\right)$$

$$= -\frac{\sin^3 x \cos x}{4} - \frac{3\sin x \cos x}{8} + \frac{3x}{8} + C$$

---

## 4.4 Reduction Formula for ∫ cosⁿx dx

$$I_n = \int \cos^n x\, dx = \frac{\cos^{n-1}x \sin x}{n} + \frac{n-1}{n} I_{n-2}$$

**Base cases:** I₀ = x + C, I₁ = sin x + C

> Note the similarity with the sinⁿx formula — only signs differ.

---

## 4.5 Reduction Formula for ∫ tanⁿx dx

$$I_n = \int \tan^n x\, dx = \frac{\tan^{n-1}x}{n-1} - I_{n-2}$$

### Derivation

Write tanⁿx = tanⁿ⁻²x · tan²x = tanⁿ⁻²x · (sec²x − 1):

$$I_n = \int \tan^{n-2}x \sec^2 x\, dx - \int \tan^{n-2}x\, dx$$

$$= \frac{\tan^{n-1}x}{n-1} - I_{n-2}$$

### Base Cases

$$I_0 = x + C, \quad I_1 = \ln|\sec x| + C$$

### Example: ∫ tan⁴x dx

$$I_4 = \frac{\tan^3 x}{3} - I_2 = \frac{\tan^3 x}{3} - \left(\frac{\tan x}{1} - I_0\right) = \frac{\tan^3 x}{3} - \tan x + x + C$$

---

## 4.6 Reduction Formula for ∫ secⁿx dx

$$I_n = \int \sec^n x\, dx = \frac{\sec^{n-2}x \tan x}{n-1} + \frac{n-2}{n-1}I_{n-2}$$

### Base Cases

$$I_0 = x + C, \quad I_1 = \ln|\sec x + \tan x| + C, \quad I_2 = \tan x + C$$

### Example: ∫ sec³x dx

$$I_3 = \frac{\sec x \tan x}{2} + \frac{1}{2}I_1 = \frac{\sec x \tan x}{2} + \frac{1}{2}\ln|\sec x + \tan x| + C$$

This is an important result used frequently.

---

## 4.7 Reduction Formula for ∫ xⁿ eᵃˣ dx

$$I_n = \int x^n e^{ax}\, dx = \frac{x^n e^{ax}}{a} - \frac{n}{a}I_{n-1}$$

### Base Case

$$I_0 = \int e^{ax}\, dx = \frac{e^{ax}}{a} + C$$

---

## 4.8 Wallis' Formulas (Definite Integral Reduction)

For the **definite** integral from 0 to π/2:

$$\int_0^{\pi/2} \sin^n x\, dx = \int_0^{\pi/2} \cos^n x\, dx$$

| n | Value |
|---|---|
| n even: 2m | $\frac{(2m-1)!!}{(2m)!!} \cdot \frac{\pi}{2}$ |
| n odd: 2m+1 | $\frac{(2m)!!}{(2m+1)!!}$ |

Where n!! denotes the **double factorial**:
- (2m)!! = 2·4·6···(2m)
- (2m−1)!! = 1·3·5···(2m−1)

### Wallis' Formula Table

```
  n │ ∫₀^{π/2} sinⁿx dx
  ──┼─────────────────────
  0 │ π/2
  1 │ 1
  2 │ π/4
  3 │ 2/3
  4 │ 3π/16
  5 │ 8/15
  6 │ 5π/32
  7 │ 16/35
```

### Pattern

```
  Even n:  (n-1)/n · (n-3)/(n-2) · ··· · 1/2 · π/2
  
  Odd n:   (n-1)/n · (n-3)/(n-2) · ··· · 2/3 · 1
```

---

## 4.9 Reduction Formulas — Master Reference

| Integral | Reduction Formula |
|---|---|
| ∫ sinⁿx dx | −sinⁿ⁻¹x cos x/n + (n−1)/n · Iₙ₋₂ |
| ∫ cosⁿx dx | cosⁿ⁻¹x sin x/n + (n−1)/n · Iₙ₋₂ |
| ∫ tanⁿx dx | tanⁿ⁻¹x/(n−1) − Iₙ₋₂ |
| ∫ cotⁿx dx | −cotⁿ⁻¹x/(n−1) − Iₙ₋₂ |
| ∫ secⁿx dx | secⁿ⁻²x tan x/(n−1) + (n−2)/(n−1) · Iₙ₋₂ |
| ∫ cscⁿx dx | −cscⁿ⁻²x cot x/(n−1) + (n−2)/(n−1) · Iₙ₋₂ |
| ∫ xⁿ eᵃˣ dx | xⁿ eᵃˣ/a − n/a · Iₙ₋₁ |
| ∫ (ln x)ⁿ dx | x(ln x)ⁿ − n · Iₙ₋₁ |

---

## 4.10 Step-by-Step Application Process

```
  Given: Iₙ = ∫ f(x, n) dx
     │
     ▼
  Step 1: Apply the reduction formula to get Iₙ in terms of Iₙ₋₂ (or Iₙ₋₁)
     │
     ▼
  Step 2: Apply again: express Iₙ₋₂ in terms of Iₙ₋₄
     │
     ▼
  Step 3: Continue until reaching I₀ or I₁ (base case)
     │
     ▼
  Step 4: Evaluate the base case directly
     │
     ▼
  Step 5: Substitute back through the chain
```

---

## Summary Table

| Concept | Key Point |
|---|---|
| Reduction formula | Relates Iₙ to Iₙ₋₁ or Iₙ₋₂ with lower index |
| Derivation method | Integration by parts or trig identities |
| Base cases | I₀ and I₁ are evaluated directly |
| Wallis' formulas | For definite integrals ∫₀^{π/2} sinⁿx dx |
| Even n (Wallis) | Ends with π/2 factor |
| Odd n (Wallis) | Ends with factor 1 (no π) |
| tanⁿ formula | Uses tan²x = sec²x − 1 (no parts needed) |
| secⁿ formula | Important: gives ∫sec³x dx = (sec x tan x + ln\|sec x + tan x\|)/2 |

---

## Quick Revision Questions

1. Derive the reduction formula for ∫ sinⁿx dx using integration by parts.

2. Use the reduction formula to evaluate ∫ cos⁴x dx.

3. Evaluate ∫ tan⁵x dx using the tan reduction formula.

4. Using Wallis' formula, find ∫₀^{π/2} sin⁶x dx.

5. Write the reduction formula for ∫ secⁿx dx and use it to find ∫ sec⁵x dx in terms of known integrals.

6. What are the base cases (I₀ and I₁) for the sinⁿx and tanⁿx reduction formulas?

---

[← Previous: Integrals of Type ∫dx/√(ax²+bx+c)](03-integrals-irrational-quadratic.md) | [Back to README](../README.md) | [Next: Riemann Sums →](../04-Definite-Integrals/01-riemann-sums.md)
