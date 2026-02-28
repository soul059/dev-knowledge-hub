# Chapter 1: Disk Method

[← Previous: Area in Polar Form](../06-Applications-Area/04-area-polar-form.md) | [Back to README](../README.md) | [Next: Washer Method →](02-washer-method.md)

---

## 1.1 Overview

The **disk method** (or disc method) calculates volumes of solids of revolution by slicing the solid into thin circular disks perpendicular to the axis of rotation. Each disk's volume is known — it's just π r² Δx — and we sum (integrate) them all.

---

## 1.2 Revolution About the x-axis

> **Formula:** When y = f(x) ≥ 0 is revolved about the x-axis on [a, b]:
>
> $$V = \pi\int_a^b [f(x)]^2\, dx$$

### Visualization

```
  y                      3D View (cross-section):
  │   ╱‾╲                
  │  ╱   ╲  y=f(x)         ╱────╲
  │ ╱     ╲              ╱│    │╲
  ├─┼──────┼── x        ╱ │ ●  │ ╲  ← circular disk
  │ a      b             ╲ │    │ ╱     radius = f(x)
  │                        ╲│    │╱     thickness = dx
  │ Rotate about x-axis     ╲────╱
  
  Each slice at position x:
  • Shape: Circle (disk)
  • Radius: r = f(x)
  • Thickness: dx
  • Volume: dV = π[f(x)]² dx
```

### Derivation

1. Slice the solid perpendicular to x-axis at position x
2. Cross-section is a circle with radius r = f(x)
3. Area of cross-section: A(x) = πr² = π[f(x)]²
4. Volume of thin disk: dV = A(x) dx = π[f(x)]² dx
5. Total volume:

$$V = \int_a^b \pi[f(x)]^2\, dx$$

---

## 1.3 Revolution About the y-axis

When x = g(y) is revolved about the y-axis:

$$V = \pi\int_c^d [g(y)]^2\, dy$$

```
  y
  d│  ╱
   │ ╱  x=g(y)
   │╱           
  c│           Rotate about y-axis
   │────────── x
   
  Each horizontal slice:
  Radius = g(y), thickness = dy
  dV = π[g(y)]² dy
```

---

## 1.4 Worked Examples

### Example 1: Cone

Revolve y = (r/h)x about the x-axis from x = 0 to x = h.

```
  y
  │     ╱
  r    ╱
  │   ╱    y = (r/h)x
  │  ╱
  │ ╱
  ├─────────── x
  0     h
```

$$V = \pi\int_0^h \left(\frac{r}{h}x\right)^2 dx = \pi\frac{r^2}{h^2}\int_0^h x^2\, dx = \pi\frac{r^2}{h^2}\cdot\frac{h^3}{3}$$

$$\boxed{V = \frac{1}{3}\pi r^2 h}$$

This is the well-known cone volume formula!

---

### Example 2: Sphere

Revolve y = √(R² − x²) (upper semicircle) about x-axis from −R to R.

$$V = \pi\int_{-R}^{R} (R^2 - x^2)\, dx = \pi\left[R^2 x - \frac{x^3}{3}\right]_{-R}^{R}$$

$$= \pi\left[\left(R^3 - \frac{R^3}{3}\right) - \left(-R^3 + \frac{R^3}{3}\right)\right] = \pi \cdot \frac{4R^3}{3}$$

$$\boxed{V = \frac{4}{3}\pi R^3}$$

---

### Example 3: Paraboloid

Revolve y = √x about the x-axis from x = 0 to x = 4.

$$V = \pi\int_0^4 (\sqrt{x})^2\, dx = \pi\int_0^4 x\, dx = \pi\left[\frac{x^2}{2}\right]_0^4 = 8\pi$$

---

### Example 4: Revolution About y-axis

Find the volume when x = y² is revolved about the y-axis from y = 0 to y = 2.

$$V = \pi\int_0^2 (y^2)^2\, dy = \pi\int_0^2 y^4\, dy = \pi\left[\frac{y^5}{5}\right]_0^2 = \frac{32\pi}{5}$$

---

## 1.5 Revolution About Other Lines

### About y = k (horizontal line)

If revolving about y = k instead of y = 0:

$$\text{Radius} = |f(x) - k|$$

$$V = \pi\int_a^b [f(x) - k]^2\, dx$$

### About x = k (vertical line)

$$\text{Radius} = |g(y) - k|$$

$$V = \pi\int_c^d [g(y) - k]^2\, dy$$

```
  Revolving about y = 2:
  
  y=2 ─ ─ ─ ─ ─ ─ ─ ─ ─
  │        ╲   r = 2 − f(x)
  │    ╱‾‾‾‾╲
  │   ╱      ╲  y=f(x)
  ├──────────── x
  
  V = π ∫ (2 − f(x))² dx
```

---

## 1.6 When to Use the Disk Method

```
  DISK METHOD CHECKLIST
  ═════════════════════
  ✓ Solid has NO HOLE through the center
  ✓ Cross-section is a full circle (disk)
  ✓ Slicing perpendicular to the axis of rotation
  ✓ The region touches the axis of rotation
  
  If there IS a hole → Use WASHER method (next chapter)
  If slicing parallel → Use SHELL method (Chapter 3)
```

---

## Summary Table

| Scenario | Formula |
|---|---|
| About x-axis | V = π ∫ₐᵇ [f(x)]² dx |
| About y-axis | V = π ∫ᶜᵈ [g(y)]² dy |
| About y = k | V = π ∫ₐᵇ [f(x) − k]² dx |
| About x = k | V = π ∫ᶜᵈ [g(y) − k]² dy |
| Cone | V = πr²h/3 |
| Sphere | V = 4πR³/3 |

---

## Quick Revision Questions

1. Derive the volume of a cone using the disk method.

2. Find the volume when y = eˣ is revolved about the x-axis from x = 0 to x = 1.

3. Find the volume when x = √y is revolved about the y-axis from y = 0 to y = 9.

4. Find the volume when y = x² is revolved about the line y = 4 from x = 0 to x = 2.

5. Explain why the disk method requires the region to touch the axis of rotation.

6. Set up (but don't evaluate) the disk integral for the volume of an ellipsoid x²/a² + y²/b² = 1 revolved about the x-axis.

---

[← Previous: Area in Polar Form](../06-Applications-Area/04-area-polar-form.md) | [Back to README](../README.md) | [Next: Washer Method →](02-washer-method.md)
