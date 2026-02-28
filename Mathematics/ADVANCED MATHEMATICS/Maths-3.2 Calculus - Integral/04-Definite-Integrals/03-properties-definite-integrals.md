# Chapter 3: Properties of Definite Integrals

[← Previous: Fundamental Theorem](02-fundamental-theorem.md) | [Back to README](../README.md) | [Next: Evaluation Techniques →](04-evaluation-techniques.md)

---

## 3.1 Overview

Definite integrals obey several powerful algebraic and order properties that allow us to manipulate, simplify, and evaluate them efficiently. These properties are fundamental tools used throughout applied mathematics, physics, and engineering.

---

## 3.2 Algebraic Properties

### Property 1: Integral over Zero-Width Interval

$$\int_a^a f(x)\, dx = 0$$

### Property 2: Reversing Limits

$$\int_a^b f(x)\, dx = -\int_b^a f(x)\, dx$$

### Property 3: Constant Multiple

$$\int_a^b k \cdot f(x)\, dx = k \int_a^b f(x)\, dx$$

### Property 4: Sum / Difference

$$\int_a^b [f(x) \pm g(x)]\, dx = \int_a^b f(x)\, dx \pm \int_a^b g(x)\, dx$$

### Property 5: Additivity over Intervals

$$\int_a^b f(x)\, dx = \int_a^c f(x)\, dx + \int_c^b f(x)\, dx$$

for any c (not necessarily between a and b).

```
   f(x)
  │    ╱╲
  │   ╱  ╲
  │  ╱    ╲
  │ ╱  A₁  ╲  A₂
  └─┬───┬───┬── x
    a   c   b
    
  ∫ₐᵇ = ∫ₐᶜ + ∫ᶜᵇ = A₁ + A₂
```

### Property 6: Dummy Variable

$$\int_a^b f(x)\, dx = \int_a^b f(t)\, dt = \int_a^b f(u)\, du$$

The variable of integration is a **dummy variable** — the name doesn't matter.

---

## 3.3 Order Properties

### Property 7: Non-negativity

If f(x) ≥ 0 on [a, b], then:
$$\int_a^b f(x)\, dx \geq 0$$

### Property 8: Monotonicity (Comparison)

If f(x) ≥ g(x) on [a, b], then:
$$\int_a^b f(x)\, dx \geq \int_a^b g(x)\, dx$$

### Property 9: Absolute Value Inequality

$$\left|\int_a^b f(x)\, dx\right| \leq \int_a^b |f(x)|\, dx$$

### Property 10: Bounding

If m ≤ f(x) ≤ M on [a, b], then:
$$m(b-a) \leq \int_a^b f(x)\, dx \leq M(b-a)$$

```
  y
  M ┤─────────────────  upper bound
    │    ╱──╲
    │   ╱    ╲   f(x)   Area is between
    │  ╱      ╲          m(b−a) and M(b−a)
  m ┤─────────────────  lower bound
    └──┬──────────┬──── x
       a          b
```

---

## 3.4 Mean Value Theorem for Integrals

> **Theorem:** If f is continuous on [a, b], there exists a point c ∈ (a, b) such that:
>
> $$\int_a^b f(x)\, dx = f(c) \cdot (b - a)$$

The **average value** of f on [a, b] is:

$$f_{avg} = \frac{1}{b-a}\int_a^b f(x)\, dx$$

```
  y
  │     ╱──╲
  │    ╱    ╲       f(x)
  │   ╱      ╲
  │──╱────────╲──── f(c) = average value
  │ ╱██████████╲
  │╱████████████╲
  └──┬──────────┬── x
     a    c     b
  
  Rectangle area = f(c)·(b−a) = ∫ₐᵇ f(x) dx
```

### Example

Find the average value of f(x) = x² on [0, 3]:

$$f_{avg} = \frac{1}{3-0}\int_0^3 x^2\, dx = \frac{1}{3} \cdot \frac{x^3}{3}\Big|_0^3 = \frac{1}{3} \cdot 9 = 3$$

---

## 3.5 Properties — Quick Reference Table

| # | Property | Formula |
|---|---|---|
| 1 | Zero width | ∫ₐᵃ f dx = 0 |
| 2 | Reverse limits | ∫ₐᵇ = −∫ᵇₐ |
| 3 | Constant multiple | ∫ₐᵇ kf dx = k∫ₐᵇ f dx |
| 4 | Sum/Difference | ∫(f±g) = ∫f ± ∫g |
| 5 | Additivity | ∫ₐᵇ = ∫ₐᶜ + ∫ᶜᵇ |
| 6 | Dummy variable | ∫f(x)dx = ∫f(t)dt |
| 7 | Non-negative | f ≥ 0 ⟹ ∫f ≥ 0 |
| 8 | Comparison | f ≥ g ⟹ ∫f ≥ ∫g |
| 9 | Triangle inequality | \|∫f\| ≤ ∫\|f\| |
| 10 | Bounding | m(b−a) ≤ ∫f ≤ M(b−a) |
| 11 | Mean Value | ∫ₐᵇ f = f(c)·(b−a) for some c |

---

## Summary Table

| Concept | Key Point |
|---|---|
| Linearity | Constants pull out; sums/differences split |
| Additivity | Can split/combine integration intervals |
| Dummy variable | Name of integration variable is irrelevant |
| Order | Integral preserves ≥ inequalities |
| Bounding | m(b−a) ≤ ∫ₐᵇ f ≤ M(b−a) |
| Average value | f_avg = (1/(b−a))·∫ₐᵇ f(x) dx |
| MVT for Integrals | Guarantees f(c) = f_avg for some c in (a,b) |

---

## Quick Revision Questions

1. If ∫₀⁵ f(x) dx = 7 and ∫₀³ f(x) dx = 4, find ∫₃⁵ f(x) dx.

2. Explain why ∫ₐᵇ f(x) dx = −∫ᵇₐ f(x) dx.

3. Find the average value of f(x) = sin x on [0, π].

4. State the bounding property and use it to estimate ∫₀¹ eˣ dx without computing.

5. If f(x) ≥ g(x) on [1, 4] and ∫₁⁴ g(x) dx = 5, what can you say about ∫₁⁴ f(x) dx?

6. Why is ∫ₐᵇ f(x) dx = ∫ₐᵇ f(t) dt? What does "dummy variable" mean?

---

[← Previous: Fundamental Theorem](02-fundamental-theorem.md) | [Back to README](../README.md) | [Next: Evaluation Techniques →](04-evaluation-techniques.md)
