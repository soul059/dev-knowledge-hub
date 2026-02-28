# Chapter 4: Applications of Multiple Integrals

[← Previous: Change of Order](03-change-of-order.md) | [Back to README](../README.md)

---

## 4.1 Overview

Multiple integrals are fundamental computational tools in science and engineering. This chapter covers the major applications: **area, volume, mass, center of mass, moments of inertia,** and **surface area** using double and triple integrals.

---

## 4.2 Area of a Plane Region

$$\boxed{A = \iint_R dA}$$

### Example 1: Area Between Curves

Find the area between y = x² and y = 2x.

Intersection: x² = 2x → x = 0, 2

$$A = \int_0^2\int_{x^2}^{2x} dy\, dx = \int_0^2 (2x - x^2)\, dx = \left[x^2 - \frac{x^3}{3}\right]_0^2 = 4 - \frac{8}{3} = \frac{4}{3}$$

### Example 2: Area in Polar — Cardioid

r = a(1 + cosθ):

$$A = \int_0^{2\pi}\int_0^{a(1+\cos\theta)} r\, dr\, d\theta = \int_0^{2\pi}\frac{a^2(1+\cos\theta)^2}{2}\, d\theta = \frac{3\pi a^2}{2}$$

---

## 4.3 Volume of a Solid

$$\boxed{V = \iint_R f(x,y)\, dA \quad \text{or} \quad V = \iiint_E dV}$$

### Example 3: Volume Under Paraboloid

Volume under z = 4 − x² − y² above the xy-plane:

Region: x² + y² ≤ 4 (where z ≥ 0)

$$V = \int_0^{2\pi}\int_0^2 (4 - r^2)\, r\, dr\, d\theta = 2\pi\int_0^2 (4r - r^3)\, dr = 2\pi\left[2r^2 - \frac{r^4}{4}\right]_0^2 = 2\pi(8 - 4) = 8\pi$$

### Example 4: Volume of Sphere (Spherical Coordinates)

$$V = \int_0^{2\pi}\int_0^{\pi}\int_0^a \rho^2\sin\phi\, d\rho\, d\phi\, d\theta$$

$$= \int_0^{2\pi} d\theta \int_0^{\pi}\sin\phi\, d\phi \int_0^a \rho^2\, d\rho = 2\pi \cdot 2 \cdot \frac{a^3}{3} = \frac{4\pi a^3}{3}$$

```
  z
  │     ╱──╲       Sphere x²+y²+z² = a²
  │    ╱    ╲
  │   │  ●   │     ρ: 0 → a
  │    ╲    ╱      φ: 0 → π  
  │     ╲──╱       θ: 0 → 2π
  ├──────────── y
  ╱
  x
```

---

## 4.4 Mass and Density

For a lamina (thin plate) with area density ρ(x,y):

$$\boxed{M = \iint_R \rho(x,y)\, dA}$$

For a solid with volume density ρ(x,y,z):

$$M = \iiint_E \rho(x,y,z)\, dV$$

### Example 5: Mass of Semicircular Plate

Density ρ(x,y) = y on the semicircle x²+y² ≤ a², y ≥ 0:

$$M = \int_0^{\pi}\int_0^a (r\sin\theta)\, r\, dr\, d\theta = \int_0^{\pi}\sin\theta\, d\theta \int_0^a r^2\, dr = 2 \cdot \frac{a^3}{3} = \frac{2a^3}{3}$$

---

## 4.5 Center of Mass

$$\boxed{\bar{x} = \frac{M_y}{M} = \frac{\iint_R x\,\rho\, dA}{\iint_R \rho\, dA}, \qquad \bar{y} = \frac{M_x}{M} = \frac{\iint_R y\,\rho\, dA}{\iint_R \rho\, dA}}$$

```
  MOMENTS:
  ════════
  Mₓ = ∬ y·ρ dA    (moment about x-axis → determines ȳ)
  M_y = ∬ x·ρ dA   (moment about y-axis → determines x̄)
  
  CENTER OF MASS:
  ● (x̄, ȳ) = (M_y/M, Mₓ/M)
```

