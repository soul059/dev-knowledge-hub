# Chapter 1: Area Under a Curve

[← Previous: Leibniz Rule](../05-Properties-of-Definite-Integrals/04-leibniz-rule.md) | [Back to README](../README.md) | [Next: Area Between Curves →](02-area-between-curves.md)

---

## 1.1 Overview

The definite integral was born from the problem of finding area. In this chapter, we formalize how to calculate the area bounded by a curve, the x-axis (or y-axis), and vertical (or horizontal) boundary lines.

---

## 1.2 Area Under y = f(x) Above the x-axis

> **Theorem:** If f(x) ≥ 0 on [a, b], then the area bounded by y = f(x), y = 0, x = a, x = b is:
>
> $$A = \int_a^b f(x)\, dx$$

```
  y
  │    ╱‾‾‾‾╲
  │   ╱██████╲        f(x) ≥ 0
  │  ╱████████╲
  │ ╱██████████╲
  ├─┼──────────┼───── x
    a          b
  
  A = ∫ₐᵇ f(x) dx  (shaded area)
```

---

## 1.3 When f(x) Takes Negative Values

When f(x) < 0, the integral gives **negative** contribution. The **geometric area** (always positive) requires:

$$A = \int_a^b |f(x)|\, dx$$

### Strategy: Split at Zeros

```
  y
  │   ╱╲
  │  ╱  ╲  (+)
  ├─╱────╲─────────── x
  │       ╲    ╱
  │   (-)  ╲╱
  a    c       b

  ∫ₐᵇ f(x) dx = (positive part) + (negative part) ← signed
  
  Geometric Area = ∫ₐᶜ f(x) dx + ∫ᶜᵇ |f(x)| dx
                 = ∫ₐᶜ f(x) dx − ∫ᶜᵇ f(x) dx
```

**Steps:**
1. Find roots c₁, c₂, ... where f(x) = 0 in [a, b]
2. On each sub-interval, determine if f is positive or negative
3. Add absolute values:

$$A = \int_a^{c_1} |f| + \int_{c_1}^{c_2} |f| + \cdots + \int_{c_n}^b |f|$$

---

## 1.4 Area Using Vertical Strips (dx)

Most common approach — integrate with respect to x:

$$A = \int_a^b f(x)\, dx$$

```
  Each vertical strip:
  
  y│    ____
   │   /    \
   │  /|    |\
   │ / |    | \
   │/  |    |  \
   ├───┼────┼──── x
       x   x+dx
  
  dA = f(x) · dx   (height × width)
  A = ∫ dA = ∫ₐᵇ f(x) dx
```

---

## 1.5 Area Using Horizontal Strips (dy)

When the curve is better expressed as x = g(y):

$$A = \int_c^d g(y)\, dy$$

where c and d are the y-limits.

```
  y
  d├─────────┐
   │█████████│
   │████ x=g(y)
   │██╱      │
  c├╱────────┘
   │              x
   └──────────────

  dA = g(y) · dy  (width × height)
  A = ∫ᶜᵈ g(y) dy
```

> **When to use dy?** When the curve is naturally x = g(y), or when using dx would require splitting the integral.

---

## 1.6 Worked Examples

### Example 1: Basic Area

Find the area under y = x² from x = 0 to x = 3.

$$A = \int_0^3 x^2\, dx = \left[\frac{x^3}{3}\right]_0^3 = \frac{27}{3} = 9$$

---

### Example 2: Area with Negative Region

Find the area bounded by y = sin x, y = 0 from x = 0 to x = 2π.

sin x is positive on [0, π], negative on [π, 2π]:

$$A = \int_0^{\pi} \sin x\, dx + \int_{\pi}^{2\pi} |\sin x|\, dx$$

$$= [-\cos x]_0^{\pi} + [\cos x]_{\pi}^{2\pi} = (1+1) + (1+1) = 4$$

---

### Example 3: Area Using Horizontal Strips

Find the area bounded by x = y², x = 4.

The curve x = y² is a rightward parabola. At x = 4: y = ±2.

$$A = \int_{-2}^{2} (4 - y^2)\, dy = 2\int_0^2 (4 - y^2)\, dy \quad \text{(even function)}$$

