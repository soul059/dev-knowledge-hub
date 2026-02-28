# Chapter 1: Double Integrals

[← Previous: Gamma & Beta Functions](../09-Improper-Integrals/04-comparison-tests.md) | [Back to README](../README.md) | [Next: Iterated Integrals →](02-iterated-integrals.md)

---

## 1.1 Overview

A **double integral** extends single-variable integration to functions of two variables. Instead of computing area under a curve, we compute **volume under a surface** over a planar region.

---

## 1.2 Motivation: From Single to Double

```
  SINGLE INTEGRAL                DOUBLE INTEGRAL
  ═══════════════                ═══════════════
  
  y                              z
  │  ╱╲                          │   ╱╲  surface z=f(x,y)
  │ ╱  ╲   curve                 │  ╱  ╲───╲
  │╱    ╲  y=f(x)               │ ╱    ╲   ╲
  ├──────╲──── x                 │╱──────╲───╲── y
  a      b                      ╱  Region R   ╲
                                x
  
  Area = ∫ₐᵇ f(x)dx             Volume = ∬_R f(x,y) dA
```

---

## 1.3 Definition via Riemann Sums

Let f(x,y) be defined on a closed, bounded region R.

**Step 1:** Partition R into n small sub-regions ΔA₁, ΔA₂, ..., ΔAₙ

**Step 2:** Choose sample point (xᵢ*, yᵢ*) in each ΔAᵢ

**Step 3:** Form the Riemann sum:

$$S_n = \sum_{i=1}^{n} f(x_i^*, y_i^*)\, \Delta A_i$$

**Step 4:** Take the limit as the mesh size → 0:

$$\boxed{\iint_R f(x,y)\, dA = \lim_{n \to \infty} \sum_{i=1}^{n} f(x_i^*, y_i^*)\, \Delta A_i}$$

```
  Partition of region R into sub-rectangles:
  
  y
  │ ┌──┬──┬──┬──┐
  d │  │  │  │  │
  │ ├──┼──┼──┼──┤
  │ │  │● │  │  │  ← sample point (xᵢ*,yᵢ*)
  │ ├──┼──┼──┼──┤     in sub-rectangle
  │ │  │  │  │  │     ΔAᵢ = Δx · Δy
  c ├──┼──┼──┼──┤
  │ │  │  │  │  │
  │ └──┴──┴──┴──┘
  ├──┤           ├── x
  a  Δx          b
```

---

## 1.4 Rectangular Regions

For R = [a,b] × [c,d] (a rectangle), dA = dx dy:

$$\iint_R f(x,y)\, dA = \int_a^b \int_c^d f(x,y)\, dy\, dx = \int_c^d \int_a^b f(x,y)\, dx\, dy$$

