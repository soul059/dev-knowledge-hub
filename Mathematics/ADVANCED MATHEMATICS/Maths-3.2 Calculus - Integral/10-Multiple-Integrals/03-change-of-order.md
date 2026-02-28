# Chapter 3: Change of Order & Coordinate Transformations

[← Previous: Iterated Integrals](02-iterated-integrals.md) | [Back to README](../README.md) | [Next: Applications →](04-applications.md)

---

## 3.1 Overview

Many double integrals become dramatically simpler via two techniques:
1. **Changing the order of integration** (dy dx ↔ dx dy)
2. **Transforming to polar, cylindrical, or spherical coordinates**

---

## 3.2 Changing Order of Integration

### Why Change Order?

| Reason | Example |
|---|---|
| Integrand not integrable in given order | e^{y²} has no elementary antiderivative in y |
| Simpler limits in new order | Complicated bounds become straight lines |
| Computational efficiency | Inner integral evaluates more easily |

### Procedure

```
  STEP-BY-STEP: CHANGE ORDER OF INTEGRATION
  ═══════════════════════════════════════════
  
  1. Identify current limits
     ∫ₐᵇ ∫_{g₁(x)}^{g₂(x)} f(x,y) dy dx
  
  2. SKETCH the region R
     (This step is mandatory!)
  
  3. Express as Type II
     - y ranges: c ≤ y ≤ d  (find min, max of y)
     - For each y: h₁(y) ≤ x ≤ h₂(y)
  
  4. Rewrite:
     ∫_c^d ∫_{h₁(y)}^{h₂(y)} f(x,y) dx dy
```

---

### Example 1: Classic Change of Order

$$\int_0^4 \int_{\sqrt{x}}^{2} \frac{1}{1+y^3}\, dy\, dx$$

**Current limits:** 0 ≤ x ≤ 4, √x ≤ y ≤ 2

**Sketch the region:**

```
  y
  2 ┌──────────●
    │         ╱│
    │       ╱  │
    │     ╱    │  y = √x  →  x = y²
    │   ╱  R   │
    │ ╱        │
  0 ●──────────┘ 
    0          4
```

**New limits (dx dy):** 0 ≤ y ≤ 2, 0 ≤ x ≤ y²

$$\int_0^2 \int_0^{y^2} \frac{1}{1+y^3}\, dx\, dy = \int_0^2 \frac{y^2}{1+y^3}\, dy = \left[\frac{1}{3}\ln(1+y^3)\right]_0^2 = \frac{\ln 9}{3}$$

---

### Example 2: Splitting Required

$$\int_0^2 \int_0^{x} f(x,y)\, dy\, dx + \int_2^4 \int_0^{4-x} f(x,y)\, dy\, dx$$

```
  y
  2 │    ●
    │   ╱ ╲
    │  ╱   ╲     Triangular region
    │ ╱  R  ╲
    │╱       ╲
  0 ●─────────● 
    0    2    4
```

**Single integral (dx dy):** 0 ≤ y ≤ 2, y ≤ x ≤ 4−y

$$\int_0^2 \int_y^{4-y} f(x,y)\, dx\, dy$$

Two integrals became one!

---

## 3.3 Polar Coordinates

For regions with circular symmetry, convert to **polar coordinates**:

$$x = r\cos\theta, \qquad y = r\sin\theta, \qquad dA = r\, dr\, d\theta$$

> **Critical:** The Jacobian factor **r** must be included:
>
> $$\boxed{\iint_R f(x,y)\, dA = \int_\alpha^\beta \int_{r_1(\theta)}^{r_2(\theta)} f(r\cos\theta, r\sin\theta)\, r\, dr\, d\theta}$$

```
  y                              r
  │    ╱ θ₂                      │
  │   ╱                          │  r₂(θ)
  │  ╱ ╱────╲  r₂(θ)            │ ╱────────╲
  │ ╱ ╱  R   ╲                  │╱    R     ╲
  │╱ ╱────────╲ r₁(θ)           │╲──────────╱ r₁(θ)
  │ ╱ θ₁                        │
  ├──────────── x                ├──────────── θ
                                 α            β
```

### Key Substitutions

| Cartesian | Polar |
|---|---|
| x² + y² | r² |
| x² + y² = a² | r = a |
| x | r cos θ |
| y | r sin θ |
| dx dy | r dr dθ |

---

### Example 3: Circle Region

$$\iint_{x^2+y^2 \leq a^2} (x^2 + y^2)\, dA$$

In polar: r ranges from 0 to a, θ from 0 to 2π:

$$\int_0^{2\pi}\int_0^a r^2 \cdot r\, dr\, d\theta = \int_0^{2\pi}\left[\frac{r^4}{4}\right]_0^a d\theta = \int_0^{2\pi}\frac{a^4}{4}\, d\theta = \frac{\pi a^4}{2}$$

---

### Example 4: Quarter Circle

$$\int_0^a\int_0^{\sqrt{a^2-x^2}} \sqrt{x^2+y^2}\, dy\, dx$$

Region: first-quadrant quarter of circle x²+y² = a².

$$= \int_0^{\pi/2}\int_0^a r \cdot r\, dr\, d\theta = \int_0^{\pi/2}\frac{a^3}{3}\, d\theta = \frac{\pi a^3}{6}$$

