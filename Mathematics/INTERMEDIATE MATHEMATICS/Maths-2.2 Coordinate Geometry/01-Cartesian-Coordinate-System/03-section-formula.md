# Chapter 1.3: Section Formula

## 📚 Chapter Overview

The section formula is a powerful tool for finding the coordinates of a point that divides a line segment in a given ratio. This concept is essential for understanding midpoints, centroids, and many geometric constructions.

---

## 📝 Types of Division

A line segment can be divided by a point in two ways:

### Internal Division
The point lies **between** the two endpoints.

```
        A●━━━━━━━━━━━━●P━━━━━━━●B
        ←────m────→ ←───n───→
        
        P divides AB internally in ratio m:n
```

### External Division
The point lies **outside** the line segment (on its extension).

```
        A●━━━━━━━━━━━━●B━━━━━━━●P
        ←─────m─────→  ←──n──→
        (Extended beyond B)
        
        P divides AB externally in ratio m:n
```

Or:

```
        P●━━━━━━━●A━━━━━━━━━━━━●B
         ←──n──→ ←─────m─────→
        (Extended beyond A)
```

---

## ⚡ Internal Section Formula

### Derivation

Let $A(x_1, y_1)$ and $B(x_2, y_2)$ be two points, and let $P(x, y)$ divide AB internally in the ratio $m:n$.

```
        Y
        ↑
        │                    B(x₂, y₂)
        │                    ●
        │                   /
        │                  /
        │            n    /
        │                /
        │         P(x,y)●
        │              /
        │             /
        │        m   /
        │           /
        │          /
        │   A(x₁, y₁)
        │   ●
    ────O────────────────────→ X
```

**Using Similar Triangles:**

Draw perpendiculars from A, P, B to the X-axis meeting it at L, M, N respectively.

Since triangles APC' and PBD' are similar:

$$\frac{AP}{PB} = \frac{m}{n} = \frac{x - x_1}{x_2 - x}$$

**Solving for x:**
$$n(x - x_1) = m(x_2 - x)$$
$$nx - nx_1 = mx_2 - mx$$
$$nx + mx = mx_2 + nx_1$$
$$x(m + n) = mx_2 + nx_1$$

$$\boxed{x = \frac{mx_2 + nx_1}{m + n}}$$

**Similarly for y:**

$$\boxed{y = \frac{my_2 + ny_1}{m + n}}$$

---

> **Internal Section Formula**: If P divides AB internally in ratio $m:n$, where $A(x_1, y_1)$ and $B(x_2, y_2)$:
>
> $$\boxed{P = \left(\frac{mx_2 + nx_1}{m + n}, \frac{my_2 + ny_1}{m + n}\right)}$$

---

## ⚡ External Section Formula

When P divides AB externally in ratio $m:n$:

> **External Section Formula**: 
>
> $$\boxed{P = \left(\frac{mx_2 - nx_1}{m - n}, \frac{my_2 - ny_1}{m - n}\right)}$$

### Understanding External Division

```
    For external division in ratio m:n (m > n):
    
        A●━━━━━━━━━━━━●B━━━━━━━●P
        
    AP : PB = m : n  (externally)
    This means: AP/PB = m/n where P is outside AB
```

> 💡 **Tip**: External division formula can be derived from internal by replacing $n$ with $-n$.

---

## 📝 Special Cases

### Case 1: Division in Equal Ratio (1:1)

When $m = n$, P is the **midpoint**:
$$P = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)$$

### Case 2: Finding the Ratio

If point P(x, y) lies on line AB, the ratio in which P divides AB can be found:

$$\frac{m}{n} = \frac{x - x_1}{x_2 - x} = \frac{y - y_1}{y_2 - y}$$

### Case 3: Trisection Points

Points dividing a segment in ratio 1:2 and 2:1:

```
        A●━━━━━━●P₁━━━━━━●P₂━━━━━━●B
         ←─1─→ ←──2──→
         
         P₁ divides AB in ratio 1:2
         P₂ divides AB in ratio 2:1
```

---

## ✏️ Worked Examples

### Example 1: Internal Division

**Problem**: Find the coordinates of point P that divides the line segment joining A(4, −3) and B(8, 5) internally in the ratio 3:1.

**Solution**:

Given: $A(4, -3)$, $B(8, 5)$, $m:n = 3:1$

Using internal section formula:

$$x = \frac{mx_2 + nx_1}{m + n} = \frac{3(8) + 1(4)}{3 + 1} = \frac{24 + 4}{4} = \frac{28}{4} = 7$$

$$y = \frac{my_2 + ny_1}{m + n} = \frac{3(5) + 1(-3)}{3 + 1} = \frac{15 - 3}{4} = \frac{12}{4} = 3$$

