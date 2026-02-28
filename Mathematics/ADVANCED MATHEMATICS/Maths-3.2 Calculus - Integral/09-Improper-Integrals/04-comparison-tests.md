# Chapter 4: Gamma and Beta Functions & Special Results

[← Previous: Convergence Tests](03-convergence-tests.md) | [Back to README](../README.md) | [Next: Double Integrals →](../10-Multiple-Integrals/01-double-integrals.md)

---

## 4.1 Overview

The **Gamma** and **Beta functions** are the most important improper integrals in advanced mathematics. They generalize the factorial, connect to trigonometry, and appear ubiquitously in physics, probability, and engineering.

---

## 4.2 The Gamma Function

> **Definition:**
>
> $$\Gamma(n) = \int_0^{\infty} x^{n-1} e^{-x}\, dx \qquad (n > 0)$$

### Key Properties

| Property | Formula |
|---|---|
| Recursion | Γ(n+1) = n · Γ(n) |
| At positive integers | Γ(n) = (n−1)! |
| Γ(1) | 1 |
| Γ(1/2) | √π |
| Half-integer formula | Γ(n+½) = [(2n)! / (4ⁿ · n!)] √π |

### Proof of Recursion: Γ(n+1) = nΓ(n)

Use integration by parts with u = xⁿ, dv = e⁻ˣ dx:

$$\Gamma(n+1) = \int_0^{\infty} x^n e^{-x}\, dx = [-x^n e^{-x}]_0^{\infty} + n\int_0^{\infty} x^{n-1} e^{-x}\, dx = n\Gamma(n)$$

### Connection to Factorial

$$\Gamma(1) = 1, \quad \Gamma(2) = 1\cdot\Gamma(1) = 1, \quad \Gamma(3) = 2\cdot\Gamma(2) = 2!, \quad \Gamma(n+1) = n!$$

```
  Γ(n) extends the factorial to ALL positive reals:
  
  n!= Γ(n+1)
  
    │         ╱
    │        ╱
  6 ●───────╱   ← Γ(4) = 3! = 6
    │      ╱
  2 ●─────╱     ← Γ(3) = 2! = 2
  1 ●────╱      ← Γ(2) = 1! = 1
    ●───╱       ← Γ(1) = 0! = 1
  √π────        ← Γ(1/2) = √π ≈ 1.77
    ├──┼──┼──┼── n
    0  1  2  3
```

---

## 4.3 The Beta Function

> **Definition:**
>
> $$B(m, n) = \int_0^1 x^{m-1}(1-x)^{n-1}\, dx \qquad (m, n > 0)$$

### Key Properties

| Property | Formula |
|---|---|
| Symmetry | B(m,n) = B(n,m) |
| Gamma connection | B(m,n) = Γ(m)Γ(n) / Γ(m+n) |
| B(1/2, 1/2) | π |
| Trigonometric form | B(m,n) = 2∫₀^{π/2} sin²ᵐ⁻¹θ cos²ⁿ⁻¹θ dθ |

### The Crucial Identity

$$\boxed{B(m, n) = \frac{\Gamma(m)\,\Gamma(n)}{\Gamma(m+n)}}$$

This connects the Beta function to the Gamma function and is one of the most useful identities in analysis.

---

## 4.4 Trigonometric Form of Beta Function

Substituting x = sin²θ:

$$B(m, n) = 2\int_0^{\pi/2} \sin^{2m-1}\theta\, \cos^{2n-1}\theta\, d\theta$$

### Connection to Wallis Integrals

$$\int_0^{\pi/2} \sin^p\theta\, \cos^q\theta\, d\theta = \frac{1}{2}B\left(\frac{p+1}{2}, \frac{q+1}{2}\right) = \frac{\Gamma\left(\frac{p+1}{2}\right)\Gamma\left(\frac{q+1}{2}\right)}{2\,\Gamma\left(\frac{p+q+2}{2}\right)}$$

---

## 4.5 Worked Examples

### Example 1: Evaluating Γ(5/2)

$$\Gamma(5/2) = \frac{3}{2}\Gamma(3/2) = \frac{3}{2}\cdot\frac{1}{2}\Gamma(1/2) = \frac{3}{4}\sqrt{\pi}$$

---

### Example 2: Beta Function Evaluation

$$B(3, 2) = \int_0^1 x^2(1-x)\, dx = \int_0^1 (x^2 - x^3)\, dx = \frac{1}{3} - \frac{1}{4} = \frac{1}{12}$$

**Verify:** B(3,2) = Γ(3)Γ(2)/Γ(5) = (2!)(1!)/(4!) = 2/24 = 1/12 ✓

---

### Example 3: Proving Γ(1/2) = √π