The order of integration can be switched for continuous functions (**Fubini's Theorem**).

---

## 1.5 Properties of Double Integrals

| Property | Formula |
|---|---|
| Linearity | ∬_R [αf + βg] dA = α∬_R f dA + β∬_R g dA |
| Additivity | If R = R₁ ∪ R₂ (non-overlapping): ∬_R f dA = ∬_{R₁} f dA + ∬_{R₂} f dA |
| Comparison | If f ≥ g on R, then ∬_R f dA ≥ ∬_R g dA |
| Constant | ∬_R c · dA = c · Area(R) |
| Absolute value | |∬_R f dA| ≤ ∬_R |f| dA |
| Area | ∬_R 1 · dA = Area(R) |

---

## 1.6 Geometric Interpretation

$$\text{Volume} = \iint_R f(x,y)\, dA$$

```
  z = f(x,y)
  │           ╱───╲
  │          ╱ ╱╲  ╲     ← surface
  │         ╱╱   ╲╲ ╲
  │        ╱───────╲ ╲
  │       ╱ VOLUME  ╲ ╲
  │      ╱───────────╲──── y
  │     ╱   Region R  ╲
  │    ╱────────────────╲
  ├──────────────────────── x
  
  Volume = ∬_R f(x,y) dA
  = stacking thin columns f(x,y)·dA
```

- If f(x,y) > 0: volume above the xy-plane
- If f(x,y) < 0: negative of volume below
- Net signed volume in general

---

## 1.7 Worked Examples

### Example 1: Over a Rectangle

$$\iint_R (x^2 + y)\, dA, \quad R = [0,1] \times [0,2]$$

$$= \int_0^1 \int_0^2 (x^2 + y)\, dy\, dx = \int_0^1 \left[x^2 y + \frac{y^2}{2}\right]_0^2 dx$$

$$= \int_0^1 (2x^2 + 2)\, dx = \left[\frac{2x^3}{3} + 2x\right]_0^1 = \frac{2}{3} + 2 = \frac{8}{3}$$

---

### Example 2: Product Form

When f(x,y) = g(x)·h(y) and R = [a,b]×[c,d]:

$$\iint_R g(x)\,h(y)\, dA = \left(\int_a^b g(x)\, dx\right)\left(\int_c^d h(y)\, dy\right)$$

**Example:** ∬_R xy dA over R = [0,1]×[0,2]

$$= \left(\int_0^1 x\, dx\right)\left(\int_0^2 y\, dy\right) = \frac{1}{2} \cdot 2 = 1$$

---

### Example 3: Volume of Solid

Find the volume under z = 4 − x² − y² above the rectangle R = [0,1]×[0,1].

$$V = \int_0^1\int_0^1 (4 - x^2 - y^2)\, dy\, dx = \int_0^1\left[4y - x^2 y - \frac{y^3}{3}\right]_0^1 dx$$

$$= \int_0^1\left(4 - x^2 - \frac{1}{3}\right) dx = \int_0^1\left(\frac{11}{3} - x^2\right) dx = \frac{11}{3} - \frac{1}{3} = \frac{10}{3}$$

---

### Example 4: Constant Function — Area

$$\iint_R 1\, dA = \text{Area}(R)$$

For R = [a,b]×[c,d]: Area = (b−a)(d−c) ✓

---

## 1.8 Real-World Application

**Heat Distribution:** The average temperature over a plate R is:

$$T_{\text{avg}} = \frac{1}{\text{Area}(R)}\iint_R T(x,y)\, dA$$

**Example:** Temperature T(x,y) = 100 − x² − y² on plate R = [0,3]×[0,4]:

$$T_{\text{avg}} = \frac{1}{12}\int_0^3\int_0^4 (100 - x^2 - y^2)\, dy\, dx$$

$$= \frac{1}{12}\int_0^3\left[100y - x^2y - \frac{y^3}{3}\right]_0^4 dx = \frac{1}{12}\int_0^3\left(400 - 4x^2 - \frac{64}{3}\right) dx$$

$$= \frac{1}{12}\int_0^3\left(\frac{1136}{3} - 4x^2\right) dx = \frac{1}{12}\left[\frac{1136x}{3} - \frac{4x^3}{3}\right]_0^3 = \frac{1}{12}\left(1136 - 36\right) = \frac{1100}{12} \approx 91.67°$$

---

## Summary Table

| Concept | Formula |
|---|---|
| Double integral (definition) | ∬_R f(x,y) dA = lim Σf(xᵢ*,yᵢ*)ΔAᵢ |
| Over rectangle [a,b]×[c,d] | ∫ₐᵇ∫_c^d f(x,y) dy dx |
| Volume under surface | V = ∬_R f(x,y) dA |
| Area of region | A = ∬_R 1 dA |
| Average value | f̄ = (1/Area(R)) ∬_R f dA |
| Product form | ∬ g(x)h(y) dA = (∫g dx)(∫h dy) |

---

## Quick Revision Questions

1. Define the double integral as a limit of Riemann sums.

2. Evaluate ∬_R (3x + 2y) dA where R = [0,2]×[1,3].

3. Show that for f(x,y) = g(x)·h(y) on a rectangle, the double integral factors.

4. Find the volume under z = x + y over [0,1]×[0,1].

5. Compute the average value of f(x,y) = xy over R = [0,2]×[0,4].

---

[← Previous: Gamma & Beta Functions](../09-Improper-Integrals/04-comparison-tests.md) | [Back to README](../README.md) | [Next: Iterated Integrals →](02-iterated-integrals.md)
