# Chapter 8.4: Straight Line in 3D

## 📚 Chapter Overview

Unlike in 2D where a single equation represents a line, in 3D a **straight line** requires either two equations (intersection of planes) or a special form called the **symmetric form**. This chapter explores various representations of lines in 3D space.

---

## 📝 Representations of a Line

A line in 3D can be defined by:

1. **Two points** on the line
2. **One point and direction ratios**
3. **Intersection of two planes**

---

## 📐 Symmetric Form (Cartesian Form)

### Line Through Point with Given DRs

A line passing through point $(x_1, y_1, z_1)$ with direction ratios $(a, b, c)$:

$$\boxed{\frac{x - x_1}{a} = \frac{y - y_1}{b} = \frac{z - z_1}{c}}$$

This is called the **symmetric form** or **standard form**.

```
                 Direction (a, b, c)
                      ↗
    ●────────────●───────────●────────→
    P₁         (x₁,y₁,z₁)     P₂
```

### Line Through Two Points

For line through $A(x_1, y_1, z_1)$ and $B(x_2, y_2, z_2)$:

$$\boxed{\frac{x - x_1}{x_2 - x_1} = \frac{y - y_1}{y_2 - y_1} = \frac{z - z_1}{z_2 - z_1}}$$

Here, DRs are $(x_2-x_1, y_2-y_1, z_2-z_1)$.

---

## ⚡ Parametric Form

From the symmetric form, let each ratio equal parameter $t$:

$$\frac{x - x_1}{a} = \frac{y - y_1}{b} = \frac{z - z_1}{c} = t$$

Then:

$$\boxed{x = x_1 + at, \quad y = y_1 + bt, \quad z = z_1 + ct}$$

where $t \in \mathbb{R}$ (any real number).

### General Point on Line

Any point on the line can be written as:

$(x_1 + at, y_1 + bt, z_1 + ct)$ for some value of $t$.

---

## 📝 Special Cases

### When One DR is Zero

If $a = 0$ (line parallel to yz-plane):

$$x = x_1, \quad \frac{y - y_1}{b} = \frac{z - z_1}{c}$$

### When Two DRs are Zero

If $a = b = 0$ (line parallel to z-axis):

$$x = x_1, \quad y = y_1$$

This is a line parallel to z-axis through $(x_1, y_1, 0)$.

---

## 📝 Line as Intersection of Two Planes

Two non-parallel planes intersect in a line:

$$a_1x + b_1y + c_1z + d_1 = 0$$
$$a_2x + b_2y + c_2z + d_2 = 0$$

### Converting to Symmetric Form

The DRs of the line of intersection are:

$$\boxed{(b_1c_2 - b_2c_1, c_1a_2 - c_2a_1, a_1b_2 - a_2b_1)}$$

This is the cross product of normal vectors: $\vec{n_1} \times \vec{n_2}$.

---

## ✏️ Worked Examples

### Example 1: Symmetric Form from Point and DRs

**Problem**: Write equation of line through $(2, 3, 4)$ with DRs $(1, -2, 3)$.

**Solution**:

$\frac{x - 2}{1} = \frac{y - 3}{-2} = \frac{z - 4}{3}$

**Answer**: $\frac{x-2}{1} = \frac{y-3}{-2} = \frac{z-4}{3}$

---

### Example 2: Line Through Two Points

**Problem**: Find equation of line through $(1, 2, 3)$ and $(4, 5, 6)$.

**Solution**:

DRs: $(4-1, 5-2, 6-3) = (3, 3, 3)$

Simplified DRs: $(1, 1, 1)$

$\frac{x - 1}{1} = \frac{y - 2}{1} = \frac{z - 3}{1}$

**Answer**: $\frac{x-1}{1} = \frac{y-2}{1} = \frac{z-3}{1}$ or $x - 1 = y - 2 = z - 3$

---

### Example 3: Parametric Form

**Problem**: Write parametric equations for line $\frac{x-1}{2} = \frac{y+2}{3} = \frac{z-3}{-1}$.

**Solution**:

Let $\frac{x-1}{2} = \frac{y+2}{3} = \frac{z-3}{-1} = t$

$x = 1 + 2t$
$y = -2 + 3t$
$z = 3 - t$

**Answer**: $x = 1 + 2t$, $y = -2 + 3t$, $z = 3 - t$

---

### Example 4: Point on Line

**Problem**: Find the point on line $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$ at distance 5 from $(1, 2, 3)$.

**Solution**:

Any point on line: $(1 + 2t, 2 + 3t, 3 + 4t)$

