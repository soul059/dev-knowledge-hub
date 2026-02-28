# 7.1 Conformal Maps

[← Previous: Integrals Involving Branch Points](../06-Residue-Theory/05-integrals-involving-branch-points.md) | [Next: Linear Fractional Transformations →](02-linear-fractional-transformations.md)

---

## Chapter Overview

A **conformal map** is an analytic function whose derivative is nonzero — it preserves angles between curves. Conformal maps are the geometric heart of complex analysis, connecting analytic structure to geometric properties. They are indispensable in fluid dynamics, electrostatics, and the study of Riemann surfaces.

---

## 1. Definition and Basic Properties

### 1.1 Conformal = Angle-Preserving

A function $f$ analytic at $z_0$ with $f'(z_0) \neq 0$ is **conformal** at $z_0$: it preserves both the **magnitude** and **sense** (orientation) of angles between curves.

### 1.2 Local Geometric Action

Near $z_0$, $f$ acts as:

$$f(z) \approx f(z_0) + f'(z_0)(z - z_0)$$

Writing $f'(z_0) = re^{i\alpha}$:
- **Rotation** by angle $\alpha = \arg f'(z_0)$
- **Scaling** by factor $r = |f'(z_0)|$
- **Translation** by $f(z_0)$

```
Local action of a conformal map:

  z-plane                              w-plane
                                       
    curve γ₂                             f(γ₂)
      ╲  θ                                ╲  θ  (same angle!)
       ╲                                    ╲
  ──────•────── curve γ₁       ──────────────•────── f(γ₁)
       z₀                                  f(z₀)

  Angles preserved ⟵ f'(z₀) ≠ 0
  Angles doubled/collapsed ⟵ f'(z₀) = 0 (critical point)
```

### 1.3 What Fails at Critical Points

If $f'(z_0) = 0$ and $f^{(k)}(z_0) \neq 0$ (first nonzero derivative), then angles are **multiplied by $k$** at $z_0$. Conformality fails.

**Example:** $f(z) = z^2$, $f'(0) = 0$, $f''(0) = 2$. Angles at the origin are doubled.

---

## 2. Univalent (Schlicht) Functions

### 2.1 Definition

$f$ is **univalent** (one-to-one) on $D$ if $f(z_1) = f(z_2) \Rightarrow z_1 = z_2$.

### 2.2 Inverse Function Theorem

If $f$ is analytic and univalent on $D$, then:
- $f'(z) \neq 0$ everywhere in $D$
- $f^{-1}$ is analytic on $f(D)$
- $(f^{-1})'(w) = \frac{1}{f'(f^{-1}(w))}$

### 2.3 Key Fact

Conformal + univalent = **biholomorphic** (analytic bijection with analytic inverse).

---

## 3. Important Conformal Maps

### 3.1 Table of Standard Maps

| Map | Action | Key Property |
|-----|--------|-------------|
| $w = z + c$ | Translation | Isometry |
| $w = e^{i\alpha}z$ | Rotation by $\alpha$ | Isometry |
| $w = kz$ ($k > 0$) | Dilation | Scaling by $k$ |
| $w = z^2$ | Doubles angles at origin | Maps half-plane to full plane |
| $w = e^z$ | Exponential | Maps strip to sector |
| $w = \log z$ | Logarithm | Maps sector to strip |
| $w = \frac{1}{z}$ | Inversion | Maps circles/lines to circles/lines |
| $w = \frac{z-a}{1-\bar{a}z}$ | Möbius (disk to disk) | Automorphism of disk |
| $w = \frac{z-i}{z+i}$ | Cayley transform | UHP → unit disk |

### 3.2 Power Maps $w = z^n$

```
w = z²: Doubles angles, maps sectors to sectors

  z-plane (first quadrant):      w-plane (upper half-plane):
       Im                              Im
        ▲                               ▲
        │ ╱╱╱╱                           │████████████
        │╱╱╱╱╱                           │████████████
        •──────► Re                      •──────────────► Re

  Quarter-plane (angle π/2) → Half-plane (angle π)
```

### 3.3 Exponential Map $w = e^z$

```
w = e^z: Maps horizontal strip to sector

  z-plane (strip 0 < y < π):    w-plane (upper half-plane):
       y=π ────────────────          Im
        │██████████████████           ▲
        │██████████████████           │████████████
       y=0 ────────────────           │████████████
                                      •──────────────► Re
```

