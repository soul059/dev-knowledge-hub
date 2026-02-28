# Chapter 3: Basic Integration Formulas

[← Previous: Indefinite Integrals](02-indefinite-integrals.md) | [Back to README](../README.md) | [Next: Properties of Integrals →](04-properties-of-integrals.md)

---

## 3.1 Overview

This chapter catalogues the **fundamental integration formulas** that every student must memorize. These are obtained by reversing standard derivative formulas and form the building blocks for all integration techniques.

---

## 3.2 Power Rule

$$\int x^n\, dx = \frac{x^{n+1}}{n+1} + C, \quad n \neq -1$$

**Special case (n = −1):**

$$\int x^{-1}\, dx = \int \frac{1}{x}\, dx = \ln|x| + C$$

### Power Rule — Visual Map

```
  n < -1          n = -1          n > -1
  ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ x⁻² → ? │    │ 1/x → ? │    │ x² → ?  │
  │ = x⁻¹/(-1)│  │ = ln|x| │    │ = x³/3  │
  │ = -1/x  │    │ + C     │    │ + C     │
  │ + C     │    └─────────┘    └─────────┘
  └─────────┘
  
  The formula xⁿ⁺¹/(n+1) works for ALL n except n = −1
```

---

## 3.3 Exponential Functions

$$\int e^x\, dx = e^x + C$$

$$\int e^{ax}\, dx = \frac{e^{ax}}{a} + C$$

$$\int a^x\, dx = \frac{a^x}{\ln a} + C, \quad a > 0,\ a \neq 1$$

---

## 3.4 Trigonometric Functions

### Basic Trigonometric Integrals

| Integral | Result |
|---|---|
| ∫ sin x dx | −cos x + C |
| ∫ cos x dx | sin x + C |
| ∫ tan x dx | −ln\|cos x\| + C = ln\|sec x\| + C |
| ∫ cot x dx | ln\|sin x\| + C |
| ∫ sec x dx | ln\|sec x + tan x\| + C |
| ∫ csc x dx | ln\|csc x − cot x\| + C |

### Squared Trigonometric Integrals

| Integral | Result |
|---|---|
| ∫ sec²x dx | tan x + C |
| ∫ csc²x dx | −cot x + C |
| ∫ sec x tan x dx | sec x + C |
| ∫ csc x cot x dx | −csc x + C |

### Derivation Map — How to Remember

```
  Derivative direction (→)         Integral direction (←)
  
  sin x  ──d/dx──►  cos x         cos x  ──∫──►  sin x
  
 −cos x  ──d/dx──►  sin x         sin x  ──∫──► −cos x
  
  tan x  ──d/dx──►  sec²x         sec²x  ──∫──►  tan x
  
 −cot x  ──d/dx──►  csc²x         csc²x  ──∫──► −cot x
  
  sec x  ──d/dx──►  sec x tan x   sec x tan x ──∫──► sec x
  
 −csc x  ──d/dx──►  csc x cot x   csc x cot x ──∫──► −csc x
```

---

## 3.5 Inverse Trigonometric Results

These arise from recognizing derivatives of inverse trig functions:

$$\int \frac{dx}{\sqrt{1-x^2}} = \sin^{-1}x + C$$

$$\int \frac{dx}{1+x^2} = \tan^{-1}x + C$$

$$\int \frac{dx}{|x|\sqrt{x^2-1}} = \sec^{-1}|x| + C$$

### Generalized Forms

$$\int \frac{dx}{\sqrt{a^2-x^2}} = \sin^{-1}\left(\frac{x}{a}\right) + C$$

$$\int \frac{dx}{a^2+x^2} = \frac{1}{a}\tan^{-1}\left(\frac{x}{a}\right) + C$$

$$\int \frac{dx}{x^2-a^2} = \frac{1}{2a}\ln\left|\frac{x-a}{x+a}\right| + C$$

$$\int \frac{dx}{a^2-x^2} = \frac{1}{2a}\ln\left|\frac{a+x}{a-x}\right| + C$$

---

## 3.6 Logarithmic Integrals

$$\int \ln x\, dx = x\ln x - x + C \quad \text{(derived via integration by parts)}$$

$$\int \log_a x\, dx = \frac{x\ln x - x}{\ln a} + C$$

---

## 3.7 Hyperbolic Functions

| Integral | Result |
|---|---|
| ∫ sinh x dx | cosh x + C |
| ∫ cosh x dx | sinh x + C |
| ∫ sech²x dx | tanh x + C |
| ∫ csch²x dx | −coth x + C |
| ∫ sech x tanh x dx | −sech x + C |
| ∫ csch x coth x dx | −csch x + C |

---

## 3.8 Master Formula Table

### Complete Reference of Standard Integrals

