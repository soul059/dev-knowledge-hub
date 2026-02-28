# 7.3 Mapping of Standard Regions

[← Previous: Linear Fractional Transformations](02-linear-fractional-transformations.md) | [Next: Applications →](04-applications.md)

---

## Chapter Overview

This chapter provides a **catalogue** of conformal maps between standard regions: half-planes, disks, strips, wedges, sectors, and slits. These building blocks are combined (via composition) to map any simply connected region to any other — as guaranteed by the Riemann Mapping Theorem.

---

## 1. Strategy: Composition of Elementary Maps

To map region $A$ to region $B$:

$$A \xrightarrow{f_1} \text{Half-plane} \xrightarrow{f_2} B$$

or chain through intermediaries:

$$A \xrightarrow{f_1} \text{UHP} \xrightarrow{\text{Cayley}} \text{Disk} \xrightarrow{f_3} B$$

---

## 2. Catalogue of Standard Conformal Maps

### 2.1 Half-Plane ↔ Disk

| Map | Direction |
|-----|-----------|
| $w = \frac{z-i}{z+i}$ | UHP → Unit disk |
| $z = i\frac{1+w}{1-w}$ | Unit disk → UHP |

```
Upper Half-Plane  ←→  Unit Disk

  Im                      ╭──╮
   ▲                     ╱    ╲
   │████████            │  ○   │
   │████████             ╲    ╱
   ┼──────► Re            ╰──╯

  Cayley: w = (z-i)/(z+i)
```

### 2.2 Sector ↔ Half-Plane

| Sector | Map | Image |
|--------|-----|-------|
| $0 < \arg z < \alpha$ | $w = z^{\pi/\alpha}$ | UHP |
| UHP | $w = z^{\alpha/\pi}$ | Sector $0 < \arg w < \alpha$ |

**Example:** First quadrant ($\alpha = \pi/2$) → UHP via $w = z^2$.

### 2.3 Strip ↔ Half-Plane

| Strip | Map | Image |
|-------|-----|-------|
| $0 < \text{Im}(z) < \pi$ | $w = e^z$ | UHP |
| $0 < \text{Im}(z) < h$ | $w = e^{\pi z/h}$ | UHP |
| $-\pi/2 < \text{Im}(z) < \pi/2$ | $w = e^z$ | Right half-plane |

```
Strip (0 < Im z < π) → UHP via w = eᶻ:

  z-plane:                    w-plane:
   Im z = π ──────────         Im
     │████████████████          ▲
     │████████████████          │█████████████
   Im z = 0 ──────────          ┼────────────► Re
```

### 2.4 Half-Plane ↔ Half-Plane

| Map | Effect |
|-----|--------|
| $w = az + b$ ($a > 0$, $b \in \mathbb{R}$) | UHP → UHP (scale + translate) |
| $w = -1/z$ | UHP → UHP |
| $w = \frac{az+b}{cz+d}$ ($a,b,c,d \in \mathbb{R}$, $ad-bc > 0$) | UHP → UHP |

### 2.5 Disk → Disk

$$w = e^{i\theta}\frac{z - a}{1 - \bar{a}z}, \quad |a| < 1$$

### 2.6 Exterior ↔ Disk/Half-Plane

| Map | Direction |
|-----|-----------|
| $w = 1/z$ | Exterior of unit disk → unit disk (reverses orientation) |
| $w = z + 1/z$ | Exterior of unit circle → $\mathbb{C} \setminus [-2, 2]$ (Joukowski) |

---

## 3. Mapping Polygonal Regions: Schwarz–Christoffel

### 3.1 The Schwarz–Christoffel Formula

Maps the upper half-plane to the interior of a polygon with vertices $w_1, \ldots, w_n$ and interior angles $\alpha_1 \pi, \ldots, \alpha_n \pi$:

$$\boxed{f(z) = A\int^z \prod_{k=1}^{n}(\zeta - x_k)^{\alpha_k - 1}\,d\zeta + B}$$

where $x_1 < x_2 < \cdots < x_n$ are the prevertices on the real axis.

```
Schwarz-Christoffel mapping:

  z-plane (UHP):                  w-plane (polygon):
    Im                              w₃
     ▲                             ╱  ╲
     │████████████████            ╱    ╲
     ┼──•──•──•──•───► Re     w₂•      •w₄
        x₁ x₂ x₃ x₄            │      │
                                 w₁────w₅
  Prevertices x_k on real axis map to vertices w_k
```

