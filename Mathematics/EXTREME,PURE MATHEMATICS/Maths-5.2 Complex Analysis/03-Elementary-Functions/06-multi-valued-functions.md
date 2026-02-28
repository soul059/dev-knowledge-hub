# 3.6 Multi-valued Functions and Riemann Surfaces

[← Previous: Power Functions](05-power-functions.md) | [Next: Unit 4 — Contours and Paths →](../04-Complex-Integration/01-contours-and-paths.md)

---

## Chapter Overview

Multi-valued "functions" like $\sqrt{z}$ and $\log z$ are not functions in the classical sense — they assign multiple values to each input. We handle this through **branch cuts** (restricting the domain to get single-valued branches) and **Riemann surfaces** (enlarging the range to make them genuinely single-valued). These concepts are essential for contour integration (Unit 4–6) and conformal mapping (Unit 7).

---

## 1. The Problem of Multi-Valuedness

### 1.1 Source

Multi-valuedness arises when inverting a function that is many-to-one:

| Function | Many-to-one? | Inverse | Multi-valued? |
|----------|-------------|---------|---------------|
| $w = z^2$ | 2-to-1 | $z = w^{1/2}$ | 2 values |
| $w = z^n$ | $n$-to-1 | $z = w^{1/n}$ | $n$ values |
| $w = e^z$ | $\infty$-to-1 | $z = \log w$ | $\infty$ values |

### 1.2 Continuous Argument Change

Consider $z = re^{i\theta}$ and $f(z) = z^{1/2} = \sqrt{r}\,e^{i\theta/2}$.

If we travel around the origin ($\theta: 0 \to 2\pi$), the argument of $f(z)$ changes by $\pi$, and $f(z)$ returns to $-f(z)$!

```
Analytic continuation of √z around origin:

  Start: z = r, √z = √r (positive real)

       θ = π/2                 θ = π
       √z = √r · e^(iπ/4)     √z = √r · i
            •                       •
           ╱                       │
          ╱                        │
  O──────•──────────── θ=0    O────┤──────── θ=0
     √z = √r                      │
          ╲                        │
           ╲                       •
            •                 θ = 3π/2
       θ = 7π/4               √z = √r · e^(i·3π/4)

  After full circuit (θ = 2π): √z = √r · e^(iπ) = -√r
  ≠ starting value!  (Branch point!)
```

### 1.3 Branch Points

A **branch point** is a point $z_0$ such that analytically continuing $f(z)$ around $z_0$ produces a different value.

For $z^{1/n}$: branch point at $z = 0$ (and $z = \infty$).
For $\log z$: branch point at $z = 0$ (and $z = \infty$).

---

## 2. Branch Cuts

### 2.1 Definition

A **branch cut** is a curve in the complex plane (usually a ray or line segment) that we remove to prevent analytic continuation from reaching different branches.

### 2.2 Standard Branch Cuts

| Function | Standard Branch Cut | Domain of Principal Branch |
|----------|--------------------|-----------------------------|
| $\sqrt{z}$ | $(-\infty, 0]$ | $\mathbb{C} \setminus (-\infty, 0]$ |
| $z^{1/n}$ | $(-\infty, 0]$ | $\mathbb{C} \setminus (-\infty, 0]$ |
| $\text{Log}(z)$ | $(-\infty, 0]$ | $\mathbb{C} \setminus (-\infty, 0]$ |
| $\sqrt{z^2-1}$ | $(-\infty, -1] \cup [1, \infty)$ | Depends on branch |

```
Standard branch cut along negative real axis:

       Im
        ▲
        │
        │  ← analytic region
  ██████O──────────────► Re
  ↑     │
  branch │  ← analytic region
  cut    │
```

### 2.3 Alternative Branch Cuts

The choice of branch cut is not unique! Any curve connecting the branch points works:

```
Different valid branch cuts for √z:

  (a) Negative real axis:    (b) Positive imaginary:   (c) Any curve to ∞:
      Im                         Im                         Im
       ▲                          ▲                          ▲
       │                          █                          │
       │                          █                          │  ╱
  █████O──────► Re                █                          │╱
       │                          O──────► Re                O──────► Re
       │                          │                         ╱│
                                  │                        █ │
```

---

## 3. Riemann Surfaces

### 3.1 Concept

A **Riemann surface** is a surface on which a multi-valued function becomes single-valued. We "stack" copies of the complex plane (called **sheets**) and connect them along branch cuts.

### 3.2 Riemann Surface for $\sqrt{z}$

$\sqrt{z}$ has 2 values → 2 sheets, connected along the branch cut.

```
Riemann surface for √z (schematic side view):

      Sheet 1 (√r e^(iθ/2),  0 ≤ θ < 2π)
    ──────────╲─────────────────────────
               ╲ branch cut seam
    ────────────╲───────────────────────
      Sheet 2 (√r e^(i(θ+2π)/2),  one full rotation)

  Crossing the branch cut takes you from Sheet 1 to Sheet 2
  Going around origin twice returns to Sheet 1
```

```
Top view of Riemann surface for √z:

    Sheet 1:              Sheet 2:
    ┌─────────────┐      ┌─────────────┐
    │             │      │             │
    │   branch    │      │   branch    │
    │═══O─────cut─┤━━━━━━┤═══O─────cut─│
    │             │      │             │
    │  f = +√z    │      │  f = -√z    │
    └─────────────┘      └─────────────┘
         ↕ connected along cuts ↕
```