Distance from $(1, 2, 3)$:

$\sqrt{(2t)^2 + (3t)^2 + (4t)^2} = 5$

$\sqrt{4t^2 + 9t^2 + 16t^2} = 5$

$\sqrt{29t^2} = 5$

$|t|\sqrt{29} = 5$

$t = \pm\frac{5}{\sqrt{29}}$

Points: $\left(1 + \frac{10}{\sqrt{29}}, 2 + \frac{15}{\sqrt{29}}, 3 + \frac{20}{\sqrt{29}}\right)$

and $\left(1 - \frac{10}{\sqrt{29}}, 2 - \frac{15}{\sqrt{29}}, 3 - \frac{20}{\sqrt{29}}\right)$

**Answer**: Two points at distance 5 on either side.

---

### Example 5: Line Parallel to Axis

**Problem**: Write equation of line through $(2, 3, 4)$ parallel to z-axis.

**Solution**:

DRs parallel to z-axis: $(0, 0, 1)$

Line: $\frac{x-2}{0} = \frac{y-3}{0} = \frac{z-4}{1}$

This means: $x = 2$, $y = 3$ (z is free)

**Answer**: $x = 2$, $y = 3$ (or parametrically: $x = 2$, $y = 3$, $z = 4 + t$)

---

### Example 6: Line from Two Planes

**Problem**: Find symmetric form of line: $x + y + z = 1$ and $2x - y + z = 3$.

**Solution**:

Normal vectors: $\vec{n_1} = (1, 1, 1)$, $\vec{n_2} = (2, -1, 1)$

DRs of line = $\vec{n_1} \times \vec{n_2}$:

$= (1 \cdot 1 - 1 \cdot (-1), 1 \cdot 2 - 1 \cdot 1, 1 \cdot (-1) - 1 \cdot 2)$
$= (1 + 1, 2 - 1, -1 - 2)$
$= (2, 1, -3)$

Find a point on both planes:
Let $z = 0$: $x + y = 1$ and $2x - y = 3$
Adding: $3x = 4 \Rightarrow x = \frac{4}{3}$
$y = 1 - \frac{4}{3} = -\frac{1}{3}$

Point: $\left(\frac{4}{3}, -\frac{1}{3}, 0\right)$

**Answer**: $\frac{x - \frac{4}{3}}{2} = \frac{y + \frac{1}{3}}{1} = \frac{z}{-3}$

---

### Example 7: Angle Between Lines

**Problem**: Find angle between lines:
$\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$ and $\frac{x-2}{1} = \frac{y-3}{2} = \frac{z-4}{-1}$

**Solution**:

DRs: $(2, 3, 4)$ and $(1, 2, -1)$

$\cos\theta = \frac{2(1) + 3(2) + 4(-1)}{\sqrt{4+9+16} \cdot \sqrt{1+4+1}}$

$= \frac{2 + 6 - 4}{\sqrt{29} \cdot \sqrt{6}} = \frac{4}{\sqrt{174}}$

$\theta = \cos^{-1}\left(\frac{4}{\sqrt{174}}\right)$

**Answer**: $\theta = \cos^{-1}\left(\frac{4}{\sqrt{174}}\right) \approx 72.3°$

---

### Example 8: Coplanar Lines

**Problem**: Check if lines $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$ and $\frac{x-2}{1} = \frac{y-4}{1} = \frac{z-5}{1}$ are coplanar.

**Solution**:

For coplanar lines:
$$\begin{vmatrix} x_2-x_1 & y_2-y_1 & z_2-z_1 \\ a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \end{vmatrix} = 0$$

Points: $(1, 2, 3)$ and $(2, 4, 5)$

$\begin{vmatrix} 1 & 2 & 2 \\ 2 & 3 & 4 \\ 1 & 1 & 1 \end{vmatrix}$

$= 1(3-4) - 2(2-4) + 2(2-3)$
$= 1(-1) - 2(-2) + 2(-1)$
$= -1 + 4 - 2 = 1 \neq 0$

**Answer**: Lines are **NOT coplanar** (they are skew lines)

---

## 📐 Skew Lines

Two lines in 3D that are neither parallel nor intersecting are called **skew lines**.

```
                Line 1
         ●─────────────●
        /
       / (lines don't 
      /   intersect)
     ●─────────────●
           Line 2
```

### Shortest Distance Between Skew Lines

For lines:
$\frac{x-x_1}{a_1} = \frac{y-y_1}{b_1} = \frac{z-z_1}{c_1}$ and $\frac{x-x_2}{a_2} = \frac{y-y_2}{b_2} = \frac{z-z_2}{c_2}$

