# Chapter 4: Work and Fluid Pressure

[← Previous: Center of Mass](03-center-of-mass.md) | [Back to README](../README.md) | [Next: Type I Improper Integrals →](../09-Improper-Integrals/01-type-I-infinite-limits.md)

---

## 4.1 Overview

Two important physics applications of integration: (1) **work** done by a variable force, and (2) **hydrostatic pressure/force** on submerged surfaces. Both require summing infinitely many small contributions.

---

## 4.2 Work Done by a Variable Force

> **Definition:** Work = Force × Distance. When force varies:
>
> $$W = \int_a^b F(x)\, dx$$

```
  Work with CONSTANT force:     Work with VARIABLE force:
  
  F│▓▓▓▓▓▓▓▓▓▓▓│               │     ╱────╲
   │▓▓▓▓▓▓▓▓▓▓▓│               │    ╱██████╲
   │▓▓▓▓▓▓▓▓▓▓▓│               │   ╱████████╲
   ├───────────── x              ├──╱██████████╲── x
   a            b                a              b
   
   W = F · (b−a)                W = ∫ₐᵇ F(x) dx
```

---

## 4.3 Common Work Problems

### Spring Work (Hooke's Law)

Force: F(x) = kx (k = spring constant, x = displacement from natural length).

$$W = \int_0^d kx\, dx = \frac{1}{2}kd^2$$

```
  Natural length          Stretched by d
  ├─────────────┤         ├─────────────────┤
  |/\/\/\/\/\/\/|    →    |/\/\/\/\/\/\/\/\/\/|
                          ├────── d ──────────┤
                          F(x) = kx
```

---

### Pumping Liquids

**Work to pump liquid out of a tank:**

$$W = \int_a^b \rho g \cdot A(y) \cdot (h - y)\, dy$$

where:
- ρg = weight density of liquid
- A(y) = cross-sectional area at height y
- (h − y) = distance each slice must be lifted to the top h

```
  ┌──────────┐ ← h (top, pump to here)
  │          │
  │  liquid  │ ← slice at height y
  │██████████│   must travel (h − y)
  │██████████│
  └──────────┘ ← 0 (bottom)
```

---

### Gravitational Work

Work to move mass m from distance r₁ to r₂ from Earth's center:

$$W = GMm\int_{r_1}^{r_2} \frac{1}{r^2}\, dr = GMm\left(\frac{1}{r_1} - \frac{1}{r_2}\right)$$

---

## 4.4 Worked Examples — Work

### Example 1: Spring

A spring has natural length 10 cm. A force of 40 N stretches it to 15 cm. Find work to stretch from 15 cm to 18 cm.

**Step 1:** Find k. F = kx: 40 = k(0.05), so k = 800 N/m.

**Step 2:** Work from x = 0.05 to x = 0.08:

$$W = \int_{0.05}^{0.08} 800x\, dx = 800\left[\frac{x^2}{2}\right]_{0.05}^{0.08} = 400(0.0064 - 0.0025) = 1.56 \text{ J}$$

---

### Example 2: Pumping Water from a Cylindrical Tank

Cylindrical tank: radius 3 m, height 5 m, full of water (ρ = 1000 kg/m³). Pump water over the top.

At height y: A(y) = π(3)² = 9π. Distance = 5 − y.

$$W = \int_0^5 1000 \cdot 9.8 \cdot 9\pi \cdot (5 - y)\, dy = 88200\pi\int_0^5 (5-y)\, dy$$

$$= 88200\pi\left[5y - \frac{y^2}{2}\right]_0^5 = 88200\pi \cdot \frac{25}{2} = 1{,}102{,}500\pi \approx 3{,}463{,}606 \text{ J}$$

---

### Example 3: Conical Tank

Inverted cone: radius 2 m at top, height 4 m, full of water. Pump to top.

At height y: radius at y is r(y) = (2/4)y = y/2. Area: A(y) = π(y/2)² = πy²/4.

$$W = \int_0^4 9800 \cdot \frac{\pi y^2}{4} \cdot (4-y)\, dy = 2450\pi\int_0^4 (4y^2 - y^3)\, dy$$

$$= 2450\pi\left[\frac{4y^3}{3} - \frac{y^4}{4}\right]_0^4 = 2450\pi\left(\frac{256}{3} - 64\right) = 2450\pi \cdot \frac{64}{3}$$

$$\approx 164{,}267 \text{ J}$$

---

