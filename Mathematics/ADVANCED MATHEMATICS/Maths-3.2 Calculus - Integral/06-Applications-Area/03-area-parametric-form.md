# Chapter 3: Area in Parametric Form

[← Previous: Area Between Curves](02-area-between-curves.md) | [Back to README](../README.md) | [Next: Area in Polar Form →](04-area-polar-form.md)

---

## 3.1 Overview

When a curve is described parametrically by x = x(t), y = y(t), we can still find areas — but the integration variable changes from dx to dt using the chain rule.

---

## 3.2 The Parametric Area Formula

Starting from the standard formula A = ∫ₐᵇ y dx, we substitute:

$$x = x(t), \quad y = y(t), \quad dx = x'(t)\, dt$$

> **Parametric Area Formula:**
>
> $$A = \int_{t_1}^{t_2} y(t)\, x'(t)\, dt$$
>
> where t₁ and t₂ are the parameter values corresponding to x = a and x = b.

```
  Standard:          Parametric:
  A = ∫ₐᵇ y dx   →   A = ∫_{t₁}^{t₂} y(t) · x'(t) dt
       │                     │         │        │
       └── variable x        └── variable t
```

> **Important:** The sign of the area depends on the direction of traversal. If x decreases as t increases, x'(t) < 0, and we may need absolute value.

---

## 3.3 Closed Curves

For a **closed curve** traced counterclockwise as t goes from t₁ to t₂:

$$A = \left|\int_{t_1}^{t_2} y(t)\, x'(t)\, dt\right|$$

Or equivalently (by integration by parts):

$$A = \left|\int_{t_1}^{t_2} x(t)\, y'(t)\, dt\right|$$

### Symmetric Form (Shoelace Formula for Curves)

$$A = \frac{1}{2}\left|\int_{t_1}^{t_2} [x(t)\, y'(t) - y(t)\, x'(t)]\, dt\right|$$

```
  Closed curve (counterclockwise):
  
       ╱‾‾‾╲
      ╱     ╲
     ╱  Area  ╲
     ╲   A    ╱
      ╲     ╱
       ╲___╱
     t₁     t₂  (t₁ = t₂ for full traversal)
  
  A = ½|∮ (x dy − y dx)|
```

---

## 3.4 Worked Examples

### Example 1: Area of an Ellipse

Parametric equations: x = a cos t, y = b sin t, 0 ≤ t ≤ 2π.

As t goes from 0 to 2π, the ellipse is traced counterclockwise.

$$A = \left|\int_0^{2\pi} y(t) \cdot x'(t)\, dt\right| = \left|\int_0^{2\pi} b\sin t \cdot (-a\sin t)\, dt\right|$$

$$= ab\left|\int_0^{2\pi} -\sin^2 t\, dt\right| = ab \cdot \pi$$

$$\boxed{A = \pi ab}$$

**Verification:** When a = b = r, this gives πr² (area of a circle). ✓

---

### Example 2: Area Under a Cycloid Arch

A cycloid is given by: x = r(t − sin t), y = r(1 − cos t), 0 ≤ t ≤ 2π.

```
  y
  │      ╱‾╲
  │     ╱   ╲      One arch of cycloid
  │    ╱     ╲
  │   ╱       ╲
  │  ╱         ╲
  ├─╱───────────╲── x
  0              2πr
```

$$A = \int_0^{2\pi} y(t) \cdot x'(t)\, dt = \int_0^{2\pi} r(1 - \cos t) \cdot r(1 - \cos t)\, dt$$

$$= r^2 \int_0^{2\pi} (1 - \cos t)^2\, dt = r^2 \int_0^{2\pi} (1 - 2\cos t + \cos^2 t)\, dt$$

$$= r^2 \left[t - 2\sin t + \frac{t}{2} + \frac{\sin 2t}{4}\right]_0^{2\pi}$$

$$= r^2 \left(2\pi + \pi\right) = 3\pi r^2$$

