# Chapter 3: Shell Method

[← Previous: Washer Method](02-washer-method.md) | [Back to README](../README.md) | [Next: Volume of Revolution Summary →](04-volume-of-revolution.md)

---

## 3.1 Overview

The **shell method** (cylindrical shell method) computes volumes by decomposing the solid into thin **cylindrical shells** — like nested Russian dolls. It's especially useful when the disk/washer approach leads to complicated integrals.

---

## 3.2 The Cylindrical Shell

```
  A thin cylindrical shell:
  
      ╱────╲          Unrolled shell:
    ╱│      │╲        ┌─────────────────┐
   │ │      │ │       │                 │  height = h
   │ │  r   │ │  →    │                 │
   │ │      │ │       └─────────────────┘
    ╲│      │╱          width = 2πr
      ╲────╱            thickness = dr
  
  Volume of shell ≈ 2πr · h · dr
  (circumference × height × thickness)
```

---

## 3.3 Shell Method — Revolution About y-axis

> **Formula:** When the region under y = f(x) on [a, b] is revolved about the **y-axis**:
>
> $$V = 2\pi\int_a^b x \cdot f(x)\, dx$$

```
  y
  │    ╱‾╲
  │   ╱   ╲  y=f(x)
  │  ╱     ╲
  ├─┼───────┼──── x     Revolve about y-axis
    a       b
  
  At position x:
  • Shell radius = x (distance to y-axis)
  • Shell height = f(x)
  • Shell thickness = dx
  
  dV = 2π · (radius) · (height) · (thickness)
     = 2π · x · f(x) · dx
```

---

## 3.4 Shell Method — Revolution About x-axis

When revolving about the x-axis, integrate with respect to y:

$$V = 2\pi\int_c^d y \cdot g(y)\, dy$$

where x = g(y) is the width of the region at height y.

---

## 3.5 Shell Method — General Axis

### About x = k (vertical line)

$$V = 2\pi\int_a^b |x - k| \cdot f(x)\, dx$$

### About y = k (horizontal line)

$$V = 2\pi\int_c^d |y - k| \cdot g(y)\, dy$$

The radius is always the **distance** from the shell to the axis.

---

## 3.6 Worked Examples

### Example 1: Basic Shell Method

Revolve y = x² (0 ≤ x ≤ 1) about the y-axis.

$$V = 2\pi\int_0^1 x \cdot x^2\, dx = 2\pi\int_0^1 x^3\, dx = 2\pi\left[\frac{x^4}{4}\right]_0^1 = \frac{\pi}{2}$$

**Verification with disk method:** x = √y, 0 ≤ y ≤ 1.
$$V = \pi\int_0^1 (\sqrt{y})^2\, dy = \pi\int_0^1 y\, dy = \frac{\pi}{2} \quad✓$$

---

### Example 2: Region Between Two Curves

Revolve the region between y = x and y = x² (0 ≤ x ≤ 1) about the y-axis.

Shell height = x − x² (top minus bottom):

$$V = 2\pi\int_0^1 x(x - x^2)\, dx = 2\pi\int_0^1 (x^2 - x^3)\, dx$$

$$= 2\pi\left[\frac{x^3}{3} - \frac{x^4}{4}\right]_0^1 = 2\pi\left(\frac{1}{3} - \frac{1}{4}\right) = \frac{\pi}{6}$$

---

### Example 3: About a Non-Standard Axis

Revolve y = x² (0 ≤ x ≤ 1) about x = 2.

```
  y│        x=2
   │   ╱     │
  1│  ╱      │
   │ ╱──r────│  r = 2 − x
   │╱        │
   ├─────────┼── x
   0    1    2
```

Shell radius = 2 − x, shell height = x²:

$$V = 2\pi\int_0^1 (2-x) \cdot x^2\, dx = 2\pi\int_0^1 (2x^2 - x^3)\, dx$$

