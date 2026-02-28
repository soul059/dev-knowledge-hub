# Chapter 1.4: Midpoint Formula

## 📚 Chapter Overview

The midpoint formula is a special case of the section formula where a point divides a line segment in the ratio 1:1. This simple yet powerful formula finds extensive applications in geometry, from finding centers of shapes to constructing perpendicular bisectors.

---

## 📝 Derivation of Midpoint Formula

### From Section Formula

The midpoint is the point that divides a line segment into two equal parts, i.e., in the ratio **1:1**.

Using the section formula with $m = n = 1$:

$$P = \left(\frac{mx_2 + nx_1}{m + n}, \frac{my_2 + ny_1}{m + n}\right)$$

$$M = \left(\frac{1 \cdot x_2 + 1 \cdot x_1}{1 + 1}, \frac{1 \cdot y_2 + 1 \cdot y_1}{1 + 1}\right)$$

$$M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)$$

---

## ⚡ The Midpoint Formula

> **Midpoint Formula**: The midpoint M of a line segment joining $A(x_1, y_1)$ and $B(x_2, y_2)$ is:
>
> $$\boxed{M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)}$$

```
        Y
        ↑
        │               B(x₂, y₂)
        │               ●
        │              /
        │             /
        │        M   ●  ← Midpoint
        │           /    ((x₁+x₂)/2, (y₁+y₂)/2)
        │          /
        │         /
        │        ●
        │       A(x₁, y₁)
    ────O──────────────────→ X
        │
```

### Key Insight

> 💡 The midpoint coordinates are simply the **average** of the corresponding coordinates of the endpoints.
>
> $$x_M = \frac{x_1 + x_2}{2} = \text{average of x-coordinates}$$
> $$y_M = \frac{y_1 + y_2}{2} = \text{average of y-coordinates}$$

---

## 📝 Properties of Midpoint

### Property 1: Equidistance

The midpoint is equidistant from both endpoints:
$$AM = MB = \frac{1}{2}AB$$

### Property 2: Verification

If M is the midpoint of AB, then:
- $x_1 + x_2 = 2x_M$
- $y_1 + y_2 = 2y_M$

### Property 3: Finding Endpoint

If M is the midpoint of AB and one endpoint is known:

Given A$(x_1, y_1)$ and midpoint M$(x_M, y_M)$, find B$(x_2, y_2)$:

$$x_2 = 2x_M - x_1$$
$$y_2 = 2y_M - y_1$$

---

## ✏️ Worked Examples

### Example 1: Basic Midpoint Calculation

**Problem**: Find the midpoint of the line segment joining A(2, 5) and B(8, 3).

**Solution**:

$$M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)$$

$$M = \left(\frac{2 + 8}{2}, \frac{5 + 3}{2}\right)$$

$$M = \left(\frac{10}{2}, \frac{8}{2}\right) = (5, 4)$$

**Answer**: Midpoint M = **(5, 4)**

```
        Y
        ↑
    5   │   ●A(2, 5)
        │    \
    4   │     ●M(5, 4)
        │      \
    3   │       ●B(8, 3)
        │
    ────O───┬───┬───┬───┬───→ X
            2   5       8
```

---

### Example 2: Finding Unknown Endpoint

**Problem**: If M(3, 4) is the midpoint of segment AB, where A = (1, 2), find the coordinates of B.

**Solution**:

Let B = $(x_2, y_2)$

Using the midpoint formula:
$$3 = \frac{1 + x_2}{2} \Rightarrow 6 = 1 + x_2 \Rightarrow x_2 = 5$$

$$4 = \frac{2 + y_2}{2} \Rightarrow 8 = 2 + y_2 \Rightarrow y_2 = 6$$

**Answer**: B = **(5, 6)**

---

### Example 3: Midpoint with Negative Coordinates

**Problem**: Find the midpoint of the line segment joining P(−4, 6) and Q(8, −2).

**Solution**:

$$M = \left(\frac{-4 + 8}{2}, \frac{6 + (-2)}{2}\right)$$