$$\boxed{d = \frac{\begin{vmatrix} x_2-x_1 & y_2-y_1 & z_2-z_1 \\ a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \end{vmatrix}}{\sqrt{(b_1c_2-b_2c_1)^2 + (c_1a_2-c_2a_1)^2 + (a_1b_2-a_2b_1)^2}}}$$

---

## 📊 Summary Table

| Form | Equation |
|------|----------|
| Symmetric | $\frac{x-x_1}{a} = \frac{y-y_1}{b} = \frac{z-z_1}{c}$ |
| Two-point | $\frac{x-x_1}{x_2-x_1} = \frac{y-y_1}{y_2-y_1} = \frac{z-z_1}{z_2-z_1}$ |
| Parametric | $x = x_1 + at$, $y = y_1 + bt$, $z = z_1 + ct$ |
| General point | $(x_1 + at, y_1 + bt, z_1 + ct)$ |

### Line Relationships

| Condition | Test |
|-----------|------|
| Parallel | DRs proportional |
| Perpendicular | $a_1a_2 + b_1b_2 + c_1c_2 = 0$ |
| Coplanar | Determinant = 0 |
| Skew | Not parallel, not intersecting |

---

## 💡 Tips and Insights

> 💡 **Parametric is Universal**: Any point on the line can be found by choosing appropriate $t$.

> ⚠️ **Zero in Denominator**: When a DR is zero, that coordinate stays constant.

> 💡 **Cross Product**: DRs of line of intersection = cross product of plane normals.

> 💡 **General Point**: Use $(x_1 + at, y_1 + bt, z_1 + ct)$ to find specific points satisfying given conditions.

---

## ❓ Quick Revision Questions

1. **Write equation of line through $(1, 2, 3)$ with DRs $(2, -1, 3)$.**

2. **Find parametric equations for $\frac{x+1}{2} = \frac{y-3}{4} = \frac{z}{-1}$.**

3. **Find the point on $\frac{x-1}{1} = \frac{y-2}{2} = \frac{z-3}{3}$ where $x = 3$.**

4. **Are lines $\frac{x}{1} = \frac{y}{2} = \frac{z}{3}$ and $\frac{x}{2} = \frac{y}{4} = \frac{z}{6}$ same or different?**

5. **Write equation of line through origin parallel to line $\frac{x-1}{2} = \frac{y+1}{3} = \frac{z-2}{4}$.**

6. **Find DRs of line of intersection of planes $x + y + z = 1$ and $x - y + z = 2$.**

<details>
<summary><b>Click to see answers</b></summary>

1. **$\frac{x-1}{2} = \frac{y-2}{-1} = \frac{z-3}{3}$**

2. Let ratio = $t$:
   **$x = -1 + 2t$, $y = 3 + 4t$, $z = -t$**

3. $\frac{3-1}{1} = 2 = \frac{y-2}{2} = \frac{z-3}{3}$
   $y - 2 = 4 \Rightarrow y = 6$
   $z - 3 = 6 \Rightarrow z = 9$
   **$(3, 6, 9)$**

4. DRs of first: $(1, 2, 3)$
   DRs of second: $(2, 4, 6) = 2(1, 2, 3)$
   Same direction, but first passes through origin, second... let's check:
   Second line: points satisfying $\frac{x}{2} = \frac{y}{4}$ gives $2x = y$, and from $\frac{y}{4} = \frac{z}{6}$ gives $3y = 2z$.
   At $x = 0$: $y = 0$, $z = 0$. So origin is on both!
   **Same line** (just different parameterization)

5. Same DRs $(2, 3, 4)$, through origin:
   **$\frac{x}{2} = \frac{y}{3} = \frac{z}{4}$**

6. Normals: $(1, 1, 1)$ and $(1, -1, 1)$
   Cross product: $(1 \cdot 1 - 1 \cdot (-1), 1 \cdot 1 - 1 \cdot 1, 1 \cdot (-1) - 1 \cdot 1)$
   $= (1 + 1, 1 - 1, -1 - 1) = (2, 0, -2)$
   Simplified: **$(1, 0, -1)$**

</details>

---

## ⏭️ Navigation

| [← Previous: Direction Cosines](03-direction-cosines.md) | [Unit 8 Home](README.md) | [Next: Equation of Plane →](05-equation-of-plane.md) |
|:--------------------------------------------------------:|:------------------------:|----------------------------------------------------:|
