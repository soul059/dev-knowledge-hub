# Chapter 4.1: Standard Form of Circle

## 📚 Chapter Overview

The **standard form** (also called center-radius form) is the most intuitive way to represent a circle. This chapter introduces the equation, its derivation, and various special cases.

---

## 📝 Definition of Circle

> A **circle** is the locus of all points in a plane that are at a fixed distance (radius) from a fixed point (center).

```
                    ●───────────●
                  /       r       \
                 /                 \
                ●         ●─────────● P(x,y)
               │      C(h,k)   r    │
                \                 /
                 \               /
                  \             /
                    ●─────────●
                    
    All points P are at distance r from center C
```

---

## ⚡ Standard Form Equation

> **Standard Form (Center-Radius Form)**:
>
> A circle with center $(h, k)$ and radius $r$ has equation:
>
> $$\boxed{(x - h)^2 + (y - k)^2 = r^2}$$

---

## 📐 Derivation

Let C$(h, k)$ be the center and $r$ be the radius.

For any point P$(x, y)$ on the circle:
$$\text{Distance CP} = r$$

Using distance formula:
$$\sqrt{(x - h)^2 + (y - k)^2} = r$$

Squaring both sides:
$$(x - h)^2 + (y - k)^2 = r^2$$

---

## 📝 Special Cases

### 1. Circle Centered at Origin

When center is $(0, 0)$:

$$\boxed{x^2 + y^2 = r^2}$$

```
        Y
        ↑
        │    ●───●
        │   /     \
        │  ●   ●   ●
        │   \  O  /
    ────┼────●───●────→ X
        │   /     \
        │  ●       ●
        │   \     /
        │    ●───●
```

### 2. Circle Passing Through Origin

If circle passes through $(0, 0)$:

Substituting in $(x - h)^2 + (y - k)^2 = r^2$:
$$h^2 + k^2 = r^2$$

### 3. Circle Touching X-axis

If circle touches x-axis:
$$r = |k|$$

Equation: $(x - h)^2 + (y - k)^2 = k^2$

```
        Y
        ↑
        │    ●───●
        │   /     \
        │  ●   ●   ●  Center (h,k)
        │   \     /
        │    ●───●
    ────┼────────●────→ X
                 ↑
           touches here
```

### 4. Circle Touching Y-axis

If circle touches y-axis:
$$r = |h|$$

Equation: $(x - h)^2 + (y - k)^2 = h^2$

### 5. Circle Touching Both Axes

If circle touches both axes:
$$r = |h| = |k|$$

Four possible positions depending on quadrant.

### 6. Circle with Diameter Endpoints

If $(x_1, y_1)$ and $(x_2, y_2)$ are endpoints of a diameter:

$$\boxed{(x - x_1)(x - x_2) + (y - y_1)(y - y_2) = 0}$$

**Derivation**: If P$(x, y)$ is any point on circle, angle APB = 90° (angle in semicircle).

So $\overrightarrow{PA} \cdot \overrightarrow{PB} = 0$, giving the equation.

---

## ✏️ Worked Examples

### Example 1: Writing Circle Equation

**Problem**: Write the equation of circle with center (3, -2) and radius 5.

**Solution**:

Using $(x - h)^2 + (y - k)^2 = r^2$:

$$(x - 3)^2 + (y - (-2))^2 = 5^2$$

$$(x - 3)^2 + (y + 2)^2 = 25$$

**Expanded form**:
$$x^2 - 6x + 9 + y^2 + 4y + 4 = 25$$
$$x^2 + y^2 - 6x + 4y - 12 = 0$$

---

### Example 2: Finding Center and Radius

**Problem**: Find the center and radius of $(x + 4)^2 + (y - 1)^2 = 49$.

**Solution**:

Rewrite as: $(x - (-4))^2 + (y - 1)^2 = 7^2$

**Center**: $(-4, 1)$
**Radius**: $7$

---

### Example 3: Circle Through Origin

**Problem**: Find the equation of circle with center (3, 4) passing through origin.

**Solution**:

**Step 1**: Find radius (distance from center to origin)
$$r = \sqrt{3^2 + 4^2} = \sqrt{25} = 5$$

**Step 2**: Write equation
$$(x - 3)^2 + (y - 4)^2 = 25$$

---

### Example 4: Circle Touching X-axis

**Problem**: Find the equation of circle with center (2, 3) touching the x-axis.

**Solution**:

Since circle touches x-axis, radius = |y-coordinate of center| = 3

$$(x - 2)^2 + (y - 3)^2 = 9$$

---

### Example 5: Circle with Given Diameter

**Problem**: Find the equation of circle with diameter endpoints A(1, 2) and B(5, 6).

**Solution**:

**Method 1**: Using diameter form
$$(x - 1)(x - 5) + (y - 2)(y - 6) = 0$$
$$x^2 - 6x + 5 + y^2 - 8y + 12 = 0$$
$$x^2 + y^2 - 6x - 8y + 17 = 0$$

**Method 2**: Find center and radius
Center = midpoint = $\left(\frac{1+5}{2}, \frac{2+6}{2}\right) = (3, 4)$
Diameter = $\sqrt{16 + 16} = 4\sqrt{2}$, so $r = 2\sqrt{2}$

$(x-3)^2 + (y-4)^2 = 8$

