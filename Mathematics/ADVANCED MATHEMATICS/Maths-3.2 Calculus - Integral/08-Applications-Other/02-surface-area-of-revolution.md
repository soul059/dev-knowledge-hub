# Chapter 2: Surface Area of Revolution

[← Previous: Arc Length](01-arc-length.md) | [Back to README](../README.md) | [Next: Center of Mass →](03-center-of-mass.md)

---

## 2.1 Overview

When a curve is revolved about an axis, it sweeps out a **surface of revolution**. The surface area combines arc length (ds) with the circumference traced by each point on the curve.

---

## 2.2 The Surface Area Formula

> **Key Idea:** A small arc element ds, at distance r from the axis, sweeps out a band (frustum) of area:
>
> $$dS = 2\pi r\, ds$$

```
  Side view:              3D band (frustum):
  
  │    ╱ ds                   ╱══════╲
  │   ╱                      ║      ║
  │  ╱←─ r (distance         ║      ║
  │ ╱     to axis)            ╲══════╱
  AXIS                     circumference = 2πr
                            width = ds
                            area dS = 2πr · ds
```

---

## 2.3 Formulas for Different Representations

### Revolution About x-axis

**Cartesian:**
$$S = 2\pi\int_a^b y\sqrt{1 + \left(\frac{dy}{dx}\right)^2}\, dx$$

**Parametric:**
$$S = 2\pi\int_{t_1}^{t_2} y(t)\sqrt{\left(\frac{dx}{dt}\right)^2 + \left(\frac{dy}{dt}\right)^2}\, dt$$

**Polar:**
$$S = 2\pi\int_{\alpha}^{\beta} r\sin\theta \sqrt{r^2 + \left(\frac{dr}{d\theta}\right)^2}\, d\theta$$

(Since y = r sin θ in polar coordinates.)

### Revolution About y-axis

Replace the distance r (= y for x-axis) with the distance to the y-axis (= x):

$$S = 2\pi\int_a^b x\sqrt{1 + \left(\frac{dy}{dx}\right)^2}\, dx$$

---

## 2.4 General Formula Summary

$$\boxed{S = 2\pi\int (\text{radius})\, ds}$$

where:
- **radius** = perpendicular distance from curve to axis
- **ds** = arc length element = √(1 + (dy/dx)²) dx, etc.

```
  dS = 2π × (distance to axis) × ds
  ═══════════════════════════════════
  
  About x-axis: distance = y (or r sin θ)
  About y-axis: distance = x (or r cos θ)
  About y = k:  distance = |y − k|
  About x = k:  distance = |x − k|
```

---

## 2.5 Worked Examples

### Example 1: Surface Area of a Sphere

Revolve y = √(R² − x²) about x-axis from −R to R.

$$dy/dx = \frac{-x}{\sqrt{R^2 - x^2}}$$

$$\sqrt{1+(dy/dx)^2} = \sqrt{1 + \frac{x^2}{R^2-x^2}} = \sqrt{\frac{R^2}{R^2-x^2}} = \frac{R}{\sqrt{R^2-x^2}}$$

$$S = 2\pi\int_{-R}^{R} \sqrt{R^2-x^2} \cdot \frac{R}{\sqrt{R^2-x^2}}\, dx = 2\pi\int_{-R}^R R\, dx = 2\pi R \cdot 2R$$

$$\boxed{S_{\text{sphere}} = 4\pi R^2}$$

> Beautiful result: the integrand simplifies completely to a constant!

---

### Example 2: Lateral Surface of a Cone

Revolve y = (r/h)x about x-axis from 0 to h.

$$dy/dx = r/h, \quad \sqrt{1+(r/h)^2} = \frac{\sqrt{h^2+r^2}}{h} = \frac{\ell}{h}$$

where ℓ = √(h² + r²) is the slant height.

$$S = 2\pi\int_0^h \frac{r}{h}x \cdot \frac{\ell}{h}\, dx = \frac{2\pi r\ell}{h^2}\left[\frac{x^2}{2}\right]_0^h = \frac{2\pi r\ell}{h^2} \cdot \frac{h^2}{2}$$

$$\boxed{S_{\text{cone}} = \pi r\ell}$$

---

### Example 3: Surface of a Torus

Revolve the circle (x − R)² + y² = r² about the y-axis. Parametrize: x = R + r cos t, y = r sin t, 0 ≤ t ≤ 2π.

Distance to y-axis = x = R + r cos t.

$$ds = \sqrt{(-r\sin t)^2 + (r\cos t)^2}\, dt = r\, dt$$

$$S = 2\pi\int_0^{2\pi} (R + r\cos t) \cdot r\, dt = 2\pi r[Rt + r\sin t]_0^{2\pi} = 2\pi r \cdot 2\pi R$$

$$\boxed{S_{\text{torus}} = 4\pi^2 Rr}$$

---

### Example 4: Paraboloid

y = x² from x = 0 to x = 1, revolved about the y-axis.

$$S = 2\pi\int_0^1 x\sqrt{1 + 4x^2}\, dx$$

Let u = 1 + 4x², du = 8x dx:

$$= 2\pi \cdot \frac{1}{8}\int_1^5 \sqrt{u}\, du = \frac{\pi}{4}\left[\frac{2u^{3/2}}{3}\right]_1^5 = \frac{\pi}{6}(5\sqrt{5} - 1)$$

---

## 2.6 Famous Surface Areas

| Surface | Area |
|---|---|
| Sphere (radius R) | 4πR² |
| Cone (lateral, radius r, slant ℓ) | πrℓ |
| Cylinder (lateral, radius r, height h) | 2πrh |
| Torus (radii R, r) | 4π²Rr |
| Gabriel's Horn (y=1/x, x≥1) | ∞ (infinite!) |

> **Paradox:** Gabriel's Horn has finite volume (π) but infinite surface area! You can fill it with paint, but you can't paint its surface.

---

## 2.7 Pappus' Theorem for Surface Area

> **Theorem:** The surface area of a surface of revolution = (arc length of curve) × (distance traveled by centroid of curve).
>
> $$S = 2\pi\bar{d} \cdot L$$

---

## Summary Table

| Rotation Axis | Surface Area Formula |
|---|---|
| x-axis (Cartesian) | S = 2π ∫ y √(1 + y'²) dx |
| y-axis (Cartesian) | S = 2π ∫ x √(1 + y'²) dx |
| x-axis (Parametric) | S = 2π ∫ y √(x'² + y'²) dt |
| Polar (about initial line) | S = 2π ∫ r sin θ √(r² + r'²) dθ |
| General principle | dS = 2π · (radius) · ds |

---

## Quick Revision Questions

1. Derive the surface area formula for a sphere of radius R.

2. Find the surface area when y = √x (0 ≤ x ≤ 1) is revolved about the x-axis.

3. Show that the lateral surface area of a cylinder (radius r, height h) is 2πrh using integration.

4. Explain Gabriel's Horn paradox: how can a solid have finite volume but infinite surface area?

5. Use Pappus' theorem to find the surface area of a torus (major radius R, minor radius r).

6. Set up the surface area integral for the cardioid r = 1 + cos θ revolved about the polar axis.

---

[← Previous: Arc Length](01-arc-length.md) | [Back to README](../README.md) | [Next: Center of Mass →](03-center-of-mass.md)
