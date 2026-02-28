# Chapter 4.4: Area of Triangle Using Determinants

[← Previous: Row and Column Operations](03-row-column-operations.md) | [Back to README](../README.md) | [Next: Cramer's Rule →](05-cramers-rule.md)

---

## 📚 Chapter Overview

One of the elegant applications of determinants is calculating the area of a triangle when the coordinates of its vertices are known. This geometric application connects linear algebra with coordinate geometry.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Derive the area formula using determinants
- Calculate triangle areas using the determinant method
- Determine if three points are collinear
- Extend the concept to other geometric applications

---

## 1. The Area Formula

### Statement

> For a triangle with vertices at $(x_1, y_1)$, $(x_2, y_2)$, and $(x_3, y_3)$:
> 
> $$\text{Area} = \frac{1}{2} \left| \begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix} \right|$$

### Note

The absolute value ensures the area is always positive, regardless of the orientation of vertices.

### Visual Representation

```
                    y
                    │
                    │      B(x₂, y₂)
                    │      ╱╲
                    │     ╱  ╲
                    │    ╱    ╲
                    │   ╱      ╲
                    │  ╱   Δ    ╲
                    │ ╱          ╲
    ────────────────A(x₁,y₁)──────C(x₃,y₃)──────────── x
                    │
                    │
                    
    Area = ½|det|  where det = | x₁  y₁  1 |
                               | x₂  y₂  1 |
                               | x₃  y₃  1 |
```

---

## 2. Derivation of the Formula

### Starting Point: Standard Area Formula

Using the cross product or shoelace formula:

$$\text{Area} = \frac{1}{2}|x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)|$$

### Connection to Determinant

Expand the determinant along the third column:

$$\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix}$$

$$= 1 \cdot \begin{vmatrix} x_2 & y_2 \\ x_3 & y_3 \end{vmatrix} - 1 \cdot \begin{vmatrix} x_1 & y_1 \\ x_3 & y_3 \end{vmatrix} + 1 \cdot \begin{vmatrix} x_1 & y_1 \\ x_2 & y_2 \end{vmatrix}$$

$$= (x_2 y_3 - y_2 x_3) - (x_1 y_3 - y_1 x_3) + (x_1 y_2 - y_1 x_2)$$

$$= x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)$$

This matches the shoelace formula! ✓

---

## 3. Step-by-Step Calculation

### Example 1: Basic Triangle

Find the area of the triangle with vertices A(1, 2), B(4, 6), and C(3, 1).

**Step 1**: Set up the determinant

$$\text{Area} = \frac{1}{2}\left|\begin{vmatrix} 1 & 2 & 1 \\ 4 & 6 & 1 \\ 3 & 1 & 1 \end{vmatrix}\right|$$

**Step 2**: Evaluate the determinant

Using expansion along column 3:

$$= 1(4 \cdot 1 - 6 \cdot 3) - 1(1 \cdot 1 - 2 \cdot 3) + 1(1 \cdot 6 - 2 \cdot 4)$$
$$= (4 - 18) - (1 - 6) + (6 - 8)$$
$$= -14 - (-5) + (-2)$$
$$= -14 + 5 - 2$$
$$= -11$$

**Step 3**: Apply the area formula

$$\text{Area} = \frac{1}{2}|-11| = \frac{11}{2} = 5.5 \text{ square units}$$

---

## 4. Alternative Calculation Method

### Using Row Operations

For the same triangle A(1, 2), B(4, 6), C(3, 1):

$$\begin{vmatrix} 1 & 2 & 1 \\ 4 & 6 & 1 \\ 3 & 1 & 1 \end{vmatrix}$$

Apply $R_2 \to R_2 - R_1$ and $R_3 \to R_3 - R_1$:

$$= \begin{vmatrix} 1 & 2 & 1 \\ 3 & 4 & 0 \\ 2 & -1 & 0 \end{vmatrix}$$

Expand along column 3:

