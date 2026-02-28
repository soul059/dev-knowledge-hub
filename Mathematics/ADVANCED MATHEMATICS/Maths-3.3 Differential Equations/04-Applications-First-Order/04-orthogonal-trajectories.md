# Chapter 4.4: Orthogonal Trajectories

[← Previous: RL and RC Circuits](03-rl-and-rc-circuits.md) | [Back to README](../README.md) | [Next: Geometric Applications →](05-geometric-applications.md)

---

## Overview

Orthogonal trajectories are families of curves that intersect a given family of curves at right angles. They arise naturally in physics (equipotential lines vs. electric field lines, isotherms vs. heat flow lines) and are found using first-order ODEs.

---

## 1. Definition

Given a one-parameter family of curves $F(x, y, c) = 0$:
- Each curve in the family has a slope given by some ODE $\dfrac{dy}{dx} = f(x, y)$
- The **orthogonal trajectories** have slopes perpendicular at every intersection point:

$$\boxed{\frac{dy}{dx}\bigg|_{\text{O.T.}} = -\frac{1}{f(x, y)}}$$

```
     y
     │    Family: y = cx²
     │   ╱  │  ╲
     │  ╱   │   ╲
     │ ╱    │    ╲
     │╱     │     ╲
  ───┼──────┼──────╲─── x
     │╲     │     ╱
     │ ╲    │    ╱     O.T.: x² + 2y² = k
     │  ╲   │   ╱       (ellipses)
     │   ╲  │  ╱
     │    ╲ │ ╱
     │
   Curves meet at RIGHT ANGLES (90°)
```

---

## 2. Method for Finding Orthogonal Trajectories

### Step-by-Step Procedure

```
┌─────────────────────────────────────┐
│ Given: Family F(x, y, c) = 0       │
├─────────────────────────────────────┤
│                                     │
│ Step 1: Differentiate to get        │
│         dy/dx = f(x, y, c)         │
│                                     │
│ Step 2: Eliminate the parameter c   │
│         using the original equation │
│         → dy/dx = f(x, y)          │
│                                     │
│ Step 3: Replace dy/dx with          │
│         -dx/dy (negative reciprocal)│
│         → dy/dx = -1/f(x, y)       │
│                                     │
│ Step 4: Solve the new ODE           │
│         → Orthogonal Trajectories   │
└─────────────────────────────────────┘
```

---

## 3. Worked Examples

### Example 1: Parabolas $y = cx^2$

**Step 1**: Differentiate: $\dfrac{dy}{dx} = 2cx$

**Step 2**: From original, $c = y/x^2$. Substitute:

$$\frac{dy}{dx} = 2\left(\frac{y}{x^2}\right)x = \frac{2y}{x}$$

**Step 3**: Replace $dy/dx$ with $-dx/dy$:

$$\frac{dy}{dx} = -\frac{x}{2y}$$

**Step 4**: Solve (separable):

$$2y\,dy = -x\,dx \implies y^2 = -\frac{x^2}{2} + C$$

$$\boxed{x^2 + 2y^2 = k} \quad (\text{family of ellipses})$$

### Example 2: Circles $x^2 + y^2 = c^2$

**Step 1**: Differentiate: $2x + 2y\dfrac{dy}{dx} = 0 \implies \dfrac{dy}{dx} = -\dfrac{x}{y}$

(No parameter to eliminate — already done!)

**Step 3**: $\dfrac{dy}{dx}\bigg|_{OT} = \dfrac{y}{x}$

**Step 4**: Solve: $\dfrac{dy}{y} = \dfrac{dx}{x} \implies \ln|y| = \ln|x| + C \implies y = kx$

$$\boxed{y = kx} \quad (\text{family of lines through origin})$$

```
     y      Circles: x² + y² = c²
     │      O.T.: y = kx (straight lines)
     │  ╱       ___
     │╱     ╱╱      ╲╲
    ╱│   ╱╱     ___   ╲╲
  ╱  │  ╱   ╱╱     ╲╲  ╲
 ╱   │ ╱  ╱╱    •    ╲╲  ╲
╱────┼╱──╱╱───────────╲╲──╲──── x
╲    │╲  ╲╲           ╱╱  ╱
 ╲   │ ╲  ╲╲   ___  ╱╱  ╱
  ╲  │  ╲   ╲╲    ╱╱   ╱
     │╲     ╲╲__╱╱
     │  ╲       
     │
```

### Example 3: Exponential Family $y = ce^{x}$

**Step 1**: $\dfrac{dy}{dx} = ce^{x}$

**Step 2**: $c = ye^{-x}$, so $\dfrac{dy}{dx} = ye^{-x} \cdot e^x = y$

**Step 3**: $\dfrac{dy}{dx}\bigg|_{OT} = -\dfrac{1}{y}$

**Step 4**: $y\,dy = -dx \implies \dfrac{y^2}{2} = -x + C$

$$\boxed{y^2 = -2x + k} \quad (\text{family of parabolas})$$

