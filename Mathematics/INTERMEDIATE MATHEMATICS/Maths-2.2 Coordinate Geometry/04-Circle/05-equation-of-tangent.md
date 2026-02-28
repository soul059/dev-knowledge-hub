# Chapter 4.5: Equation of Tangent to Circle

## 📚 Chapter Overview

A **tangent** to a circle is a line that touches the circle at exactly one point. This chapter develops multiple methods for finding tangent equations, including tangent at a point, tangent with given slope, and tangents from external points.

---

## 📝 Definition and Properties

> A **tangent** to a circle:
> - Touches the circle at exactly one point (point of tangency/contact)
> - Is perpendicular to the radius at the point of contact
> - Has distance from center equal to radius

```
                Tangent line
                    │
          ┌────────●──────────
         /         │T          \
        /          │            \
       │           │r            │
       │           ●C            │
       │        (center)         │
        \                       /
         \                     /
          └───────────────────┘
          
    CT ⊥ Tangent, CT = r
```

---

## ⚡ Tangent at a Point on Circle

### For Circle $x^2 + y^2 = r^2$

> **Tangent at $(x_1, y_1)$**:
>
> $$\boxed{xx_1 + yy_1 = r^2}$$

### For Circle $x^2 + y^2 + 2gx + 2fy + c = 0$

> **Tangent at $(x_1, y_1)$**:
>
> $$\boxed{xx_1 + yy_1 + g(x + x_1) + f(y + y_1) + c = 0}$$

---

## 📐 Derivation

### Method 1: Using Perpendicularity

For circle $x^2 + y^2 = r^2$ and point $(x_1, y_1)$ on it:

Slope of radius OT: $m_r = \frac{y_1}{x_1}$

Slope of tangent (perpendicular): $m_t = -\frac{x_1}{y_1}$

Tangent through $(x_1, y_1)$:
$$y - y_1 = -\frac{x_1}{y_1}(x - x_1)$$

$$yy_1 - y_1^2 = -xx_1 + x_1^2$$

$$xx_1 + yy_1 = x_1^2 + y_1^2 = r^2$$

### Method 2: Using Calculus

For $x^2 + y^2 = r^2$, differentiate implicitly:
$$2x + 2y\frac{dy}{dx} = 0$$
$$\frac{dy}{dx} = -\frac{x}{y}$$

At $(x_1, y_1)$: slope = $-\frac{x_1}{y_1}$

This leads to the same result.

---

## 📝 The "T = 0" Notation

For any conic, replacing:
- $x^2$ with $xx_1$
- $y^2$ with $yy_1$  
- $2x$ with $(x + x_1)$
- $2y$ with $(y + y_1)$
- $2xy$ with $(xy_1 + x_1y)$

gives the tangent equation at $(x_1, y_1)$.

