# Chapter 2.3: Equations Reducible to Homogeneous

[← Previous: Homogeneous Equations](02-homogeneous-equations.md) | [Back to README](../README.md) | [Next: Linear Equations →](04-linear-equations.md)

---

## Overview

Many first-order ODEs of the form $\dfrac{dy}{dx} = \dfrac{ax + by + c}{dx + ey + f}$ look almost homogeneous but have **constant terms** $c$ and $f$. By a suitable **translation of axes**, we can reduce these to homogeneous form.

---

## 1. The Problem

The equation $\dfrac{dy}{dx} = F\!\left(\dfrac{ax + by + c}{dx + ey + f}\right)$ is NOT homogeneous because of the constants $c, f$.

**Goal**: Shift the origin to eliminate the constants.

---

## 2. Case 1: $\dfrac{a}{d} \neq \dfrac{b}{e}$ (Lines Intersect)

### Method

The two lines $ax + by + c = 0$ and $dx + ey + f = 0$ intersect at some point $(h, k)$.

**Step 1**: Solve the system:
$$ah + bk + c = 0$$
$$dh + ek + f = 0$$

**Step 2**: Substitute $X = x - h$, $Y = y - k$ (shift origin to $(h, k)$):
$$dX = dx, \quad dY = dy$$

**Step 3**: The equation becomes:
$$\frac{dY}{dX} = \frac{aX + bY}{dX + eY}$$

This is now **homogeneous**!

**Step 4**: Solve using $Y = VX$ substitution.

**Step 5**: Back-substitute $X = x - h$, $Y = y - k$.

### Worked Example

**Solve**: $\dfrac{dy}{dx} = \dfrac{x - y + 3}{x + y - 1}$

Here $a = 1, b = -1, c = 3, d = 1, e = 1, f = -1$.

Check: $a/d = 1$, $b/e = -1$ → not equal → **Case 1** applies.

**Step 1** — Find intersection:
$$h - k + 3 = 0 \quad \ldots (i)$$
$$h + k - 1 = 0 \quad \ldots (ii)$$

Add: $2h + 2 = 0 \implies h = -1$
From (ii): $-1 + k = 1 \implies k = 2$

**Step 2** — Let $X = x + 1$, $Y = y - 2$:
$$\frac{dY}{dX} = \frac{X - Y}{X + Y}$$

**Step 3** — Now homogeneous. Let $Y = VX$:
$$V + X\frac{dV}{dX} = \frac{1 - V}{1 + V}$$

$$X\frac{dV}{dX} = \frac{1 - V}{1 + V} - V = \frac{1 - V - V - V^2}{1 + V} = \frac{1 - 2V - V^2}{1 + V}$$

**Step 4** — Separate:
$$\frac{(1+V) \, dV}{1 - 2V - V^2} = \frac{dX}{X}$$

Note: $1 - 2V - V^2 = 2 - (1 + V)^2$. Let $u = 1 + V$:

$$\frac{u \, du}{2 - u^2} = \frac{dX}{X}$$

$$-\frac{1}{2}\ln|2 - u^2| = \ln|X| + C_0$$

$$\ln|2 - u^2| = -2\ln|X| + C_1$$

$$(2 - u^2)X^2 = C$$

**Step 5** — Back-substitute: $u = 1 + V = 1 + Y/X = (X + Y)/X$

$$(2 - (X+Y)^2/X^2) \cdot X^2 = C$$

$$2X^2 - (X+Y)^2 = C$$

Now $X = x+1$, $Y = y-2$, $X + Y = x + y - 1$:

$$2(x+1)^2 - (x+y-1)^2 = C$$

---

## 3. Case 2: $\dfrac{a}{d} = \dfrac{b}{e}$ (Lines Parallel)

When the coefficients are proportional, the lines are **parallel** and don't intersect. The translation method fails.

### Method

If $\dfrac{a}{d} = \dfrac{b}{e} = \dfrac{1}{m}$ (say), then $a = d/m$, $b = e/m$.

Let $v = dx + ey$ (grouping the common linear part).

Then $\dfrac{dv}{dx} = d + e\dfrac{dy}{dx}$, so $\dfrac{dy}{dx} = \dfrac{1}{e}\left(\dfrac{dv}{dx} - d\right)$.

Substitute and the equation becomes separable in $v$ and $x$.

### Worked Example

**Solve**: $\dfrac{dy}{dx} = \dfrac{2x + 4y + 1}{x + 2y + 3}$

