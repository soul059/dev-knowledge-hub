# Chapter 3: Center of Mass

[← Previous: Surface Area of Revolution](02-surface-area-of-revolution.md) | [Back to README](../README.md) | [Next: Work and Fluid Pressure →](04-work-and-fluid-pressure.md)

---

## 3.1 Overview

Integration allows us to find the **center of mass** (centroid) of continuous distributions — regions, curves, and solids. The center of mass is the "balance point" where the object would balance perfectly on a pin.

---

## 3.2 Discrete vs Continuous

### Discrete System

For n point masses m₁, m₂, ..., mₙ at positions (x₁, y₁), (x₂, y₂), ...:

$$\bar{x} = \frac{\sum m_i x_i}{\sum m_i}, \qquad \bar{y} = \frac{\sum m_i y_i}{\sum m_i}$$

### Continuous System

Replace sums with integrals:

$$\bar{x} = \frac{\int x\, dm}{\int dm}, \qquad \bar{y} = \frac{\int y\, dm}{\int dm}$$

```
  Discrete:              Continuous:
  
  ●    ●                 ╱‾‾‾‾╲
     ●                  ╱██████╲
       ●   ●           ╱████████╲
  ──●──────────      ──╱██████████╲──
  balance at x̄         balance at x̄
```

---

## 3.3 Centroid of a Plane Region

For a region bounded by y = f(x) ≥ 0 on [a, b] with uniform density:

$$\bar{x} = \frac{\int_a^b x \cdot f(x)\, dx}{\int_a^b f(x)\, dx}$$

$$\bar{y} = \frac{\int_a^b \frac{1}{2}[f(x)]^2\, dx}{\int_a^b f(x)\, dx}$$

> The denominator is just the area A of the region.

### Why ½[f(x)]² for ȳ?

```
  At position x, the vertical strip goes from 0 to f(x).
  Its centroid is at height f(x)/2 (midpoint of strip).
  Its mass element is dA = f(x) dx.
  
  Moment about x-axis:
  dMₓ = (f(x)/2) · f(x) dx = ½[f(x)]² dx
```

---

## 3.4 Centroid Between Two Curves

For the region between y = f(x) (top) and y = g(x) (bottom) on [a, b]:

$$\bar{x} = \frac{1}{A}\int_a^b x[f(x) - g(x)]\, dx$$

$$\bar{y} = \frac{1}{2A}\int_a^b [f(x)^2 - g(x)^2]\, dx$$

where $A = \int_a^b [f(x) - g(x)]\, dx$.

---

## 3.5 Worked Examples

### Example 1: Triangle

Find the centroid of the triangular region with vertices (0,0), (b,0), (0,h).

The hypotenuse: y = h(1 − x/b), so f(x) = h(1 − x/b), x ∈ [0, b].

Area: A = ½bh.

$$\bar{x} = \frac{1}{A}\int_0^b x \cdot h\left(1 - \frac{x}{b}\right)\, dx = \frac{2}{bh} \cdot h\int_0^b \left(x - \frac{x^2}{b}\right)\, dx$$

$$= \frac{2}{b}\left[\frac{x^2}{2} - \frac{x^3}{3b}\right]_0^b = \frac{2}{b}\left(\frac{b^2}{2} - \frac{b^2}{3}\right) = \frac{2}{b} \cdot \frac{b^2}{6} = \frac{b}{3}$$

Similarly: $\bar{y} = h/3$.

$$\boxed{(\bar{x}, \bar{y}) = \left(\frac{b}{3}, \frac{h}{3}\right)}$$

> The centroid of a triangle is at 1/3 of each dimension — the intersection of medians.

---

### Example 2: Semicircular Region

Find the centroid of the upper semicircle y = √(R² − x²), x ∈ [−R, R].

By symmetry: $\bar{x} = 0$.

Area: A = πR²/2.

$$\bar{y} = \frac{1}{A}\int_{-R}^{R} \frac{1}{2}(R^2 - x^2)\, dx = \frac{2}{\pi R^2}\left[\frac{R^2 x}{2} - \frac{x^3}{6}\right]_{-R}^R$$

