# Chapter 8.2: Section Formula in 3D

## 📚 Chapter Overview

This chapter extends the **section formula** from 2D to 3D, showing how to find the coordinates of a point that divides a line segment in a given ratio. We also explore the centroid and conditions for collinearity.

---

## 📝 Internal Division

### Formula

If point $P$ divides the line segment joining $A(x_1, y_1, z_1)$ and $B(x_2, y_2, z_2)$ **internally** in the ratio $m:n$, then:

$$\boxed{P = \left(\frac{mx_2 + nx_1}{m+n}, \frac{my_2 + ny_1}{m+n}, \frac{mz_2 + nz_1}{m+n}\right)}$$

```
    A(x₁,y₁,z₁)----m----P----n----B(x₂,y₂,z₂)
                   ↑
              Internal point P
              AP:PB = m:n
```

### Derivation

Let $P(x, y, z)$ divide AB in ratio $m:n$ internally.

Using similar approach as 2D (position vectors):

$\vec{OP} = \frac{m \cdot \vec{OB} + n \cdot \vec{OA}}{m + n}$

Component-wise:
- $x = \frac{mx_2 + nx_1}{m+n}$
- $y = \frac{my_2 + ny_1}{m+n}$
- $z = \frac{mz_2 + nz_1}{m+n}$

---

## 📝 External Division

If point $P$ divides AB **externally** in ratio $m:n$:

$$\boxed{P = \left(\frac{mx_2 - nx_1}{m-n}, \frac{my_2 - ny_1}{m-n}, \frac{mz_2 - nz_1}{m-n}\right)}$$

```
    P----n----A(x₁,y₁,z₁)----m----B(x₂,y₂,z₂)
    ↑
    External point P
    (P is beyond A, on extended line)
```

Or:

```
    A(x₁,y₁,z₁)----m----B(x₂,y₂,z₂)----n----P
                                        ↑
                                   External point P
                                   (P is beyond B)
```

---

## ⚡ Special Cases

### Midpoint Formula

When $m = n$ (ratio 1:1), the midpoint is:

$$\boxed{M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}, \frac{z_1 + z_2}{2}\right)}$$

### Trisection Points

Points dividing AB in ratio 1:2 and 2:1:

$P_1 = \left(\frac{x_1 + 2x_1}{3}, \frac{y_2 + 2y_1}{3}, \frac{z_2 + 2z_1}{3}\right)$

Wait, let me correct:

For ratio 1:2: $P_1 = \left(\frac{1 \cdot x_2 + 2 \cdot x_1}{3}, \frac{y_2 + 2y_1}{3}, \frac{z_2 + 2z_1}{3}\right)$

For ratio 2:1: $P_2 = \left(\frac{2x_2 + x_1}{3}, \frac{2y_2 + y_1}{3}, \frac{2z_2 + z_1}{3}\right)$

---

## 📝 Centroid of Triangle

For triangle with vertices $A(x_1, y_1, z_1)$, $B(x_2, y_2, z_2)$, $C(x_3, y_3, z_3)$:

$$\boxed{G = \left(\frac{x_1 + x_2 + x_3}{3}, \frac{y_1 + y_2 + y_3}{3}, \frac{z_1 + z_2 + z_3}{3}\right)}$$

### Property

The centroid divides each median in ratio 2:1 from vertex.

```
              A
             /|\
            / | \
           /  G  \
          /   |   \
         B----+----C
              M (midpoint of BC)
              
   G divides AM in ratio 2:1
```

---

## 📝 Centroid of Tetrahedron

For tetrahedron with vertices $A(x_1, y_1, z_1)$, $B(x_2, y_2, z_2)$, $C(x_3, y_3, z_3)$, $D(x_4, y_4, z_4)$:

$$\boxed{G = \left(\frac{x_1+x_2+x_3+x_4}{4}, \frac{y_1+y_2+y_3+y_4}{4}, \frac{z_1+z_2+z_3+z_4}{4}\right)}$$

---

## 📝 Collinearity of Three Points

Three points $A$, $B$, $C$ are **collinear** if one divides the segment of the other two in some ratio.

### Condition for Collinearity

Points $A(x_1, y_1, z_1)$, $B(x_2, y_2, z_2)$, $C(x_3, y_3, z_3)$ are collinear if:

$$\boxed{\frac{x_3 - x_1}{x_2 - x_1} = \frac{y_3 - y_1}{y_2 - y_1} = \frac{z_3 - z_1}{z_2 - z_1}}$$

Or equivalently, if there exists scalar $k$ such that:
$$\vec{AC} = k \cdot \vec{AB}$$

---

## ✏️ Worked Examples

### Example 1: Internal Division

**Problem**: Find the point dividing the join of $(2, 3, 4)$ and $(4, 5, 6)$ internally in ratio 2:3.

**Solution**:

$A = (2, 3, 4)$, $B = (4, 5, 6)$, $m:n = 2:3$

$x = \frac{2(4) + 3(2)}{2+3} = \frac{8 + 6}{5} = \frac{14}{5}$