$$= 2\pi\left[\frac{2x^3}{3} - \frac{x^4}{4}\right]_0^1 = 2\pi\left(\frac{2}{3} - \frac{1}{4}\right) = 2\pi \cdot \frac{5}{12} = \frac{5\pi}{6}$$

---

### Example 4: Shell About x-axis

Find the volume when x = 4 − y² (0 ≤ y ≤ 2) is revolved about the x-axis.

Shell radius = y, shell height (width) = 4 − y²:

$$V = 2\pi\int_0^2 y(4 - y^2)\, dy = 2\pi\int_0^2 (4y - y^3)\, dy$$

$$= 2\pi\left[2y^2 - \frac{y^4}{4}\right]_0^2 = 2\pi(8 - 4) = 8\pi$$

---

## 3.7 Disk/Washer vs Shell — Decision Guide

```
  CHOOSING THE RIGHT METHOD
  ═════════════════════════

  Axis of revolution    Integrate w.r.t.    Method
  ────────────────────────────────────────────────
  x-axis (y=0)          dx                  Disk/Washer
  x-axis (y=0)          dy                  Shell
  y-axis (x=0)          dx                  Shell
  y-axis (x=0)          dy                  Disk/Washer
  
  RULE OF THUMB:
  ┌─────────────────────────────────────────────┐
  │ Integration variable SAME direction as axis │
  │   → Disk/Washer                             │
  │ Integration variable PERPENDICULAR to axis  │
  │   → Shell                                   │
  └─────────────────────────────────────────────┘
  
  When in doubt: Try both, pick the simpler integral!
```

### Comparison Table

| Feature | Disk/Washer | Shell |
|---|---|---|
| Slicing | Perpendicular to axis | Parallel to axis |
| Cross-section shape | Circle/Annulus | Cylinder |
| Formula | π∫R² (or π∫(R²−r²)) | 2π∫r·h |
| Best for | Simple curves | Curves that need splitting for disk |
| About y-axis | Need x = g(y) | Use y = f(x) directly |

---

## 3.8 Same Problem, Two Methods

**Problem:** Volume of y = √x, 0 ≤ x ≤ 4, revolved about the x-axis.

**Disk (dx):**
$$V = \pi\int_0^4 (\sqrt{x})^2\, dx = \pi\int_0^4 x\, dx = 8\pi$$

**Shell (dy):** x ranges from y² to 4, shell radius = y, height = 4 − y²:
$$V = 2\pi\int_0^2 y(4 - y^2)\, dy = 2\pi[2y^2 - y^4/4]_0^2 = 2\pi(8-4) = 8\pi \quad✓$$

Both give the same answer — the method doesn't change the result!

---

## Summary Table

| Scenario | Formula |
|---|---|
| About y-axis | V = 2π ∫ₐᵇ x·f(x) dx |
| About x-axis | V = 2π ∫ᶜᵈ y·g(y) dy |
| About x = k | V = 2π ∫ \|x−k\|·f(x) dx |
| About y = k | V = 2π ∫ \|y−k\|·g(y) dy |
| Between curves | V = 2π ∫ (radius)·(height difference) dx |
| Key: shell volume | dV = 2πr · h · dr |

---

## Quick Revision Questions

1. Find the volume when y = sin x (0 ≤ x ≤ π) is revolved about the y-axis using the shell method.

2. Find the volume when y = 1/x (1 ≤ x ≤ 3) is revolved about the y-axis.

3. Set up the shell integral for y = x³ (0 ≤ x ≤ 1) revolved about x = −1.

4. Show that the disk and shell methods give the same volume for y = x² (0 ≤ x ≤ 2) revolved about the y-axis.

5. When is the shell method clearly preferable to the disk/washer method? Give a specific example.

6. Find the volume of a sphere of radius R using the shell method (revolve about the x-axis).

---

[← Previous: Washer Method](02-washer-method.md) | [Back to README](../README.md) | [Next: Volume of Revolution Summary →](04-volume-of-revolution.md)
