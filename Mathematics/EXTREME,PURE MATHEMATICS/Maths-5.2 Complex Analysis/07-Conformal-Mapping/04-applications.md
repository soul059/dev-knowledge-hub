# 7.4 Applications of Conformal Mapping

[← Previous: Mapping of Standard Regions](03-mapping-of-standard-regions.md) | [Next: Argument Principle →](../08-Advanced-Topics/01-argument-principle.md)

---

## Chapter Overview

Conformal mapping transforms complicated **boundary-value problems** (heat, fluid, electrostatics) in irregular domains into **tractable problems** in standard domains (half-plane, disk). This chapter demonstrates the method on Laplace's equation, fluid dynamics, and electrostatics.

---

## 1. The Key Principle

If $\phi$ is harmonic in $D$ and $f: D' \to D$ is conformal, then

$$\Phi = \phi \circ f \quad \text{is harmonic in } D'.$$

```
Complicated domain D'          Standard domain D
  ╭────╮                         ▲ Im
 ╱      ╲      f  →              │████████
│  BVP?  │    ─────→             │████████
 ╲      ╱    conformal            ┼──────► Re
  ╰────╯          
   Hard to solve               Easy to solve

Solve in D, pull back to D' via f.
```

---

## 2. Heat Conduction (Steady-State)

### 2.1 The Problem

Steady-state temperature satisfies Laplace's equation:

$$\nabla^2 T = \frac{\partial^2 T}{\partial x^2} + \frac{\partial^2 T}{\partial y^2} = 0$$

with boundary conditions $T = T_0$ or $T = T_1$ on boundary segments.

### 2.2 Method

1. Identify domain $D'$ and boundary conditions
2. Find conformal map $w = f(z)$ from $D'$ to a standard domain $D$
3. Solve the Dirichlet problem in $D$ (often trivial)
4. Pull back: $T(x, y) = T_{\text{solution}}(u(x,y), v(x,y))$

### 2.3 Example: Temperature in a Wedge

**Problem:** Find steady-state temperature in $\{0 < \arg z < \pi/4\}$ with $T = 0$ on $\arg z = 0$ and $T = 100$ on $\arg z = \pi/4$.

**Step 1.** Map wedge to UHP: $w = z^4$ (since $\pi/(\pi/4) = 4$).

**Step 2.** Boundary maps:
- $\arg z = 0$ (positive real axis) → positive real axis ($T = 0$)
- $\arg z = \pi/4$ → negative real axis ($T = 100$)

**Step 3.** In UHP, harmonic function with $T = 0$ on positive real, $T = 100$ on negative real:

$$T = \frac{100}{\pi}\arg(w) = \frac{100}{\pi}\text{Arg}(w)$$

**Step 4.** Pull back: $w = z^4$, so $\arg w = 4\arg z = 4\theta$.

$$\boxed{T(r, \theta) = \frac{400\theta}{\pi}}$$

```
  T = 100 on this edge
       ╱
      ╱  ∇²T = 0
     ╱   here
    ╱ θ = π/4
   ╱_________  T = 0 on this edge
   O          (θ = 0)
```

---

## 3. Fluid Flow (2D Ideal Flow)

### 3.1 Complex Potential

For irrotational, incompressible 2D flow:

$$\Omega(z) = \phi(x,y) + i\psi(x,y)$$

- $\phi$ = velocity potential (harmonic)
- $\psi$ = stream function (harmonic)
- $\frac{d\Omega}{dz} = u - iv$ = complex velocity

### 3.2 Elementary Flows

| Complex Potential | Flow Type |
|-------------------|-----------|
| $\Omega = Uz$ | Uniform flow (speed $U$, rightward) |
| $\Omega = \frac{m}{2\pi}\log z$ | Source/sink at origin ($m > 0$: source) |
| $\Omega = \frac{-i\Gamma}{2\pi}\log z$ | Point vortex at origin |
| $\Omega = Uz + \frac{Ua^2}{z}$ | Flow around a cylinder of radius $a$ |

### 3.3 Flow via Conformal Mapping

If $\Omega_0(w)$ describes flow in a simple domain and $w = f(z)$ is conformal:

$$\Omega(z) = \Omega_0(f(z))$$

gives the flow in the $z$-plane.

```
Flow past a cylinder:         Joukowski map         Flow past an airfoil:

     →  →  →                                         →  →  →
    →  ╭──╮  →        w = z + 1/z                →  ╭────────╮  →
   →  │  ○ │  →      ───────────→               →  ╱          ╲  →
    →  ╰──╯  →                                    →  ╰────────╯  →
     →  →  →                                         →  →  →
```

### 3.4 Example: Flow Around an Obstacle

**Problem:** Find the flow in the upper half-plane above a step.

Schwarz–Christoffel maps the UHP to half-planes with steps, translating the geometry to uniform flow $\Omega_0 = Uw$ in the image half-plane. Pull back via the inverse map to get the flow in the physical domain.

