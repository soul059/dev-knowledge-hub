# Chapter 4.6: Equation of Normal to Circle

## 📚 Chapter Overview

The **normal** to a circle at any point is the line perpendicular to the tangent at that point. For circles, normals have a special property: they always pass through the center. This chapter explores normal equations and their applications.

---

## 📝 Definition and Properties

> A **normal** to a circle at a point:
> - Is perpendicular to the tangent at that point
> - Passes through the center of the circle
> - Is the line containing the radius to that point

```
                    Tangent
                       │
              ─────────●P──────────
                      /│
                     / │ Normal
                    /  │
                   /   │
                  /    ●C (center)
                 /
                /
               ↙
          Normal line
          
    Normal = Line through C and P
    Normal ⊥ Tangent at P
```

---

## ⚡ Normal Equation Formulas

### For Circle $x^2 + y^2 = r^2$

> **Normal at $(x_1, y_1)$**:
>
> $$\boxed{\frac{x}{x_1} = \frac{y}{y_1}}$$
>
> Or equivalently: $xy_1 - x_1y = 0$ (passes through origin)

### For Circle $x^2 + y^2 + 2gx + 2fy + c = 0$

> **Normal at $(x_1, y_1)$**:
>
> $$\boxed{\frac{x - x_1}{x_1 + g} = \frac{y - y_1}{y_1 + f}}$$

---

## 📐 Derivation

### For Circle Centered at Origin

Center: $(0, 0)$, Point on circle: $(x_1, y_1)$

The normal is the line joining center to the point:
$$\frac{x - 0}{x_1 - 0} = \frac{y - 0}{y_1 - 0}$$

$$\frac{x}{x_1} = \frac{y}{y_1}$$

### For General Circle

Center: $(-g, -f)$, Point on circle: $(x_1, y_1)$

The normal is the line joining center to the point:
$$\frac{x - x_1}{x_1 - (-g)} = \frac{y - y_1}{y_1 - (-f)}$$

$$\frac{x - x_1}{x_1 + g} = \frac{y - y_1}{y_1 + f}$$

---

## 📝 Alternative Forms of Normal

### Using Slope

At point $(x_1, y_1)$ on circle $x^2 + y^2 = r^2$:

- Slope of tangent: $-\frac{x_1}{y_1}$
- Slope of normal: $\frac{y_1}{x_1}$ (perpendicular)

Normal equation: $y - y_1 = \frac{y_1}{x_1}(x - x_1)$

Simplifying: $xy_1 - y_1 x_1 = x_1 y - x_1 y_1$

$$xy_1 = x_1 y$$

### Parametric Form

For circle $x^2 + y^2 = r^2$, at point $(r\cos\theta, r\sin\theta)$:

$$\boxed{y = x\tan\theta}$$

Or: $x\sin\theta - y\cos\theta = 0$

---

## ✏️ Worked Examples

### Example 1: Normal at Point (Simple Circle)

**Problem**: Find the equation of normal to $x^2 + y^2 = 25$ at point (3, 4).

**Solution**:

**Method 1**: Using formula
$$\frac{x}{3} = \frac{y}{4}$$
$$4x = 3y$$
$$4x - 3y = 0$$

**Method 2**: Line through (0,0) and (3,4)
Slope = $\frac{4}{3}$
$y = \frac{4}{3}x$

**Answer**: $4x - 3y = 0$ or $y = \frac{4}{3}x$

---

### Example 2: Normal at Point (General Circle)

**Problem**: Find normal to $x^2 + y^2 - 4x + 6y - 12 = 0$ at (5, 1).

**Solution**:

Here: $g = -2$, $f = 3$

Using $\frac{x - x_1}{x_1 + g} = \frac{y - y_1}{y_1 + f}$:

$$\frac{x - 5}{5 + (-2)} = \frac{y - 1}{1 + 3}$$

$$\frac{x - 5}{3} = \frac{y - 1}{4}$$

$$4(x - 5) = 3(y - 1)$$

$$4x - 20 = 3y - 3$$

$$4x - 3y = 17$$

**Verification**: This line passes through center $(2, -3)$:
$4(2) - 3(-3) = 8 + 9 = 17$ ✓

---

### Example 3: Normal at Parameter θ

**Problem**: Find normal to $x^2 + y^2 = 16$ at $\theta = \frac{\pi}{6}$.

**Solution**:

Point: $(4\cos\frac{\pi}{6}, 4\sin\frac{\pi}{6}) = (2\sqrt{3}, 2)$

