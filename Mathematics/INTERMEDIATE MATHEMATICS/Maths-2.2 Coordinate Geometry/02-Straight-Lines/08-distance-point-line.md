# Chapter 2.8: Distance of a Point from a Line

## 📚 Chapter Overview

The perpendicular distance from a point to a line is one of the most frequently used formulas in coordinate geometry. This chapter derives the formula and explores its numerous applications in geometry problems.

---

## 📝 Concept Introduction

The **distance from a point to a line** is defined as the length of the perpendicular segment from the point to the line.

```
                Y
                ↑
                │
           P ● 
                │\
                │ \  d = perpendicular distance
                │  \
                │   ↘
    ────────────│────\──────→ X
                │     \  Line: ax + by + c = 0
                │      \
                │       \
```

The perpendicular distance is the **shortest distance** from the point to any point on the line.

---

## ⚡ Distance Formula

> **Distance from Point to Line**:
>
> The perpendicular distance from point $(x_1, y_1)$ to line $ax + by + c = 0$ is:
>
> $$\boxed{d = \frac{|ax_1 + by_1 + c|}{\sqrt{a^2 + b^2}}}$$

---

## 📐 Derivation

### Method 1: Using Perpendicular Foot

```
                Y
                ↑
                │
           P(x₁,y₁) ●───────┐
                │    \      │ d
                │     \     │
                │      \    │
    ────────────│───────●───│────→ X
                │       N   Line L
                │     (h,k)
```

Let P$(x_1, y_1)$ be the point and L: $ax + by + c = 0$ be the line.

**Step 1**: Let N(h, k) be the foot of perpendicular from P to L.

Since N lies on line L:
$$ah + bk + c = 0 \quad \ldots (1)$$

**Step 2**: Slope of line L is $-\frac{a}{b}$

Slope of PN (perpendicular) is $\frac{b}{a}$

Using slope formula for PN:
$$\frac{k - y_1}{h - x_1} = \frac{b}{a}$$

$$a(k - y_1) = b(h - x_1)$$
$$ak - ay_1 = bh - bx_1$$
$$bh - ak = bx_1 - ay_1 \quad \ldots (2)$$

**Step 3**: From equations (1) and (2), solve for h and k:

From (1): $ah + bk = -c$
From (2): $bh - ak = bx_1 - ay_1$

Solving simultaneously:
$$h = \frac{b^2x_1 - aby_1 - ac}{a^2 + b^2}$$
$$k = \frac{a^2y_1 - abx_1 - bc}{a^2 + b^2}$$

**Step 4**: Find distance PN:
$$d = \sqrt{(x_1 - h)^2 + (y_1 - k)^2}$$

After simplification:
$$d = \frac{|ax_1 + by_1 + c|}{\sqrt{a^2 + b^2}}$$

---

### Method 2: Using Normal Form (Elegant Proof)

Convert line to normal form: $x\cos\alpha + y\sin\alpha = p$

where $\cos\alpha = \frac{a}{\sqrt{a^2+b^2}}$, $\sin\alpha = \frac{b}{\sqrt{a^2+b^2}}$, $p = \frac{-c}{\sqrt{a^2+b^2}}$

Distance from P$(x_1, y_1)$ = |Substituting P in LHS − RHS|

$$d = |x_1\cos\alpha + y_1\sin\alpha - p| = \frac{|ax_1 + by_1 + c|}{\sqrt{a^2 + b^2}}$$

---

## 📝 Understanding the Formula

```
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │            |ax₁ + by₁ + c|                       │
    │       d = ─────────────────                      │
    │              √(a² + b²)                          │
    │                                                  │
    │    ┌─────────────┬─────────────────────────────┐ │
    │    │ Numerator   │ Value when P is substituted │ │
    │    │             │ into line equation          │ │
    │    ├─────────────┼─────────────────────────────┤ │
    │    │ Denominator │ Normalizing factor          │ │
    │    │             │ (makes coefficients unit)   │ │
    │    └─────────────┴─────────────────────────────┘ │
    │                                                  │
    └──────────────────────────────────────────────────┘
```

---

## ✏️ Worked Examples

### Example 1: Basic Distance Calculation

**Problem**: Find the distance from point (3, 4) to the line $3x + 4y - 5 = 0$.

**Solution**:

Here: $a = 3$, $b = 4$, $c = -5$, $(x_1, y_1) = (3, 4)$

$$d = \frac{|3(3) + 4(4) - 5|}{\sqrt{9 + 16}}$$

$$d = \frac{|9 + 16 - 5|}{\sqrt{25}} = \frac{|20|}{5} = 4$$

**Answer**: Distance = **4 units**

---

### Example 2: Distance from Origin

**Problem**: Find the perpendicular distance from origin to the line $5x - 12y + 26 = 0$.

**Solution**:

From origin (0, 0) to line $5x - 12y + 26 = 0$:

