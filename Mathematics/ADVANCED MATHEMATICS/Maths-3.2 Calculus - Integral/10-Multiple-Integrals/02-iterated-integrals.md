# Chapter 2: Iterated Integrals & Fubini's Theorem

[← Previous: Double Integrals](01-double-integrals.md) | [Back to README](../README.md) | [Next: Change of Order →](03-change-of-order.md)

---

## 2.1 Overview

Computing a double integral directly from the Riemann sum definition is impractical. **Fubini's Theorem** allows us to evaluate double integrals as **iterated (repeated) single integrals** — integrating one variable at a time.

---

## 2.2 Fubini's Theorem (Rectangular Regions)

> **Theorem (Fubini):** If f(x,y) is continuous on the rectangle R = [a,b] × [c,d], then:
>
> $$\iint_R f(x,y)\, dA = \int_a^b\left(\int_c^d f(x,y)\, dy\right) dx = \int_c^d\left(\int_a^b f(x,y)\, dx\right) dy$$

Both orders of integration give the **same result**.

```
  ORDER 1: dy dx                ORDER 2: dx dy
  ══════════════                ══════════════
  
  y│  ┌────────┐               y│  ┌────────┐
  d│  │ ══════ │ ← integrate   d│  │ ║║║║║║ │ ← integrate
   │  │ ══════ │   y first      │  │ ║║║║║║ │   x first
   │  │ ══════ │   (inner)      │  │ ║║║║║║ │   (inner)
  c│  └────────┘                c│  └────────┘
   └──┤────────┤── x            └──┤────────┤── x
      a        b                   a        b
  
  Then sweep x                  Then sweep y
  (outer integral)              (outer integral)
```

---

## 2.3 Type I Regions (Vertical Slices)

A **Type I** region is bounded:

$$R = \{(x,y) : a \leq x \leq b, \quad g_1(x) \leq y \leq g_2(x)\}$$

```
  y
  │     ╱── y = g₂(x) (upper)
  │    ╱  ╲
  │   ╱    ╲
  │  ╱══════╲  ← integrate y from g₁(x) to g₂(x)
  │ ╱   R    ╲    for each fixed x
  │╱──────────╲── y = g₁(x) (lower)
  ├──┤────────┤── x
     a        b
```

$$\iint_R f(x,y)\, dA = \int_a^b \int_{g_1(x)}^{g_2(x)} f(x,y)\, dy\, dx$$

---

## 2.4 Type II Regions (Horizontal Slices)

A **Type II** region is bounded:

$$R = \{(x,y) : c \leq y \leq d, \quad h_1(y) \leq x \leq h_2(y)\}$$

```
  y
  d ──╲────────╱
  │    ╲══════╱   ← integrate x from h₁(y) to h₂(y)
  │     ╲  R ╱      for each fixed y
  │      ╲══╱
  c ──────╲╱
  │  
  ├───────────── x
     h₁(y)  h₂(y)
```

$$\iint_R f(x,y)\, dA = \int_c^d \int_{h_1(y)}^{h_2(y)} f(x,y)\, dx\, dy$$

---

## 2.5 Setting Up Limits: Step-by-Step

### For dy dx (Type I):

1. **Outer limits** (x): Find the range of x over the full region → constants a to b
2. **Inner limits** (y): For a fixed x, y runs from bottom curve to top curve → g₁(x) to g₂(x)

### For dx dy (Type II):

1. **Outer limits** (y): Find the range of y over the full region → constants c to d
2. **Inner limits** (x): For a fixed y, x runs from left curve to right curve → h₁(y) to h₂(y)

```
  DECISION GUIDE
  ═══════════════
  
  Draw the region R
       │
       ▼
  Can you express y-bounds   ──YES──→  Use dy dx (Type I)
  as functions of x?                   ∫ₐᵇ ∫_{g₁(x)}^{g₂(x)} f dy dx
       │NO
       ▼
  Can you express x-bounds   ──YES──→  Use dx dy (Type II)
  as functions of y?                   ∫_c^d ∫_{h₁(y)}^{h₂(y)} f dx dy
       │NO
       ▼
  Split R into sub-regions
  (each Type I or Type II)
```

---

## 2.6 Worked Examples

### Example 1: Triangular Region (Type I)

Evaluate ∬_R x dA over the triangle with vertices (0,0), (1,0), (1,1).

```
  y
  1 │      ╱│
    │    ╱  │
    │  ╱  R │
    │╱──────│
  0 └───────┤── x
    0       1
```

Region: 0 ≤ x ≤ 1, 0 ≤ y ≤ x

$$\int_0^1 \int_0^x x\, dy\, dx = \int_0^1 [xy]_0^x\, dx = \int_0^1 x^2\, dx = \frac{1}{3}$$

---

### Example 2: Same Region (Type II)

Using horizontal slices: 0 ≤ y ≤ 1, y ≤ x ≤ 1