$z = x + iy$ → $w = e^x e^{iy}$: the strip $\{0 < y < \pi\}$ maps to $\{\text{Im}(w) > 0\}$.

### 3.4 Joukowski Map

$$w = z + \frac{1}{z}$$

Maps the exterior of the unit circle to $\mathbb{C} \setminus [-2, 2]$. Critical in **aerodynamics** (airfoil theory).

```
Joukowski map:

  z-plane:                      w-plane:
       ╭──╮                    
      ╱    ╲                    ─────•═══════════•─────
     │  ○   │  exterior    →       -2    slit     2
      ╲    ╱  of |z|=1         
       ╰──╯                     maps to ℂ \ [-2, 2]
```

---

## 4. The Riemann Mapping Theorem

### 4.1 Statement

> **Riemann Mapping Theorem.** Every **simply connected** domain $D \subsetneq \mathbb{C}$ (i.e., $D \neq \mathbb{C}$) is conformally equivalent to the open unit disk $\mathbb{D} = \{|z| < 1\}$.

Moreover, if we fix $z_0 \in D$ and require $f(z_0) = 0$, $f'(z_0) > 0$, then the conformal map is **unique**.

### 4.2 Significance

- Any two simply connected proper subdomains of $\mathbb{C}$ are conformally equivalent.
- Reduces problems on complicated domains to problems on the disk.
- Existence is guaranteed, but the explicit map is usually hard to find.

### 4.3 What the Theorem Does NOT Cover

- $D = \mathbb{C}$ (Liouville's theorem: the only biholomorphism $\mathbb{C} \to \mathbb{D}$ would be bounded + entire = constant, contradiction).
- Multiply connected domains (not equivalent to disk).
- The theorem is non-constructive (gives existence, not formula).

---

## 5. Design Example

### Example: Find a conformal map from the first quadrant $Q = \{x > 0, y > 0\}$ to the upper half-plane $\mathbb{H} = \{\text{Im}(w) > 0\}$.

**Step 1.** $Q$ has angle $\pi/2$ at the origin. We need to open it to angle $\pi$.

**Step 2.** Use $w = z^2$:
- At $z = x$ (positive real): $w = x^2 > 0$ (positive real).
- At $z = iy$ (positive imaginary): $w = -y^2 < 0$ (negative real).
- Interior of $Q$ maps to $\mathbb{H}$.

**Step 3.** Verify conformality: $w' = 2z \neq 0$ for $z \in Q$ (but $= 0$ at origin, a boundary point — acceptable since the origin maps to the boundary).

$$\boxed{w = z^2 : Q \to \mathbb{H}}$$

---

## 6. Real-World Applications

| Application | Conformal Map Used |
|-------------|-------------------|
| **Aerodynamics** | Joukowski map: circle → airfoil shape |
| **Electrostatics** | Map complicated geometry to simple one; solve Laplace eq. |
| **Fluid dynamics** | Map flow around obstacles to flow in half-plane |
| **Heat conduction** | Steady-state temperature via conformal map → harmonic function |
| **Cartography** | Stereographic projection, Mercator (conformal maps of sphere) |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Conformal | Analytic + $f' \neq 0$ → angle-preserving |
| Local action | Rotation by $\arg f'$ + scaling by $|f'|$ |
| Critical point | $f' = 0$ → angles multiplied |
| Univalent | One-to-one analytic → biholomorphic |
| Riemann Mapping Theorem | Any simply connected $D \neq \mathbb{C}$ is conformally equivalent to $\mathbb{D}$ |
| Joukowski map | $z + 1/z$ maps circle to airfoil |
| Power map $z^n$ | Multiplies angles by $n$ |
| Exponential | Strip → sector/half-plane |

---

## Quick Revision Questions

1. **Define** conformal map and explain why $f'(z_0) \neq 0$ is required.

2. **What** happens to angles at a zero of $f'$? Illustrate with $f(z) = z^3$ at $z = 0$.

3. **State** the Riemann Mapping Theorem and explain why $D = \mathbb{C}$ is excluded.

4. **Find** a conformal map from the sector $\{0 < \arg z < \pi/3\}$ to the upper half-plane.

5. **Describe** the image of the horizontal strip $\{0 < \text{Im}(z) < \pi\}$ under $w = e^z$.

6. **Explain** the Joukowski map and its application to aerodynamics.

---

[← Previous: Integrals Involving Branch Points](../06-Residue-Theory/05-integrals-involving-branch-points.md) | [Next: Linear Fractional Transformations →](02-linear-fractional-transformations.md)
