# Chapter 4: Volume of Revolution — Comprehensive Summary

[← Previous: Shell Method](03-shell-method.md) | [Back to README](../README.md) | [Next: Arc Length →](../08-Applications-Other/01-arc-length.md)

---

## 4.1 Overview

This chapter consolidates all volume-of-revolution methods and adds cross-section methods for non-rotational solids. Use this as your go-to reference when setting up volume integrals.

---

## 4.2 Master Comparison Table

| Method | Formula | Cross-section | Slicing |
|---|---|---|---|
| **Disk** | V = π∫R² dx | Full circle | ⊥ to axis |
| **Washer** | V = π∫(R²−r²) dx | Annular ring | ⊥ to axis |
| **Shell** | V = 2π∫r·h dx | Cylindrical shell | ∥ to axis |

---

## 4.3 Decision Flowchart

```
  VOLUME OF REVOLUTION — WHICH METHOD?
  ═════════════════════════════════════
  
  START
    │
    ├── Does region touch the axis of rotation?
    │   ├── YES → DISK method possible
    │   └── NO  → WASHER method needed (has hole)
    │
    ├── Would disk/washer require splitting the integral?
    │   ├── YES → Consider SHELL method (often simpler)
    │   └── NO  → Disk/Washer is fine
    │
    ├── Is the curve easier as y=f(x) or x=g(y)?
    │   ├── y=f(x) + revolve about x-axis → DISK (dx)
    │   ├── y=f(x) + revolve about y-axis → SHELL (dx)
    │   ├── x=g(y) + revolve about y-axis → DISK (dy)
    │   └── x=g(y) + revolve about x-axis → SHELL (dy)
    │
    └── When in doubt: SET UP BOTH and pick the simpler one
```

---

## 4.4 Quick Reference — All Formulas

### Revolving About x-axis (or y = k)

| Variable | Method | Formula |
|---|---|---|
| dx | Disk | V = π∫ₐᵇ [f(x)]² dx |
| dx | Washer | V = π∫ₐᵇ {[f(x)]² − [g(x)]²} dx |
| dy | Shell | V = 2π∫ᶜᵈ y·[x_right − x_left] dy |

### Revolving About y-axis (or x = k)

| Variable | Method | Formula |
|---|---|---|
| dy | Disk | V = π∫ᶜᵈ [g(y)]² dy |
| dy | Washer | V = π∫ᶜᵈ {[R(y)]² − [r(y)]²} dy |
| dx | Shell | V = 2π∫ₐᵇ x·[f(x) − g(x)] dx |

---

## 4.5 Cross-Sectional Method (Non-Rotational)

For solids with known cross-sections:

$$V = \int_a^b A(x)\, dx$$

where A(x) is the area of the cross-section at position x.

### Common Cross-Section Shapes

```
  Cross-section    Area formula        Example
  ─────────────    ────────────        ───────
  Square           A = s²              s = side length at x
  Equilateral △    A = (√3/4)s²        s = side length at x
  Semicircle       A = (π/2)r²         r = radius at x
  Isosceles right △ A = ½s²            s = leg length at x
```

### Example: Square Cross-Sections

Find the volume of the solid whose base is the circle x² + y² = 1 with square cross-sections perpendicular to the x-axis.

At position x: the base width is 2√(1−x²), so the square has side s = 2√(1−x²).

$$V = \int_{-1}^1 [2\sqrt{1-x^2}]^2\, dx = \int_{-1}^1 4(1-x^2)\, dx = 4\left[x - \frac{x^3}{3}\right]_{-1}^1 = \frac{16}{3}$$

---

## 4.6 Pappus' Theorem

> **Theorem (Pappus):** The volume of a solid of revolution = (area of region) × (distance traveled by centroid).
>
> $$V = 2\pi \bar{d} \cdot A$$
>
> where $\bar{d}$ is the distance from the centroid to the axis of rotation, and A is the area.

### Applications

| Solid | Centroid Distance | Area | Volume |
|---|---|---|---|
| Torus | R | πr² | 2π²Rr² |
| Semicircle about diameter | 4r/(3π) | πr²/2 | ⁴⁄₃πr³ (sphere!) |

This is powerful when you know the centroid but the integration is complicated.

---

## 4.7 Volume in Parametric and Polar Forms

### Parametric (Revolution about x-axis)

$$V = \pi\int_{t_1}^{t_2} [y(t)]^2 \cdot x'(t)\, dt$$

### Polar (Revolution about the initial line, θ = 0)

$$V = \frac{2\pi}{3}\int_{\alpha}^{\beta} r^3 \sin\theta\, d\theta$$

---

## 4.8 Famous Volumes

| Solid | Volume |
|---|---|
| Sphere (radius R) | 4πR³/3 |
| Cone (radius r, height h) | πr²h/3 |
| Ellipsoid (a, b, c) | 4πabc/3 |
| Torus (R, r) | 2π²Rr² |
| Paraboloid (radius R, height h) | πR²h/2 |
| Cylinder (radius R, height h) | πR²h |
| Gabriel's Horn (y=1/x, x≥1) | π |

---

## 4.9 Worked Example — Choosing the Best Method

**Problem:** Region bounded by y = x² and y = 2x, revolved about the x-axis.

**Step 1:** Find intersections. x² = 2x → x = 0, 2.

**Step 2:** Which method?
- About x-axis with dx → Washer (region doesn't touch axis at most points... actually it does for part). Let's use washer.

**Washer:** Outer R = 2x, inner r = x²:

$$V = \pi\int_0^2 [(2x)^2 - (x^2)^2]\, dx = \pi\int_0^2 (4x^2 - x^4)\, dx$$

$$= \pi\left[\frac{4x^3}{3} - \frac{x^5}{5}\right]_0^2 = \pi\left(\frac{32}{3} - \frac{32}{5}\right) = \pi \cdot \frac{64}{15} = \frac{64\pi}{15}$$

---

## Summary Table

| Method | When to Use | Key Formula |
|---|---|---|
| Disk | Full circle cross-section, region touches axis | V = π∫R² |
| Washer | Ring cross-section, hole present | V = π∫(R²−r²) |
| Shell | Parallel slicing, avoids splitting | V = 2π∫r·h |
| Cross-section | Known cross-sectional shapes | V = ∫A(x) dx |
| Pappus | Centroid known | V = 2πd̄·A |
| Parametric | Curve in parametric form | V = π∫y²·x' dt |

---

## Quick Revision Questions

1. For the region between y = √x and y = x/2 (0 ≤ x ≤ 4), set up volume integrals using (a) disk/washer about x-axis, (b) shell about x-axis, (c) shell about y-axis.

2. The base of a solid is the region between y = x² and y = 1. Cross-sections perpendicular to the y-axis are equilateral triangles. Find the volume.

3. Use Pappus' theorem to find the centroid of a semicircular region, given that revolving it produces a sphere.

4. Find the volume of Gabriel's Horn: y = 1/x for x ≥ 1 revolved about the x-axis.

5. Compare the three methods (disk, washer, shell) for the region y = sin x, 0 ≤ x ≤ π revolved about the y-axis. Which is simplest?

6. An ellipse x²/9 + y²/4 = 1 is revolved about the x-axis. Find the volume of the resulting ellipsoid.

---

[← Previous: Shell Method](03-shell-method.md) | [Back to README](../README.md) | [Next: Arc Length →](../08-Applications-Other/01-arc-length.md)