$$\boxed{A_{\text{cycloid}} = 3\pi r^2}$$

> The area under one arch of a cycloid equals **3 times** the area of the generating circle!

---

### Example 3: Astroid

An astroid: x = a cos³t, y = a sin³t, 0 ≤ t ≤ 2π.

```
       │
    ╱──┼──╲
   ╱  ─┼─  ╲
  ─────┼─────    Astroid (4-cusped)
   ╲  ─┼─  ╱
    ╲──┼──╱
       │
```

By symmetry, total area = 4 × (area in first quadrant, t: 0 → π/2):

$$A = 4\left|\int_{\pi/2}^{0} a\sin^3 t \cdot (-3a\cos^2 t \sin t)\, dt\right|$$

$$= 12a^2 \int_0^{\pi/2} \sin^4 t \cos^2 t\, dt$$

Using the Beta function / Wallis formula:

$$\int_0^{\pi/2} \sin^4 t \cos^2 t\, dt = \frac{3 \cdot 1 \cdot 1}{6 \cdot 4 \cdot 2} \cdot \frac{\pi}{2} = \frac{3}{48} \cdot \frac{\pi}{2} = \frac{\pi}{32}$$

$$A = 12a^2 \cdot \frac{\pi}{32} = \frac{3\pi a^2}{8}$$

$$\boxed{A_{\text{astroid}} = \frac{3\pi a^2}{8}}$$

---

### Example 4: Cardioid (Parametric Form)

x = a(2cos t − cos 2t), y = a(2sin t − sin 2t), 0 ≤ t ≤ 2π.

Using the symmetric formula:

$$A = \frac{1}{2}\left|\int_0^{2\pi} (x\, y' - y\, x')\, dt\right|$$

After computation (expanding and using trig identities):

$$\boxed{A_{\text{cardioid}} = 6\pi a^2}$$

---

## 3.5 Important Parametric Areas

| Curve | Parametrization | Area |
|---|---|---|
| Circle | x = r cos t, y = r sin t | πr² |
| Ellipse | x = a cos t, y = b sin t | πab |
| Cycloid (one arch) | x = r(t − sin t), y = r(1 − cos t) | 3πr² |
| Astroid | x = a cos³t, y = a sin³t | 3πa²/8 |

---

## 3.6 Step-by-Step Procedure

```
  PARAMETRIC AREA CALCULATION
  ═══════════════════════════
  1. Identify x(t), y(t) and the range [t₁, t₂]
  2. Compute x'(t) = dx/dt
  3. Set up A = ∫_{t₁}^{t₂} y(t) · x'(t) dt
  4. Watch the sign! (direction of traversal)
  5. For closed curves, use |A|
  6. Exploit symmetry when possible
     (e.g., area = 4 × first quadrant)
```

---

## Summary Table

| Concept | Formula |
|---|---|
| Basic parametric area | A = ∫_{t₁}^{t₂} y(t) x'(t) dt |
| Alternative form | A = −∫_{t₁}^{t₂} x(t) y'(t) dt |
| Symmetric form | A = ½\|∫(x y' − y x') dt\| |
| Sign convention | CCW traversal → positive area |
| Symmetry | Use if curve is symmetric about axes |

---

## Quick Revision Questions

1. Derive the parametric area formula from A = ∫ y dx.

2. Find the area of the ellipse x = 3cos t, y = 2sin t.

3. Show that the area under one arch of a cycloid is 3πr².

4. Find the area enclosed by x = cos t, y = sin 2t for 0 ≤ t ≤ π. (Hint: be careful about the shape.)

5. For the astroid x = a cos³t, y = a sin³t, verify the area formula using the Wallis integral.

6. Explain why the sign of x'(t) matters in parametric area calculations.

---

[← Previous: Area Between Curves](02-area-between-curves.md) | [Back to README](../README.md) | [Next: Area in Polar Form →](04-area-polar-form.md)
