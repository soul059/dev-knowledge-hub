# Chapter 8.5: Equation of Plane

## 📚 Chapter Overview

A **plane** is a flat, two-dimensional surface that extends infinitely in 3D space. This chapter explores various forms of plane equations, including general form, normal form, intercept form, and how to find angles between planes.

---

## 📝 General Equation of Plane

The most general form of a plane equation is:

$$\boxed{ax + by + cz + d = 0}$$

where $(a, b, c)$ are the **direction ratios of the normal** to the plane.

```
              Normal vector (a,b,c)
                    ↑
                   /
                  /
    ─────────────/──────────── Plane
               / │
              /  │
                 │
                 
   The normal is perpendicular to the plane
```

---

## 📐 Normal Form

If the perpendicular distance from origin to plane is $p$, and the normal has direction cosines $(l, m, n)$:

$$\boxed{lx + my + nz = p}$$

where $l^2 + m^2 + n^2 = 1$ and $p > 0$.

### Converting General to Normal Form

From $ax + by + cz + d = 0$:

$$\frac{ax + by + cz + d}{\pm\sqrt{a^2+b^2+c^2}} = 0$$

$$\frac{a}{\sqrt{a^2+b^2+c^2}}x + \frac{b}{\sqrt{a^2+b^2+c^2}}y + \frac{c}{\sqrt{a^2+b^2+c^2}}z = \frac{-d}{\sqrt{a^2+b^2+c^2}}$$

Choose sign so that $p > 0$.

---

## ⚡ Point-Normal Form

Plane through point $(x_1, y_1, z_1)$ with normal direction $(a, b, c)$:

$$\boxed{a(x - x_1) + b(y - y_1) + c(z - z_1) = 0}$$

This expands to: $ax + by + cz = ax_1 + by_1 + cz_1$

---

## 📝 Intercept Form

If a plane makes intercepts $a$, $b$, $c$ on x, y, z axes respectively:

$$\boxed{\frac{x}{a} + \frac{y}{b} + \frac{z}{c} = 1}$$

```
              z
              |
              | (0,0,c)
              |/
    ─────────●───────────── y
            /|
           / |
          /  |
     (a,0,0) ● ─────● (0,b,0)
            x
            
   Plane intersects axes at (a,0,0), (0,b,0), (0,0,c)
```

---

## 📝 Three-Point Form

Plane through three non-collinear points $A(x_1, y_1, z_1)$, $B(x_2, y_2, z_2)$, $C(x_3, y_3, z_3)$:

$$\boxed{\begin{vmatrix} x-x_1 & y-y_1 & z-z_1 \\ x_2-x_1 & y_2-y_1 & z_2-z_1 \\ x_3-x_1 & y_3-y_1 & z_3-z_1 \end{vmatrix} = 0}$$

---

## 📝 Special Planes

### Coordinate Planes

| Plane | Equation |
|-------|----------|
| xy-plane | $z = 0$ |
| yz-plane | $x = 0$ |
| xz-plane | $y = 0$ |

### Planes Parallel to Coordinate Planes

| Parallel to | Equation |
|-------------|----------|
| xy-plane | $z = k$ |
| yz-plane | $x = k$ |
| xz-plane | $y = k$ |

### Planes Through Coordinate Axes

| Through | Equation Form |
|---------|---------------|
| x-axis | $by + cz = 0$ |
| y-axis | $ax + cz = 0$ |
| z-axis | $ax + by = 0$ |

---

## 📝 Angle Between Two Planes

For planes $a_1x + b_1y + c_1z + d_1 = 0$ and $a_2x + b_2y + c_2z + d_2 = 0$:

$$\boxed{\cos\theta = \frac{|a_1a_2 + b_1b_2 + c_1c_2|}{\sqrt{a_1^2+b_1^2+c_1^2} \cdot \sqrt{a_2^2+b_2^2+c_2^2}}}$$

### Parallel Planes

$$\frac{a_1}{a_2} = \frac{b_1}{b_2} = \frac{c_1}{c_2}$$

### Perpendicular Planes

$$a_1a_2 + b_1b_2 + c_1c_2 = 0$$

---

## 📝 Distance from Point to Plane

Distance from point $(x_1, y_1, z_1)$ to plane $ax + by + cz + d = 0$:

$$\boxed{D = \frac{|ax_1 + by_1 + cz_1 + d|}{\sqrt{a^2 + b^2 + c^2}}}$$

