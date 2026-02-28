# Chapter 7.4: Beam Deflection

[← Previous: LCR Circuits](03-lcr-circuits.md) | [Back to README](../README.md) | [Next: Systems of ODEs — Introduction →](../08-Systems-of-ODEs/01-introduction-and-first-order-systems.md)

---

## Overview

The Euler-Bernoulli beam equation governs the deflection of beams under loading. This fourth-order ODE is fundamental to structural engineering and connects differential equations to real-world design problems.

---

## 1. The Euler-Bernoulli Beam Equation

$$\boxed{EI\frac{d^4y}{dx^4} = w(x)}$$

where:
- $y(x)$ = deflection of the beam at position $x$
- $E$ = Young's modulus (material property, Pa)
- $I$ = second moment of area (cross-section property, m⁴)
- $EI$ = flexural rigidity
- $w(x)$ = distributed load (N/m)

### Related Quantities

$$\frac{dy}{dx} = \theta \quad (\text{slope})$$

$$EI\frac{d^2y}{dx^2} = M(x) \quad (\text{bending moment})$$

$$EI\frac{d^3y}{dx^3} = V(x) \quad (\text{shear force})$$

$$EI\frac{d^4y}{dx^4} = w(x) \quad (\text{load intensity})$$

```
  BEAM DEFLECTION HIERARCHY

  Load w(x) ──differentiate──→ Shear V(x)
       ↑                            ↑
  d⁴y/dx⁴                       d³y/dx³
                                     ↑
  ──differentiate──→ Moment M(x) ←───┘
                        ↑
                     d²y/dx²
                        ↑
  ──differentiate──→ Slope θ(x)
                        ↑
                     dy/dx
                        ↑
                   Deflection y(x)
```

---

## 2. Boundary Conditions

### Common Support Types

```
  SIMPLY SUPPORTED (PINNED)     FIXED (CLAMPED)         FREE END
                                                        
  y = 0, M = 0                 y = 0, dy/dx = 0        M = 0, V = 0
                                                        
    ╱│                            ║                         │
   ╱ │                            ║█████                    │
  ╱  │═════════                   ║═════════               ═══════│
   △                              ║                               ↓ (may have load)
  (pin support)                (wall/clamp)                 (free end)
```

| Support Type | Conditions |
|-------------|-----------|
| Simply supported (pinned) | $y = 0$, $y'' = 0$ (zero deflection, zero moment) |
| Fixed (clamped) | $y = 0$, $y' = 0$ (zero deflection, zero slope) |
| Free end | $y'' = 0$, $y''' = 0$ (zero moment, zero shear) |

---

## 3. Worked Examples

### Example 1: Simply Supported Beam with Uniform Load

```
  w (N/m) uniform load
  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
  ═══════════════════════
  △                     △
  x=0                  x=L
```

$EI\,y'''' = w$ (constant)

**Integrate four times**:

$$EI\,y''' = wx + C_1$$
$$EI\,y'' = \frac{wx^2}{2} + C_1 x + C_2$$
$$EI\,y' = \frac{wx^3}{6} + \frac{C_1 x^2}{2} + C_2 x + C_3$$
$$EI\,y = \frac{wx^4}{24} + \frac{C_1 x^3}{6} + \frac{C_2 x^2}{2} + C_3 x + C_4$$

**Boundary conditions**: $y(0) = 0$, $y''(0) = 0$, $y(L) = 0$, $y''(L) = 0$

From $y(0) = 0$: $C_4 = 0$

From $y''(0) = 0$: $C_2 = 0$

From $y''(L) = 0$: $\dfrac{wL^2}{2} + C_1 L = 0 \implies C_1 = -\dfrac{wL}{2}$

From $y(L) = 0$: $\dfrac{wL^4}{24} - \dfrac{wL^4}{12} + C_3 L = 0 \implies C_3 = \dfrac{wL^3}{24}$

$$\boxed{y(x) = \frac{w}{24EI}(x^4 - 2Lx^3 + L^3 x) = \frac{wx}{24EI}(L^3 - 2Lx^2 + x^3)}$$

**Maximum deflection** at $x = L/2$:

$$\boxed{y_{max} = \frac{5wL^4}{384EI}}$$

### Example 2: Cantilever with Point Load at Free End

```
  ║
  ║███████════════════════════ →P
  ║                           
  ║                           x=L
  x=0
```

$EI\,y'''' = 0$ (no distributed load between support and point)

But $V(L) = -P$ (shear at free end) and $M(L) = 0$ (moment at free end).