Check: $a/d = 2/1 = 2$, $b/e = 4/2 = 2$ → equal → **Case 2**.

**Step 1** — Notice numerator = $2(\text{denominator}) - 5$:
$$\frac{dy}{dx} = \frac{2(x + 2y + 3) - 5}{x + 2y + 3} = 2 - \frac{5}{x + 2y + 3}$$

**Step 2** — Let $v = x + 2y$. Then $\dfrac{dv}{dx} = 1 + 2\dfrac{dy}{dx}$.

$$\frac{dy}{dx} = \frac{1}{2}\left(\frac{dv}{dx} - 1\right)$$

**Step 3** — Substitute:
$$\frac{1}{2}\left(\frac{dv}{dx} - 1\right) = 2 - \frac{5}{v + 3}$$

$$\frac{dv}{dx} = 5 - \frac{10}{v + 3} = \frac{5(v+3) - 10}{v + 3} = \frac{5v + 5}{v + 3}$$

**Step 4** — Separate:
$$\frac{v + 3}{5v + 5} \, dv = dx$$

$$\frac{v + 3}{5(v + 1)} \, dv = dx$$

Split by partial fractions: $\dfrac{v+3}{5(v+1)} = \dfrac{1}{5} + \dfrac{2}{5(v+1)}$

**Step 5** — Integrate:
$$\frac{1}{5}v + \frac{2}{5}\ln|v+1| = x + C$$

$$v + 2\ln|v+1| = 5x + C'$$

**Step 6** — Back-substitute $v = x + 2y$:

$$(x + 2y) + 2\ln|x + 2y + 1| = 5x + C$$

$$2y - 4x + 2\ln|x + 2y + 1| = C$$

---

## 4. Decision Tree

```
Given: dy/dx = (ax + by + c)/(dx + ey + f)
                    │
                    ▼
            Is a/d = b/e ?
           ┌────┴────┐
          NO         YES
           │          │
           ▼          ▼
        Case 1:     Case 2:
        Lines       Lines
        intersect   parallel
           │          │
           ▼          ▼
        Find (h,k)  Let v = dx + ey
        X = x-h     (or v = ax + by)
        Y = y-k     
           │          │
           ▼          ▼
        Homogeneous  Separable
        in X, Y      in v, x
           │          │
           ▼          ▼
        Y = VX      Integrate
           │          │
           ▼          ▼
        Separate    Back-
        & Integrate substitute
           │
           ▼
        Back-substitute
        X, Y, V → x, y
```

---

## 5. Quick Reference: When to Use What

| Condition | Lines | Method | Substitution |
|-----------|-------|--------|-------------|
| $a/d \neq b/e$ | Intersect at $(h,k)$ | Translate origin | $X = x-h$, $Y = y-k$ |
| $a/d = b/e$, $c/f = a/d$ | Identical (same line) | Simplify algebraically | Direct simplification |
| $a/d = b/e$, $c/f \neq a/d$ | Parallel (distinct) | Substitution | $v = ax + by$ or $v = dx + ey$ |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Form** | $dy/dx = (ax + by + c)/(dx + ey + f)$ |
| **Case 1** | $a/d \neq b/e$: translate axes to intersection point |
| **Case 2** | $a/d = b/e$: substitute $v = ax + by$ |
| **After Case 1** | Becomes homogeneous → use $Y = VX$ |
| **After Case 2** | Becomes separable → integrate directly |
| **Key skill** | Solving $2 \times 2$ linear system to find $(h, k)$ |

---

## Quick Revision Questions

1. Classify and solve: $\dfrac{dy}{dx} = \dfrac{x + y + 1}{x + y + 3}$. Is this Case 1 or Case 2?

2. Find the point $(h, k)$ for: $\dfrac{dy}{dx} = \dfrac{2x - y + 1}{4x + y - 3}$

3. Why can't the translation method be applied when the lines are parallel?

4. Solve: $\dfrac{dy}{dx} = \dfrac{3x + 6y + 2}{x + 2y + 1}$

5. What substitution would you use for $\dfrac{dy}{dx} = \dfrac{x - 2y + 5}{2x - 4y - 1}$?

6. Verify that shifting the origin to $(-1, 2)$ eliminates the constants from $\dfrac{dy}{dx} = \dfrac{x - y + 3}{x + y - 1}$.

---

[← Previous: Homogeneous Equations](02-homogeneous-equations.md) | [Back to README](../README.md) | [Next: Linear Equations →](04-linear-equations.md)