---

## 📝 Distance Between Parallel Planes

For parallel planes $ax + by + cz + d_1 = 0$ and $ax + by + cz + d_2 = 0$:

$$\boxed{D = \frac{|d_1 - d_2|}{\sqrt{a^2 + b^2 + c^2}}}$$

---

## ✏️ Worked Examples

### Example 1: General Equation

**Problem**: Find equation of plane through $(1, 2, 3)$ with normal $(2, -1, 3)$.

**Solution**:

Using point-normal form:
$2(x - 1) + (-1)(y - 2) + 3(z - 3) = 0$

$2x - 2 - y + 2 + 3z - 9 = 0$

$2x - y + 3z - 9 = 0$

**Answer**: $2x - y + 3z = 9$

---

### Example 2: Intercept Form

**Problem**: Find equation of plane with intercepts 2, 3, 4 on x, y, z axes.

**Solution**:

$\frac{x}{2} + \frac{y}{3} + \frac{z}{4} = 1$

Multiply by 12:
$6x + 4y + 3z = 12$

**Answer**: $6x + 4y + 3z = 12$ or $\frac{x}{2} + \frac{y}{3} + \frac{z}{4} = 1$

---

### Example 3: Three-Point Form

**Problem**: Find equation of plane through $(1, 1, 1)$, $(1, 2, 3)$, $(2, 1, 1)$.

**Solution**:

$\begin{vmatrix} x-1 & y-1 & z-1 \\ 0 & 1 & 2 \\ 1 & 0 & 0 \end{vmatrix} = 0$

Expanding along third row:
$1(1 \cdot 2 - 0 \cdot 1) - 0 + 0 = 0$... wait, let me recalculate.

$= 1 \cdot \begin{vmatrix} y-1 & z-1 \\ 1 & 2 \end{vmatrix} - 0 + 0$

$= 1[(y-1)(2) - (z-1)(1)]$

$= 2(y-1) - (z-1) = 0$

$2y - 2 - z + 1 = 0$

$2y - z - 1 = 0$

Verify: $(1,1,1)$: $2 - 1 - 1 = 0$ ✓
$(1,2,3)$: $4 - 3 - 1 = 0$ ✓
$(2,1,1)$: $2 - 1 - 1 = 0$ ✓

**Answer**: $2y - z = 1$

---

### Example 4: Distance from Origin

**Problem**: Find distance from origin to plane $2x + 3y + 6z = 21$.

**Solution**:

$D = \frac{|2(0) + 3(0) + 6(0) - 21|}{\sqrt{4+9+36}}$

$= \frac{21}{\sqrt{49}} = \frac{21}{7} = 3$

**Answer**: 3 units

---

### Example 5: Angle Between Planes

**Problem**: Find angle between planes $x + y + z = 1$ and $2x - y + 2z = 5$.

**Solution**:

$\cos\theta = \frac{|1(2) + 1(-1) + 1(2)|}{\sqrt{1+1+1} \cdot \sqrt{4+1+4}}$

$= \frac{|2 - 1 + 2|}{\sqrt{3} \cdot \sqrt{9}} = \frac{3}{3\sqrt{3}} = \frac{1}{\sqrt{3}}$

$\theta = \cos^{-1}\left(\frac{1}{\sqrt{3}}\right) \approx 54.7°$

**Answer**: $\theta = \cos^{-1}\left(\frac{1}{\sqrt{3}}\right)$

---

### Example 6: Parallel Planes

**Problem**: Find equation of plane parallel to $2x + 3y + 4z = 5$ passing through $(1, 1, 1)$.

**Solution**:

Parallel plane has same normal: $(2, 3, 4)$

Equation: $2x + 3y + 4z = k$

Through $(1, 1, 1)$: $2 + 3 + 4 = k = 9$

**Answer**: $2x + 3y + 4z = 9$

---

### Example 7: Perpendicular Planes

**Problem**: Find equation of plane through $(1, 2, 3)$ perpendicular to line $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$.

**Solution**:

The line's DRs $(2, 3, 4)$ are normal to the plane.

Plane: $2(x-1) + 3(y-2) + 4(z-3) = 0$

$2x - 2 + 3y - 6 + 4z - 12 = 0$

$2x + 3y + 4z = 20$

**Answer**: $2x + 3y + 4z = 20$

---

### Example 8: Distance Between Parallel Planes