| # | Integral | Result |
|---|---|---|
| 1 | ∫ xⁿ dx (n ≠ −1) | xⁿ⁺¹/(n+1) + C |
| 2 | ∫ 1/x dx | ln\|x\| + C |
| 3 | ∫ eˣ dx | eˣ + C |
| 4 | ∫ aˣ dx | aˣ/ln a + C |
| 5 | ∫ sin x dx | −cos x + C |
| 6 | ∫ cos x dx | sin x + C |
| 7 | ∫ tan x dx | ln\|sec x\| + C |
| 8 | ∫ cot x dx | ln\|sin x\| + C |
| 9 | ∫ sec x dx | ln\|sec x + tan x\| + C |
| 10 | ∫ csc x dx | ln\|csc x − cot x\| + C |
| 11 | ∫ sec²x dx | tan x + C |
| 12 | ∫ csc²x dx | −cot x + C |
| 13 | ∫ sec x tan x dx | sec x + C |
| 14 | ∫ csc x cot x dx | −csc x + C |
| 15 | ∫ dx/√(a²−x²) | sin⁻¹(x/a) + C |
| 16 | ∫ dx/(a²+x²) | (1/a)tan⁻¹(x/a) + C |
| 17 | ∫ dx/(x²−a²) | (1/2a)ln\|(x−a)/(x+a)\| + C |
| 18 | ∫ dx/(a²−x²) | (1/2a)ln\|(a+x)/(a−x)\| + C |
| 19 | ∫ dx/√(x²+a²) | ln\|x + √(x²+a²)\| + C |
| 20 | ∫ dx/√(x²−a²) | ln\|x + √(x²−a²)\| + C |

---

## 3.9 Worked Examples

### Example 1: Mixed Terms

$$\int \left(3x^4 - \frac{2}{x} + 5e^x\right) dx$$

$$= 3 \cdot \frac{x^5}{5} - 2\ln|x| + 5e^x + C = \frac{3x^5}{5} - 2\ln|x| + 5e^x + C$$

---

### Example 2: Rewriting Before Integrating

$$\int \frac{x^3 + 2x - 1}{x^2}\, dx = \int \left(x + 2x^{-1} - x^{-2}\right) dx$$

$$= \frac{x^2}{2} + 2\ln|x| + \frac{1}{x} + C$$

---

### Example 3: Trigonometric Identity

$$\int \sin^2 x\, dx$$

Use the identity: sin²x = (1 − cos 2x)/2

$$= \int \frac{1 - \cos 2x}{2}\, dx = \frac{1}{2}\left(x - \frac{\sin 2x}{2}\right) + C = \frac{x}{2} - \frac{\sin 2x}{4} + C$$

---

### Example 4: Completing the Square

$$\int \frac{dx}{x^2 + 4x + 13}$$

Complete the square: x² + 4x + 13 = (x+2)² + 9 = (x+2)² + 3²

$$= \int \frac{dx}{(x+2)^2 + 3^2} = \frac{1}{3}\tan^{-1}\left(\frac{x+2}{3}\right) + C$$

This uses formula #16 with a = 3 and x replaced by (x+2).

---

### Example 5: Algebraic Manipulation

$$\int \frac{1+\cos x}{\sin^2 x}\, dx = \int \left(\csc^2 x + \frac{\cos x}{\sin^2 x}\right) dx$$

$$= \int \csc^2 x\, dx + \int \csc x \cot x\, dx = -\cot x - \csc x + C$$

---

## 3.10 Strategy: When to Use Which Formula

```
  Given ∫ f(x) dx
     │
     ├─── Is f(x) a power of x?  ──► Use Power Rule (#1 or #2)
     │
     ├─── Is f(x) exponential?   ──► Use #3 or #4
     │
     ├─── Is f(x) a basic trig?  ──► Use #5–#14
     │
     ├─── Is f(x) of the form     ──► Use Inverse Trig (#15–#16)
     │     1/√(a²−x²) or 1/(a²+x²)?
     │
     ├─── Is denominator a quadratic? ──► Complete the square,
     │                                    then use #15–#20
     │
     └─── None of the above?     ──► Need techniques from Unit 2
                                      (substitution, parts, etc.)
```

---

## Summary Table

| Category | Key Formulas |
|---|---|
| Power | xⁿ → xⁿ⁺¹/(n+1); 1/x → ln\|x\| |
| Exponential | eˣ → eˣ; aˣ → aˣ/ln a |
| Trig (basic) | sin → −cos; cos → sin |
| Trig (squared) | sec² → tan; csc² → −cot |
| Inverse Trig | 1/√(a²−x²) → sin⁻¹(x/a); 1/(a²+x²) → (1/a)tan⁻¹(x/a) |
| Strategy | Recognize form → apply matching formula → verify |

---

## Quick Revision Questions

1. Write from memory the integrals of sin x, cos x, sec²x, and csc²x.

2. What is ∫ tan x dx? Derive it by writing tan x = sin x / cos x and substituting.

3. Evaluate ∫ dx/(9 + x²) using the standard formula for ∫ dx/(a² + x²).

4. Evaluate ∫ (x² + 1/x²) dx.

5. Rewrite and evaluate: ∫ (√x + 1)² / x dx.

6. Which standard formula would you use for ∫ dx / √(16 − x²)? State the result.

---

[← Previous: Indefinite Integrals](02-indefinite-integrals.md) | [Back to README](../README.md) | [Next: Properties of Integrals →](04-properties-of-integrals.md)