$y = \frac{2(5) + 3(3)}{5} = \frac{10 + 9}{5} = \frac{19}{5}$

$z = \frac{2(6) + 3(4)}{5} = \frac{12 + 12}{5} = \frac{24}{5}$

**Answer**: $\left(\frac{14}{5}, \frac{19}{5}, \frac{24}{5}\right)$

---

### Example 2: External Division

**Problem**: Find point dividing $(1, 2, 3)$ and $(4, 5, 6)$ externally in ratio 2:1.

**Solution**:

$A = (1, 2, 3)$, $B = (4, 5, 6)$, $m:n = 2:1$

$x = \frac{2(4) - 1(1)}{2-1} = \frac{8 - 1}{1} = 7$

$y = \frac{2(5) - 1(2)}{1} = 10 - 2 = 8$

$z = \frac{2(6) - 1(3)}{1} = 12 - 3 = 9$

**Answer**: $(7, 8, 9)$

---

### Example 3: Midpoint

**Problem**: Find midpoint of $(1, -1, 2)$ and $(3, 5, 4)$.

**Solution**:

$M = \left(\frac{1+3}{2}, \frac{-1+5}{2}, \frac{2+4}{2}\right)$

$= \left(\frac{4}{2}, \frac{4}{2}, \frac{6}{2}\right)$

$= (2, 2, 3)$

**Answer**: $(2, 2, 3)$

---

### Example 4: Finding Ratio

**Problem**: In what ratio does the point $(3, 2, 5)$ divide the join of $(2, 1, 3)$ and $(5, 4, 9)$?

**Solution**:

Let the ratio be $k:1$ (internal division).

$3 = \frac{k(5) + 1(2)}{k+1}$

$3(k+1) = 5k + 2$

$3k + 3 = 5k + 2$

$1 = 2k$

$k = \frac{1}{2}$

Verify with y-coordinate:
$2 = \frac{\frac{1}{2}(4) + 1(1)}{\frac{1}{2}+1} = \frac{2 + 1}{\frac{3}{2}} = \frac{3}{\frac{3}{2}} = 2$ ✓

Verify with z-coordinate:
$5 = \frac{\frac{1}{2}(9) + 1(3)}{\frac{3}{2}} = \frac{4.5 + 3}{1.5} = \frac{7.5}{1.5} = 5$ ✓

**Answer**: Ratio is $\frac{1}{2}:1$ or **1:2**

---

### Example 5: Centroid of Triangle

**Problem**: Find centroid of triangle with vertices $(1, 2, 3)$, $(3, 4, 5)$, $(5, 6, 7)$.

**Solution**:

$G = \left(\frac{1+3+5}{3}, \frac{2+4+6}{3}, \frac{3+5+7}{3}\right)$

$= \left(\frac{9}{3}, \frac{12}{3}, \frac{15}{3}\right)$

$= (3, 4, 5)$

**Answer**: $(3, 4, 5)$

---

### Example 6: Collinearity Check

**Problem**: Check if $(1, 2, 3)$, $(4, 5, 6)$, $(7, 8, 9)$ are collinear.

**Solution**:

$\vec{AB} = (4-1, 5-2, 6-3) = (3, 3, 3)$

$\vec{AC} = (7-1, 8-2, 9-3) = (6, 6, 6)$

Check: Is $\vec{AC} = k \cdot \vec{AB}$?

$\frac{6}{3} = \frac{6}{3} = \frac{6}{3} = 2$

Yes! $\vec{AC} = 2 \cdot \vec{AB}$

**Answer**: Points are **collinear**

---

### Example 7: Point on Line Segment

**Problem**: Find ratio in which xy-plane divides the join of $(2, 3, 4)$ and $(1, 2, -3)$.

**Solution**:

xy-plane: $z = 0$

Let ratio be $k:1$. Using section formula for z:

$0 = \frac{k(-3) + 1(4)}{k+1}$

$0 = -3k + 4$

$k = \frac{4}{3}$

The ratio is $\frac{4}{3}:1 = 4:3$

**Answer**: xy-plane divides in ratio **4:3** internally

---

### Example 8: Finding Fourth Vertex

**Problem**: Three vertices of a parallelogram are $A(1, 2, 3)$, $B(4, 5, 6)$, $C(7, 8, 9)$. Find the fourth vertex D such that ABCD is a parallelogram.

**Solution**:

In parallelogram, diagonals bisect each other.

Midpoint of AC = Midpoint of BD

$\left(\frac{1+7}{2}, \frac{2+8}{2}, \frac{3+9}{2}\right) = \left(\frac{4+x}{2}, \frac{5+y}{2}, \frac{6+z}{2}\right)$

$(4, 5, 6) = \left(\frac{4+x}{2}, \frac{5+y}{2}, \frac{6+z}{2}\right)$

$4 + x = 8 \Rightarrow x = 4$
$5 + y = 10 \Rightarrow y = 5$
$6 + z = 12 \Rightarrow z = 6$

**Answer**: $D = (4, 5, 6)$

Hmm, this is same as B! Let me reconsider - ABC might be collinear (we showed this in Example 6).

