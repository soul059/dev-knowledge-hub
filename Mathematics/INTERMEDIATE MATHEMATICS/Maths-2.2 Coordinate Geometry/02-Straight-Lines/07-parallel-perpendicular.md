# Chapter 2.7: Parallel and Perpendicular Lines

## 📚 Chapter Overview

Parallel and perpendicular lines are fundamental concepts in geometry. This chapter explores the conditions for lines to be parallel or perpendicular, and provides methods for finding equations of such lines.

---

## 📝 Parallel Lines

### Definition

Two lines are **parallel** if they:
- Never intersect
- Have the same slope
- Have the same direction

```
        Y
        ↑
        │    /  /  /
        │   /  /  /   Three parallel lines
        │  /  /  /    Same slope, different intercepts
        │ /  /  /
        │/  /  /
    ────O──/──/────→ X
       /  /  /
      /  /  /
```

---

## ⚡ Condition for Parallel Lines

> **Parallel Lines Condition**:
>
> Two non-vertical lines are parallel if and only if they have **equal slopes**:
>
> $$\boxed{m_1 = m_2}$$

### For General Form

Lines $a_1x + b_1y + c_1 = 0$ and $a_2x + b_2y + c_2 = 0$ are parallel if:

$$\boxed{\frac{a_1}{a_2} = \frac{b_1}{b_2} \neq \frac{c_1}{c_2}}$$

Or equivalently: $a_1 b_2 = a_2 b_1$

---

## 📝 Writing Equation of Parallel Line

### Method 1: Using Slope

If a line is parallel to $y = mx + c$, its equation is:
$$y = mx + k$$ (different intercept, same slope)

### Method 2: Using General Form

If a line is parallel to $ax + by + c = 0$, its equation is:
$$ax + by + k = 0$$ (same coefficients of x and y, different constant)

---

## ✏️ Examples: Parallel Lines

### Example 1: Finding Parallel Line Through a Point

**Problem**: Find the equation of a line parallel to $2x + 3y = 6$ passing through (1, 4).

**Solution**:

**Method 1**: Using slope
Slope of given line: $m = -\frac{2}{3}$

Line through (1, 4) with slope $-\frac{2}{3}$:
$$y - 4 = -\frac{2}{3}(x - 1)$$
$$3y - 12 = -2x + 2$$
$$2x + 3y = 14$$

**Method 2**: Direct form
Parallel line has form: $2x + 3y = k$
Since it passes through (1, 4): $2(1) + 3(4) = k$
$k = 14$

**Answer**: $2x + 3y = 14$

---

### Example 2: Distance Between Parallel Lines

**Problem**: Find the distance between parallel lines $3x + 4y = 5$ and $3x + 4y = 10$.

**Solution**:

For parallel lines $ax + by + c_1 = 0$ and $ax + by + c_2 = 0$:

$$\text{Distance} = \frac{|c_1 - c_2|}{\sqrt{a^2 + b^2}}$$

Lines: $3x + 4y - 5 = 0$ and $3x + 4y - 10 = 0$

$$d = \frac{|-5 - (-10)|}{\sqrt{9 + 16}} = \frac{5}{5} = 1$$

**Answer**: Distance = **1 unit**

---

### Example 3: Checking if Lines are Parallel

**Problem**: Determine if lines $4x - 6y + 10 = 0$ and $6x - 9y - 15 = 0$ are parallel.

**Solution**:

Line 1: $4x - 6y + 10 = 0$ → slope $m_1 = \frac{4}{6} = \frac{2}{3}$

Line 2: $6x - 9y - 15 = 0$ → slope $m_2 = \frac{6}{9} = \frac{2}{3}$

Since $m_1 = m_2 = \frac{2}{3}$, the lines are **parallel**.

---

## 📝 Perpendicular Lines

### Definition

Two lines are **perpendicular** if they:
- Intersect at a 90° angle
- Have slopes whose product is −1

```
        Y
        ↑
        │      /│
        │     / │
        │    /  │
        │   /   │
        │  /90° │
        │ /     │
    ────O───────│────→ X
        │       │
        │       │
```

---

## ⚡ Condition for Perpendicular Lines

> **Perpendicular Lines Condition**:
>
> Two non-vertical lines are perpendicular if and only if the product of their slopes is **−1**:
>
> $$\boxed{m_1 \cdot m_2 = -1}$$
>
> Or equivalently: $m_2 = -\frac{1}{m_1}$ (negative reciprocal)

### For General Form

Lines $a_1x + b_1y + c_1 = 0$ and $a_2x + b_2y + c_2 = 0$ are perpendicular if:

$$\boxed{a_1 a_2 + b_1 b_2 = 0}$$