### Example 4: Hyperbolas $xy = c$

**Step 1**: $y + x\dfrac{dy}{dx} = 0 \implies \dfrac{dy}{dx} = -\dfrac{y}{x}$

**Step 3**: $\dfrac{dy}{dx}\bigg|_{OT} = \dfrac{x}{y}$

**Step 4**: $y\,dy = x\,dx \implies y^2 = x^2 + C$

$$\boxed{x^2 - y^2 = k} \quad (\text{rectangular hyperbolas rotated 45°})$$

### Example 5: Family of Lines $y = mx + c$ with Fixed Slope $m$

If the family is $y = x + c$ (parallel lines with slope 1):

$\dfrac{dy}{dx} = 1 \implies \dfrac{dy}{dx}\bigg|_{OT} = -1$

Solution: $y = -x + k$ (perpendicular parallel lines)

---

## 4. Orthogonal Trajectories in Polar Coordinates

For family $f(r, \theta, c) = 0$ with $\dfrac{dr}{d\theta} = g(r, \theta)$:

Replace $\dfrac{dr}{d\theta}$ with $-r^2 \dfrac{d\theta}{dr}$ (equivalently, $\dfrac{dr}{d\theta}$ becomes $-\dfrac{r^2}{g(r,\theta)}$):

$$\boxed{\frac{dr}{d\theta}\bigg|_{OT} = -\frac{r^2}{g(r,\theta)}}$$

### Example 6: Cardioids $r = c(1 + \cos\theta)$

**Step 1**: $\dfrac{dr}{d\theta} = -c\sin\theta$

**Step 2**: $c = \dfrac{r}{1 + \cos\theta}$, so $\dfrac{dr}{d\theta} = \dfrac{-r\sin\theta}{1 + \cos\theta}$

**Step 3**: Replace: $\dfrac{dr}{d\theta}\bigg|_{OT} = \dfrac{r(1 + \cos\theta)}{\sin\theta}$

**Step 4**: Separable: $\dfrac{dr}{r} = \dfrac{(1 + \cos\theta)}{\sin\theta}d\theta$

Using $\dfrac{1+\cos\theta}{\sin\theta} = \cot(\theta/2)$:

$$\ln r = 2\ln\sin(\theta/2) + C'$$

$$\boxed{r = k(1 - \cos\theta)} \quad (\text{cardioids, reflected})$$

---

## 5. Physical Applications

| Given Family | O.T. Family | Physical Context |
|-------------|-------------|------------------|
| Equipotential lines | Electric field lines | Electrostatics |
| Isotherms ($T = $ const) | Heat flow lines | Thermodynamics |
| Stream lines | Velocity potential lines | Fluid mechanics |
| Magnetic field lines | Equipotential surfaces | Magnetostatics |

```
  PHYSICAL INTERPRETATION

  Isotherms (T = const)          Heat Flow Lines
        ╱│╲                         ── ── ──
       ╱ │ ╲                        ── ── ──
  ────╱──│──╲────               ────────────────
     ╱   │   ╲                      ── ── ──
    ╱    │    ╲                     ── ── ──

        ⊗ heat source

  Heat flows PERPENDICULAR to isotherms!
```

---

## 6. Self-Orthogonal Families

A family is **self-orthogonal** if its orthogonal trajectories belong to the same family.

**Example**: The family of parabolas $y^2 = 4c(x + c)$

After finding the ODE and replacing $dy/dx$ with $-1/(dy/dx)$, we get the same ODE — hence self-orthogonal!

---

## Summary Table

| Concept | Details |
|---------|---------|
| **O.T. in Cartesian** | Replace $dy/dx = f$ with $dy/dx = -1/f$ |
| **O.T. in Polar** | Replace $dr/d\theta$ with $-r^2/(dr/d\theta)$ |
| **Method** | Differentiate → eliminate $c$ → negate reciprocal → solve |
| **$y = cx^2$** | O.T.: $x^2 + 2y^2 = k$ (ellipses) |
| **$x^2 + y^2 = c^2$** | O.T.: $y = kx$ (lines) |
| **$xy = c$** | O.T.: $x^2 - y^2 = k$ (hyperbolas) |
| **Physical meaning** | Equipotential ⊥ field lines, isotherms ⊥ heat flow |

---

## Quick Revision Questions

1. Find the orthogonal trajectories of the family $y = cx^3$.

2. Show that the family of circles $x^2 + y^2 = 2cx$ has orthogonal trajectories $x^2 + y^2 = 2ky$.

3. Find the O.T. of the family $y^2 = 4cx$ (parabolas). What type of curves are they?

4. In polar coordinates, find the O.T. of the family $r = c\cos\theta$.

5. Why are equipotential lines and electric field lines orthogonal? Relate this to the gradient.

6. Can a family of straight lines be self-orthogonal? If so, give an example.

---

[← Previous: RL and RC Circuits](03-rl-and-rc-circuits.md) | [Back to README](../README.md) | [Next: Geometric Applications →](05-geometric-applications.md)