$$M = \left(\frac{4}{2}, \frac{4}{2}\right) = (2, 2)$$

**Answer**: Midpoint M = **(2, 2)**

---

### Example 4: Proving a Parallelogram

**Problem**: Show that the diagonals of a parallelogram with vertices A(1, 2), B(5, 4), C(7, 10), and D(3, 8) bisect each other.

**Solution**:

For diagonals to bisect each other, they must have the same midpoint.

**Midpoint of diagonal AC:**
$$M_{AC} = \left(\frac{1 + 7}{2}, \frac{2 + 10}{2}\right) = (4, 6)$$

**Midpoint of diagonal BD:**
$$M_{BD} = \left(\frac{5 + 3}{2}, \frac{4 + 8}{2}\right) = (4, 6)$$

Since $M_{AC} = M_{BD} = (4, 6)$, the diagonals bisect each other.

**Conclusion**: ABCD is a **parallelogram**.

```
          C(7, 10)
            ●
           /|\
          / | \
         /  |  \
    D(3,8)  |   \
        ●   |    \
         \  ●(4,6)\
          \ |      ●B(5, 4)
           \|     /
            \    /
             \  /
              \/
               ●
             A(1, 2)
```

---

### Example 5: Finding Center of a Rectangle

**Problem**: ABCD is a rectangle with vertices A(0, 0), B(8, 0), C(8, 6), and D(0, 6). Find its center.

**Solution**:

The center of a rectangle is the intersection point of its diagonals, which is also the midpoint of each diagonal.

**Midpoint of AC:**
$$\text{Center} = \left(\frac{0 + 8}{2}, \frac{0 + 6}{2}\right) = (4, 3)$$

**Verification with BD:**
$$\text{Center} = \left(\frac{8 + 0}{2}, \frac{0 + 6}{2}\right) = (4, 3)$$ ✓

**Answer**: Center = **(4, 3)**

```
    D(0, 6)●━━━━━━━━━━━━●C(8, 6)
           │           │
           │     ●     │
           │   (4,3)   │
           │           │
    A(0, 0)●━━━━━━━━━━━━●B(8, 0)
```

---

### Example 6: Line Segment on Coordinate Axes

**Problem**: A line segment has its midpoint at (6, 4). If one endpoint lies on the X-axis and the other on the Y-axis, find the endpoints.

**Solution**:

Let the endpoint on X-axis be A(a, 0) and on Y-axis be B(0, b).

**Midpoint condition:**
$$6 = \frac{a + 0}{2} \Rightarrow a = 12$$

$$4 = \frac{0 + b}{2} \Rightarrow b = 8$$

**Answer**: Endpoints are A = **(12, 0)** and B = **(0, 8)**

```
        Y
        ↑
    8   ●B(0, 8)
        |\
        | \
    4   │  ●M(6, 4)
        │   \
        │    \
    ────O─────●────────●────→ X
              6       12
                    A(12, 0)
```

---

## 📝 Applications of Midpoint Formula

### 1. Finding Centroid Using Midpoints

The centroid of a triangle can be found using the midpoint of a side and the opposite vertex.

```
            A
            ●
           /|\
          / | \
         /  |  \
        /   ●G  \   ← G divides median in 2:1
       /   /|\   \
      /   / | \   \
     ●───●──┼──●───●
     B   D  |  E   C
            |
    D = midpoint of BC
    E = midpoint of AC
```

### 2. Perpendicular Bisector

The perpendicular bisector of a segment passes through its midpoint and is perpendicular to the segment.

### 3. Medians of a Triangle

A median connects a vertex to the midpoint of the opposite side.

**Median from A to side BC:**
- Find midpoint D of BC
- Line AD is the median

---

## 📝 Midpoint and Distance Relationship

If M is the midpoint of AB:

$$AM = MB = \frac{AB}{2}$$

**Proof:**
$$AM = \sqrt{\left(\frac{x_1+x_2}{2} - x_1\right)^2 + \left(\frac{y_1+y_2}{2} - y_1\right)^2}$$