$$\int_0^1 \int_y^1 x\, dx\, dy = \int_0^1 \left[\frac{x^2}{2}\right]_y^1 dy = \int_0^1 \frac{1-y^2}{2}\, dy = \frac{1}{2}\left[y - \frac{y^3}{3}\right]_0^1 = \frac{1}{2}\cdot\frac{2}{3} = \frac{1}{3}$$

Same answer ✓ — Fubini's theorem confirmed.

---

### Example 3: Curved Boundary

$$\int_0^1 \int_0^{\sqrt{x}} e^{y/\sqrt{x}}\, dy\, dx$$

Region: 0 ≤ x ≤ 1, 0 ≤ y ≤ √x

Inner integral: $\int_0^{\sqrt{x}} e^{y/\sqrt{x}}\, dy = [\sqrt{x}\, e^{y/\sqrt{x}}]_0^{\sqrt{x}} = \sqrt{x}(e - 1)$

$$= (e-1)\int_0^1 \sqrt{x}\, dx = (e-1)\cdot\frac{2}{3} = \frac{2(e-1)}{3}$$

---

### Example 4: Region Between Curves

Evaluate ∬_R (x+y) dA where R is bounded by y = x² and y = x.

```
  y
  1 │   ●
    │  ╱│╲
    │ ╱R│ ╲  y = x²
    │╱──│──╲
  0 ●───┴───── x
    0       1
    
    y = x (line above)
    y = x² (parabola below)
    Intersect: x² = x → x = 0, 1
```

Type I: 0 ≤ x ≤ 1, x² ≤ y ≤ x

$$\int_0^1\int_{x^2}^{x} (x+y)\, dy\, dx = \int_0^1 \left[xy + \frac{y^2}{2}\right]_{x^2}^{x} dx$$

$$= \int_0^1 \left[\left(x^2 + \frac{x^2}{2}\right) - \left(x^3 + \frac{x^4}{2}\right)\right] dx = \int_0^1 \left(\frac{3x^2}{2} - x^3 - \frac{x^4}{2}\right) dx$$

$$= \left[\frac{x^3}{2} - \frac{x^4}{4} - \frac{x^5}{10}\right]_0^1 = \frac{1}{2} - \frac{1}{4} - \frac{1}{10} = \frac{3}{20}$$

---

### Example 5: Integral That Requires Type II

$$\int_0^1\int_x^1 e^{y^2}\, dy\, dx$$

Cannot integrate e^{y²} with respect to y! Switch order.

```
  y                         y
  1 ┌──────┐               1 ┌──────┐
    │╲     │                 │══════│
    │ ╲ R  │   ──switch──→   │════╱ │
    │  ╲   │                 │══╱   │
  0 └──────┘               0 └─────┘
    0      1                 0      1
    
  dy dx: x to 1            dx dy: 0 to y
```

New limits: 0 ≤ y ≤ 1, 0 ≤ x ≤ y

$$\int_0^1\int_0^y e^{y^2}\, dx\, dy = \int_0^1 ye^{y^2}\, dy = \left[\frac{e^{y^2}}{2}\right]_0^1 = \frac{e-1}{2}$$

---

## 2.7 Common Mistakes

| Mistake | Why It's Wrong |
|---|---|
| Wrong limit order | Inner limits must be functions of the outer variable only |
| Constants in inner limits when region is curved | Inner limits vary with outer variable |
| Integrating inner variable in outer integral | Must fully evaluate inner integral first |
| Forgetting to check integrability | Sometimes one order works, the other doesn't |

---

## Summary Table

| Concept | Formula |
|---|---|
| Fubini (rectangle) | ∬_R f dA = ∫ₐᵇ∫_c^d f dy dx = ∫_c^d∫ₐᵇ f dx dy |
| Type I (dy dx) | ∫ₐᵇ ∫_{g₁(x)}^{g₂(x)} f(x,y) dy dx |
| Type II (dx dy) | ∫_c^d ∫_{h₁(y)}^{h₂(y)} f(x,y) dx dy |
| Switching order | Sketch region, re-express bounds |
| Key rule | Inner limits → functions of outer variable; outer limits → constants |

---

## Quick Revision Questions

1. State Fubini's Theorem and its conditions.

2. Evaluate ∬_R xy dA where R is the triangle with vertices (0,0), (2,0), (0,3).

3. Set up ∬_R f dA over the region bounded by y = x² and y = 4 in both orders.

4. Evaluate ∫₀¹ ∫ₓ¹ sin(y²) dy dx by switching the order of integration.

5. Find ∬_R y dA over the region between y = √x and y = x².

6. Explain why sometimes one order of integration is preferable to the other.

---

[← Previous: Double Integrals](01-double-integrals.md) | [Back to README](../README.md) | [Next: Change of Order →](03-change-of-order.md)