### Example 6: Centroid of Quarter Circle

Uniform density (ρ = 1), region: x²+y² ≤ a², x ≥ 0, y ≥ 0.

$$M = \frac{\pi a^2}{4}$$

$$M_y = \int_0^{\pi/2}\int_0^a r\cos\theta\cdot r\, dr\, d\theta = \int_0^{\pi/2}\cos\theta\, d\theta\int_0^a r^2\, dr = 1\cdot\frac{a^3}{3} = \frac{a^3}{3}$$

$$\bar{x} = \frac{a^3/3}{\pi a^2/4} = \frac{4a}{3\pi} \approx 0.424a$$

By symmetry: $\bar{y} = \frac{4a}{3\pi}$

$$(\bar{x}, \bar{y}) = \left(\frac{4a}{3\pi}, \frac{4a}{3\pi}\right)$$

---

## 4.6 Moments of Inertia

$$\boxed{I_x = \iint_R y^2\,\rho\, dA, \qquad I_y = \iint_R x^2\,\rho\, dA, \qquad I_0 = \iint_R (x^2+y^2)\,\rho\, dA}$$

| Moment | Axis | Formula |
|---|---|---|
| Iₓ | About x-axis | ∬ y²ρ dA |
| I_y | About y-axis | ∬ x²ρ dA |
| I₀ | About origin (polar) | ∬ (x²+y²)ρ dA = Iₓ + I_y |

### Example 7: Moment of Inertia of Disk

Uniform disk of radius a, ρ = ρ₀:

$$I_0 = \int_0^{2\pi}\int_0^a r^2\cdot\rho_0\cdot r\, dr\, d\theta = 2\pi\rho_0\int_0^a r^3\, dr = 2\pi\rho_0\cdot\frac{a^4}{4} = \frac{\pi\rho_0 a^4}{2}$$

Since M = πρ₀a²:

$$I_0 = \frac{1}{2}Ma^2$$

By symmetry: $I_x = I_y = \frac{1}{4}Ma^2$ (since $I_0 = I_x + I_y$)

---

## 4.7 Surface Area

The area of a surface z = f(x,y) over region R:

$$\boxed{S = \iint_R \sqrt{1 + \left(\frac{\partial f}{\partial x}\right)^2 + \left(\frac{\partial f}{\partial y}\right)^2}\, dA}$$

```
  z      surface z = f(x,y)
  │     ╱╲  ╱╲
  │    ╱  ╲╱  ╲         dS (surface element)
  │   ╱────────╲         = √(1 + fₓ² + f_y²) · dA
  │  ╱          ╲
  ├───────────────── y
  ╱   ┌──────┐
  x   │  R   │  ← projection onto xy-plane
      └──────┘     dA = dx dy
```

### Example 8: Surface Area of Hemisphere

z = √(a² − x² − y²) over the disk x²+y² ≤ a²:

$$\frac{\partial z}{\partial x} = \frac{-x}{\sqrt{a^2-x^2-y^2}}, \qquad \frac{\partial z}{\partial y} = \frac{-y}{\sqrt{a^2-x^2-y^2}}$$

$$1 + z_x^2 + z_y^2 = 1 + \frac{x^2+y^2}{a^2-x^2-y^2} = \frac{a^2}{a^2-x^2-y^2}$$

$$S = \int_0^{2\pi}\int_0^a \frac{a}{\sqrt{a^2-r^2}}\, r\, dr\, d\theta = 2\pi a\left[-\sqrt{a^2-r^2}\right]_0^a = 2\pi a^2$$

Full sphere: S = 2 × 2πa² = **4πa²** ✓

---

## 4.8 Key Applications Table

