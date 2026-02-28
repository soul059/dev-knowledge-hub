# Chapter 8.6: Line and Plane Interactions

## 📚 Chapter Overview

This chapter explores the relationships between **lines and planes** in 3D space, including angles between them, intersection points, parallel and perpendicular conditions, and the concept of coplanarity.

---

## 📝 Angle Between Line and Plane

For line $\frac{x-x_1}{a} = \frac{y-y_1}{b} = \frac{z-z_1}{c}$ and plane $Ax + By + Cz + D = 0$:

The angle $\phi$ between the line and plane:

$$\boxed{\sin\phi = \frac{|Aa + Bb + Cc|}{\sqrt{A^2+B^2+C^2} \cdot \sqrt{a^2+b^2+c^2}}}$$

```
              Normal to plane
                  ↑
                 /
                / θ (angle with normal)
               /
    ──────────/───────────── Plane
             / φ (angle with plane)
            /
           Line
           
   φ = 90° - θ
   sin φ = cos θ
```

Note: The angle between line and plane = complement of angle between line and normal.

---

## 📐 Special Cases

### Line Parallel to Plane

The line is parallel to plane when it's perpendicular to the normal:

$$\boxed{Aa + Bb + Cc = 0}$$

### Line Perpendicular to Plane

The line is perpendicular to plane when it's parallel to the normal:

$$\boxed{\frac{A}{a} = \frac{B}{b} = \frac{C}{c}}$$

### Line Lies in Plane

The line lies entirely in the plane if:
1. It's parallel to the plane ($Aa + Bb + Cc = 0$)
2. Any point of the line lies on the plane

---

## 📝 Intersection of Line and Plane

To find intersection of line $\frac{x-x_1}{a} = \frac{y-y_1}{b} = \frac{z-z_1}{c} = t$ and plane $Ax + By + Cz + D = 0$:

### Method

1. Express line in parametric form: $(x_1 + at, y_1 + bt, z_1 + ct)$
2. Substitute into plane equation
3. Solve for $t$
4. Find coordinates using $t$

### Formula for t

$$t = \frac{-(Ax_1 + By_1 + Cz_1 + D)}{Aa + Bb + Cc}$$

---

## 📝 Image of Point in Plane

To find image of point $P(x_1, y_1, z_1)$ in plane $ax + by + cz + d = 0$:

$$\boxed{\frac{x - x_1}{a} = \frac{y - y_1}{b} = \frac{z - z_1}{c} = \frac{-2(ax_1 + by_1 + cz_1 + d)}{a^2 + b^2 + c^2}}$$

### Foot of Perpendicular

The foot of perpendicular from $P$ to the plane is the midpoint of $P$ and its image.

$$\frac{x - x_1}{a} = \frac{y - y_1}{b} = \frac{z - z_1}{c} = \frac{-(ax_1 + by_1 + cz_1 + d)}{a^2 + b^2 + c^2}$$

---

## 📝 Coplanar Lines

Two lines are **coplanar** if they either:
1. Intersect at a point, or
2. Are parallel

### Test for Coplanarity

Lines $\frac{x-x_1}{a_1} = \frac{y-y_1}{b_1} = \frac{z-z_1}{c_1}$ and $\frac{x-x_2}{a_2} = \frac{y-y_2}{b_2} = \frac{z-z_2}{c_2}$ are coplanar iff:

$$\boxed{\begin{vmatrix} x_2-x_1 & y_2-y_1 & z_2-z_1 \\ a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \end{vmatrix} = 0}$$

---

## 📝 Skew Lines

Lines that are neither parallel nor intersecting are called **skew lines**.

```
           Line L₁
    ●─────────────●
   (not in same plane)
           
              Line L₂
         ●─────────────●
```

### Shortest Distance Between Skew Lines

For lines L₁ and L₂ with points $(x_1, y_1, z_1)$, $(x_2, y_2, z_2)$ and DRs $(a_1, b_1, c_1)$, $(a_2, b_2, c_2)$:

$$\boxed{d = \frac{\left|\begin{vmatrix} x_2-x_1 & y_2-y_1 & z_2-z_1 \\ a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \end{vmatrix}\right|}{\sqrt{(b_1c_2-b_2c_1)^2 + (c_1a_2-c_2a_1)^2 + (a_1b_2-a_2b_1)^2}}}$$

---

## ✏️ Worked Examples

### Example 1: Angle Between Line and Plane

**Problem**: Find angle between line $\frac{x-1}{2} = \frac{y+1}{3} = \frac{z-2}{6}$ and plane $x + y + z = 5$.

**Solution**:

Line DRs: $(2, 3, 6)$
Plane normal: $(1, 1, 1)$

$\sin\phi = \frac{|2(1) + 3(1) + 6(1)|}{\sqrt{4+9+36} \cdot \sqrt{3}}$

$= \frac{|2 + 3 + 6|}{7\sqrt{3}} = \frac{11}{7\sqrt{3}}$

$\phi = \sin^{-1}\left(\frac{11}{7\sqrt{3}}\right)$

**Answer**: $\phi = \sin^{-1}\left(\frac{11\sqrt{3}}{21}\right)$