Solving from $M(x) = -P(L - x)$:

$$EI\,y'' = -P(L - x) = -PL + Px$$

$$EI\,y' = -PLx + \frac{Px^2}{2} + C_1$$

$y'(0) = 0$: $C_1 = 0$

$$EI\,y = -\frac{PLx^2}{2} + \frac{Px^3}{6} + C_2$$

$y(0) = 0$: $C_2 = 0$

$$\boxed{y(x) = \frac{P}{6EI}(x^3 - 3Lx^2) = \frac{Px^2}{6EI}(x - 3L)}$$

**Maximum deflection** at $x = L$:

$$\boxed{y_{max} = \frac{-PL^3}{3EI}}$$

(Negative = downward deflection)

### Example 3: Cantilever with Uniform Load

```
  ║  w (N/m)
  ║  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
  ║██════════════════════
  ║                       
  x=0                  x=L
```

$EI\,y'''' = w$

BCs: $y(0) = 0$, $y'(0) = 0$, $y''(L) = 0$, $y'''(L) = 0$

From $y'''(L) = 0$: $wL + C_1 = 0 \implies C_1 = -wL$

From $y''(L) = 0$: $\dfrac{wL^2}{2} - wL^2 + C_2 = 0 \implies C_2 = \dfrac{wL^2}{2}$

From $y'(0) = 0$: $C_3 = 0$

From $y(0) = 0$: $C_4 = 0$

$$\boxed{y(x) = \frac{w}{24EI}(x^4 - 4Lx^3 + 6L^2x^2)}$$

$$y_{max} = y(L) = \frac{wL^4}{8EI}$$

---

## 4. Summary of Standard Results

| Configuration | Max Deflection | Location |
|--------------|---------------|----------|
| Simply supported, uniform $w$ | $\dfrac{5wL^4}{384EI}$ | Center |
| Simply supported, center point $P$ | $\dfrac{PL^3}{48EI}$ | Center |
| Cantilever, end point $P$ | $\dfrac{PL^3}{3EI}$ | Free end |
| Cantilever, uniform $w$ | $\dfrac{wL^4}{8EI}$ | Free end |

---

## 5. Deflection Shapes

```
  SIMPLY SUPPORTED, UNIFORM LOAD

  ═══════════════════════════
  △           │             △
              │
              ↓ y_max = 5wL⁴/384EI


  CANTILEVER, POINT LOAD AT END

  ║███══════════════════╲
  ║                      ╲
  ║                       ╲
                           ↓ y_max = PL³/3EI


  CANTILEVER, UNIFORM LOAD

  ║██═══════════════╲
  ║                   ╲╲
  ║                      ╲╲
                            ↓ y_max = wL⁴/8EI
```

---

## 6. Design Significance

- **Stiffness**: Deflection $\propto 1/EI$. Increase $E$ (stronger material) or $I$ (bigger cross-section) to reduce deflection.
- **Length**: Deflection $\propto L^3$ or $L^4$ — doubling length increases deflection 8–16× !
- **Support conditions**: Clamped ends are stiffer than simply supported (factor of ~5).

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Beam equation** | $EI\,y'''' = w(x)$ |
| **Moment** | $M = EI\,y''$ |
| **Shear** | $V = EI\,y'''$ |
| **Pin support** | $y = 0$, $y'' = 0$ |
| **Fixed support** | $y = 0$, $y' = 0$ |
| **Free end** | $y'' = 0$, $y''' = 0$ |
| **Solution method** | Direct integration (4 times) |

---

## Quick Revision Questions

1. Derive the deflection curve for a simply supported beam with a concentrated load $P$ at the center.

2. A cantilever of length 2 m carries a uniform load of 5 kN/m. If $EI = 10^4$ kN·m², find the maximum deflection.

3. Why does the beam equation have four boundary conditions while second-order ODEs have two?

4. Show that for a simply supported beam with uniform load, the maximum moment occurs at the center and equals $wL^2/8$.

5. If a beam's length is doubled (all else constant), by what factor does the maximum deflection increase for (a) uniform load on a simply supported beam? (b) point load on a cantilever?

6. Solve $EI\,y'''' = 0$ (no load) with $y(0) = 0$, $y'(0) = 0$, $y(L) = \delta$, $y'(L) = 0$ (beam forced to deflect $\delta$ at the far end).

---

[← Previous: LCR Circuits](03-lcr-circuits.md) | [Back to README](../README.md) | [Next: Systems of ODEs — Introduction →](../08-Systems-of-ODEs/01-introduction-and-first-order-systems.md)