| Application | 2D (Double Integral) | 3D (Triple Integral) |
|---|---|---|
| Area / Volume | A = ∬_R dA | V = ∭_E dV |
| Mass | M = ∬ ρ dA | M = ∭ ρ dV |
| Center of mass x̄ | (∬ xρ dA) / M | (∭ xρ dV) / M |
| Center of mass ȳ | (∬ yρ dA) / M | (∭ yρ dV) / M |
| Center of mass z̄ | — | (∭ zρ dV) / M |
| Moment Iₓ | ∬ y²ρ dA | ∭ (y²+z²)ρ dV |
| Moment I_y | ∬ x²ρ dA | ∭ (x²+z²)ρ dV |
| Surface area | ∬ √(1+fₓ²+f_y²) dA | — |

---

## 4.9 Famous Results via Multiple Integrals

| Result | Value | Method |
|---|---|---|
| Area of ellipse x²/a²+y²/b²=1 | πab | Polar/Jacobian |
| Volume of sphere radius a | 4πa³/3 | Spherical coords |
| Volume of cone h, radius R | πR²h/3 | Cylindrical coords |
| Volume of ellipsoid x²/a²+y²/b²+z²/c²=1 | 4πabc/3 | Scaled spherical |
| Surface area of sphere | 4πa² | Surface integral |
| Moment of inertia, solid ball | (2/5)Ma² | Spherical coords |
| Moment of inertia, solid cylinder | (1/2)Ma² | Cylindrical coords |
| Gaussian integral ∫e^{−x²}dx | √π | Double integral trick |

---

## 4.10 The Gaussian Integral via Double Integrals

The most famous application: proving $\int_{-\infty}^{\infty} e^{-x^2}\, dx = \sqrt{\pi}$.

Let $I = \int_{-\infty}^{\infty} e^{-x^2}\, dx$. Then:

$$I^2 = \int_{-\infty}^{\infty} e^{-x^2}\, dx \int_{-\infty}^{\infty} e^{-y^2}\, dy = \iint_{\mathbb{R}^2} e^{-(x^2+y^2)}\, dA$$

Convert to polar:

$$I^2 = \int_0^{2\pi}\int_0^{\infty} e^{-r^2}\, r\, dr\, d\theta = 2\pi\left[-\frac{e^{-r^2}}{2}\right]_0^{\infty} = 2\pi\cdot\frac{1}{2} = \pi$$

$$\boxed{I = \sqrt{\pi}}$$

```
  This is one of the most elegant proofs in mathematics:
  
  1D integral (impossible directly)
       │
       ▼
  Square it: I² = ∬ e^{-(x²+y²)} dA
       │
       ▼
  Convert to polar (x²+y²=r²)
       │
       ▼
  Trivial integral: ∫ re^{-r²} dr = -½e^{-r²}
       │
       ▼
  I² = π  →  I = √π  ✓
```

---

## Summary Table

| Application | Formula | Key Idea |
|---|---|---|
| Area | ∬_R dA | f = 1 |
| Volume (double) | ∬_R f(x,y) dA | Height × base area |
| Volume (triple) | ∭_E dV | Integrate 1 over solid |
| Mass | ∬ ρ dA or ∭ ρ dV | Density × element |
| Centroid | x̄ = ∬ x dA / ∬ dA | Uniform density |
| Center of mass | x̄ = ∬ xρ dA / M | Variable density |
| Moment of inertia | I = ∬ r²ρ dA | r = distance to axis |
| Surface area | ∬ √(1+fₓ²+f_y²) dA | Tilted area element |
| Gaussian integral | ∬ e^{−(x²+y²)} dA = π | Polar trick |

---

## Quick Revision Questions

1. Find the volume enclosed by the paraboloid z = x²+y² and the plane z = 4.

2. Find the mass of a circular plate of radius a with density ρ(x,y) = x²+y².

3. Compute the center of mass of the triangle with vertices (0,0), (1,0), (0,1) with uniform density.

4. Find the moment of inertia I₀ of a uniform thin ring (annulus) with inner radius a and outer radius b.

5. Compute the surface area of the paraboloid z = x²+y² for 0 ≤ z ≤ 1.

6. Derive the volume of a cone of height h and base radius R using cylindrical coordinates.

---

[← Previous: Change of Order](03-change-of-order.md) | [Back to README](../README.md)
