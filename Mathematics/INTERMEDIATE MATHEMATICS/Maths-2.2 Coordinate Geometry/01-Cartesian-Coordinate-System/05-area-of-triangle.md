# Chapter 1.5: Area of Triangle

## 📚 Chapter Overview

Calculating the area of a triangle when the coordinates of its vertices are known is a fundamental application of coordinate geometry. This chapter presents the formula for finding area using coordinates and extends it to finding areas of general polygons.

---

## 📝 Derivation of Area Formula

### Method 1: Using Trapezoids

Consider a triangle with vertices $A(x_1, y_1)$, $B(x_2, y_2)$, and $C(x_3, y_3)$.

```
        Y
        ↑
        │           B(x₂, y₂)
        │           ●
        │          /|\
        │         / | \
        │        /  |  \
        │       /   |   \
        │   A  /    |    \  C
        │   ●───────┼─────●
        │   |       |     |
        │   |       |     |
    ────O───●───────●─────●────→ X
           L(x₁,0)  M    N(x₃,0)
```

**Area of triangle ABC** = Area of trapezoid ALNB + Area of trapezoid BMNC − Area of trapezoid ALMC

### Method 2: Direct Formula (Cross Product Approach)

The area can be computed using the determinant formula derived from vector cross products.

---

## ⚡ Area Formula for Triangle

> **Area Formula**: For a triangle with vertices $A(x_1, y_1)$, $B(x_2, y_2)$, and $C(x_3, y_3)$:
>
> $$\boxed{\text{Area} = \frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|}$$

### Alternative Forms

**Form 1: Cyclic Form**
$$\text{Area} = \frac{1}{2}|(x_1y_2 - x_2y_1) + (x_2y_3 - x_3y_2) + (x_3y_1 - x_1y_3)|$$

**Form 2: Matrix/Determinant Form**
$$\text{Area} = \frac{1}{2}\left|\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix}\right|$$

**Form 3: Table Method (Shoelace Formula)**

Arrange coordinates in a table and cross-multiply:

```
    x₁  y₁
    x₂  y₂
    x₃  y₃
    x₁  y₁   (repeat first point)
    
    Area = ½|sum of (↘ products) - sum of (↙ products)|
         = ½|(x₁y₂ + x₂y₃ + x₃y₁) - (x₂y₁ + x₃y₂ + x₁y₃)|
```

---

## 📝 Understanding the Shoelace Method

### Visual Representation

```
    List vertices in order (counterclockwise or clockwise):
    
    (x₁, y₁) → (x₂, y₂) → (x₃, y₃) → back to (x₁, y₁)
    
    ┌───────┬───────┐
    │  x₁   │  y₁   │
    │   ╲   │   ╱   │
    │    ╲  │  ╱    │
    ├───────┼───────┤
    │  x₂   │  y₂   │
    │   ╲   │   ╱   │
    │    ╲  │  ╱    │
    ├───────┼───────┤
    │  x₃   │  y₃   │
    │   ╲   │   ╱   │
    │    ╲  │  ╱    │
    ├───────┼───────┤
    │  x₁   │  y₁   │
    └───────┴───────┘
    
    Multiply diagonally:
    ↘ direction: x₁y₂ + x₂y₃ + x₃y₁  (positive sum)
    ↙ direction: x₂y₁ + x₃y₂ + x₁y₃  (negative sum)
```

---

## ✏️ Worked Examples

### Example 1: Basic Area Calculation

**Problem**: Find the area of the triangle with vertices A(1, 2), B(4, 6), and C(7, 2).

**Solution**:

**Method 1: Using Direct Formula**
$$\text{Area} = \frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|$$

$$= \frac{1}{2}|1(6 - 2) + 4(2 - 2) + 7(2 - 6)|$$

$$= \frac{1}{2}|1(4) + 4(0) + 7(-4)|$$

$$= \frac{1}{2}|4 + 0 - 28|$$

$$= \frac{1}{2}|-24| = \frac{1}{2} \times 24 = 12$$

**Method 2: Shoelace Method**
```
    1    2
    4    6
    7    2
    1    2
    
    ↘ products: (1×6) + (4×2) + (7×2) = 6 + 8 + 14 = 28
    ↙ products: (4×2) + (7×6) + (1×2) = 8 + 42 + 2 = 52
    
    Area = ½|28 - 52| = ½ × 24 = 12
```

**Answer**: Area = **12 square units**

```
        Y
        ↑
    6   │       ●B(4, 6)
        │      /|\
        │     / | \
        │    /  |  \
        │   /   |   \
    2   │  ●────┼────●
        │ A(1,2)│   C(7,2)
    ────O───┬───┴───┬───→ X
            1   4   7
```