---

## 📝 Writing Equation of Perpendicular Line

### Method 1: Using Negative Reciprocal Slope

If a line has slope $m$, a perpendicular line has slope $-\frac{1}{m}$.

### Method 2: Coefficient Swap

If a line is $ax + by + c = 0$, a perpendicular line has the form:
$$bx - ay + k = 0$$

(Swap coefficients of x and y, change one sign)

---

## ✏️ Examples: Perpendicular Lines

### Example 4: Finding Perpendicular Line Through a Point

**Problem**: Find the equation of a line perpendicular to $3x - 4y + 7 = 0$ passing through (2, 5).

**Solution**:

Given line slope: $m_1 = \frac{3}{4}$

Perpendicular slope: $m_2 = -\frac{4}{3}$

Line through (2, 5):
$$y - 5 = -\frac{4}{3}(x - 2)$$
$$3y - 15 = -4x + 8$$
$$4x + 3y = 23$$

**Alternative**: Using coefficient swap
Given: $3x - 4y + 7 = 0$ → Perpendicular form: $4x + 3y + k = 0$
Through (2, 5): $8 + 15 + k = 0$ → $k = -23$

**Answer**: $4x + 3y - 23 = 0$ or $4x + 3y = 23$

---

### Example 5: Verifying Perpendicularity

**Problem**: Show that lines $5x + 12y = 13$ and $12x - 5y = 7$ are perpendicular.

**Solution**:

**Method 1**: Using slopes
- $m_1 = -\frac{5}{12}$
- $m_2 = \frac{12}{5}$

$$m_1 \cdot m_2 = -\frac{5}{12} \cdot \frac{12}{5} = -1$$ ✓

**Method 2**: Using general form condition
$a_1 a_2 + b_1 b_2 = (5)(12) + (12)(-5) = 60 - 60 = 0$ ✓

**Answer**: Lines are **perpendicular**.

---

### Example 6: Perpendicular Bisector

**Problem**: Find the equation of the perpendicular bisector of the segment joining A(2, 3) and B(6, 7).

**Solution**:

**Step 1**: Find midpoint M
$$M = \left(\frac{2+6}{2}, \frac{3+7}{2}\right) = (4, 5)$$

**Step 2**: Find slope of AB
$$m_{AB} = \frac{7-3}{6-2} = \frac{4}{4} = 1$$

**Step 3**: Find perpendicular slope
$$m_{\perp} = -1$$

**Step 4**: Write equation through M
$$y - 5 = -1(x - 4)$$
$$y = -x + 9$$

**Answer**: $x + y = 9$

```
        Y
        ↑
    9   │   ●
        │  /│\
    7   │ / │ ●B(6,7)
        │/  │/
    5   ●───●M(4,5)── Perpendicular bisector
       /│  /│
    3 ●─│─/ │
   A(2,3)  │
    ────O──┼───┼───┼──→ X
           2   4   6
```

---

### Example 7: Foot of Perpendicular

**Problem**: Find the foot of the perpendicular from point P(3, 4) to the line $2x + y = 6$.

**Solution**:

**Step 1**: Find slope of given line
$m = -2$

**Step 2**: Perpendicular from P has slope $\frac{1}{2}$
$$y - 4 = \frac{1}{2}(x - 3)$$
$$2y - 8 = x - 3$$
$$x - 2y + 5 = 0$$

**Step 3**: Find intersection with $2x + y = 6$

From perpendicular: $x = 2y - 5$

Substitute: $2(2y - 5) + y = 6$
$$4y - 10 + y = 6$$
$$5y = 16$$
$$y = \frac{16}{5}$$

$$x = 2 \cdot \frac{16}{5} - 5 = \frac{32 - 25}{5} = \frac{7}{5}$$

**Answer**: Foot of perpendicular is $\left(\frac{7}{5}, \frac{16}{5}\right)$

---

## 📝 Special Perpendicular Pairs

| Line Type | Perpendicular |
|-----------|---------------|
| Horizontal ($y = k$) | Vertical ($x = h$) |
| $y = x$ (45° line) | $y = -x$ (135° line) |
| $y = mx$ | $y = -\frac{1}{m}x$ |
| X-axis | Y-axis |

---

## 📝 Important Geometric Constructions

### 1. Altitude of Triangle

The altitude from a vertex to the opposite side is perpendicular to that side.

```
        A
        ●
       /|\
      / | \
     /  |  \   AH ⊥ BC
    /   H   \
   /    ●    \
  /     |     \
 ●──────┴──────●
 B      H      C
```

### 2. Perpendicular from Point to Line

Used in finding shortest distance from a point to a line.