---

### Example 2: Line-Plane Intersection

**Problem**: Find where $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4} = t$ meets $x + y + z = 12$.

**Solution**:

Point on line: $(1 + 2t, 2 + 3t, 3 + 4t)$

Substituting in plane:
$(1 + 2t) + (2 + 3t) + (3 + 4t) = 12$

$6 + 9t = 12$

$t = \frac{6}{9} = \frac{2}{3}$

Point: $\left(1 + \frac{4}{3}, 2 + 2, 3 + \frac{8}{3}\right) = \left(\frac{7}{3}, 4, \frac{17}{3}\right)$

**Answer**: $\left(\frac{7}{3}, 4, \frac{17}{3}\right)$

---

### Example 3: Parallel Check

**Problem**: Is line $\frac{x}{1} = \frac{y}{2} = \frac{z}{3}$ parallel to plane $x + 2y + 3z = 5$?

**Solution**:

Line DRs: $(1, 2, 3)$
Plane normal: $(1, 2, 3)$

Check: $1(1) + 2(2) + 3(3) = 1 + 4 + 9 = 14 \neq 0$

**Answer**: **No**, line is NOT parallel (it's actually perpendicular to the plane!)

---

### Example 4: Line Perpendicular to Plane

**Problem**: Find equation of line through $(1, 2, 3)$ perpendicular to plane $2x + 3y + 4z = 5$.

**Solution**:

Normal to plane $(2, 3, 4)$ = DRs of line

Line: $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$

**Answer**: $\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$

---

### Example 5: Foot of Perpendicular

**Problem**: Find foot of perpendicular from $(1, 2, 3)$ to plane $x + y + z = 6$.

**Solution**:

Using formula:
$\frac{x-1}{1} = \frac{y-2}{1} = \frac{z-3}{1} = \frac{-(1+2+3-6)}{1+1+1} = \frac{0}{3} = 0$

This gives: $x = 1$, $y = 2$, $z = 3$

Wait - this means the point $(1, 2, 3)$ is already on the plane!

Check: $1 + 2 + 3 = 6$ ✓

**Answer**: Point $(1, 2, 3)$ lies on the plane itself!

Let's try another point $(1, 1, 1)$:

$\frac{x-1}{1} = \frac{y-1}{1} = \frac{z-1}{1} = \frac{-(1+1+1-6)}{3} = \frac{3}{3} = 1$

Foot: $(1+1, 1+1, 1+1) = (2, 2, 2)$

**Answer**: Foot of perpendicular from $(1, 1, 1)$ is $(2, 2, 2)$

---

### Example 6: Image of Point

**Problem**: Find image of $(1, 2, 3)$ in plane $x + y + z = 9$.

**Solution**:

$\frac{x-1}{1} = \frac{y-2}{1} = \frac{z-3}{1} = \frac{-2(1+2+3-9)}{3} = \frac{-2(-3)}{3} = 2$

Image: $(1+2, 2+2, 3+2) = (3, 4, 5)$

**Answer**: $(3, 4, 5)$

Verify: Midpoint of $(1,2,3)$ and $(3,4,5)$ is $(2, 3, 4)$
Check if on plane: $2+3+4 = 9$ ✓

---

### Example 7: Coplanarity Test

**Problem**: Check if lines $\frac{x-1}{1} = \frac{y-2}{2} = \frac{z-3}{3}$ and $\frac{x-2}{2} = \frac{y-3}{3} = \frac{z-4}{4}$ are coplanar.

**Solution**:

Points: $(1, 2, 3)$ and $(2, 3, 4)$
DRs: $(1, 2, 3)$ and $(2, 3, 4)$

$\begin{vmatrix} 1 & 1 & 1 \\ 1 & 2 & 3 \\ 2 & 3 & 4 \end{vmatrix}$

$= 1(8-9) - 1(4-6) + 1(3-4)$
$= -1 + 2 - 1 = 0$

**Answer**: **Yes, lines are coplanar**

---

### Example 8: Shortest Distance

**Problem**: Find shortest distance between lines:
$\frac{x-1}{2} = \frac{y-2}{3} = \frac{z-3}{4}$ and $\frac{x-2}{3} = \frac{y-4}{4} = \frac{z-5}{5}$

**Solution**:

Points: $(1, 2, 3)$ and $(2, 4, 5)$
DRs: $(2, 3, 4)$ and $(3, 4, 5)$

Numerator:
$\begin{vmatrix} 1 & 2 & 2 \\ 2 & 3 & 4 \\ 3 & 4 & 5 \end{vmatrix}$

$= 1(15-16) - 2(10-12) + 2(8-9)$
$= -1 + 4 - 2 = 1$

Denominator:
$\vec{d_1} \times \vec{d_2} = (3 \cdot 5 - 4 \cdot 4, 4 \cdot 3 - 2 \cdot 5, 2 \cdot 4 - 3 \cdot 3)$
$= (15-16, 12-10, 8-9) = (-1, 2, -1)$

$|\vec{d_1} \times \vec{d_2}| = \sqrt{1+4+1} = \sqrt{6}$

$d = \frac{|1|}{\sqrt{6}} = \frac{1}{\sqrt{6}}$

**Answer**: $\frac{1}{\sqrt{6}} = \frac{\sqrt{6}}{6}$ units

---

## 📊 Summary Table

| Relationship | Condition |
|--------------|-----------|
| Line ∥ Plane | $Aa + Bb + Cc = 0$ |
| Line ⊥ Plane | $\frac{A}{a} = \frac{B}{b} = \frac{C}{c}$ |
| Angle between line and plane | $\sin\phi = \frac{|Aa+Bb+Cc|}{\sqrt{A^2+B^2+C^2}\sqrt{a^2+b^2+c^2}}$ |
| Lines coplanar | Determinant = 0 |
| Lines skew | Not parallel AND not intersecting |

### Key Formulas

| Quantity | Formula |
|----------|---------|
| Foot of perpendicular | Use $t = \frac{-(ax_1+by_1+cz_1+d)}{a^2+b^2+c^2}$ |
| Image in plane | Use $t = \frac{-2(ax_1+by_1+cz_1+d)}{a^2+b^2+c^2}$ |
| Shortest distance | $\frac{|\text{det}|}{|\vec{d_1} \times \vec{d_2}|}$ |

---

## 💡 Tips and Insights

> 💡 **Angle Complement**: Angle with plane = 90° - angle with normal.

> 💡 **Line in Plane**: Check both parallel condition AND point on plane.

> ⚠️ **Skew Lines**: Always check if lines are parallel first before computing shortest distance.

> 💡 **Image Formula**: Image formula uses -2 times the distance, foot uses -1 times.

---

## 🎉 Unit 8 Complete!

Congratulations on completing **3D Coordinate Geometry**! Key concepts covered:

✅ 3D coordinate system and octants  
✅ Distance and section formulas in 3D  
✅ Direction cosines and ratios  
✅ Equations of lines (symmetric, parametric)  
✅ Equations of planes (various forms)  
✅ Line-plane interactions and coplanarity  

---

## ❓ Quick Revision Questions

1. **Find angle between line $\frac{x}{1} = \frac{y}{1} = \frac{z}{1}$ and plane $x + y = 0$.**

2. **Find intersection of $\frac{x-2}{1} = \frac{y-3}{2} = \frac{z-4}{3}$ and plane $x + y + z = 15$.**

3. **Is line $\frac{x}{2} = \frac{y}{3} = \frac{z}{4}$ parallel to $2x + 3y + 4z = 1$?**

4. **Find foot of perpendicular from origin to $x + 2y + 2z = 9$.**

5. **Check if lines $\frac{x}{1} = \frac{y}{2} = \frac{z}{3}$ and $\frac{x}{2} = \frac{y}{4} = \frac{z}{6}$ are coplanar.**

6. **Find image of $(3, 4, 5)$ in plane $x + y + z = 3$.**

<details>
<summary><b>Click to see answers</b></summary>

1. DRs: $(1, 1, 1)$, Normal: $(1, 1, 0)$
   $\sin\phi = \frac{|1+1+0|}{\sqrt{3}\sqrt{2}} = \frac{2}{\sqrt{6}}$
   **$\phi = \sin^{-1}\left(\frac{2\sqrt{6}}{6}\right) = \sin^{-1}\left(\frac{\sqrt{6}}{3}\right)$**

2. Point: $(2+t, 3+2t, 4+3t)$
   $(2+t) + (3+2t) + (4+3t) = 15$
   $9 + 6t = 15 \Rightarrow t = 1$
   **$(3, 5, 7)$**

3. Check: $2(2) + 3(3) + 4(4) = 4 + 9 + 16 = 29 \neq 0$
   **No, not parallel** (line will intersect plane)

4. Perpendicular from O: along normal $(1, 2, 2)$
   $t = \frac{-(0-9)}{1+4+4} = 1$
   Foot: $(1, 2, 2)$
   **$(1, 2, 2)$**

5. DRs: $(1, 2, 3)$ and $(2, 4, 6) = 2(1, 2, 3)$
   Lines are parallel! (same direction)
   Both pass through origin.
   **Yes, coplanar** (actually the same line!)

6. $t = \frac{-2(3+4+5-3)}{3} = \frac{-2(9)}{3} = -6$
   Image: $(3-6, 4-6, 5-6) = (-3, -2, -1)$
   **$(-3, -2, -1)$**

</details>

---

## 🎊 Course Complete!

**Congratulations!** You have completed the entire **Coordinate Geometry** course!

### Units Covered:
1. ✅ Cartesian Coordinate System
2. ✅ Straight Lines
3. ✅ Pair of Straight Lines
4. ✅ Circle
5. ✅ Parabola
6. ✅ Ellipse
7. ✅ Hyperbola
8. ✅ 3D Coordinate Geometry

---

## ⏭️ Navigation

| [← Previous: Equation of Plane](05-equation-of-plane.md) | [Unit 8 Home](README.md) | [Course Home →](../README.md) |
|:--------------------------------------------------------:|:------------------------:|:-----------------------------:|