---

### Example 2: Triangle with Origin

**Problem**: Find the area of the triangle formed by the origin and points (4, 0) and (0, 3).

**Solution**:

Vertices: O(0, 0), A(4, 0), B(0, 3)

$$\text{Area} = \frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|$$

$$= \frac{1}{2}|0(0 - 3) + 4(3 - 0) + 0(0 - 0)|$$

$$= \frac{1}{2}|0 + 12 + 0| = 6$$

**Or simply:** For a right triangle at origin with legs on axes:
$$\text{Area} = \frac{1}{2} \times \text{base} \times \text{height} = \frac{1}{2} \times 4 \times 3 = 6$$

**Answer**: Area = **6 square units**

---

### Example 3: Negative Coordinates

**Problem**: Find the area of the triangle with vertices P(−2, −3), Q(4, 2), and R(−1, 5).

**Solution**:

Using the formula:
$$\text{Area} = \frac{1}{2}|(-2)(2 - 5) + 4(5 - (-3)) + (-1)((-3) - 2)|$$

$$= \frac{1}{2}|(-2)(-3) + 4(8) + (-1)(-5)|$$

$$= \frac{1}{2}|6 + 32 + 5|$$

$$= \frac{1}{2} \times 43 = 21.5$$

**Answer**: Area = **21.5 square units**

---

### Example 4: Finding Unknown Vertex

**Problem**: The vertices of a triangle are A(2, 3), B(5, 7), and C(x, 3). If the area is 6 square units, find x.

**Solution**:

$$6 = \frac{1}{2}|2(7 - 3) + 5(3 - 3) + x(3 - 7)|$$

$$6 = \frac{1}{2}|2(4) + 5(0) + x(-4)|$$

$$6 = \frac{1}{2}|8 - 4x|$$

$$12 = |8 - 4x|$$

**Case 1:** $8 - 4x = 12$
$$-4x = 4 \Rightarrow x = -1$$

**Case 2:** $8 - 4x = -12$
$$-4x = -20 \Rightarrow x = 5$$

But if $x = 5$, point C would be (5, 3), and we need to check if this forms a valid triangle with A(2, 3) and B(5, 7). Since A and C would have the same x-coordinate (No, wait - A is at x=2, C at x=5), it's valid.

**Answer**: $x = -1$ or $x = 5$

---

### Example 5: Area of Quadrilateral

**Problem**: Find the area of the quadrilateral with vertices A(1, 1), B(5, 1), C(6, 4), and D(2, 4).

**Solution**:

Divide the quadrilateral into two triangles: ABC and ACD.

**Or use the extended Shoelace formula for quadrilaterals:**

```
    1    1
    5    1
    6    4
    2    4
    1    1  (repeat first)
    
    ↘: (1×1) + (5×4) + (6×4) + (2×1) = 1 + 20 + 24 + 2 = 47
    ↙: (5×1) + (6×1) + (2×4) + (1×4) = 5 + 6 + 8 + 4 = 23
    
    Area = ½|47 - 23| = ½ × 24 = 12
```

**Answer**: Area = **12 square units**

```
    D(2,4)●────────────●C(6,4)
          │            │
          │            │
          │            │
    A(1,1)●────────────●B(5,1)
```

---

### Example 6: Triangle with Line Intercepts

**Problem**: A line makes intercepts of 4 units on the X-axis and 3 units on the Y-axis. Find the area of the triangle formed with the coordinate axes.

**Solution**:

The line intercepts:
- X-axis at (4, 0)
- Y-axis at (0, 3)
- Origin at (0, 0)

Triangle vertices: O(0, 0), A(4, 0), B(0, 3)

$$\text{Area} = \frac{1}{2} \times |4| \times |3| = 6 \text{ sq units}$$

**General Formula**: For a line making intercepts $a$ and $b$ on the axes:
$$\text{Area} = \frac{1}{2}|a \times b|$$

**Answer**: Area = **6 square units**

---

## 📝 Special Cases and Extensions

### Area with Origin as Vertex

If one vertex is at the origin, the formula simplifies:

For triangle with O(0,0), A$(x_1, y_1)$, B$(x_2, y_2)$:

$$\text{Area} = \frac{1}{2}|x_1y_2 - x_2y_1|$$

### Area of Polygon (Shoelace Formula)

For a polygon with n vertices $(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)$:

$$\text{Area} = \frac{1}{2}\left|\sum_{i=1}^{n}(x_iy_{i+1} - x_{i+1}y_i)\right|$$

