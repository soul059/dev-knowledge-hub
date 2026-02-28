# Chapter 2: Area Between Curves

[← Previous: Area Under a Curve](01-area-under-curve.md) | [Back to README](../README.md) | [Next: Area in Parametric Form →](03-area-parametric-form.md)

---

## 2.1 Overview

One of the most common applications of integration is finding the area enclosed between two curves. The key idea: the area between two curves equals the integral of the **upper curve minus the lower curve**.

---

## 2.2 Basic Formula (Vertical Strips)

> If f(x) ≥ g(x) on [a, b], then the area between y = f(x) and y = g(x) is:
>
> $$A = \int_a^b [f(x) - g(x)]\, dx$$

```
  y
  │    ╱‾‾f(x)‾╲
  │   ╱██████████╲
  │  ╱████████████╲      ← f(x) on top
  │ ╱██████████╱   ╲
  │  ‾‾g(x)‾‾╱     ╲
  ├──────────────────── x
    a                b
  
  dA = [f(x) − g(x)] dx
  A  = ∫ₐᵇ [f(x) − g(x)] dx
```

> **Key insight:** We're NOT computing "area under f" and "area under g" separately. We're integrating the **vertical distance** between the curves.

---

## 2.3 When Curves Cross

If f and g intersect within [a, b], we must split at the intersection points:

```
  y
  │     f(x)
  │   ╱╲
  │  ╱++╲    ╱ g(x)
  │ ╱++++╲╳╱          ← intersection at x = c
  │       ╲╱╲
  │    --  ╲╱
  ├──────────────── x
  a    c       b
  
  On [a,c]: f ≥ g  → ∫ₐᶜ [f − g] dx
  On [c,b]: g ≥ f  → ∫ᶜᵇ [g − f] dx
```

**General formula:**

$$A = \int_a^b |f(x) - g(x)|\, dx$$

Or split at crossings:

$$A = \int_a^{c_1} |f - g|\, dx + \int_{c_1}^{c_2} |f - g|\, dx + \cdots$$

---

## 2.4 Using Horizontal Strips (dy)

When curves are more naturally expressed as x = g(y):

$$A = \int_c^d [x_{\text{right}}(y) - x_{\text{left}}(y)]\, dy$$

```
  y
  d├────────────┐
   │  x_left    │ x_right
   │    ╲███████╱
   │     ╲█████╱
   │      ╲███╱
  c├───────╳───── 
   │              x
   └──────────────
   
  dA = [x_right − x_left] dy
```

---

## 2.5 Choosing dx vs dy

```
  DECISION GUIDE
  ═══════════════
  
  Use dx (vertical strips) when:
  ✓ Both curves are y = f(x) form
  ✓ "Top" and "bottom" don't switch (or switch few times)
  
  Use dy (horizontal strips) when:
  ✓ Curves are x = g(y) form
  ✓ Using dx would require multiple integrals
  ✓ "Right" and "left" stay consistent
  
  RULE OF THUMB: Pick the variable that requires
  fewer integrals (fewer split points).
```

---

## 2.6 Worked Examples

### Example 1: Simple Area Between Curves

Find the area between y = x² and y = x from x = 0 to x = 1.

On [0, 1]: x ≥ x² (since 0 ≤ x ≤ 1), so x is the upper curve.

$$A = \int_0^1 (x - x^2)\, dx = \left[\frac{x^2}{2} - \frac{x^3}{3}\right]_0^1 = \frac{1}{2} - \frac{1}{3} = \frac{1}{6}$$

---

### Example 2: Finding Intersection Points

Find the area enclosed by y = x² and y = 2x + 3.

**Step 1:** Find intersections: x² = 2x + 3 → x² − 2x − 3 = 0 → (x−3)(x+1) = 0 → x = −1, 3

**Step 2:** Determine which is on top. At x = 0: line gives 3, parabola gives 0. So line is on top.

$$A = \int_{-1}^3 [(2x + 3) - x^2]\, dx = \left[x^2 + 3x - \frac{x^3}{3}\right]_{-1}^3$$

$$= \left(9 + 9 - 9\right) - \left(1 - 3 + \frac{1}{3}\right) = 9 - \left(-\frac{5}{3}\right) = 9 + \frac{5}{3} = \frac{32}{3}$$