$$d = \frac{|5(0) - 12(0) + 26|}{\sqrt{25 + 144}}$$

$$d = \frac{26}{\sqrt{169}} = \frac{26}{13} = 2$$

**Answer**: Distance = **2 units**

---

### Example 3: Finding Point on Line at Given Distance

**Problem**: Find the point on line $2x + y = 5$ that is at distance 3 from point (1, 1).

**Solution**:

Let the point be P(a, b) on the line.

Since P is on line: $2a + b = 5$ → $b = 5 - 2a$

Distance from (1, 1) to P equals 3:
$$\sqrt{(a-1)^2 + (b-1)^2} = 3$$

$$(a-1)^2 + (5-2a-1)^2 = 9$$
$$(a-1)^2 + (4-2a)^2 = 9$$
$$a^2 - 2a + 1 + 16 - 16a + 4a^2 = 9$$
$$5a^2 - 18a + 17 - 9 = 0$$
$$5a^2 - 18a + 8 = 0$$

Using quadratic formula:
$$a = \frac{18 \pm \sqrt{324 - 160}}{10} = \frac{18 \pm \sqrt{164}}{10}$$

**Answer**: Two points exist: $\left(\frac{18 + \sqrt{164}}{10}, 5 - 2 \cdot \frac{18 + \sqrt{164}}{10}\right)$ and corresponding point with minus sign.

---

### Example 4: Distance Between Parallel Lines

**Problem**: Find the distance between parallel lines $3x - 4y + 7 = 0$ and $3x - 4y - 8 = 0$.

**Solution**:

For parallel lines $ax + by + c_1 = 0$ and $ax + by + c_2 = 0$:

$$d = \frac{|c_1 - c_2|}{\sqrt{a^2 + b^2}}$$

$$d = \frac{|7 - (-8)|}{\sqrt{9 + 16}} = \frac{15}{5} = 3$$

**Answer**: Distance = **3 units**

```
        │      Line 1: 3x - 4y + 7 = 0
        │    /
        │   /
    ────│──/──────────────────→
        │ /
        │/     ←───── d = 3 ─────→
        /
       /│      Line 2: 3x - 4y - 8 = 0
      / │    /
```

---

### Example 5: Finding Value of c

**Problem**: For what value of c is the distance from point (1, 2) to line $3x + 4y + c = 0$ equal to 2 units?

**Solution**:

$$2 = \frac{|3(1) + 4(2) + c|}{\sqrt{9 + 16}}$$

$$2 = \frac{|11 + c|}{5}$$

$$|11 + c| = 10$$

**Case 1**: $11 + c = 10$ → $c = -1$

**Case 2**: $11 + c = -10$ → $c = -21$

**Answer**: $c = -1$ or $c = -21$

---

### Example 6: Equidistant Point from Two Lines

**Problem**: Find the locus of points equidistant from lines $3x - 4y + 5 = 0$ and $5x + 12y - 7 = 0$.

**Solution**:

Let P(x, y) be equidistant from both lines.

$$\frac{|3x - 4y + 5|}{\sqrt{9 + 16}} = \frac{|5x + 12y - 7|}{\sqrt{25 + 144}}$$

$$\frac{|3x - 4y + 5|}{5} = \frac{|5x + 12y - 7|}{13}$$

**Case 1**: Taking positive values
$$13(3x - 4y + 5) = 5(5x + 12y - 7)$$
$$39x - 52y + 65 = 25x + 60y - 35$$
$$14x - 112y + 100 = 0$$
$$7x - 56y + 50 = 0$$

**Case 2**: Taking negative for one
$$13(3x - 4y + 5) = -5(5x + 12y - 7)$$
$$39x - 52y + 65 = -25x - 60y + 35$$
$$64x + 8y + 30 = 0$$
$$32x + 4y + 15 = 0$$

**Answer**: Locus is two lines (angle bisectors):
- $7x - 56y + 50 = 0$
- $32x + 4y + 15 = 0$

---

### Example 7: Area of Rhombus

**Problem**: Find the area of rhombus formed by lines $3x + 4y = 5$, $3x + 4y = 15$, $4x - 3y = 5$, and $4x - 3y = 15$.

**Solution**:

**Step 1**: Find distances between parallel lines

Distance between $3x + 4y = 5$ and $3x + 4y = 15$:
$$d_1 = \frac{|5 - 15|}{5} = \frac{10}{5} = 2$$

Distance between $4x - 3y = 5$ and $4x - 3y = 15$:
$$d_2 = \frac{|5 - 15|}{5} = \frac{10}{5} = 2$$

**Step 2**: Since pairs are perpendicular (product of slopes = −1):
The diagonals are $d_1$ and $d_2$

Area = $d_1 \times d_2 = 2 \times 2 = 4$ (for rhombus with perpendicular sides)