### 3.2 Examples

| Polygon | Interior Angles | S-C Map |
|---------|-----------------|---------|
| Half-strip | $\pi/2, \pi/2$ (one vertex at $\infty$) | $w = \sin^{-1}(z)$ |
| Equilateral triangle | $\pi/3, \pi/3, \pi/3$ | Involves $\int(\zeta^3-1)^{-2/3}\,d\zeta$ |
| Rectangle | $\pi/2, \pi/2, \pi/2, \pi/2$ | Involves elliptic integrals |

---

## 4. Design Example

### Example: Map the quarter-disk $\{|z| < 1, \text{Re}(z) > 0, \text{Im}(z) > 0\}$ to the UHP.

**Step 1.** Map first quadrant to UHP: $\zeta = z^2$.

Under this map, the quarter-disk $\{|z| < 1, 0 < \arg z < \pi/2\}$ becomes:
- $|z| < 1$ → $|\zeta| < 1$ (still inside unit circle, but only the upper half of it)
- First quadrant ($\arg z \in (0, \pi/2)$) → upper half-plane ($\arg \zeta \in (0, \pi)$)
- Image: upper half of the unit disk

**Step 2.** Map upper half-disk to UHP. First map disk to disk sending a convenient point to center, then compose appropriately.

A more direct approach: $\zeta = z^2$ maps the quarter-disk to the upper half-disk. Then:

$$w = \frac{\zeta - i|\zeta|}{\cdots}$$

This becomes involved. Instead, compose:
1. $z^2$: quarter-disk → upper half of disk
2. Cayley inverse $i(1+\zeta)/(1-\zeta)$: disk → UHP maps the full disk, but we can use Möbius cleverly.

The exact composite formula is complex, but the methodology is: **break into standard pieces, compose**.

---

## 5. Quick Reference: Map Lookup

| From → To | Map |
|-----------|-----|
| UHP → Disk | $(z-i)/(z+i)$ |
| Disk → UHP | $i(1+w)/(1-w)$ |
| 1st quadrant → UHP | $z^2$ |
| Sector (angle $\alpha$) → UHP | $z^{\pi/\alpha}$ |
| Strip ($0 < \text{Im} < \pi$) → UHP | $e^z$ |
| Strip ($0 < \text{Im} < h$) → UHP | $e^{\pi z/h}$ |
| UHP → Strip ($0 < \text{Im} < \pi$) | $\log z$ |
| Right HP → Strip ($-\pi/2 < \text{Im} < \pi/2$) | $\log z$ |
| Ext. of disk → Disk | $1/z$ |
| UHP → Polygon | Schwarz–Christoffel |

---

## 6. Real-World Applications

| Application | Map Used |
|-------------|---------|
| **Heat flow in a wedge** | $z^{\pi/\alpha}$ maps wedge to half-plane |
| **Electrostatic parallel plates** | Strip → half-plane via $e^z$ |
| **Flow around a corner** | Sector → half-plane via power map |
| **Airfoil design** | Joukowski and Kármán–Trefftz maps |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Strategy | Compose elementary maps through standard intermediaries |
| UHP ↔ Disk | Cayley transform and inverse |
| Sector → UHP | $z^{\pi/\alpha}$ |
| Strip → UHP | $e^{\pi z/h}$ |
| Disk automorphisms | $e^{i\theta}(z-a)/(1-\bar{a}z)$ |
| Schwarz–Christoffel | UHP → arbitrary polygon |
| Joukowski | Ext. circle → slit plane |

---

## Quick Revision Questions

1. **Find** a conformal map from the half-strip $\{x > 0, 0 < y < \pi\}$ to the upper half-plane.

2. **Map** the sector $\{0 < \arg z < \pi/4\}$ to the unit disk (via composition).

3. **State** the Schwarz–Christoffel formula and describe its geometric meaning.

4. **Explain** why composition is the key technique for building conformal maps.

5. **Find** a conformal map from the upper half-disk $\{|z| < 1, \text{Im}(z) > 0\}$ to the upper half-plane.

6. **Map** the horizontal strip $\{a < \text{Im}(z) < b\}$ to the upper half-plane.

---

[← Previous: Linear Fractional Transformations](02-linear-fractional-transformations.md) | [Next: Applications →](04-applications.md)
