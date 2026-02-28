# Chapter 1: Arc Length

[← Previous: Volume of Revolution](../07-Applications-Volume/04-volume-of-revolution.md) | [Back to README](../README.md) | [Next: Surface Area of Revolution →](02-surface-area-of-revolution.md)

---

## 1.1 Overview

How long is a curved line? Unlike straight lines, measuring the length of a curve requires calculus. We approximate the curve with tiny straight segments and sum their lengths via integration.

---

## 1.2 Arc Length in Cartesian Form

> **Formula:** The length of y = f(x) from x = a to x = b:
>
> $$L = \int_a^b \sqrt{1 + \left(\frac{dy}{dx}\right)^2}\, dx$$

### Derivation

```
  A small piece of the curve:
  
  │         ╱(x+dx, y+dy)
  │        ╱
  │       ╱ ds
  │      ╱
  │     ╱───── dy
  │    (x,y)
  │     ├─── dx
  │
  
  By Pythagoras: ds² = dx² + dy²
  
  ds = √(dx² + dy²) = √(1 + (dy/dx)²) dx
  
  L = ∫ ds = ∫ₐᵇ √(1 + (dy/dx)²) dx
```

### Alternative: Integrate w.r.t. y

If x = g(y):

$$L = \int_c^d \sqrt{1 + \left(\frac{dx}{dy}\right)^2}\, dy$$

---

## 1.3 Arc Length in Parametric Form

For x = x(t), y = y(t), from t₁ to t₂:

$$L = \int_{t_1}^{t_2} \sqrt{\left(\frac{dx}{dt}\right)^2 + \left(\frac{dy}{dt}\right)^2}\, dt$$

This is the most general form: ds² = (dx)² + (dy)² = [(dx/dt)² + (dy/dt)²] dt².

---

## 1.4 Arc Length in Polar Form

For r = f(θ), from θ = α to θ = β:

$$L = \int_{\alpha}^{\beta} \sqrt{r^2 + \left(\frac{dr}{d\theta}\right)^2}\, d\theta$$

**Derivation:** In polar, x = r cos θ, y = r sin θ. Computing dx/dθ and dy/dθ:
$$ds^2 = (dx)^2 + (dy)^2 = (r^2 + r'^2)\, d\theta^2$$

---

## 1.5 Worked Examples

### Example 1: Parabola

Find the length of y = x² from x = 0 to x = 1.

dy/dx = 2x, so:

$$L = \int_0^1 \sqrt{1 + 4x^2}\, dx$$

Using the formula $\int \sqrt{1+u^2}\, du = \frac{u\sqrt{1+u^2}}{2} + \frac{\ln(u + \sqrt{1+u^2})}{2}$ with u = 2x:

$$L = \frac{1}{2}\left[2x\sqrt{1+4x^2} + \ln(2x + \sqrt{1+4x^2})\right]_0^1$$

$$= \frac{1}{2}\left[2\sqrt{5} + \ln(2 + \sqrt{5})\right] \approx \frac{1}{2}(4.472 + 1.444) \approx 2.958$$

---

### Example 2: Catenary

The catenary y = a cosh(x/a) from x = 0 to x = b.

dy/dx = sinh(x/a), so:

$$L = \int_0^b \sqrt{1 + \sinh^2(x/a)}\, dx = \int_0^b \cosh(x/a)\, dx$$

(Using the identity 1 + sinh²u = cosh²u)

$$= [a\sinh(x/a)]_0^b = a\sinh(b/a)$$

> The catenary is one of the rare curves whose arc length formula simplifies beautifully!

---

### Example 3: Circle (Parametric)

x = R cos t, y = R sin t, 0 ≤ t ≤ 2π.

$$dx/dt = -R\sin t, \quad dy/dt = R\cos t$$

$$L = \int_0^{2\pi} \sqrt{R^2\sin^2 t + R^2\cos^2 t}\, dt = \int_0^{2\pi} R\, dt = 2\pi R \quad✓$$

---

### Example 4: Cycloid

x = r(t − sin t), y = r(1 − cos t), 0 ≤ t ≤ 2π.

$$dx/dt = r(1 - \cos t), \quad dy/dt = r\sin t$$

$$ds = r\sqrt{(1-\cos t)^2 + \sin^2 t}\, dt = r\sqrt{2 - 2\cos t}\, dt = r\sqrt{4\sin^2(t/2)}\, dt = 2r|\sin(t/2)|\, dt$$

Since 0 ≤ t ≤ 2π, we have sin(t/2) ≥ 0:

$$L = \int_0^{2\pi} 2r\sin(t/2)\, dt = 2r[-2\cos(t/2)]_0^{2\pi} = 2r(2 + 2) = 8r$$

$$\boxed{L_{\text{cycloid}} = 8r}$$

---

### Example 5: Cardioid (Polar)

r = a(1 + cos θ), 0 ≤ θ ≤ 2π.

dr/dθ = −a sin θ. So:

$$L = \int_0^{2\pi} \sqrt{a^2(1+\cos\theta)^2 + a^2\sin^2\theta}\, d\theta = a\int_0^{2\pi}\sqrt{2 + 2\cos\theta}\, d\theta$$

$$= a\int_0^{2\pi} \sqrt{4\cos^2(\theta/2)}\, d\theta = 2a\int_0^{2\pi} |\cos(\theta/2)|\, d\theta$$

$$= 2a \cdot 2\int_0^{\pi} \cos(\theta/2)\, d\theta = 4a[2\sin(\theta/2)]_0^{\pi} = 8a$$

$$\boxed{L_{\text{cardioid}} = 8a}$$

---

## 1.6 Famous Arc Lengths

| Curve | Arc Length |
|---|---|
| Circle (radius R) | 2πR |
| Cycloid (one arch) | 8r |
| Cardioid r = a(1+cos θ) | 8a |
| Astroid x²/³ + y²/³ = a²/³ | 6a |
| Catenary y = a cosh(x/a), [0, b] | a sinh(b/a) |

---

## 1.7 Why Arc Length Integrals Are Hard

Most arc length integrals do NOT have closed-form solutions! The expression √(1 + f'²) rarely simplifies. Notable exceptions:
- **Catenary** (cosh): perfectly simplifies
- **Cycloid/Cardioid**: half-angle tricks
- **y = x^{3/2}**: power rule works

For most curves, numerical methods (Simpson's rule, etc.) are needed.

---

## Summary Table

| Form | Arc Length Formula |
|---|---|
| Cartesian (dx) | L = ∫ₐᵇ √(1 + (dy/dx)²) dx |
| Cartesian (dy) | L = ∫ᶜᵈ √(1 + (dx/dy)²) dy |
| Parametric | L = ∫_{t₁}^{t₂} √((dx/dt)² + (dy/dt)²) dt |
| Polar | L = ∫_α^β √(r² + (dr/dθ)²) dθ |
| Key element | ds = √(dx² + dy²) |

---

## Quick Revision Questions

1. Find the arc length of y = (2/3)x^{3/2} from x = 0 to x = 3.

2. Derive the arc length formula in polar form from the parametric formula.

3. Find the circumference of the astroid x = a cos³t, y = a sin³t.

4. Set up the arc length integral for y = ln(sec x), 0 ≤ x ≤ π/4, and show it simplifies nicely.

5. Why does the arc length of y = x² not have a nice closed form while y = cosh x does?

6. Find the arc length of the spiral r = eθ from θ = 0 to θ = 2π.

---

[← Previous: Volume of Revolution](../07-Applications-Volume/04-volume-of-revolution.md) | [Back to README](../README.md) | [Next: Surface Area of Revolution →](02-surface-area-of-revolution.md)