```
    ┌────────────────────────────────────────────────────────┐
    │                    T = 0 RULE                          │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │   Circle: x² + y² + 2gx + 2fy + c = 0                  │
    │                                                        │
    │   Tangent at (x₁,y₁):                                  │
    │                                                        │
    │   xx₁ + yy₁ + g(x+x₁) + f(y+y₁) + c = 0                │
    │                                                        │
    │   Replace: x² → xx₁, y² → yy₁                          │
    │            2x → (x+x₁), 2y → (y+y₁)                    │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

---

## ✏️ Examples: Tangent at a Point

### Example 1: Tangent on Simple Circle

**Problem**: Find the equation of tangent to $x^2 + y^2 = 25$ at point (3, 4).

**Solution**:

Using $xx_1 + yy_1 = r^2$:

$$3x + 4y = 25$$

**Answer**: $3x + 4y = 25$

---

### Example 2: Tangent on General Circle

**Problem**: Find tangent to $x^2 + y^2 - 4x + 6y - 12 = 0$ at (5, 1).

**Solution**:

Here: $g = -2$, $f = 3$, $c = -12$

Using $xx_1 + yy_1 + g(x + x_1) + f(y + y_1) + c = 0$:

$$5x + y - 2(x + 5) + 3(y + 1) - 12 = 0$$

$$5x + y - 2x - 10 + 3y + 3 - 12 = 0$$

$$3x + 4y - 19 = 0$$

**Answer**: $3x + 4y = 19$

---

## ⚡ Tangent with Given Slope

### For Circle $x^2 + y^2 = r^2$

> **Tangent with slope m**:
>
> $$\boxed{y = mx \pm r\sqrt{1 + m^2}}$$

### For Circle $(x-h)^2 + (y-k)^2 = r^2$

> $$\boxed{(y - k) = m(x - h) \pm r\sqrt{1 + m^2}}$$

---

## 📐 Derivation: Tangent with Slope

Let tangent be $y = mx + c$.

For tangency, distance from center (0, 0) to line = radius:
$$\frac{|c|}{\sqrt{1 + m^2}} = r$$

$$c = \pm r\sqrt{1 + m^2}$$

Hence: $y = mx \pm r\sqrt{1 + m^2}$

---

### Example 3: Tangent with Given Slope

**Problem**: Find tangents to $x^2 + y^2 = 16$ with slope $\frac{3}{4}$.

**Solution**:

$r = 4$, $m = \frac{3}{4}$

$c = \pm 4\sqrt{1 + \frac{9}{16}} = \pm 4 \times \frac{5}{4} = \pm 5$

**Answer**: $y = \frac{3}{4}x + 5$ and $y = \frac{3}{4}x - 5$

Or: $3x - 4y + 20 = 0$ and $3x - 4y - 20 = 0$

---

### Example 4: Tangent Parallel to Given Line

**Problem**: Find tangents to $(x-1)^2 + (y-2)^2 = 9$ parallel to $3x + 4y = 7$.

**Solution**:

Slope of given line: $m = -\frac{3}{4}$

Center: (1, 2), Radius: 3

$(y - 2) = -\frac{3}{4}(x - 1) \pm 3\sqrt{1 + \frac{9}{16}}$

$(y - 2) = -\frac{3}{4}(x - 1) \pm 3 \times \frac{5}{4}$

$(y - 2) = -\frac{3}{4}(x - 1) \pm \frac{15}{4}$

Multiply by 4:
$4y - 8 = -3x + 3 \pm 15$

$3x + 4y = 11 + 15 = 26$ or $3x + 4y = 11 - 15 = -4$

**Answer**: $3x + 4y = 26$ and $3x + 4y = -4$

---

## ⚡ Tangents from External Point

### Finding Tangents from Point $(x_1, y_1)$ Outside Circle

**Method 1**: Using Slope Form

Let tangent be $y - y_1 = m(x - x_1)$, i.e., $mx - y + (y_1 - mx_1) = 0$

For tangency: $\frac{|mh - k + y_1 - mx_1|}{\sqrt{m^2 + 1}} = r$

Solve for m (gives two values).

**Method 2**: Using Chord of Contact

If tangents from P$(x_1, y_1)$ touch circle at A and B, then line AB (chord of contact) has equation:
$$T = 0$$
$$xx_1 + yy_1 + g(x + x_1) + f(y + y_1) + c = 0$$

```
              P(x₁,y₁)
             / \
            /   \
           /     \
          /       \
         A●───────●B
         │         │
         │    ●    │
         │  center │
          \       /
           \     /
            \   /
             \_/
             
    Line AB = Chord of Contact