$$[\Gamma(1/2)]^2 = \left(\int_0^{\infty} x^{-1/2}e^{-x}\, dx\right)^2$$

Let x = u²: $\Gamma(1/2) = \int_0^{\infty} \frac{e^{-u^2}}{u} \cdot 2u\, du = 2\int_0^{\infty} e^{-u^2}\, du = \sqrt{\pi}$

(Using the Gaussian integral result.)

$$\boxed{\Gamma(1/2) = \sqrt{\pi}}$$

---

### Example 4: Wallis-Type Integral

$$\int_0^{\pi/2} \sin^4\theta\, \cos^2\theta\, d\theta = \frac{1}{2}B\left(\frac{5}{2}, \frac{3}{2}\right) = \frac{\Gamma(5/2)\,\Gamma(3/2)}{2\,\Gamma(4)}$$

$$= \frac{(3\sqrt{\pi}/4)(\sqrt{\pi}/2)}{2 \cdot 6} = \frac{3\pi/8}{12} = \frac{\pi}{32}$$

---

### Example 5: Non-Standard Integral

$$\int_0^1 \sqrt{x}\sqrt{1-x}\, dx = \int_0^1 x^{1/2}(1-x)^{1/2}\, dx = B(3/2, 3/2)$$

$$= \frac{[\Gamma(3/2)]^2}{\Gamma(3)} = \frac{(\sqrt{\pi}/2)^2}{2!} = \frac{\pi/4}{2} = \frac{\pi}{8}$$

---

## 4.6 Important Special Results

| Integral | Value | Via |
|---|---|---|
| ∫₀^∞ e⁻ˣ² dx | √π/2 | Gaussian |
| ∫₀^∞ xⁿe⁻ˣ dx | n! | Γ(n+1) |
| ∫₀^∞ x^{n-1}e⁻ᵃˣ dx | Γ(n)/aⁿ | Substitution |
| ∫₀¹ xᵐ⁻¹(1−x)ⁿ⁻¹ dx | B(m,n) | Definition |
| ∫₀^{π/2} sinⁿθ dθ | Wallis = ½B((n+1)/2, 1/2) | Trig Beta |
| ∫₀^∞ xᵖ⁻¹/(1+x) dx | π/sin(pπ), 0<p<1 | Euler reflection |

### Euler's Reflection Formula

$$\Gamma(p)\,\Gamma(1-p) = \frac{\pi}{\sin(p\pi)} \qquad (0 < p < 1)$$

Setting p = 1/2: Γ(1/2)² = π/sin(π/2) = π, confirming Γ(1/2) = √π.

---

## 4.7 Decision Guide

```
  WHICH SPECIAL FUNCTION TO USE?
  ═══════════════════════════════
  
  ∫₀^∞ x^{n-1} e⁻ˣ dx     →  Gamma function Γ(n)
  
  ∫₀¹ x^{m-1}(1−x)^{n-1} dx  →  Beta function B(m,n)
  
  ∫₀^{π/2} sinᵖθ cosᵠθ dθ  →  Trig Beta: ½B((p+1)/2, (q+1)/2)
  
  ∫₀^∞ x^{n-1} e⁻ᵃˣ dx    →  Γ(n)/aⁿ (substitute u = ax)
  
  Product Γ(m)Γ(n)?        →  Convert to B(m,n)·Γ(m+n)
```

---

## Summary Table

| Function | Definition | Key Formula |
|---|---|---|
| Gamma | Γ(n) = ∫₀^∞ xⁿ⁻¹e⁻ˣ dx | Γ(n+1) = n·Γ(n) = n! |
| Beta | B(m,n) = ∫₀¹ xᵐ⁻¹(1−x)ⁿ⁻¹ dx | B(m,n) = Γ(m)Γ(n)/Γ(m+n) |
| Γ(1/2) | √π | From Gaussian integral |
| Trig form | 2∫₀^{π/2} sin²ᵐ⁻¹θ cos²ⁿ⁻¹θ dθ | = B(m,n) |
| Reflection | Γ(p)Γ(1−p) | = π/sin(pπ) |

---

## Quick Revision Questions

1. Evaluate Γ(7/2) using the recursion formula.

2. Show that B(m,n) = B(n,m) from the definition.

3. Evaluate ∫₀¹ x³(1−x)⁴ dx using the Beta function.

4. Express ∫₀^{π/2} sin⁶θ dθ in terms of the Beta function and evaluate.

5. Prove the Euler reflection formula for p = 1/2 and verify Γ(1/2) = √π.

6. Evaluate ∫₀^∞ x⁴e⁻²ˣ dx using the Gamma function.

---

[← Previous: Convergence Tests](03-convergence-tests.md) | [Back to README](../README.md) | [Next: Double Integrals →](../10-Multiple-Integrals/01-double-integrals.md)