---

### Example 3: Curves That Cross

Find the area between y = sin x and y = cos x from x = 0 to x = π.

They cross at x = π/4 (where sin x = cos x). On [0, π/4]: cos x ≥ sin x. On [π/4, π]: sin x ≥ cos x.

$$A = \int_0^{\pi/4} (\cos x - \sin x)\, dx + \int_{\pi/4}^{\pi} (\sin x - \cos x)\, dx$$

$$= [\sin x + \cos x]_0^{\pi/4} + [-\cos x - \sin x]_{\pi/4}^{\pi}$$

$$= \left(\frac{\sqrt{2}}{2} + \frac{\sqrt{2}}{2} - 0 - 1\right) + \left(1 - 0 + \frac{\sqrt{2}}{2} + \frac{\sqrt{2}}{2}\right)$$

$$= (\sqrt{2} - 1) + (1 + \sqrt{2}) = 2\sqrt{2}$$

---

### Example 4: Using Horizontal Strips

Find the area enclosed by y² = x and y = x − 2.

**Using dy** (avoids splitting):
- From y² = x: x = y²
- From y = x − 2: x = y + 2

Intersections: y² = y + 2 → y² − y − 2 = 0 → (y − 2)(y + 1) = 0 → y = −1, 2

On [−1, 2]: the line x = y + 2 is to the right of the parabola x = y².

$$A = \int_{-1}^2 [(y + 2) - y^2]\, dy = \left[\frac{y^2}{2} + 2y - \frac{y^3}{3}\right]_{-1}^2$$

$$= \left(2 + 4 - \frac{8}{3}\right) - \left(\frac{1}{2} - 2 + \frac{1}{3}\right) = \left(\frac{10}{3}\right) - \left(-\frac{7}{6}\right) = \frac{20}{6} + \frac{7}{6} = \frac{27}{6} = \frac{9}{2}$$

---

### Example 5: Three Curves

Find the area bounded by y = x², y = 0, and x = 2.

$$A = \int_0^2 x^2\, dx = \left[\frac{x^3}{3}\right]_0^2 = \frac{8}{3}$$

---

## 2.7 Common Curve Pairs

| Curves | Intersections | Area |
|---|---|---|
| y = x² and y = x | x = 0, 1 | 1/6 |
| y = x² and y = √x | x = 0, 1 | 1/3 |
| y = x² and y = 2x − x² | x = 0, 1 | 1/3 |
| y = sin x and y = 0, [0,π] | x = 0, π | 2 |
| x² + y² = r² (circle) | — | πr² |

---

## 2.8 Step-by-Step Procedure

```
  FINDING AREA BETWEEN CURVES
  ════════════════════════════
  1. Sketch the curves (even roughly)
  2. Find ALL intersection points
  3. Determine which curve is "top/right"
     on each sub-interval
  4. Choose dx or dy based on simplicity
  5. Set up integral(s)
  6. Evaluate
  7. Check: Area must be POSITIVE!
```

---

## Summary Table

| Situation | Formula |
|---|---|
| f(x) ≥ g(x) on [a,b] | A = ∫ₐᵇ [f(x) − g(x)] dx |
| Curves cross | Split at crossings, use \|f − g\| |
| Horizontal strips | A = ∫ᶜᵈ [x_right − x_left] dy |
| General | A = ∫ₐᵇ \|f(x) − g(x)\| dx |
| Enclosed region | Find intersection points first |

---

## Quick Revision Questions

1. Find the area enclosed between y = x² and y = x³.

2. Find the area between y = eˣ and y = e⁻ˣ from x = 0 to x = 1.

3. Find the area enclosed by x = y² and x = 2y + 3 using horizontal strips.

4. The curves y = x² and y = 6 − x² enclose a region. Find its area.

5. Find the area between y = |x| and y = x² − 2. (Hint: use symmetry.)

6. Explain with a diagram why using horizontal strips might require one integral while vertical strips might require two for the same region.

---

[← Previous: Area Under a Curve](01-area-under-curve.md) | [Back to README](../README.md) | [Next: Area in Parametric Form →](03-area-parametric-form.md)