**Answer**: P = **(7, 3)**

```
        Y
        ↑
    5   │               ●B(8, 5)
        │              /
    3   │          ●P(7, 3)
        │         /
        │        /
        │       /
    ────O──────/──────────→ X
        │     /
   -3   │    ●A(4, -3)
        │
```

---

### Example 2: External Division

**Problem**: Find the coordinates of point P that divides the line joining A(2, 3) and B(6, 8) externally in the ratio 2:1.

**Solution**:

Given: $A(2, 3)$, $B(6, 8)$, $m:n = 2:1$ (external)

Using external section formula:

$$x = \frac{mx_2 - nx_1}{m - n} = \frac{2(6) - 1(2)}{2 - 1} = \frac{12 - 2}{1} = 10$$

$$y = \frac{my_2 - ny_1}{m - n} = \frac{2(8) - 1(3)}{2 - 1} = \frac{16 - 3}{1} = 13$$

**Answer**: P = **(10, 13)**

```
        Y
        ↑
   13   │                    ●P(10, 13)
        │                   /
        │                  /
    8   │          ●B(6, 8)
        │         /
        │        /
    3   │   ●A(2, 3)
        │
    ────O────────────────────→ X
            2    6      10
```

---

### Example 3: Finding the Ratio

**Problem**: In what ratio does the point (−2, 3) divide the line segment joining (−3, 5) and (4, −9)?

**Solution**:

Let the ratio be $m:n$ (assuming internal division).

Let $A(-3, 5)$, $B(4, -9)$, $P(-2, 3)$

Using the section formula:
$$-2 = \frac{m(4) + n(-3)}{m + n}$$

$$-2(m + n) = 4m - 3n$$
$$-2m - 2n = 4m - 3n$$
$$n = 6m$$
$$\frac{m}{n} = \frac{1}{6}$$

**Verification with y-coordinate:**
$$3 = \frac{m(-9) + n(5)}{m + n} = \frac{m(-9) + 6m(5)}{m + 6m} = \frac{-9m + 30m}{7m} = \frac{21m}{7m} = 3$$ ✓

**Answer**: The ratio is **1:6** (internal division)

---

### Example 4: Trisection Points

**Problem**: Find the coordinates of the trisection points of the line segment joining (1, −2) and (−3, 4).

**Solution**:

Let $A(1, -2)$ and $B(-3, 4)$

**Point P₁** divides AB in ratio **1:2**:
$$P_1 = \left(\frac{1(-3) + 2(1)}{1 + 2}, \frac{1(4) + 2(-2)}{1 + 2}\right)$$
$$P_1 = \left(\frac{-3 + 2}{3}, \frac{4 - 4}{3}\right) = \left(\frac{-1}{3}, 0\right)$$

**Point P₂** divides AB in ratio **2:1**:
$$P_2 = \left(\frac{2(-3) + 1(1)}{2 + 1}, \frac{2(4) + 1(-2)}{2 + 1}\right)$$
$$P_2 = \left(\frac{-6 + 1}{3}, \frac{8 - 2}{3}\right) = \left(\frac{-5}{3}, 2\right)$$

**Answer**: Trisection points are $\left(-\frac{1}{3}, 0\right)$ and $\left(-\frac{5}{3}, 2\right)$

```
    A(1,-2)●━━━━━━●P₁(-1/3,0)━━━━━━●P₂(-5/3,2)━━━━━━●B(-3,4)
           ←─1─→   ←────1────→     ←────1────→
```

---

### Example 5: Points on Axes

**Problem**: In what ratio does the X-axis divide the line segment joining (2, −3) and (5, 6)?

**Solution**:

Any point on the X-axis has y-coordinate = 0.

Let the ratio be $k:1$, and let P(x, 0) be the point of division.

$$0 = \frac{k(6) + 1(-3)}{k + 1}$$

$$0 = 6k - 3$$
$$k = \frac{1}{2}$$

So the ratio is $\frac{1}{2}:1 = 1:2$

**Finding the point:**
$$x = \frac{\frac{1}{2}(5) + 1(2)}{\frac{1}{2} + 1} = \frac{2.5 + 2}{1.5} = \frac{4.5}{1.5} = 3$$

**Answer**: X-axis divides the segment in ratio **1:2** at point **(3, 0)**

---

### Example 6: Centroid of Triangle

**Problem**: Find the centroid of the triangle with vertices A(1, 2), B(3, 4), and C(5, 6).

**Solution**:

The centroid divides each median in ratio 2:1 from the vertex.

**Centroid Formula:**
$$G = \left(\frac{x_1 + x_2 + x_3}{3}, \frac{y_1 + y_2 + y_3}{3}\right)$$