where $(x_{n+1}, y_{n+1}) = (x_1, y_1)$

---

## 📝 Sign of the Area Expression

The expression inside the absolute value can be positive or negative:

| Sign | Vertex Order |
|------|--------------|
| Positive | Vertices in counterclockwise order |
| Negative | Vertices in clockwise order |
| Zero | Points are collinear |

> 💡 **Note**: We always take the absolute value for area, but the signed value has applications in determining orientation.

---

## 📝 Relationship to Collinearity

**Key Insight**: If three points are collinear, the area of the "triangle" they form is **zero**.

$$\text{Collinearity Condition: } x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2) = 0$$

This provides a method to check if three points lie on the same line.

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: Remember "1/2 × base × height" as a check — your coordinate formula should give the same result.

> 💡 **Shoelace Pattern**: Write vertices in order, repeat the first vertex at the end, then cross-multiply.

> ⚠️ **Common Mistake**: Forgetting the absolute value — area is always positive!

> ⚠️ **Order Matters**: For polygons, vertices must be listed in consecutive order (either all clockwise or all counterclockwise).

---

## 🌍 Real-World Applications

1. **Surveying**: Calculating land area from GPS coordinates
2. **Computer Graphics**: Filling polygons, calculating surface areas
3. **Architecture**: Computing floor areas and material requirements
4. **GIS (Geographic Information Systems)**: Calculating regions on maps
5. **Physics**: Computing work done (area under force-displacement graph)
6. **Game Development**: Collision detection and area calculations

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Area of triangle | $\frac{1}{2}\|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)\|$ |
| Determinant form | $\frac{1}{2}\left\|\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix}\right\|$ |
| With origin | $\frac{1}{2}\|x_1y_2 - x_2y_1\|$ |
| Triangle with intercepts a, b | $\frac{1}{2}\|ab\|$ |
| Collinearity test | Area = 0 |
| Quadrilateral (shoelace) | $\frac{1}{2}\|(x_1y_2 - x_2y_1) + (x_2y_3 - x_3y_2) + (x_3y_4 - x_4y_3) + (x_4y_1 - x_1y_4)\|$ |

---

## ❓ Quick Revision Questions

1. **Find the area of the triangle with vertices (0, 0), (5, 0), and (3, 4).**

2. **Calculate the area of a triangle formed by points (−2, 1), (3, 4), and (5, −2).**

3. **If the area of triangle with vertices (k, 0), (4, 0), and (0, 2) is 4 sq units, find k.**

4. **Find the area of the quadrilateral with vertices (0, 0), (4, 0), (4, 3), and (0, 3).**

5. **A triangle has vertices A(1, 1), B(7, 1), and C(4, 5). Find its area.**

6. **Using the area formula, check if points (1, 2), (3, 6), and (5, 10) are collinear.**

<details>
<summary><b>Click to see answers</b></summary>

1. Area = $\frac{1}{2}|0(0-4) + 5(4-0) + 3(0-0)| = \frac{1}{2}|20| = 10$ sq units

2. Area = $\frac{1}{2}|(-2)(4-(-2)) + 3((-2)-1) + 5(1-4)|$
   = $\frac{1}{2}|(-2)(6) + 3(-3) + 5(-3)|$
   = $\frac{1}{2}|-12 - 9 - 15| = \frac{1}{2} \times 36 = 18$ sq units

3. $4 = \frac{1}{2}|k(0-2) + 4(2-0) + 0(0-0)|$
   $8 = |-2k + 8|$
   $-2k + 8 = 8$ or $-2k + 8 = -8$
   $k = 0$ or $k = 8$

4. Using shoelace: $\frac{1}{2}|(0×0 + 4×3 + 4×3 + 0×0) - (4×0 + 4×0 + 0×3 + 0×3)|$
   = $\frac{1}{2}|24 - 0| = 12$ sq units
   (This is a rectangle 4×3 = 12) ✓

5. Area = $\frac{1}{2}|1(1-5) + 7(5-1) + 4(1-1)|$
   = $\frac{1}{2}|(-4) + 28 + 0| = 12$ sq units

6. Area = $\frac{1}{2}|1(6-10) + 3(10-2) + 5(2-6)|$
   = $\frac{1}{2}|(-4) + 24 + (-20)| = \frac{1}{2}|0| = 0$
   Since area = 0, the points are **collinear**.

</details>

---

## ⏭️ Navigation

| [← Previous: Midpoint Formula](04-midpoint-formula.md) | [Next: Collinearity →](06-collinearity.md) |
|:------------------------------------------------------|------------------------------------------:|