$$= 2\left[4y - \frac{y^3}{3}\right]_0^2 = 2\left(8 - \frac{8}{3}\right) = 2 \cdot \frac{16}{3} = \frac{32}{3}$$

---

### Example 4: Choosing the Right Variable

Find the area enclosed by y² = 4x and y = 2x − 4.

**Using dy (simpler):** Express both as x = f(y):
- From y² = 4x: x = y²/4
- From y = 2x − 4: x = (y + 4)/2

Find intersections: y²/4 = (y + 4)/2 → y² = 2y + 8 → y² − 2y − 8 = 0 → (y − 4)(y + 2) = 0

So y = −2 and y = 4.

$$A = \int_{-2}^{4} \left[\frac{y+4}{2} - \frac{y^2}{4}\right]\, dy = \int_{-2}^{4} \frac{2y + 8 - y^2}{4}\, dy$$

$$= \frac{1}{4}\left[y^2 + 8y - \frac{y^3}{3}\right]_{-2}^{4} = \frac{1}{4}\left[\left(16 + 32 - \frac{64}{3}\right) - \left(4 - 16 + \frac{8}{3}\right)\right]$$

$$= \frac{1}{4}\left[48 - \frac{64}{3} - 4 + 16 - \frac{8}{3}\right] = \frac{1}{4}\left[60 - 24\right] = \frac{1}{4} \cdot 36 = 9$$

Wait, let me recalculate:
$$= \frac{1}{4}\left[(16+32-\frac{64}{3})-(4-16+\frac{8}{3})\right]$$
$$= \frac{1}{4}\left[48-\frac{64}{3}-4+16-\frac{8}{3}\right] = \frac{1}{4}\left[60 - \frac{72}{3}\right] = \frac{1}{4}[60 - 24] = 9$$

$$\boxed{A = 9}$$

---

## 1.7 Decision Flowchart

```
  Area under a curve:
  ═══════════════════
       │
       ├── Curve given as y = f(x)?
       │   ├── f(x) ≥ 0 on [a,b] → A = ∫ₐᵇ f(x) dx
       │   └── f changes sign → Split at zeros, use |f|
       │
       ├── Curve given as x = g(y)?
       │   └── A = ∫ᶜᵈ g(y) dy  (horizontal strips)
       │
       └── Both forms available?
           └── Choose whichever avoids splitting
               the integral or gives simpler integrand
```

---

## 1.8 Real-World Applications

| Application | Formula |
|---|---|
| **Distance from velocity** | d = ∫₀ᵗ v(t) dt |
| **Work from force** | W = ∫ₐᵇ F(x) dx |
| **Probability** | P(a ≤ X ≤ b) = ∫ₐᵇ f(x) dx (pdf) |
| **Economics** | Consumer surplus = ∫₀^Q [D(q) − p*] dq |

---

## Summary Table

| Concept | Formula | When to Use |
|---|---|---|
| Area (f ≥ 0) | ∫ₐᵇ f(x) dx | Curve above x-axis |
| Area (signed) | ∫ₐᵇ \|f(x)\| dx | Curve crosses x-axis |
| Vertical strips | dA = f(x) dx | y = f(x) form |
| Horizontal strips | dA = g(y) dy | x = g(y) form |
| Split at zeros | Find roots, integrate piecewise | f changes sign |

---

## Quick Revision Questions

1. Find the area under y = x³ from x = 0 to x = 2.

2. Find the total area between y = x² − 4 and the x-axis from x = 0 to x = 3. (Be careful about sign changes!)

3. Find the area bounded by x = y² and x = 9 using horizontal strips.

4. Explain when it is advantageous to use dy instead of dx for area calculations.

5. Find the area bounded by y = sin x and y = 0 from x = 0 to x = 3π.

6. A velocity function is v(t) = t² − 4t + 3. Find the total distance traveled from t = 0 to t = 4.

---

[← Previous: Leibniz Rule](../05-Properties-of-Definite-Integrals/04-leibniz-rule.md) | [Back to README](../README.md) | [Next: Area Between Curves →](02-area-between-curves.md)