$$G = \left(\frac{1 + 3 + 5}{3}, \frac{2 + 4 + 6}{3}\right) = \left(\frac{9}{3}, \frac{12}{3}\right) = (3, 4)$$

**Answer**: Centroid G = **(3, 4)**

---

## 📝 Important Points and Formulas

### Centroid of a Triangle

The **centroid** is the point of intersection of medians:

$$G = \left(\frac{x_1 + x_2 + x_3}{3}, \frac{y_1 + y_2 + y_3}{3}\right)$$

```
            A
            ●
           /|\
          / | \
         /  |  \
        /   |G  \
       /    ●    \
      /   / | \   \
     /  /   |   \  \
    ●───────●───────●
    B       M       C
    
    G divides each median in ratio 2:1 from vertex
```

### Incenter of a Triangle

The **incenter** is the intersection of angle bisectors:

$$I = \left(\frac{ax_1 + bx_2 + cx_3}{a + b + c}, \frac{ay_1 + by_2 + cy_3}{a + b + c}\right)$$

where $a$, $b$, $c$ are the lengths of sides opposite to vertices A, B, C.

---

## 📝 Division by Coordinate Axes

### Division by X-axis

For X-axis division, set y = 0 in section formula:
- Ratio $= -\frac{y_1}{y_2}$
- If ratio is positive → internal division
- If ratio is negative → external division

### Division by Y-axis

For Y-axis division, set x = 0 in section formula:
- Ratio $= -\frac{x_1}{x_2}$

---

## 💡 Tips and Insights

> 💡 **Memory Tip**: In section formula, the coordinate of the point being approached (B) is multiplied by the first part of the ratio (m).

> 💡 **Quick Check**: For internal division, verify that P lies between A and B.

> ⚠️ **Common Mistake**: Don't confuse internal and external division formulas (+ vs −).

> ⚠️ **Sign Convention**: For external division, ensure $m \neq n$ to avoid division by zero.

---

## 🌍 Real-World Applications

1. **Construction**: Finding points to divide beams or supports in specific ratios
2. **Computer Graphics**: Linear interpolation between points
3. **Navigation**: Finding intermediate waypoints along a route
4. **Physics**: Center of mass calculations
5. **Engineering**: Load distribution calculations

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Internal Division ($m:n$) | $P = \left(\frac{mx_2 + nx_1}{m + n}, \frac{my_2 + ny_1}{m + n}\right)$ |
| External Division ($m:n$) | $P = \left(\frac{mx_2 - nx_1}{m - n}, \frac{my_2 - ny_1}{m - n}\right)$ |
| Midpoint (1:1) | $M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)$ |
| Centroid | $G = \left(\frac{x_1 + x_2 + x_3}{3}, \frac{y_1 + y_2 + y_3}{3}\right)$ |
| Find ratio k:1 at P(x,y) | $k = \frac{x - x_1}{x_2 - x}$ or $k = \frac{y - y_1}{y_2 - y}$ |

---

## ❓ Quick Revision Questions

1. **Find the point dividing the join of (1, 2) and (4, 5) internally in ratio 2:1.**

2. **In what ratio does (2, 4) divide the line segment joining (1, 2) and (4, 8)?**

3. **Find the point dividing A(−1, 2) and B(3, 5) externally in ratio 5:2.**

4. **Find the trisection points of the line joining (−4, 2) and (8, 6).**

5. **In what ratio does the Y-axis divide the join of (−3, 4) and (5, −2)?**

6. **Find the centroid of triangle with vertices (0, 0), (6, 0), and (3, 9).**

<details>
<summary><b>Click to see answers</b></summary>

1. $P = \left(\frac{2(4) + 1(1)}{3}, \frac{2(5) + 1(2)}{3}\right) = \left(3, 4\right)$

2. Using $\frac{m}{n} = \frac{2-1}{4-2} = \frac{1}{2}$, so ratio is **1:2**

3. $P = \left(\frac{5(3) - 2(-1)}{5-2}, \frac{5(5) - 2(2)}{5-2}\right) = \left(\frac{17}{3}, 7\right)$

4. P₁ (ratio 1:2): $\left(0, \frac{10}{3}\right)$
   P₂ (ratio 2:1): $\left(4, \frac{14}{3}\right)$

5. For Y-axis, x = 0. Ratio $= \frac{0-(-3)}{5-0} = \frac{3}{5}$ → **3:5** internally

6. $G = \left(\frac{0+6+3}{3}, \frac{0+0+9}{3}\right) = (3, 3)$

</details>

---

## ⏭️ Navigation

| [← Previous: Distance Formula](02-distance-formula.md) | [Next: Midpoint Formula →](04-midpoint-formula.md) |
|:------------------------------------------------------|--------------------------------------------------:|