### 3. Orthocenter

The intersection of altitudes of a triangle.

---

## 📝 Summary of Relationships

```
    ┌─────────────────────────────────────────────────────┐
    │           PARALLEL vs PERPENDICULAR                 │
    ├─────────────────────────────────────────────────────┤
    │                                                     │
    │   PARALLEL LINES              PERPENDICULAR LINES   │
    │   ═══════════════             ════════════════════  │
    │                                                     │
    │   • m₁ = m₂                   • m₁ · m₂ = -1        │
    │                                                     │
    │   • a₁b₂ = a₂b₁               • a₁a₂ + b₁b₂ = 0     │
    │                                                     │
    │   • Same direction            • Right angle         │
    │                                                     │
    │   • Never intersect           • Intersect at 90°    │
    │                                                     │
    │   • ax + by + c₁ = 0          • ax + by + c = 0     │
    │     ax + by + c₂ = 0            bx - ay + k = 0     │
    │                                                     │
    └─────────────────────────────────────────────────────┘
```

---

## 💡 Tips and Insights

> 💡 **Negative Reciprocal**: For perpendicular lines, flip the fraction and change the sign.

> 💡 **Quick Check for ⊥**: Multiply slopes; if product is −1, lines are perpendicular.

> ⚠️ **Vertical Lines**: A vertical line (undefined slope) is perpendicular to any horizontal line (slope = 0).

> 💡 **Coefficient Swap**: For line $ax + by = c$, perpendicular is $bx - ay = k$.

---

## 🌍 Real-World Applications

1. **Construction**: Building corners at 90° angles
2. **Road Design**: Crossroads and perpendicular intersections
3. **Electronics**: Perpendicular magnetic fields in motors
4. **Architecture**: Structural support systems
5. **Navigation**: Course corrections at right angles
6. **Carpentry**: Ensuring corners are square

---

## 📋 Summary Table

| Concept | Parallel Lines | Perpendicular Lines |
|---------|---------------|---------------------|
| Slope condition | $m_1 = m_2$ | $m_1 \cdot m_2 = -1$ |
| General form | $a_1 b_2 = a_2 b_1$ | $a_1 a_2 + b_1 b_2 = 0$ |
| Angle between | 0° | 90° |
| Line form | $ax + by + k = 0$ | $bx - ay + k = 0$ |
| Example | $y = 2x + 1$ ∥ $y = 2x + 5$ | $y = 2x$ ⊥ $y = -\frac{1}{2}x$ |

---

## ❓ Quick Revision Questions

1. **Find the equation of a line parallel to $5x - 3y = 7$ through (1, 2).**

2. **Find the equation of a line perpendicular to $2x + 5y = 10$ through origin.**

3. **Are the lines $3x + 4y = 5$ and $6x + 8y = 15$ parallel, perpendicular, or neither?**

4. **Find the perpendicular distance between $4x + 3y = 10$ and $4x + 3y = 25$.**

5. **Find the equation of the perpendicular bisector of segment joining (−1, 2) and (3, 4).**

6. **The lines $kx + 2y = 5$ and $3x + y = 2$ are perpendicular. Find k.**

<details>
<summary><b>Click to see answers</b></summary>

1. Parallel line: $5x - 3y = k$
   Through (1, 2): $5(1) - 3(2) = k$ → $k = -1$
   **Answer**: $5x - 3y = -1$ or $5x - 3y + 1 = 0$

2. Given slope: $m = -\frac{2}{5}$
   Perpendicular slope: $m = \frac{5}{2}$
   Through origin: $y = \frac{5}{2}x$
   **Answer**: $5x - 2y = 0$

3. Slopes: $m_1 = -\frac{3}{4}$, $m_2 = -\frac{6}{8} = -\frac{3}{4}$
   Same slopes → **Parallel**

4. Distance = $\frac{|10 - 25|}{\sqrt{16 + 9}} = \frac{15}{5} = 3$ units

5. Midpoint: $(1, 3)$
   Slope of segment: $\frac{4-2}{3-(-1)} = \frac{2}{4} = \frac{1}{2}$
   Perpendicular slope: $-2$
   Line: $y - 3 = -2(x - 1)$ → **$2x + y = 5$**

6. For perpendicular: $a_1 a_2 + b_1 b_2 = 0$
   $(k)(3) + (2)(1) = 0$
   $3k + 2 = 0$
   **k = $-\frac{2}{3}$**

</details>

---

## ⏭️ Navigation

| [← Previous: Angle Between Lines](06-angle-between-lines.md) | [Next: Distance of Point from Line →](08-distance-point-line.md) |
|:-------------------------------------------------------------|----------------------------------------------------------------:|