Actually, for a rhombus formed by two pairs of parallel lines:
$$\text{Area} = \frac{d_1 \times d_2}{\sin\theta}$$

Since lines are perpendicular, $\sin\theta = 1$

**Answer**: Area = **4 square units**

---

## 📝 Distance Between Parallel Lines

> **Formula for Parallel Lines**:
>
> Distance between $ax + by + c_1 = 0$ and $ax + by + c_2 = 0$:
>
> $$\boxed{d = \frac{|c_1 - c_2|}{\sqrt{a^2 + b^2}}}$$

**Note**: Coefficients of x and y must be identical for this formula.

---

## 📝 Special Cases

### 1. Distance from Origin to Line

$$d = \frac{|c|}{\sqrt{a^2 + b^2}}$$

### 2. Distance from (α, β) to Line through Origin

Line: $ax + by = 0$

$$d = \frac{|a\alpha + b\beta|}{\sqrt{a^2 + b^2}}$$

### 3. Position of Point Relative to Line

For point P$(x_1, y_1)$ and line $ax + by + c = 0$:

- If $ax_1 + by_1 + c > 0$: P is on one side
- If $ax_1 + by_1 + c < 0$: P is on opposite side
- If $ax_1 + by_1 + c = 0$: P is on the line

---

## 💡 Tips and Insights

> 💡 **Sign Matters for Position**: The sign of $(ax_1 + by_1 + c)$ tells which side of the line the point is on.

> 💡 **Same Side Test**: Two points are on the same side if they give the same sign when substituted.

> ⚠️ **Ensure Same Coefficients**: When finding distance between parallel lines, make sure coefficients of x and y are identical (not just proportional).

> 💡 **Shortcut for $3x + 4y = c$ Lines**: Since $\sqrt{3^2 + 4^2} = 5$, the distance is simply $\frac{|c_1 - c_2|}{5}$.

---

## 🌍 Real-World Applications

1. **Navigation**: Finding shortest route to a road
2. **Computer Graphics**: Collision detection
3. **Civil Engineering**: Offset distances in surveying
4. **Physics**: Distance of charge from a boundary
5. **Robotics**: Path planning and obstacle avoidance
6. **GIS Systems**: Finding closest features

---

## 📋 Summary Table

| Scenario | Formula |
|----------|---------|
| Point $(x_1, y_1)$ to line $ax + by + c = 0$ | $d = \frac{\|ax_1 + by_1 + c\|}{\sqrt{a^2 + b^2}}$ |
| Origin to line $ax + by + c = 0$ | $d = \frac{\|c\|}{\sqrt{a^2 + b^2}}$ |
| Between parallel lines $ax + by + c_1 = 0$ and $ax + by + c_2 = 0$ | $d = \frac{\|c_1 - c_2\|}{\sqrt{a^2 + b^2}}$ |
| Point $(x_1, y_1)$ to line $y = mx + c$ | $d = \frac{\|y_1 - mx_1 - c\|}{\sqrt{1 + m^2}}$ |

---

## ❓ Quick Revision Questions

1. **Find the distance from point (2, 3) to the line $4x - 3y + 5 = 0$.**

2. **Find the distance from origin to the line $8x - 15y + 34 = 0$.**

3. **Find the distance between parallel lines $2x + y = 3$ and $2x + y = 8$.**

4. **A point moves such that its distance from line $3x + 4y = 5$ is always 2. Find the locus.**

5. **Find the value of k if distance from (1, -1) to line $3x + 4y + k = 0$ is 2.**

6. **Find the area of triangle formed by line $3x + 4y = 12$ with coordinate axes.**

<details>
<summary><b>Click to see answers</b></summary>

1. $d = \frac{|4(2) - 3(3) + 5|}{\sqrt{16 + 9}} = \frac{|8 - 9 + 5|}{5} = \frac{4}{5}$ units

2. $d = \frac{|34|}{\sqrt{64 + 225}} = \frac{34}{\sqrt{289}} = \frac{34}{17} = 2$ units

3. $d = \frac{|3 - 8|}{\sqrt{4 + 1}} = \frac{5}{\sqrt{5}} = \sqrt{5}$ units

4. Locus is two lines parallel to given line at distance 2:
   $3x + 4y = 5 \pm 2(5)$ → **$3x + 4y = 15$ and $3x + 4y = -5$**

5. $2 = \frac{|3(1) + 4(-1) + k|}{5}$
   $10 = |k - 1|$
   $k = 11$ or $k = -9$

6. x-intercept: (4, 0), y-intercept: (0, 3)
   Area = $\frac{1}{2} \times 4 \times 3 = 6$ square units

</details>

---

## ⏭️ Navigation

| [← Previous: Parallel and Perpendicular Lines](07-parallel-perpendicular.md) | [Next: Family of Lines →](09-family-of-lines.md) |
|:-----------------------------------------------------------------------------|------------------------------------------------:|