**Problem**: Find distance between planes $x + 2y - 2z + 1 = 0$ and $x + 2y - 2z + 4 = 0$.

**Solution**:

$D = \frac{|4 - 1|}{\sqrt{1+4+4}} = \frac{3}{3} = 1$

**Answer**: 1 unit

---

## 📐 Plane Through Line of Intersection

The equation of plane passing through the line of intersection of planes $P_1: a_1x + b_1y + c_1z + d_1 = 0$ and $P_2: a_2x + b_2y + c_2z + d_2 = 0$ is:

$$\boxed{P_1 + \lambda P_2 = 0}$$

i.e., $(a_1 + \lambda a_2)x + (b_1 + \lambda b_2)y + (c_1 + \lambda c_2)z + (d_1 + \lambda d_2) = 0$

---

## 💡 Tips and Insights

> 💡 **Normal to Plane**: Coefficients of x, y, z in general equation give normal direction.

> 💡 **Parallel Planes**: Same normal direction, different constant terms.

> ⚠️ **Distance Formula**: Don't forget absolute value in numerator!

> 💡 **Intercepts**: If plane passes through origin, it has no intercept form.

---

## 📋 Summary Table

| Form | Equation |
|------|----------|
| General | $ax + by + cz + d = 0$ |
| Normal | $lx + my + nz = p$ ($l^2+m^2+n^2=1$) |
| Point-normal | $a(x-x_1) + b(y-y_1) + c(z-z_1) = 0$ |
| Intercept | $\frac{x}{a} + \frac{y}{b} + \frac{z}{c} = 1$ |
| Three-point | Determinant form |

### Key Formulas

| Concept | Formula |
|---------|---------|
| Angle between planes | $\cos\theta = \frac{|a_1a_2+b_1b_2+c_1c_2|}{\sqrt{a_1^2+b_1^2+c_1^2}\sqrt{a_2^2+b_2^2+c_2^2}}$ |
| Point to plane distance | $\frac{|ax_1+by_1+cz_1+d|}{\sqrt{a^2+b^2+c^2}}$ |
| Between parallel planes | $\frac{|d_1-d_2|}{\sqrt{a^2+b^2+c^2}}$ |
| Parallel condition | $\frac{a_1}{a_2} = \frac{b_1}{b_2} = \frac{c_1}{c_2}$ |
| Perpendicular condition | $a_1a_2 + b_1b_2 + c_1c_2 = 0$ |

---

## ❓ Quick Revision Questions

1. **Find equation of plane through $(2, 3, 4)$ parallel to $x + y + z = 1$.**

2. **Find intercepts of plane $2x + 3y + 6z = 6$.**

3. **Find distance from $(1, 2, 3)$ to plane $x + y + z = 10$.**

4. **Are planes $x + y + z = 1$ and $2x + 2y + 2z = 5$ parallel?**

5. **Find equation of plane perpendicular to z-axis passing through $(1, 2, 3)$.**

6. **Find angle between planes $x = 0$ and $y = 0$.**

<details>
<summary><b>Click to see answers</b></summary>

1. Parallel plane: $x + y + z = k$
   Through $(2,3,4)$: $k = 2 + 3 + 4 = 9$
   **$x + y + z = 9$**

2. $\frac{x}{3} + \frac{y}{2} + \frac{z}{1} = 1$
   **x-intercept: 3, y-intercept: 2, z-intercept: 1**

3. $D = \frac{|1+2+3-10|}{\sqrt{3}} = \frac{4}{\sqrt{3}} = \frac{4\sqrt{3}}{3}$
   **$\frac{4\sqrt{3}}{3}$ units**

4. First: normal $(1, 1, 1)$
   Second: normal $(2, 2, 2) = 2(1, 1, 1)$
   **Yes, parallel** (same normal direction)

5. Perpendicular to z-axis means normal is $(0, 0, 1)$
   Plane: $z = 3$
   **$z = 3$**

6. $x = 0$: normal $(1, 0, 0)$
   $y = 0$: normal $(0, 1, 0)$
   $\cos\theta = 0 \Rightarrow \theta = 90°$
   **90°** (they're the yz and xz planes, perpendicular)

</details>

---

## ⏭️ Navigation

| [← Previous: Straight Line](04-straight-line.md) | [Unit 8 Home](README.md) | [Next: Line and Plane →](06-line-and-plane.md) |
|:------------------------------------------------:|:------------------------:|----------------------------------------------:|