Normal: $y = x\tan\frac{\pi}{6} = \frac{x}{\sqrt{3}}$

$$\sqrt{3}y = x$$

**Answer**: $x - \sqrt{3}y = 0$

---

### Example 4: Normal Parallel to Given Line

**Problem**: Find the point on circle $x^2 + y^2 = 52$ where normal is parallel to line $2x + 3y = 9$.

**Solution**:

Slope of given line: $m = -\frac{2}{3}$

For normal at $(x_1, y_1)$ on $x^2 + y^2 = r^2$:
Slope of normal = $\frac{y_1}{x_1} = -\frac{2}{3}$

So: $y_1 = -\frac{2}{3}x_1$

Also, $(x_1, y_1)$ lies on circle:
$$x_1^2 + \frac{4}{9}x_1^2 = 52$$
$$\frac{13x_1^2}{9} = 52$$
$$x_1^2 = 36$$
$$x_1 = \pm 6$$

If $x_1 = 6$: $y_1 = -4$ → Point (6, -4)
If $x_1 = -6$: $y_1 = 4$ → Point (-6, 4)

**Answer**: Points are $(6, -4)$ and $(-6, 4)$

---

### Example 5: Normal Through External Point

**Problem**: Find the normals to $x^2 + y^2 = 9$ that pass through point (5, 0).

**Solution**:

Any normal to this circle passes through center (0, 0).

For normal through (5, 0) and (0, 0):
The line is simply $y = 0$ (x-axis).

But let's verify: Does the normal (x-axis) meet the circle?
At $y = 0$: $x^2 = 9$, so $x = \pm 3$

Points of normalcy: $(3, 0)$ and $(-3, 0)$

**Answer**: $y = 0$ (only one line, which is normal at both $(\pm 3, 0)$)

---

### Example 6: Number of Normals from a Point

**Problem**: How many normals can be drawn from point (4, 3) to circle $x^2 + y^2 = 4$?

**Solution**:

Every normal passes through center (0, 0).

Line through (0, 0) and (4, 3): $y = \frac{3}{4}x$, i.e., $3x - 4y = 0$

This line intersects circle at:
Substitute $y = \frac{3x}{4}$: $x^2 + \frac{9x^2}{16} = 4$
$\frac{25x^2}{16} = 4$
$x^2 = \frac{64}{25}$
$x = \pm\frac{8}{5}$

So points are $\left(\frac{8}{5}, \frac{6}{5}\right)$ and $\left(-\frac{8}{5}, -\frac{6}{5}\right)$

But wait - can we draw normals from (4, 3) to both these points?

Check: Is (4, 3) on the line joining center to each point?
Line through $(0,0)$ and $(\frac{8}{5}, \frac{6}{5})$: passes through (4, 3)? 
$\frac{3}{4} = \frac{6/5}{8/5} = \frac{6}{8} = \frac{3}{4}$ ✓

Both normals are on the same line through center!

**Answer**: **1 normal** (the line through center and external point)

Actually, let me reconsider. From an external point, we can draw normals to multiple points where the "normal from those points passes through the external point."

Any point P on circle has its normal passing through center. For this normal to also pass through (4, 3), the center (0, 0), point P, and (4, 3) must be collinear.

So only points on line OQ (where Q = (4,3)) qualify. This gives 2 points on circle.

**Corrected Answer**: **2 normals** from (4, 3), at points $\left(\frac{8}{5}, \frac{6}{5}\right)$ and $\left(-\frac{8}{5}, -\frac{6}{5}\right)$

---

## 📝 Key Properties of Normals

### 1. Normals Always Pass Through Center

This is unique to circles. For other conics, normals don't generally pass through the center.

### 2. Normals from External Point

From a point outside the circle:
- If point is at distance > 2r from center: 2 normals (both on same side)
- If point is at distance ≤ 2r: varies based on position

### 3. Foot of Normal

The point where normal meets the circle is called the **foot of normal**.

---

## 📝 Relationship Between Tangent and Normal

