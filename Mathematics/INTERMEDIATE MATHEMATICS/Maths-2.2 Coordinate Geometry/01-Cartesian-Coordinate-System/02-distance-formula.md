# Chapter 1.2: Distance Formula

## 📚 Chapter Overview

The distance formula is one of the most fundamental tools in coordinate geometry. It allows us to calculate the exact distance between any two points in the Cartesian plane using their coordinates. This formula is derived from the Pythagorean theorem and forms the basis for many advanced concepts.

---

## 📝 Derivation of Distance Formula

### The Concept

To find the distance between two points, we construct a right triangle and apply the Pythagorean theorem.

### Geometric Construction

Consider two points $P(x_1, y_1)$ and $Q(x_2, y_2)$ in the coordinate plane:

```
        Y
        ↑
        │                    Q(x₂, y₂)
        │                    ●
        │                   /│
        │                  / │
        │                 /  │
        │           d    /   │ (y₂ - y₁)
        │               /    │
        │              /     │
        │             /      │
        │   P(x₁, y₁)●───────●R(x₂, y₁)
        │             (x₂ - x₁)
        │
    ────O────────────────────────────→ X
        │
```

### Step-by-Step Derivation

**Step 1**: Construct point R at $(x_2, y_1)$ to form a right angle at R.

**Step 2**: Calculate the horizontal distance (PR):
$$PR = |x_2 - x_1|$$

**Step 3**: Calculate the vertical distance (QR):
$$QR = |y_2 - y_1|$$

**Step 4**: Apply the Pythagorean theorem:
$$PQ^2 = PR^2 + QR^2$$

**Step 5**: Substitute and simplify:
$$PQ^2 = (x_2 - x_1)^2 + (y_2 - y_1)^2$$

$$PQ = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

---

## ⚡ The Distance Formula

> **Distance Formula**: The distance $d$ between two points $P(x_1, y_1)$ and $Q(x_2, y_2)$ is:
> 
> $$\boxed{d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}}$$

### Alternative Forms

The formula can also be written as:

$$d = \sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2}$$

> 💡 **Note**: Since we're squaring the differences, it doesn't matter which point comes first: $(x_2 - x_1)^2 = (x_1 - x_2)^2$

---

## 📝 Distance from Origin

### Special Case

When one point is the origin $O(0, 0)$, the formula simplifies:

$$d = \sqrt{(x - 0)^2 + (y - 0)^2} = \sqrt{x^2 + y^2}$$

> **Distance from Origin**: For point $P(x, y)$:
> $$\boxed{OP = \sqrt{x^2 + y^2}}$$

```
        Y
        ↑
        │           P(x, y)
        │           ●
        │          /│
        │         / │
        │    d   /  │ y
        │       /   │
        │      /    │
        │     /     │
    ────O────●──────●────→ X
             0      x
```

---

## ✏️ Worked Examples

### Example 1: Basic Distance Calculation

**Problem**: Find the distance between points A(3, 4) and B(7, 1).

**Solution**:
$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

$$d = \sqrt{(7 - 3)^2 + (1 - 4)^2}$$

$$d = \sqrt{(4)^2 + (-3)^2}$$

$$d = \sqrt{16 + 9} = \sqrt{25} = 5$$

**Answer**: The distance is **5 units**.

```
        Y
        ↑
    4   │   A●─────────┐
        │   │          │ 3
    1   │   │          ●B
        │   │    4     │
    ────O───┴──────────┴──→ X
            3          7
```

---

### Example 2: Distance from Origin

**Problem**: Find the distance of point P(−5, 12) from the origin.

**Solution**:
$$OP = \sqrt{x^2 + y^2}$$

$$OP = \sqrt{(-5)^2 + (12)^2}$$

$$OP = \sqrt{25 + 144} = \sqrt{169} = 13$$

**Answer**: The distance from origin is **13 units**.

---

### Example 3: Finding Unknown Coordinate

**Problem**: If the distance between points A(2, −3) and B(x, 5) is 10 units, find the value of x.

**Solution**:
Given: $AB = 10$

