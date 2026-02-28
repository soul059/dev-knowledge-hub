# Chapter 2: Washer Method

[← Previous: Disk Method](01-disk-method.md) | [Back to README](../README.md) | [Next: Shell Method →](03-shell-method.md)

---

## 2.1 Overview

The **washer method** extends the disk method to handle solids of revolution that have a **hole** through the center — like a donut, pipe, or vase with a hollow core. Cross-sections are annular rings (washers) rather than full disks.

---

## 2.2 The Washer — A Disk with a Hole

```
  Disk (full circle):      Washer (ring):
  
      ╱────╲                 ╱────╲
    ╱████████╲             ╱██╱──╲██╲
   │██████████│           │██│    │██│
    ╲████████╱             ╲██╲──╱██╱
      ╲────╱                 ╲────╱
  
  Area = πR²              Area = πR² − πr²
                                = π(R² − r²)
  
  R = outer radius
  r = inner radius
```

---

## 2.3 Washer Formula — Revolution About x-axis

> If the region between y = f(x) (outer) and y = g(x) (inner), where f(x) ≥ g(x) ≥ 0, is revolved about the x-axis:
>
> $$V = \pi\int_a^b \left\{[f(x)]^2 - [g(x)]^2\right\}\, dx$$

```
  y
  │   ╱‾‾‾f(x)‾‾╲
  │  ╱  ╱g(x)╲   ╲
  │ ╱  ╱       ╲   ╲    Revolve about x-axis
  │╱  ╱         ╲   ╲
  ├──────────────────── x
  a                  b
  
  Cross-section at x:
  ○ Outer radius R(x) = f(x)
  ○ Inner radius r(x) = g(x)
  
  dV = π[R² − r²] dx = π[f(x)² − g(x)²] dx
```

---

## 2.4 Washer Method — Other Axes

### About the y-axis

$$V = \pi\int_c^d \left\{[R(y)]^2 - [r(y)]^2\right\}\, dy$$

### About y = k

$$R(x) = |f(x) - k|, \quad r(x) = |g(x) - k|$$

$$V = \pi\int_a^b \left\{[f(x) - k]^2 - [g(x) - k]^2\right\}\, dx$$

### About x = k

$$V = \pi\int_c^d \left\{[R(y)]^2 - [r(y)]^2\right\}\, dy$$

where radii are horizontal distances to x = k.

---

## 2.5 Worked Examples

### Example 1: Region Between Two Curves

Find the volume when the region between y = x² and y = x (for 0 ≤ x ≤ 1) is revolved about the x-axis.

Here y = x is the outer curve (larger), y = x² is the inner:

$$V = \pi\int_0^1 [x^2 - (x^2)^2]\, dx = \pi\int_0^1 (x^2 - x^4)\, dx$$

$$= \pi\left[\frac{x^3}{3} - \frac{x^5}{5}\right]_0^1 = \pi\left(\frac{1}{3} - \frac{1}{5}\right) = \frac{2\pi}{15}$$

---

### Example 2: Revolve About a Line Other Than Axis

Revolve the region between y = x and y = x² (0 ≤ x ≤ 1) about y = 2.

```
  y=2 ─ ─ ─ ─ ─ ─ ─ ─
       R = 2 − x²  ↕
       r = 2 − x    ↕
  y=x  ╱
  y=x²  ╱  (below y=x for 0<x<1)
  ─────────────── x
```

Outer radius: R = 2 − x² (distance from y = x² to y = 2)
Inner radius: r = 2 − x (distance from y = x to y = 2)

$$V = \pi\int_0^1 [(2 - x^2)^2 - (2 - x)^2]\, dx$$

$$= \pi\int_0^1 [(4 - 4x^2 + x^4) - (4 - 4x + x^2)]\, dx$$

$$= \pi\int_0^1 (x^4 - 5x^2 + 4x)\, dx$$