---

## 4. Electrostatics

### 4.1 2D Electrostatic Potential

Electric potential $\phi$ in a charge-free region satisfies $\nabla^2 \phi = 0$.

Conductors have constant potential on their surfaces → **Dirichlet problem**.

### 4.2 Capacitance of Parallel Plates

**Setup:** Plates at $y = 0$ ($\phi = 0$) and $y = d$ ($\phi = V_0$).

**Domain:** Strip $0 < y < d$ → map via $w = e^{\pi z / d}$ to UHP.

But the linear solution is already obvious: $\phi = V_0 y/d$.

### 4.3 Capacitance of Coaxial Cylinders

**Setup:** Inner radius $a$ ($\phi = 0$), outer radius $b$ ($\phi = V_0$).

Domain: annulus $a < |z| < b$. Solution:

$$\phi = V_0 \frac{\log(r/a)}{\log(b/a)}, \quad r = |z|$$

This is $\text{Re}\left(\frac{V_0}{\log(b/a)}\log(z/a)\right)$.

---

## 5. Design Example

### Example: Temperature in a semicircular plate

**Problem:** Find steady-state temperature in the upper half of the unit disk ($|z| < 1$, $\text{Im}(z) > 0$) with:
- $T = 0$ on the diameter $[-1, 1]$ (real axis)
- $T = 1$ on the semicircular arc

**Step 1.** Map the upper half-disk to the upper half-plane.

Use $w = \frac{(1+z)^2}{(1-z)^2}$, which maps:
- Center point $z = i$ to $w = -1$
- Real diameter to negative real axis
- Arc to positive real axis

Actually, a simpler approach: Use $w = i\frac{1-z^2}{(z-1)^2 + z^2}$... these get complicated. 

**Practical approach:** Use the Poisson integral directly.

For the upper half-disk with $T = 1$ on the arc and $T = 0$ on the diameter:

By symmetry and separation of variables in polar:

$$\boxed{T(r, \theta) = \frac{2}{\pi}\left[\arctan\!\left(\frac{2r\sin\theta}{1-r^2}\right)\right]}$$

This uses the identity: the Poisson kernel on the disk with half-boundary unity.

---

## 6. Summary of the Conformal Mapping Method

```
┌────────────────────┐     conformal map f    ┌────────────────────┐
│  Complicated       │  ──────────────────→   │  Standard          │
│  domain D'         │                        │  domain D          │
│                    │                        │  (disk / UHP)      │
│  Hard BVP          │                        │  Easy BVP          │
└────────────────────┘                        └────────────────────┘
         ▲                                             │
         │               pull back                     │
         └─────────────── T' = T ∘ f ────────── SOLVE ─┘
```

---

## 7. Real-World Applications

| Application | Domain | Map Used |
|-------------|--------|----------|
| **Aerodynamics** | Flow past airfoil | Joukowski / Kármán–Trefftz |
| **VLSI design** | Heat in L-shaped region | Schwarz–Christoffel |
| **Groundwater flow** | Seepage under a dam | Strip → half-plane |
| **Capacitor design** | Fringing field at plate edge | S-C to half-strip |
| **Magnetic shielding** | Field in a gap | Disk automorphisms |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Harmonic pull-back | $\phi \circ f$ is harmonic if $\phi$ is harmonic and $f$ is conformal |
| Heat / electrostatics | Solve Laplace $\nabla^2 T = 0$ with Dirichlet BCs |
| Fluid flow | Complex potential $\Omega = \phi + i\psi$, velocity $= d\Omega/dz$ |
| Method outline | Map to standard domain → solve → pull back |
| Wedge problems | $z^{\pi/\alpha}$ opens wedge to half-plane |
| Joukowski | $z + 1/z$ maps cylinder flow to airfoil flow |
| Schwarz–Christoffel | Maps UHP to polygonal domains for BVPs |

---

## Quick Revision Questions

1. **Explain** why $\phi \circ f$ is harmonic whenever $\phi$ is harmonic and $f$ is conformal.

2. **Find** the steady-state temperature in the sector $0 < \arg z < \pi/3$ with $T = 0$ on $\theta = 0$ and $T = 50$ on $\theta = \pi/3$.

3. **Write** the complex potential for uniform flow of speed $U$ past a circular cylinder of radius $a$.

4. **Describe** how the Joukowski map is used in aerodynamics.

5. **Outline** the conformal mapping method for solving a Dirichlet problem in an irregular domain.

6. **Solve** for the electrostatic potential between two concentric cylinders of radii $a$ and $b$ with potentials $0$ and $V_0$.

---

[← Previous: Mapping of Standard Regions](03-mapping-of-standard-regions.md) | [Next: Argument Principle →](../08-Advanced-Topics/01-argument-principle.md)