---

### Example 5: Semi-Annular Region

$$\iint_R \frac{dA}{x^2+y^2}, \qquad R: 1 \leq x^2+y^2 \leq 4, \; y \geq 0$$

```
  y
  │    ╱────╲  r=2
  │  ╱╱ R  ╲╲
  │╱╱───────╲╲─ r=1
  ├────────────── x
```

$$\int_0^{\pi}\int_1^2 \frac{1}{r^2}\cdot r\, dr\, d\theta = \int_0^{\pi}\int_1^2 \frac{1}{r}\, dr\, d\theta = \int_0^{\pi} \ln 2\, d\theta = \pi\ln 2$$

---

## 3.4 General Change of Variables

For a transformation x = x(u,v), y = y(u,v):

$$\iint_R f(x,y)\, dx\, dy = \iint_{R'} f(x(u,v), y(u,v))\, |J|\, du\, dv$$

where the **Jacobian** is:

$$J = \frac{\partial(x,y)}{\partial(u,v)} = \begin{vmatrix} \frac{\partial x}{\partial u} & \frac{\partial x}{\partial v} \\ \frac{\partial y}{\partial u} & \frac{\partial y}{\partial v} \end{vmatrix}$$

### Common Jacobians

| Transformation | Jacobian |J| |
|---|---|
| Polar: x=rcosθ, y=rsinθ | r |
| Scaling: x=au, y=bv | ab |
| Rotation: x=ucosα−vsinα, y=usinα+vcosα | 1 |
| Elliptical: x=arcosθ, y=brsinθ | abr |

### Example 6: Elliptical Region

$$\iint_{x^2/a^2 + y^2/b^2 \leq 1} (1)\, dA$$

Let x = ar cosθ, y = br sinθ, then |J| = abr:

$$\int_0^{2\pi}\int_0^1 abr\, dr\, d\theta = ab\int_0^{2\pi}\frac{1}{2}\, d\theta = \pi ab$$

confirms the area of an ellipse.

---

## 3.5 Triple Integrals (Preview)

For functions of three variables:

$$\iiint_V f(x,y,z)\, dV$$

### Cylindrical Coordinates

$$x = r\cos\theta, \quad y = r\sin\theta, \quad z = z, \quad dV = r\, dr\, d\theta\, dz$$

```
  z
  │    ┌───┐
  │    │   │  height z
  │    │   │
  ├────┘   └──── y
  ╱  r, θ
  x
```

### Spherical Coordinates

$$x = \rho\sin\phi\cos\theta, \quad y = \rho\sin\phi\sin\theta, \quad z = \rho\cos\phi$$

$$dV = \rho^2\sin\phi\, d\rho\, d\phi\, d\theta$$

```
  z
  │   ╱ P(ρ,φ,θ)
  │  ╱
  │ ╱ φ (polar angle from z-axis)
  │╱───────── y
  ╱ θ (azimuth)
  x
  
  ρ = distance from origin
  φ = angle from positive z-axis (0 ≤ φ ≤ π)
  θ = angle in xy-plane (0 ≤ θ ≤ 2π)
```

---

## 3.6 When to Use Which Coordinates

```
  COORDINATE SELECTION GUIDE
  ══════════════════════════
  
  Region involves circles,       ──→  POLAR (2D) or
  cylinders, x²+y²?                  CYLINDRICAL (3D)
  
  Region involves spheres,       ──→  SPHERICAL
  cones, ρ = const?
  
  Integrand has x²+y²?           ──→  POLAR
  
  Integrand has x²+y²+z²?        ──→  SPHERICAL
  
  Elliptical boundary?           ──→  ELLIPTICAL
  x²/a² + y²/b² = 1                  (x=arcosθ, y=brsinθ)
  
  Rectangular/no symmetry?       ──→  CARTESIAN
```

---

## Summary Table

| Technique | When to Use | Key Formula |
|---|---|---|
| Change order | Inner integral not evaluable | Sketch region, re-derive limits |
| Polar | Circular symmetry | dA = r dr dθ |
| Cylindrical | Cylindrical symmetry (3D) | dV = r dr dθ dz |
| Spherical | Spherical symmetry (3D) | dV = ρ²sinφ dρ dφ dθ |
| General substitution | Non-standard regions | dA → |J| du dv |
| Jacobian | All transformations | J = ∂(x,y)/∂(u,v) |

---

## Quick Revision Questions

1. Change the order of integration: ∫₀¹ ∫ₓ¹ f(x,y) dy dx.

2. Convert ∬_R (x²+y²) dA to polar, where R is the disk x²+y² ≤ 9.

3. Compute the Jacobian for x = u² − v², y = 2uv.

4. Evaluate ∫₀² ∫₀^{√(4−x²)} e^{−(x²+y²)} dy dx using polar coordinates.

5. Find the volume of a sphere of radius a using spherical coordinates.

6. Set up ∬_R f dA in both orders for the region bounded by y = x and y = x³ (first quadrant).

---

[← Previous: Iterated Integrals](02-iterated-integrals.md) | [Back to README](../README.md) | [Next: Applications →](04-applications.md)