$$= 1 \cdot \begin{vmatrix} 3 & 4 \\ 2 & -1 \end{vmatrix}$$

$$= 3(-1) - 4(2) = -3 - 8 = -11$$

Area = $\frac{1}{2}|-11| = 5.5$ ✓

---

## 5. Special Case: Collinear Points

### When Are Points Collinear?

> Three points are **collinear** (lie on the same line) if and only if the area of the triangle they form is **zero**.

### Condition

$$\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix} = 0$$

### Visual

```
                    Collinear Points
                    
        ──────●────────●────────●──────
             A          B          C
             
        Area = 0 (no triangle formed!)
```

### Example 2: Testing Collinearity

Are points A(1, 2), B(2, 4), and C(3, 6) collinear?

$$\begin{vmatrix} 1 & 2 & 1 \\ 2 & 4 & 1 \\ 3 & 6 & 1 \end{vmatrix}$$

Notice: Row 2 = 2 × Row 1 (in the first two columns)... Let's verify:

$R_2 \to R_2 - 2R_1$:
$$\begin{vmatrix} 1 & 2 & 1 \\ 0 & 0 & -1 \\ 3 & 6 & 1 \end{vmatrix}$$

$R_3 \to R_3 - 3R_1$:
$$\begin{vmatrix} 1 & 2 & 1 \\ 0 & 0 & -1 \\ 0 & 0 & -2 \end{vmatrix}$$

$R_3 \to R_3 - 2R_2$:
$$\begin{vmatrix} 1 & 2 & 1 \\ 0 & 0 & -1 \\ 0 & 0 & 0 \end{vmatrix}$$

Determinant = 0 (zero row)

**Conclusion**: Yes, the points are collinear! ✓

(They all lie on the line $y = 2x$)

---

## 6. Signed Area and Orientation

### Understanding the Sign

The determinant (without absolute value) gives a **signed area**:

- **Positive**: Vertices are in counterclockwise order
- **Negative**: Vertices are in clockwise order

### Example

```
    Counterclockwise (CCW)              Clockwise (CW)
    Determinant > 0                     Determinant < 0
    
           B                                  A
          /\                                 /\
         /  \                               /  \
        /    \                             /    \
       A──────C                           B──────C
```

### Application

The sign can be used to determine the orientation of vertices, which is useful in:
- Computer graphics
- Computational geometry
- Determining if a point is inside a polygon

---

## 7. More Examples

### Example 3: Origin as a Vertex

Find the area of the triangle with vertices O(0, 0), A(4, 0), and B(0, 3).

$$\text{Area} = \frac{1}{2}\left|\begin{vmatrix} 0 & 0 & 1 \\ 4 & 0 & 1 \\ 0 & 3 & 1 \end{vmatrix}\right|$$

Expand along Row 1:
$$= \frac{1}{2}\left|0 - 0 + 1\begin{vmatrix} 4 & 0 \\ 0 & 3 \end{vmatrix}\right|$$
$$= \frac{1}{2}|12| = 6 \text{ square units}$$

**Verification**: Base = 4, Height = 3, Area = $\frac{1}{2}(4)(3) = 6$ ✓

### Example 4: Finding Unknown Vertex

If points A(1, 1), B(5, 3), and C(k, 7) are collinear, find k.

**Solution**: Set the determinant equal to 0:

$$\begin{vmatrix} 1 & 1 & 1 \\ 5 & 3 & 1 \\ k & 7 & 1 \end{vmatrix} = 0$$

Expand:
$$1(3-7) - 1(5-k) + 1(35-3k) = 0$$
$$-4 - 5 + k + 35 - 3k = 0$$
$$26 - 2k = 0$$
$$k = 13$$

---

## 8. Area of Quadrilateral

### Method: Divide into Two Triangles

For a quadrilateral with vertices A, B, C, D:

$$\text{Area} = \text{Area}(\triangle ABC) + \text{Area}(\triangle ACD)$$

### Alternative: Direct Formula