$$= \pi\left[\frac{x^5}{5} - \frac{5x^3}{3} + 2x^2\right]_0^1 = \pi\left(\frac{1}{5} - \frac{5}{3} + 2\right) = \pi \cdot \frac{8}{15} = \frac{8\pi}{15}$$

---

### Example 3: Torus (Donut)

Revolve a circle of radius r centered at (R, 0) about the y-axis (R > r).

The circle: (x − R)² + y² = r² → x = R ± √(r² − y²)

- Outer: x_out = R + √(r² − y²)
- Inner: x_in = R − √(r² − y²)

$$V = \pi\int_{-r}^{r} [(R + \sqrt{r^2 - y^2})^2 - (R - \sqrt{r^2 - y^2})^2]\, dy$$

Expanding: $(A+B)^2 - (A-B)^2 = 4AB$

$$= \pi\int_{-r}^{r} 4R\sqrt{r^2 - y^2}\, dy = 4\pi R \cdot \frac{\pi r^2}{2} = 2\pi^2 R r^2$$

(Since $\int_{-r}^r \sqrt{r^2 - y^2}\, dy = \frac{\pi r^2}{2}$, the area of a semicircle.)

$$\boxed{V_{\text{torus}} = 2\pi^2 R r^2}$$

> This is also **Pappus' theorem**: V = 2π × d × A, where d = R (distance of centroid from axis) and A = πr².

---

### Example 4: About the y-axis

Region bounded by x = y² and x = y, revolved about the y-axis.

Intersections: y² = y → y = 0, 1. On [0,1]: x = y ≥ x = y² (line is outer).

$$V = \pi\int_0^1 [y^2 - (y^2)^2]\, dy = \pi\int_0^1 (y^2 - y^4)\, dy = \pi\left(\frac{1}{3} - \frac{1}{5}\right) = \frac{2\pi}{15}$$

---

## 2.6 Disk vs Washer Decision

```
  Does the region touch the axis of rotation?
  ├── YES → DISK method
  │        (cross-section is full circle)
  │        V = π∫ R² dx
  │
  └── NO → WASHER method
           (cross-section has a hole)
           V = π∫ (R² − r²) dx
  
  Also consider:
  ├── Slicing perpendicular to axis? → Disk/Washer
  └── Slicing parallel to axis? → Shell (next chapter)
```

---

## 2.7 Common Mistakes

```
  ⚠ Don't subtract radii: π∫(R − r)² ≠ π∫(R² − r²)
     CORRECT: V = π∫(R² − r²) dx
  
  ⚠ Don't confuse inner and outer curves
     → Check which is farther from axis
  
  ⚠ When revolving about y = k, radii are DISTANCES:
     R = |curve₁ − k|,  r = |curve₂ − k|
     → The farther curve gives outer radius
```

---

## Summary Table

| Scenario | Volume Formula |
|---|---|
| About x-axis | V = π ∫ₐᵇ [f²(x) − g²(x)] dx |
| About y-axis | V = π ∫ᶜᵈ [R²(y) − r²(y)] dy |
| About y = k | V = π ∫ [(f−k)² − (g−k)²] dx |
| Torus | V = 2π²Rr² |
| Key identity | (A+B)² − (A−B)² = 4AB |

---

## Quick Revision Questions

1. Set up the washer integral for the region between y = √x and y = x revolved about the x-axis.

2. Find the volume when the region between y = x² and y = 4 is revolved about y = 4.

3. Derive the volume of a torus using the washer method.

4. Why is π∫(R − r)² dx WRONG for the washer method? Explain with a counterexample.

5. The region bounded by y = x, y = 0, x = 1 is revolved about x = 2. Set up the washer integral.

6. Find the volume when the region between y = sin x and y = cos x (0 ≤ x ≤ π/4) is revolved about y = 1.

---

[← Previous: Disk Method](01-disk-method.md) | [Back to README](../README.md) | [Next: Shell Method →](03-shell-method.md)