## 4.5 Hydrostatic Pressure and Force

### Pascal's Law

Pressure at depth h below the surface:

$$P = \rho g h$$

where ρ is the fluid density, g is gravitational acceleration.

### Hydrostatic Force on a Submerged Plate

> $$F = \int_a^b \rho g \cdot h(y) \cdot w(y)\, dy$$
>
> where h(y) = depth of the strip at position y, w(y) = width of the plate at y.

```
  Surface ═══════════════════
         │          │
    h(y) │   ┌──────┐←w(y)
         ↓   │██████│  ← horizontal strip at depth h
              │██████│
              │██████│
              └──────┘ ← submerged plate
  
  dF = P · dA = ρgh(y) · w(y) · dy
```

---

## 4.6 Worked Examples — Fluid Pressure

### Example 4: Rectangular Plate

A rectangular plate 3 m wide, 2 m tall, submerged vertically with top edge 1 m below surface. Water (ρ = 1000 kg/m³).

Set y = 0 at top edge, y = 2 at bottom. Depth at y: h = 1 + y. Width: w = 3.

$$F = \int_0^2 1000 \cdot 9.8 \cdot (1+y) \cdot 3\, dy = 29400\int_0^2 (1+y)\, dy$$

$$= 29400\left[y + \frac{y^2}{2}\right]_0^2 = 29400 \cdot 4 = 117{,}600 \text{ N}$$

---

### Example 5: Triangular Plate

Equilateral triangle with side 2 m, vertex down, top edge at the water surface.

```
  Surface ════════════
          ├────2m────┤
          ╲          ╱
           ╲        ╱
            ╲      ╱
             ╲    ╱
              ╲  ╱
               ╲╱ ← vertex at depth h = √3 m
```

Set y = 0 at surface (top edge), y = √3 at vertex. Width at depth y: w(y) = 2(1 − y/√3).

$$F = \int_0^{\sqrt{3}} 9800 \cdot y \cdot 2\left(1 - \frac{y}{\sqrt{3}}\right)\, dy = 19600\int_0^{\sqrt{3}} \left(y - \frac{y^2}{\sqrt{3}}\right)\, dy$$

$$= 19600\left[\frac{y^2}{2} - \frac{y^3}{3\sqrt{3}}\right]_0^{\sqrt{3}} = 19600\left(\frac{3}{2} - \frac{3\sqrt{3}}{3\sqrt{3}}\right) = 19600\left(\frac{3}{2} - 1\right)$$

$$= 19600 \cdot \frac{1}{2} = 9{,}800 \text{ N}$$

---

## 4.7 Application Summary

| Application | Key Formula |
|---|---|
| Work (variable force) | W = ∫ F(x) dx |
| Spring work | W = ½kx² |
| Pumping liquid | W = ∫ ρg·A(y)·[distance] dy |
| Pressure at depth h | P = ρgh |
| Hydrostatic force | F = ∫ ρg·h(y)·w(y) dy |
| Gravitational work | W = GMm(1/r₁ − 1/r₂) |

---

## Summary Table

| Concept | Formula | Key Variable |
|---|---|---|
| Work | W = ∫ₐᵇ F(x) dx | F = variable force |
| Hooke's law | F = kx | k = spring constant |
| Pumping | W = ∫ ρgA(y)(h−y) dy | h−y = lift distance |
| Pressure | P = ρgh | h = depth |
| Hydrostatic force | F = ∫ ρg·h·w(y) dy | w = plate width |

---

## Quick Revision Questions

1. A spring requires 100 J to stretch from natural length to 0.5 m. Find the spring constant k and the work to stretch from 0.5 m to 1 m.

2. A hemispherical bowl of radius 3 m is full of water. Set up the integral for the work needed to pump all water to the rim.

3. A trapezoidal dam face has top width 10 m, bottom width 6 m, and height 4 m. If water reaches the top, find the hydrostatic force on the dam.

4. Explain why pumping work depends on the shape of the tank even if two tanks hold the same volume of water.

5. A chain weighing 5 kg/m hangs from a 20 m tall building. Find the work to pull the entire chain to the top.

6. A circular plate of radius 2 m is submerged vertically with its center 5 m below the surface. Set up the integral for the hydrostatic force.

---

[← Previous: Center of Mass](03-center-of-mass.md) | [Back to README](../README.md) | [Next: Type I Improper Integrals →](../09-Improper-Integrals/01-type-I-infinite-limits.md)