---

### Example 6: Circle Touching Both Axes

**Problem**: Find the equation of circle with radius 4, touching both coordinate axes, center in first quadrant.

**Solution**:

Since touches both axes and center in first quadrant:
- Center = $(4, 4)$
- Radius = $4$

$$(x - 4)^2 + (y - 4)^2 = 16$$

---

### Example 7: Circle Through Three Points

**Problem**: Find the equation of circle through points (0, 0), (1, 0), and (0, 1).

**Solution**:

Let equation be $(x - h)^2 + (y - k)^2 = r^2$

**From (0, 0)**: $h^2 + k^2 = r^2$ ... (1)
**From (1, 0)**: $(1-h)^2 + k^2 = r^2$ ... (2)
**From (0, 1)**: $h^2 + (1-k)^2 = r^2$ ... (3)

From (1) and (2):
$(1-h)^2 + k^2 = h^2 + k^2$
$1 - 2h + h^2 = h^2$
$h = \frac{1}{2}$

From (1) and (3):
$h^2 + (1-k)^2 = h^2 + k^2$
$1 - 2k = 0$
$k = \frac{1}{2}$

From (1): $r^2 = \frac{1}{4} + \frac{1}{4} = \frac{1}{2}$

**Answer**: $\left(x - \frac{1}{2}\right)^2 + \left(y - \frac{1}{2}\right)^2 = \frac{1}{2}$

---

## 📝 Parametric Equations

A circle with center $(h, k)$ and radius $r$ can be expressed parametrically:

$$\boxed{x = h + r\cos\theta, \quad y = k + r\sin\theta}$$

where $\theta \in [0, 2\pi)$ is the parameter (angle from positive x-direction).

```
        Y
        ↑
        │        ● P(h+rcosθ, k+rsinθ)
        │       /│
        │      / │
        │     /  │ r·sinθ
        │    /   │
        │   ●────┼───
        │  C(h,k) r·cosθ
    ────┼────────────────→ X
        │
```

---

## 💡 Tips and Insights

> 💡 **Sign Convention**: In $(x - h)^2$, the center x-coordinate is $h$, not $-h$.

> 💡 **Touching Axis**: When circle touches an axis, radius equals perpendicular distance from center to that axis.

> ⚠️ **Radius Must Be Positive**: If calculation gives $r^2 < 0$, the equation doesn't represent a real circle.

> 💡 **Quick Check**: A point lies on circle if it satisfies the equation exactly.

---

## 🌍 Real-World Applications

1. **Engineering**: Wheel design, gear teeth
2. **Architecture**: Circular arches, domes
3. **Physics**: Orbital paths, wave propagation
4. **Sports**: Track design, ball trajectories
5. **Manufacturing**: Precision circular parts
6. **Navigation**: Radar range circles

---

## 📋 Summary Table

| Circle Type | Center | Radius | Equation |
|-------------|--------|--------|----------|
| General | $(h, k)$ | $r$ | $(x-h)^2 + (y-k)^2 = r^2$ |
| At origin | $(0, 0)$ | $r$ | $x^2 + y^2 = r^2$ |
| Touching x-axis | $(h, r)$ | $r$ | $(x-h)^2 + (y-r)^2 = r^2$ |
| Touching y-axis | $(r, k)$ | $r$ | $(x-r)^2 + (y-k)^2 = r^2$ |
| Touching both axes | $(r, r)$ | $r$ | $(x-r)^2 + (y-r)^2 = r^2$ |
| Through origin | $(h, k)$ | $\sqrt{h^2+k^2}$ | $(x-h)^2 + (y-k)^2 = h^2+k^2$ |

---

## ❓ Quick Revision Questions

1. **Write the equation of circle with center (-2, 5) and radius 3.**

2. **Find center and radius of $(x - 1)^2 + (y + 3)^2 = 16$.**

3. **Find equation of circle with center (4, -3) passing through (1, 1).**

4. **A circle has diameter with endpoints (-1, 2) and (3, 4). Find its equation.**

5. **Find equation of circle touching both axes with radius 5, center in third quadrant.**

6. **The circle $x^2 + y^2 = 25$ is shifted so its center moves to (3, 4). Find new equation.**

<details>
<summary><b>Click to see answers</b></summary>

1. $(x + 2)^2 + (y - 5)^2 = 9$

2. Center = $(1, -3)$, Radius = $4$

3. $r = \sqrt{(4-1)^2 + (-3-1)^2} = \sqrt{9 + 16} = 5$
   **$(x - 4)^2 + (y + 3)^2 = 25$**

4. Center = $(1, 3)$
   $r = \frac{1}{2}\sqrt{16 + 4} = \sqrt{5}$
   **$(x - 1)^2 + (y - 3)^2 = 5$**
   Or: $(x + 1)(x - 3) + (y - 2)(y - 4) = 0$

5. Third quadrant: center = $(-5, -5)$
   **$(x + 5)^2 + (y + 5)^2 = 25$**

6. New center = $(3, 4)$, radius stays 5
   **$(x - 3)^2 + (y - 4)^2 = 25$**

</details>

---

## ⏭️ Navigation

| [← Back to Unit 4 Overview](README.md) | [Next: General Equation →](02-general-equation.md) |
|:--------------------------------------|--------------------------------------------------:|