```
    ┌────────────────────────────────────────────────────────┐
    │              TANGENT vs NORMAL at Point P              │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │   Property          Tangent           Normal           │
    │   ────────          ───────           ──────           │
    │   Direction         ⊥ to radius       Along radius     │
    │   Passes through    P only            P and center     │
    │   Slope (at P)      -x₁/y₁            y₁/x₁            │
    │   Equation          xx₁+yy₁=r²        xy₁=x₁y          │
    │                                                        │
    │   Product of slopes = -1 (perpendicular)               │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

---

## 💡 Tips and Insights

> 💡 **Simple Rule**: Normal to circle = Line through center and point of contact.

> 💡 **Slope Relationship**: If tangent slope is m, normal slope is $-\frac{1}{m}$.

> ⚠️ **Special Points**: At points where $x_1 = 0$ or $y_1 = 0$, use the appropriate form (vertical/horizontal lines).

> 💡 **Origin Circle**: For $x^2 + y^2 = r^2$, all normals pass through origin.

---

## 🌍 Real-World Applications

1. **Optics**: Reflection from curved mirrors
2. **Mechanics**: Force direction at contact points
3. **Engineering**: Stress analysis in circular structures
4. **Physics**: Centripetal force direction
5. **Manufacturing**: Tool orientation for circular cutting

---

## 📋 Summary Table

| Circle | Normal Equation at $(x_1, y_1)$ |
|--------|--------------------------------|
| $x^2 + y^2 = r^2$ | $\frac{x}{x_1} = \frac{y}{y_1}$ or $xy_1 - x_1y = 0$ |
| $(x-h)^2 + (y-k)^2 = r^2$ | $\frac{x-x_1}{x_1-h} = \frac{y-y_1}{y_1-k}$ |
| General form | $\frac{x-x_1}{x_1+g} = \frac{y-y_1}{y_1+f}$ |
| At parameter θ | $y = x\tan\theta$ (for origin-centered) |
| Slope of normal | $\frac{y_1}{x_1}$ (origin-centered) |

---

## ❓ Quick Revision Questions

1. **Find normal to $x^2 + y^2 = 36$ at (0, 6).**

2. **Find normal to $x^2 + y^2 = 50$ at point (5, 5).**

3. **Find normal to $x^2 + y^2 - 2x - 4y = 0$ at origin.**

4. **At what points on $x^2 + y^2 = 25$ is the normal parallel to $x + 2y = 3$?**

5. **Find equation of normal at $\theta = \frac{2\pi}{3}$ on $x^2 + y^2 = 9$.**

6. **The normal at point (4, 3) to a circle passes through (8, 6). Find center if radius = 5.**

<details>
<summary><b>Click to see answers</b></summary>

1. At (0, 6), x-coordinate is 0
   Normal is vertical line: **$x = 0$** (y-axis)

2. $\frac{x}{5} = \frac{y}{5}$
   **$x = y$** or **$x - y = 0$**

3. First verify (0,0) is on circle: $0 - 0 - 0 = 0$ ✓
   $g = -1$, $f = -2$
   $\frac{x - 0}{0 + (-1)} = \frac{y - 0}{0 + (-2)}$
   $\frac{x}{-1} = \frac{y}{-2}$
   **$2x - y = 0$** or **$y = 2x$**

4. Normal slope = $\frac{y_1}{x_1} = -\frac{1}{2}$ (parallel to given line)
   So $y_1 = -\frac{x_1}{2}$
   $x_1^2 + \frac{x_1^2}{4} = 25$
   $\frac{5x_1^2}{4} = 25$
   $x_1 = \pm 2\sqrt{5}$
   Points: **$(2\sqrt{5}, -\sqrt{5})$ and $(-2\sqrt{5}, \sqrt{5})$**

5. $y = x\tan\frac{2\pi}{3} = -\sqrt{3}x$
   **$\sqrt{3}x + y = 0$**

6. Normal passes through (4, 3) and (8, 6), so center is on this line.
   Line: $\frac{y-3}{x-4} = \frac{6-3}{8-4} = \frac{3}{4}$
   $y - 3 = \frac{3}{4}(x - 4)$, i.e., $3x - 4y = 0$
   Center $(h, k)$ satisfies $3h - 4k = 0$, so $k = \frac{3h}{4}$
   Distance from center to (4, 3) = 5:
   $(h-4)^2 + (k-3)^2 = 25$
   $(h-4)^2 + (\frac{3h}{4}-3)^2 = 25$
   $(h-4)^2 + \frac{9(h-4)^2}{16} = 25$
   $\frac{25(h-4)^2}{16} = 25$
   $(h-4)^2 = 16$
   $h = 8$ or $h = 0$
   **Centers: $(8, 6)$ or $(0, 0)$**

</details>

---

## ⏭️ Navigation

| [← Previous: Equation of Tangent](05-equation-of-tangent.md) | [Next: Family of Circles →](07-family-of-circles.md) |
|:------------------------------------------------------------|----------------------------------------------------:|