Actually, if A, B, C are collinear, they cannot form a parallelogram. Let's try with different vertices.

For valid parallelogram with $A(1, 2, 1)$, $B(2, 3, 2)$, $C(4, 4, 3)$:

Midpoint of AC: $\left(\frac{5}{2}, 3, 2\right)$

$\frac{5}{2} = \frac{2+x}{2} \Rightarrow x = 3$
$3 = \frac{3+y}{2} \Rightarrow y = 3$
$2 = \frac{2+z}{2} \Rightarrow z = 2$

**D = (3, 3, 2)**

---

## 📊 Section Formula Summary

| Division | Formula |
|----------|---------|
| Internal $(m:n)$ | $\left(\frac{mx_2+nx_1}{m+n}, \frac{my_2+ny_1}{m+n}, \frac{mz_2+nz_1}{m+n}\right)$ |
| External $(m:n)$ | $\left(\frac{mx_2-nx_1}{m-n}, \frac{my_2-ny_1}{m-n}, \frac{mz_2-nz_1}{m-n}\right)$ |
| Midpoint | $\left(\frac{x_1+x_2}{2}, \frac{y_1+y_2}{2}, \frac{z_1+z_2}{2}\right)$ |
| Centroid (triangle) | $\left(\frac{x_1+x_2+x_3}{3}, \frac{y_1+y_2+y_3}{3}, \frac{z_1+z_2+z_3}{3}\right)$ |
| Centroid (tetrahedron) | $\left(\frac{\sum x_i}{4}, \frac{\sum y_i}{4}, \frac{\sum z_i}{4}\right)$ |

---

## 💡 Tips and Insights

> 💡 **Same as 2D**: The section formula in 3D has the same structure as 2D, just with an extra z-coordinate.

> 💡 **Ratio Check**: When finding ratio, solve for k using any coordinate, then verify with others.

> ⚠️ **External Division**: Remember to use minus signs in denominator and numerator for external division.

> 💡 **Plane Intersection**: To find where a line segment crosses a coordinate plane, set that coordinate to zero.

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Internal division $m:n$ | $P = \frac{m \cdot B + n \cdot A}{m+n}$ (each coordinate) |
| External division $m:n$ | $P = \frac{m \cdot B - n \cdot A}{m-n}$ (each coordinate) |
| Collinearity condition | $\frac{x_3-x_1}{x_2-x_1} = \frac{y_3-y_1}{y_2-y_1} = \frac{z_3-z_1}{z_2-z_1}$ |
| Centroid of triangle | Average of coordinates |
| Parallelogram diagonal | Midpoints equal |

---

## ❓ Quick Revision Questions

1. **Find point dividing $(1, 2, 3)$ and $(7, 8, 9)$ in ratio 1:2 internally.**

2. **Find midpoint of $(0, 0, 0)$ and $(4, 6, 8)$.**

3. **In what ratio does yz-plane divide the join of $(2, 3, 4)$ and $(-6, 1, 2)$?**

4. **Find centroid of triangle with vertices $(0, 0, 0)$, $(3, 0, 0)$, $(0, 4, 0)$.**

5. **Are points $(1, 1, 1)$, $(2, 3, 2)$, $(3, 5, 3)$ collinear?**

6. **Find point dividing $(1, 2, 3)$ and $(3, 4, 5)$ externally in ratio 3:1.**

<details>
<summary><b>Click to see answers</b></summary>

1. $x = \frac{1(7)+2(1)}{3} = \frac{9}{3} = 3$
   $y = \frac{1(8)+2(2)}{3} = \frac{12}{3} = 4$
   $z = \frac{1(9)+2(3)}{3} = \frac{15}{3} = 5$
   **$(3, 4, 5)$**

2. $M = \left(\frac{0+4}{2}, \frac{0+6}{2}, \frac{0+8}{2}\right) = (2, 3, 4)$
   **(2, 3, 4)**

3. yz-plane: $x = 0$
   $0 = \frac{k(-6)+1(2)}{k+1}$
   $-6k + 2 = 0$
   $k = \frac{1}{3}$
   Ratio: **1:3**

4. $G = \left(\frac{0+3+0}{3}, \frac{0+0+4}{3}, \frac{0+0+0}{3}\right) = (1, \frac{4}{3}, 0)$
   **$(1, \frac{4}{3}, 0)$**

5. $\vec{AB} = (1, 2, 1)$, $\vec{AC} = (2, 4, 2) = 2(1, 2, 1)$
   **Yes, collinear**

6. $x = \frac{3(3)-1(1)}{3-1} = \frac{8}{2} = 4$
   $y = \frac{3(4)-1(2)}{2} = \frac{10}{2} = 5$
   $z = \frac{3(5)-1(3)}{2} = \frac{12}{2} = 6$
   **(4, 5, 6)**

</details>

---

## ⏭️ Navigation

| [← Previous: Coordinate System](01-coordinate-system.md) | [Unit 8 Home](README.md) | [Next: Direction Cosines →](03-direction-cosines.md) |
|:--------------------------------------------------------:|:------------------------:|----------------------------------------------------:|