### 3.3 Riemann Surface for $\log z$

$\log z$ has infinitely many values → infinitely many sheets. The Riemann surface is a **helicoidal surface** (like a spiral staircase).

```
Riemann surface for log z (spiral staircase):

                    ╭──── Sheet k+1 ────╮
                   ╱                     ╲
    ╭──── Sheet k ╱───────────────────────╲────╮
   ╱             ╱                         ╲    ╲
  ╱    Sheet k-1╱─────────────────────────────╲   ╲
 │             ╱                               ╲   │
 │            O (branch point at origin)        │   │
 │             ╲                               ╱   │
  ╲    Sheet k-2╲─────────────────────────────╱   ╱
   ╲             ╲                         ╱    ╱
    ╰─────────────╲───────────────────────╱────╯

  Each circuit around the origin moves up one sheet
  Im(log z) increases by 2π per circuit
```

---

## 4. Order of Branch Points

| Function | Branch point | Order | Sheets |
|----------|-------------|-------|--------|
| $z^{1/2}$ | $z = 0$ | 2 | 2 (return after 2 circuits) |
| $z^{1/3}$ | $z = 0$ | 3 | 3 |
| $z^{1/n}$ | $z = 0$ | $n$ | $n$ |
| $\log z$ | $z = 0$ | $\infty$ | $\infty$ (logarithmic branch point) |
| $\sqrt{z-a}$ | $z = a$ | 2 | 2 |
| $\sqrt{(z-a)(z-b)}$ | $z = a, b$ | 2 each | 2 |

---

## 5. Functions with Multiple Branch Points

### 5.1 $\sqrt{z^2 - 1} = \sqrt{(z-1)(z+1)}$

Branch points at $z = 1$ and $z = -1$. The branch cut must connect them or go to infinity.

**Standard choices:**
- Branch cut along $[-1, 1]$ (finite segment)
- Branch cut along $(-\infty, -1] \cup [1, \infty)$ (complement)

```
Branch cut options for √(z²-1):

  (a) Along [-1, 1]:            (b) Along (-∞,-1] ∪ [1,∞):

       Im                            Im
        ▲                             ▲
        │                             │
  ──────┤═══•═══•═══┤──► Re    ██████•───────•██████► Re
        │  -1    1                   -1       1
        │                             │
```

---

## 6. Design Example

### Example: Determine the branches of $f(z) = \sqrt{z}$ with branch cut along $[0, \infty)$ (positive real axis).

**Step 1.** Write $z = re^{i\theta}$ with $0 < \theta < 2\pi$ (argument measured from positive real axis, going counterclockwise).

**Step 2.** The two branches:
- Branch 1: $f_1(z) = \sqrt{r}\, e^{i\theta/2}$, $\theta \in (0, 2\pi)$
- Branch 2: $f_2(z) = \sqrt{r}\, e^{i(\theta+2\pi)/2} = -f_1(z)$

**Step 3.** On the upper imaginary axis ($z = iy$, $y > 0$, $\theta = \pi/2$):
$$f_1(iy) = \sqrt{y}\, e^{i\pi/4} = \sqrt{y}\left(\frac{1+i}{\sqrt{2}}\right) = \frac{\sqrt{y}(1+i)}{\sqrt{2}}$$

---

## 7. Real-World Applications

| Application | Multi-valued Function Used |
|-------------|--------------------------|
| **Fluid around obstacles** | $\sqrt{z^2 - a^2}$ for flow past a slit |
| **Acoustics/Diffraction** | Branch cuts model diffraction by half-planes |
| **Quantum mechanics** | Multi-sheeted energy surfaces (Riemann sheets) |
| **String theory** | Riemann surfaces as worldsheets |
| **Algebraic geometry** | Riemann surfaces of algebraic curves |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Branch point | Point where analytic continuation produces different values |
| Branch cut | Curve removed to make function single-valued |
| Branch | Single-valued selection on cut domain |
| Riemann surface | Multi-sheeted surface making function globally single-valued |
| $\sqrt{z}$ | 2 sheets, branch point at $0$ and $\infty$ |
| $\log z$ | $\infty$ sheets (logarithmic branch point) |
| $z^{p/q}$ | $q$ sheets (algebraic branch point) |
| Choice of cut | Not unique; any curve from branch point to $\infty$ |

---

## Quick Revision Questions

1. **Define** branch point, branch cut, and branch precisely.

2. **Show** that going around $z = 0$ once changes $\sqrt{z}$ to $-\sqrt{z}$.

3. **Describe** the Riemann surface for $z^{1/3}$ — how many sheets and how are they connected?

4. **Identify** all branch points of $f(z) = \sqrt{z(z-1)(z-2)}$.

5. **Explain** why the Riemann surface for $\log z$ needs infinitely many sheets while $\sqrt{z}$ needs only two.

6. **Give** two different valid branch cuts for $\sqrt{z-1}$ and describe the domain of analyticity for each.

---

[← Previous: Power Functions](05-power-functions.md) | [Next: Unit 4 — Contours and Paths →](../04-Complex-Integration/01-contours-and-paths.md)