```

---

### Example 5: Tangents from External Point

**Problem**: Find the equations of tangents from (4, 3) to circle $x^2 + y^2 = 9$.

**Solution**:

Let tangent be: $y - 3 = m(x - 4)$, i.e., $mx - y + 3 - 4m = 0$

Distance from (0, 0) = 3:
$$\frac{|3 - 4m|}{\sqrt{m^2 + 1}} = 3$$

$$(3 - 4m)^2 = 9(m^2 + 1)$$

$$9 - 24m + 16m^2 = 9m^2 + 9$$

$$7m^2 - 24m = 0$$

$$m(7m - 24) = 0$$

$m = 0$ or $m = \frac{24}{7}$

**Tangent 1** (m = 0): $y = 3$

**Tangent 2** (m = 24/7): $y - 3 = \frac{24}{7}(x - 4)$
$7y - 21 = 24x - 96$
$24x - 7y - 75 = 0$

**Answer**: $y = 3$ and $24x - 7y = 75$

---

### Example 6: Chord of Contact

**Problem**: Find the chord of contact of tangents from (5, 3) to circle $x^2 + y^2 = 16$.

**Solution**:

Chord of contact: $xx_1 + yy_1 = r^2$

$$5x + 3y = 16$$

**Answer**: $5x + 3y = 16$

---

## 📝 Pair of Tangents from External Point

The combined equation of pair of tangents from $(x_1, y_1)$ to circle $S = 0$ is:

> **Pair of Tangents**:
>
> $$\boxed{SS_1 = T^2}$$
>
> where:
> - $S = x^2 + y^2 + 2gx + 2fy + c$
> - $S_1 = x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c$
> - $T = xx_1 + yy_1 + g(x + x_1) + f(y + y_1) + c$

---

## 📝 Tangent at Point θ (Parametric)

For circle $x^2 + y^2 = r^2$, tangent at point $(r\cos\theta, r\sin\theta)$:

$$\boxed{x\cos\theta + y\sin\theta = r}$$

---

## 💡 Tips and Insights

> 💡 **T = 0 Rule**: The same substitution pattern works for all conics.

> 💡 **Two Tangents**: From any external point, exactly two tangents can be drawn.

> ⚠️ **Verify Point on Circle**: Before finding tangent at a point, verify the point lies on the circle.

> 💡 **Slope Form**: Gives two parallel tangents, one on each side of center.

---

## 🌍 Real-World Applications

1. **Optics**: Light ray tangent to curved mirror
2. **Engineering**: Belt around pulleys
3. **Architecture**: Smooth curve transitions
4. **Physics**: Instantaneous velocity direction
5. **Navigation**: Approach paths to circular runways

---

## 📋 Summary Table

| Scenario | Formula |
|----------|---------|
| Tangent at $(x_1, y_1)$ on $x^2 + y^2 = r^2$ | $xx_1 + yy_1 = r^2$ |
| Tangent at $(x_1, y_1)$ on general circle | $xx_1 + yy_1 + g(x+x_1) + f(y+y_1) + c = 0$ |
| Tangent with slope m to $x^2 + y^2 = r^2$ | $y = mx \pm r\sqrt{1+m^2}$ |
| Tangent at parameter θ | $x\cos\theta + y\sin\theta = r$ |
| Chord of contact | $T = 0$ |
| Pair of tangents | $SS_1 = T^2$ |
| Length of tangent | $\sqrt{S_1}$ |

---

## ❓ Quick Revision Questions

1. **Find tangent to $x^2 + y^2 = 13$ at (2, 3).**

2. **Find tangents to $x^2 + y^2 = 25$ with slope 2.**

3. **Find tangent to $x^2 + y^2 = 49$ at $\theta = \frac{\pi}{3}$.**

4. **Find chord of contact from (6, 8) to $x^2 + y^2 = 25$.**

5. **Find tangents from (5, 2) to $x^2 + y^2 = 20$.**

6. **Find tangent to $(x-1)^2 + (y-2)^2 = 4$ at point (1, 4).**

<details>
<summary><b>Click to see answers</b></summary>

1. $2x + 3y = 13$

2. $y = 2x \pm 5\sqrt{5}$
   Or: $2x - y \pm 5\sqrt{5} = 0$

3. $x\cos\frac{\pi}{3} + y\sin\frac{\pi}{3} = 7$
   $\frac{x}{2} + \frac{y\sqrt{3}}{2} = 7$
   **$x + \sqrt{3}y = 14$**

4. $6x + 8y = 25$

5. Let $y - 2 = m(x - 5)$, $mx - y + 2 - 5m = 0$
   $\frac{|2 - 5m|}{\sqrt{m^2+1}} = \sqrt{20}$
   $(2 - 5m)^2 = 20(m^2 + 1)$
   $4 - 20m + 25m^2 = 20m^2 + 20$
   $5m^2 - 20m - 16 = 0$
   $m = \frac{20 \pm \sqrt{400 + 320}}{10} = \frac{20 \pm 12\sqrt{5}}{10} = 2 \pm \frac{6\sqrt{5}}{5}$
   **Two tangents with these slopes through (5, 2)**

6. Point (1, 4): Verify $(1-1)^2 + (4-2)^2 = 0 + 4 = 4$ ✓
   Center (1, 2), radius at point is vertical
   **Tangent: $y = 4$**

</details>

---

## ⏭️ Navigation

| [← Previous: Position of Point and Line](04-point-line-position.md) | [Next: Equation of Normal →](06-equation-of-normal.md) |
|:-------------------------------------------------------------------|------------------------------------------------------:|