$$= \frac{2}{\pi R^2}\left[R^3 - \frac{R^3}{3}\right] = \frac{2}{\pi R^2} \cdot \frac{2R^3}{3} = \frac{4R}{3\pi}$$

$$\boxed{(\bar{x}, \bar{y}) = \left(0, \frac{4R}{3\pi}\right)}$$

---

### Example 3: Region Between Curves

Centroid of the region between y = x² and y = √x.

Intersection: x² = √x → x⁴ = x → x(x³−1) = 0 → x = 0, 1.

Area: $A = \int_0^1 (\sqrt{x} - x^2)\, dx = [2x^{3/2}/3 - x^3/3]_0^1 = 2/3 - 1/3 = 1/3$

$$\bar{x} = 3\int_0^1 x(\sqrt{x} - x^2)\, dx = 3\int_0^1 (x^{3/2} - x^3)\, dx = 3\left[\frac{2x^{5/2}}{5} - \frac{x^4}{4}\right]_0^1 = 3\left(\frac{2}{5} - \frac{1}{4}\right) = 3 \cdot \frac{3}{20} = \frac{9}{20}$$

$$\bar{y} = \frac{3}{2}\int_0^1 (x - x^4)\, dx = \frac{3}{2}\left[\frac{x^2}{2} - \frac{x^5}{5}\right]_0^1 = \frac{3}{2} \cdot \frac{3}{10} = \frac{9}{20}$$

$$\boxed{(\bar{x}, \bar{y}) = \left(\frac{9}{20}, \frac{9}{20}\right)}$$

Note: the centroid lies on the line y = x — consistent with the symmetry of the region.

---

## 3.6 Moments of Inertia (Second Moments)

The **moment of inertia** measures resistance to rotation:

$$I_x = \int y^2\, dm, \qquad I_y = \int x^2\, dm$$

For a plane region with density ρ:

$$I_x = \rho\int_a^b \frac{[f(x)]^3}{3}\, dx, \qquad I_y = \rho\int_a^b x^2 f(x)\, dx$$

---

## 3.7 Famous Centroids

| Shape | Centroid (x̄, ȳ) |
|---|---|
| Rectangle (w × h) | (w/2, h/2) |
| Triangle (base b, height h) | (b/3, h/3) from vertex |
| Semicircle (radius R) | (0, 4R/(3π)) |
| Quarter circle | (4R/(3π), 4R/(3π)) |
| Semicircular arc | (0, 2R/π) |
| Cone (height h) | h/4 from base |
| Hemisphere | 3R/8 from flat face |

---

## 3.8 Connection to Pappus' Theorem

> **Pappus' Theorem (Volume):** V = 2π × $\bar{d}$ × A
>
> **Pappus' Theorem (Surface Area):** S = 2π × $\bar{d}$ × L

This powerful connection lets you:
- Know the centroid → find the volume/surface area of revolution
- Know the volume → find the centroid

---

## Summary Table

| Quantity | Formula |
|---|---|
| Centroid x̄ (single curve) | (1/A) ∫ x·f(x) dx |
| Centroid ȳ (single curve) | (1/A) ∫ ½[f(x)]² dx |
| Between curves x̄ | (1/A) ∫ x[f−g] dx |
| Between curves ȳ | (1/2A) ∫ [f²−g²] dx |
| Moment of inertia Iy | ∫ x² dm |
| Pappus' theorem | V = 2πd̄·A |

---

## Quick Revision Questions

1. Find the centroid of the region bounded by y = x², y = 0, x = 2.

2. Show that the centroid of a semicircular region of radius R is at ȳ = 4R/(3π).

3. Use Pappus' theorem to find the volume of the solid generated by revolving a triangle with vertices (0,0), (3,0), (0,4) about the x-axis.

4. Find the centroid of the region between y = sin x and y = 0 from x = 0 to x = π.

5. Explain why the centroid of a symmetric region lies on the line of symmetry.

6. A semicircular wire (not region) of radius R has its centroid at ȳ = 2R/π. Use Pappus' theorem to find the surface area of a sphere.

---

[← Previous: Surface Area of Revolution](02-surface-area-of-revolution.md) | [Back to README](../README.md) | [Next: Work and Fluid Pressure →](04-work-and-fluid-pressure.md)