Using the distance formula:
$$\sqrt{(x - 2)^2 + (5 - (-3))^2} = 10$$

$$\sqrt{(x - 2)^2 + (8)^2} = 10$$

Squaring both sides:
$$(x - 2)^2 + 64 = 100$$

$$(x - 2)^2 = 36$$

$$x - 2 = \pm 6$$

$$x = 2 + 6 = 8 \quad \text{or} \quad x = 2 - 6 = -4$$

**Answer**: $x = 8$ or $x = -4$

---

### Example 4: Proving an Isosceles Triangle

**Problem**: Show that points A(4, 3), B(6, 4), and C(5, 6) form an isosceles triangle.

**Solution**:

Calculate all three sides:

**Side AB:**
$$AB = \sqrt{(6-4)^2 + (4-3)^2} = \sqrt{4 + 1} = \sqrt{5}$$

**Side BC:**
$$BC = \sqrt{(5-6)^2 + (6-4)^2} = \sqrt{1 + 4} = \sqrt{5}$$

**Side CA:**
$$CA = \sqrt{(4-5)^2 + (3-6)^2} = \sqrt{1 + 9} = \sqrt{10}$$

Since $AB = BC = \sqrt{5}$, the triangle has two equal sides.

**Conclusion**: Triangle ABC is **isosceles**.

```
            Y
            ↑
        6   │       C●
            │       / \
        5   │      /   \
            │     /     \
        4   │   A●───────●B
            │
        3   │
            │
    ────────O───┬───┬───┬───→ X
                4   5   6
```

---

### Example 5: Equidistant Points

**Problem**: Find a point on the Y-axis that is equidistant from A(4, 3) and B(2, 5).

**Solution**:

Let the point on Y-axis be P(0, y).

Given: PA = PB

$$\sqrt{(0-4)^2 + (y-3)^2} = \sqrt{(0-2)^2 + (y-5)^2}$$

Squaring both sides:
$$16 + (y-3)^2 = 4 + (y-5)^2$$

$$16 + y^2 - 6y + 9 = 4 + y^2 - 10y + 25$$

$$25 - 6y = 29 - 10y$$

$$4y = 4$$

$$y = 1$$

**Answer**: The point is **P(0, 1)**.

---

### Example 6: Type of Triangle

**Problem**: Determine the type of triangle formed by vertices A(0, 0), B(3, 4), and C(7, 7).

**Solution**:

Calculate all sides:

$$AB = \sqrt{(3-0)^2 + (4-0)^2} = \sqrt{9 + 16} = \sqrt{25} = 5$$

$$BC = \sqrt{(7-3)^2 + (7-4)^2} = \sqrt{16 + 9} = \sqrt{25} = 5$$

$$CA = \sqrt{(0-7)^2 + (0-7)^2} = \sqrt{49 + 49} = \sqrt{98} = 7\sqrt{2}$$

Check for right angle using Pythagorean theorem:
$$AB^2 + BC^2 = 25 + 25 = 50$$
$$CA^2 = 98$$

Since $AB^2 + BC^2 \neq CA^2$, it's not a right triangle.

Since $AB = BC = 5$, the triangle is **isosceles** (not equilateral, not right-angled).

---

## 📝 Properties and Applications

### Properties of Distance

| Property | Description | Mathematical Form |
|----------|-------------|-------------------|
| Non-negative | Distance is always ≥ 0 | $d \geq 0$ |
| Zero distance | Points coincide | $d = 0 \Leftrightarrow P = Q$ |
| Symmetry | Order doesn't matter | $d(P, Q) = d(Q, P)$ |
| Triangle inequality | Sum of two sides > third side | $d(P, R) \leq d(P, Q) + d(Q, R)$ |

### Identifying Triangle Types

| Condition | Type of Triangle |
|-----------|------------------|
| All three sides equal | Equilateral |
| Two sides equal | Isosceles |
| All sides different | Scalene |
| $a^2 + b^2 = c^2$ (longest side c) | Right-angled |
| $a^2 + b^2 > c^2$ | Acute-angled |
| $a^2 + b^2 < c^2$ | Obtuse-angled |