$$AM = \sqrt{\left(\frac{x_2-x_1}{2}\right)^2 + \left(\frac{y_2-y_1}{2}\right)^2}$$

$$AM = \frac{1}{2}\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2} = \frac{AB}{2}$$

---

## 📝 Extension: Midpoint in 3D

For three-dimensional space, if $A(x_1, y_1, z_1)$ and $B(x_2, y_2, z_2)$:

$$M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}, \frac{z_1 + z_2}{2}\right)$$

---

## 💡 Tips and Insights

> 💡 **Quick Check**: The midpoint must lie "between" the two endpoints when visualized.

> 💡 **Shortcut**: Midpoint = (Average of x's, Average of y's)

> ⚠️ **Common Mistake**: Don't divide by 2 twice — only divide the sum by 2, not each coordinate separately before adding.

> 💡 **Reverse Problem**: To find an endpoint given midpoint and one endpoint, use: $x_2 = 2x_M - x_1$

---

## 🌍 Real-World Applications

1. **Navigation**: Finding the halfway point between two locations
2. **Construction**: Locating the center point for balanced structures
3. **Computer Graphics**: Calculating centers of objects and shapes
4. **Physics**: Finding the center of mass of uniform objects
5. **Sports**: Determining the midfield point in games
6. **Map Reading**: Finding equidistant points between landmarks

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Midpoint of $(x_1, y_1)$ and $(x_2, y_2)$ | $M = \left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}\right)$ |
| Finding endpoint B given A and M | $B = (2x_M - x_1, 2y_M - y_1)$ |
| Distance to midpoint | $AM = MB = \frac{1}{2}AB$ |
| Midpoint x-coordinate | Average of x-coordinates |
| Midpoint y-coordinate | Average of y-coordinates |
| 3D Midpoint | $\left(\frac{x_1 + x_2}{2}, \frac{y_1 + y_2}{2}, \frac{z_1 + z_2}{2}\right)$ |

---

## ❓ Quick Revision Questions

1. **Find the midpoint of the line segment joining (4, 8) and (10, 2).**

2. **If M(5, −1) is the midpoint of AB where A = (2, 3), find B.**

3. **The vertices of a triangle are (0, 0), (6, 0), and (0, 8). Find the midpoints of all three sides.**

4. **Prove that the quadrilateral with vertices (−1, 2), (3, 0), (5, 4), and (1, 6) is a parallelogram using the midpoint concept.**

5. **A diameter of a circle has endpoints at (−3, 4) and (5, −2). Find the center of the circle.**

6. **If three consecutive vertices of a parallelogram are (1, 2), (4, 6), and (6, 4), find the fourth vertex.**

<details>
<summary><b>Click to see answers</b></summary>

1. $M = \left(\frac{4+10}{2}, \frac{8+2}{2}\right) = (7, 5)$

2. $x_B = 2(5) - 2 = 8$, $y_B = 2(-1) - 3 = -5$. So B = **(8, −5)**

3. Midpoints:
   - Between (0,0) and (6,0): **(3, 0)**
   - Between (6,0) and (0,8): **(3, 4)**
   - Between (0,8) and (0,0): **(0, 4)**

4. Midpoint of diagonal (−1, 2) to (5, 4) = (2, 3)
   Midpoint of diagonal (3, 0) to (1, 6) = (2, 3)
   Since diagonals bisect each other, it's a **parallelogram**.

5. Center = midpoint = $\left(\frac{-3+5}{2}, \frac{4-2}{2}\right) = (1, 1)$

6. In parallelogram ABCD, diagonals bisect each other.
   If A(1,2), B(4,6), C(6,4), let D(x,y).
   Midpoint of AC = Midpoint of BD
   $(3.5, 3) = \left(\frac{4+x}{2}, \frac{6+y}{2}\right)$
   So $x = 3$, $y = 0$. D = **(3, 0)**

</details>

---

## ⏭️ Navigation

| [← Previous: Section Formula](03-section-formula.md) | [Next: Area of Triangle →](05-area-of-triangle.md) |
|:----------------------------------------------------|--------------------------------------------------:|
