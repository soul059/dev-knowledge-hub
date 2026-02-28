# Chapter 1: Symmetry Properties

[← Previous: Evaluation Techniques](../04-Definite-Integrals/04-evaluation-techniques.md) | [Back to README](../README.md) | [Next: King Property →](02-king-property.md)

---

## 1.1 Overview

Symmetry properties of definite integrals exploit the **even** or **odd** nature of the integrand or the symmetry of the integration interval to simplify calculations dramatically — often reducing an integral by half or making it vanish entirely.

---

## 1.2 Even and Odd Functions — Review

| Type | Definition | Graphical Symmetry | Examples |
|---|---|---|---|
| **Even** | f(−x) = f(x) | Symmetric about y-axis | x², cos x, \|x\|, 1 |
| **Odd** | f(−x) = −f(x) | Symmetric about origin | x³, sin x, tan x, x |

### Products of Even/Odd Functions

| f(x) | g(x) | f(x)·g(x) |
|---|---|---|
| Even | Even | **Even** |
| Odd | Odd | **Even** |
| Even | Odd | **Odd** |
| Odd | Even | **Odd** |

> **Memory aid:** Same parity → Even; Different parity → Odd. (Like multiplication of signs!)

---

## 1.3 The Symmetry Theorem

> **Theorem:** If f is continuous on [−a, a], then:
>
> **(a)** If f is **even**: $\int_{-a}^{a} f(x)\, dx = 2\int_0^a f(x)\, dx$
>
> **(b)** If f is **odd**: $\int_{-a}^{a} f(x)\, dx = 0$

### Visual Proof

```
  EVEN function f(−x) = f(x):

  y
  │     ╱╲        ╱╲
  │    ╱  ╲      ╱  ╲
  │   ╱    ╲    ╱    ╲      Both halves are EQUAL
  │  ╱  A₁  ╲  ╱  A₂  ╲    A₁ = A₂
  │ ╱        ╲╱        ╲   
  └─┬────────┬────────┬── x
   −a        0        a
   
  Total = A₁ + A₂ = 2A₂ = 2∫₀ᵃ f(x) dx
```

```
  ODD function f(−x) = −f(x):

  y
  │         ╱╲
  │        ╱  ╲        A₂ (positive)
  │       ╱    ╲
  ├──────╱──────╲───── x
  │     ╱   −a   0   a
  │    ╱
  │   ╱    A₁ (negative)
  │  ╱
  
  A₁ = −A₂, so Total = A₁ + A₂ = 0
```

### Proof (for odd functions)

$$\int_{-a}^{a} f(x)\, dx = \int_{-a}^{0} f(x)\, dx + \int_0^a f(x)\, dx$$

In the first integral, let x = −t, dx = −dt:

$$\int_{-a}^{0} f(x)\, dx = \int_{a}^{0} f(-t)(-dt) = \int_0^a f(-t)\, dt = -\int_0^a f(t)\, dt$$

(since f is odd: f(−t) = −f(t))

$$\therefore \int_{-a}^{a} f(x)\, dx = -\int_0^a f(t)\, dt + \int_0^a f(x)\, dx = 0$$

---

## 1.4 Worked Examples

### Example 1: Quick Evaluation

$$\int_{-3}^{3} x^5 \sin x\, dx$$

- x⁵ is **odd**
- sin x is **odd**
- Product: odd × odd = **even**

$$= 2\int_0^3 x^5 \sin x\, dx$$

(This doesn't vanish — it's even! You'd still need to evaluate, but it's half the work.)

---

### Example 2: Vanishing Integral

$$\int_{-\pi}^{\pi} x \cos x\, dx$$

- x is **odd**
- cos x is **even**
- Product: odd × even = **odd**

$$= 0$$

---

### Example 3: Simplification

$$\int_{-2}^{2} (x^4 + x^3 + x^2 + x)\, dx$$

Split into even and odd parts:
- Even terms: x⁴ + x². Odd terms: x³ + x.

$$= \int_{-2}^{2}(x^4 + x^2)\, dx + \underbrace{\int_{-2}^{2}(x^3+x)\, dx}_{= 0}$$

$$= 2\int_0^2 (x^4 + x^2)\, dx = 2\left[\frac{x^5}{5} + \frac{x^3}{3}\right]_0^2 = 2\left(\frac{32}{5} + \frac{8}{3}\right) = 2 \cdot \frac{136}{15} = \frac{272}{15}$$

---

### Example 4: Symmetry in Trigonometric Integrals

$$\int_{-\pi/2}^{\pi/2} \cos^3 x\, dx$$

cos³x = (cos x)³ — since cos x is even, cos³x is **even**.

$$= 2\int_0^{\pi/2} \cos^3 x\, dx = 2 \cdot \frac{2}{3} = \frac{4}{3}$$

(Using Wallis/reduction formula for ∫₀^{π/2} cos³x dx = 2/3.)

---

## 1.5 General Decomposition: Any Function f(x)

Any function can be decomposed into even and odd parts:

$$f(x) = \underbrace{\frac{f(x) + f(-x)}{2}}_{\text{even part}} + \underbrace{\frac{f(x) - f(-x)}{2}}_{\text{odd part}}$$

When integrating over [−a, a]:
- The odd part contributes **zero**.
- Only the even part contributes.

---

## 1.6 Extended Symmetry Property

For integration from a to b where a + b = 2c (symmetric about x = c):

$$\int_a^b f(x)\, dx \stackrel{?}{=} \text{Use substitution } x = 2c - t$$

This connects to the **King Property** (next chapter), which generalizes symmetry to non-zero-centered intervals.

---

## Summary Table

| Concept | Key Point |
|---|---|
| Even function | f(−x) = f(x); symmetric about y-axis |
| Odd function | f(−x) = −f(x); symmetric about origin |
| Even × Even | Even |
| Odd × Odd | Even |
| Even × Odd | Odd |
| ∫₋ₐᵃ (even) dx | = 2∫₀ᵃ f dx |
| ∫₋ₐᵃ (odd) dx | = 0 |
| Strategy | Decompose integrand into even + odd parts |

---

## Quick Revision Questions

1. Classify each as even, odd, or neither: (a) x⁴ + 1, (b) x³ − sin x, (c) eˣ.

2. Evaluate ∫₋₅⁵ x⁷ dx without computing an antiderivative.

3. Compute ∫₋π^π (sin²x + x sin x) dx using symmetry arguments.

4. Show that ∫₋₁¹ eˣ dx = 2∫₀¹ cosh x dx by decomposing eˣ into even and odd parts.

5. If f is even and ∫₀³ f(x) dx = 7, what is ∫₋₃³ f(x) dx?

6. Explain why ∫₋π^π cos(nx) sin(mx) dx = 0 for any integers m, n.

---

[← Previous: Evaluation Techniques](../04-Definite-Integrals/04-evaluation-techniques.md) | [Back to README](../README.md) | [Next: King Property →](02-king-property.md)