---

## 📝 Identifying Quadrilateral Types

Using the distance formula, we can identify special quadrilaterals:

```
    Square                 Rectangle              Rhombus
    D────────C             D──────────C           D
    │        │             │          │          /\
    │        │             │          │         /  \
    │        │             │          │        /    \
    A────────B             A──────────B       A      C
                                               \    /
    AB=BC=CD=DA            AB=CD, BC=DA         \  /
    AC=BD                  AC=BD                  \/
                                                  B
                                              AB=BC=CD=DA
```

| Quadrilateral | Side Conditions | Diagonal Conditions |
|--------------|-----------------|---------------------|
| Square | All sides equal | Diagonals equal |
| Rectangle | Opposite sides equal | Diagonals equal |
| Rhombus | All sides equal | Diagonals unequal |
| Parallelogram | Opposite sides equal | Diagonals unequal |

---

## 💡 Tips and Insights

> 💡 **Calculation Tip**: Always simplify inside the square root before computing.

> 💡 **Verification**: For right triangles, verify using the Pythagorean theorem.

> ⚠️ **Common Mistake**: Don't forget to square the differences before adding them.

> ⚠️ **Sign Error**: Remember that squaring eliminates negative signs, so $(−3)^2 = 9$, not $−9$.

---

## 🌍 Real-World Applications

1. **GPS Navigation**: Calculating straight-line distances between locations
2. **Architecture**: Determining diagonal measurements in construction
3. **Physics**: Computing displacement in motion problems
4. **Computer Graphics**: Calculating distances for collision detection
5. **Surveying**: Measuring distances between survey points
6. **Astronomy**: Calculating distances between celestial objects

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Distance between $(x_1, y_1)$ and $(x_2, y_2)$ | $d = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$ |
| Distance from origin to $(x, y)$ | $d = \sqrt{x^2 + y^2}$ |
| Horizontal distance | $\|x_2 - x_1\|$ |
| Vertical distance | $\|y_2 - y_1\|$ |
| Points coincide if | $d = 0$ |

---

## ❓ Quick Revision Questions

1. **Find the distance between points (5, 7) and (1, 4).**

2. **What is the distance of point (−8, 6) from the origin?**

3. **If the distance between (3, k) and (3, 5) is 7 units, find k.**

4. **Prove that triangle with vertices (1, 1), (4, 4), and (1, 4) is isosceles and right-angled.**

5. **Find the point on the X-axis equidistant from (5, 4) and (−2, 3).**

6. **If A(2, 3), B(6, 3), C(6, 7), D(2, 7) are vertices of a quadrilateral, identify its type.**

<details>
<summary><b>Click to see answers</b></summary>

1. $d = \sqrt{(1-5)^2 + (4-7)^2} = \sqrt{16 + 9} = \sqrt{25} = 5$ units

2. $d = \sqrt{(-8)^2 + 6^2} = \sqrt{64 + 36} = \sqrt{100} = 10$ units

3. Distance = $|k - 5| = 7$, so $k = 12$ or $k = -2$

4. AB = $\sqrt{9+9} = 3\sqrt{2}$, BC = $\sqrt{9+0} = 3$, CA = $\sqrt{0+9} = 3$
   - BC = CA → Isosceles
   - $BC^2 + CA^2 = 9 + 9 = 18 = AB^2$ → Right-angled at C

5. Let point be (a, 0). Then $\sqrt{(a-5)^2 + 16} = \sqrt{(a+2)^2 + 9}$
   Solving: $a = 2$. Point is **(2, 0)**

6. AB = CD = 4, BC = DA = 4, so all sides equal.
   AC = $\sqrt{16 + 16} = 4\sqrt{2}$, BD = $\sqrt{16 + 16} = 4\sqrt{2}$
   Diagonals equal. **Square**.

</details>

---

## ⏭️ Navigation

| [← Previous: Coordinate Axes](01-coordinate-axes-quadrants.md) | [Next: Section Formula →](03-section-formula.md) |
|:--------------------------------------------------------------|------------------------------------------------:|