$$\text{Area} = \frac{1}{2}|x_1(y_2 - y_4) + x_2(y_3 - y_1) + x_3(y_4 - y_2) + x_4(y_1 - y_3)|$$

### Example

Find the area of quadrilateral with vertices A(0,0), B(4,0), C(4,3), D(0,3).

$$\text{Area}(\triangle ABC) = \frac{1}{2}\left|\begin{vmatrix} 0 & 0 & 1 \\ 4 & 0 & 1 \\ 4 & 3 & 1 \end{vmatrix}\right| = \frac{1}{2}|0 - 0 + 1(12-0)| = 6$$

$$\text{Area}(\triangle ACD) = \frac{1}{2}\left|\begin{vmatrix} 0 & 0 & 1 \\ 4 & 3 & 1 \\ 0 & 3 & 1 \end{vmatrix}\right| = \frac{1}{2}|0 - 0 + 1(12-0)| = 6$$

Total Area = $6 + 6 = 12$ square units

(This is a 4×3 rectangle, so Area = 12 ✓)

---

## 9. Summary Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                    AREA CALCULATION FLOWCHART                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Given: Three points (x₁,y₁), (x₂,y₂), (x₃,y₃)                │
│                          │                                       │
│                          ▼                                       │
│   ┌─────────────────────────────────────────┐                   │
│   │ Calculate: D = | x₁  y₁  1 |            │                   │
│   │                | x₂  y₂  1 |            │                   │
│   │                | x₃  y₃  1 |            │                   │
│   └─────────────────┬───────────────────────┘                   │
│                     │                                            │
│           ┌─────────┴─────────┐                                 │
│           │                   │                                  │
│           ▼                   ▼                                  │
│       D = 0?              D ≠ 0?                                │
│           │                   │                                  │
│           ▼                   ▼                                  │
│   Points are           Area = ½|D|                              │
│   COLLINEAR                                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Situation | Formula | Result |
|-----------|---------|--------|
| Area of triangle | $\frac{1}{2}\|\text{det}\|$ | Area in square units |
| Collinearity test | det = 0 | Points are collinear |
| Signed area > 0 | - | CCW orientation |
| Signed area < 0 | - | CW orientation |

---

## ❓ Quick Revision Questions

1. **What is the area formula using determinants?**
   <details>
   <summary>Click for Answer</summary>
   Area = $\frac{1}{2}\left|\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix}\right|$
   </details>

2. **How do you test if three points are collinear?**
   <details>
   <summary>Click for Answer</summary>
   Calculate the determinant. If it equals zero, the points are collinear.
   </details>

3. **Find the area of triangle with vertices (0,0), (3,0), (0,4).**
   <details>
   <summary>Click for Answer</summary>
   det = $\begin{vmatrix} 0 & 0 & 1 \\ 3 & 0 & 1 \\ 0 & 4 & 1 \end{vmatrix} = 12$. Area = $\frac{1}{2}(12) = 6$ sq units.
   </details>

4. **What does a negative determinant mean in the area formula?**
   <details>
   <summary>Click for Answer</summary>
   It indicates clockwise orientation of vertices. The absolute value is still used for area.
   </details>

5. **Why is the factor 1/2 in the area formula?**
   <details>
   <summary>Click for Answer</summary>
   The determinant gives twice the signed area (related to the parallelogram formed by the vectors), so we divide by 2.
   </details>

6. **If A(1,1), B(2,k), C(3,5) are collinear, find k.**
   <details>
   <summary>Click for Answer</summary>
   $\begin{vmatrix} 1 & 1 & 1 \\ 2 & k & 1 \\ 3 & 5 & 1 \end{vmatrix} = 0$. Solving: $1(k-5) - 1(2-3) + 1(10-3k) = 0$, gives $k - 5 + 1 + 10 - 3k = 0$, so $-2k + 6 = 0$, thus $k = 3$.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Row and Column Operations](03-row-column-operations.md) | [Unit 4: Applications](./README.md) | [Cramer's Rule →](05-cramers-rule.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
