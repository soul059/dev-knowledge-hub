# Chapter 2.5: General Equation of a Line

## 📚 Chapter Overview

The **general equation** (or **standard form**) of a line provides a unified representation that encompasses all possible lines, including vertical lines. This form is particularly useful for algebraic manipulations and finding intersections.

---

## 📝 The General Equation

### Definition

> **General Equation of a Line**: Every straight line can be represented by an equation of the form:
>
> $$\boxed{ax + by + c = 0}$$
>
> Where $a$, $b$, $c$ are constants, and $a$ and $b$ are not both zero.

### Conditions

- At least one of $a$ or $b$ must be non-zero
- If $a = 0$: Horizontal line ($by + c = 0$ → $y = -\frac{c}{b}$)
- If $b = 0$: Vertical line ($ax + c = 0$ → $x = -\frac{c}{a}$)

---

## 📝 Properties from General Form

### 1. Slope

For the line $ax + by + c = 0$ (where $b \neq 0$):

$$\boxed{m = -\frac{a}{b}}$$

**Derivation:**
$$by = -ax - c$$
$$y = -\frac{a}{b}x - \frac{c}{b}$$

Comparing with $y = mx + k$: slope $m = -\frac{a}{b}$

### 2. Y-Intercept

$$\boxed{\text{y-intercept} = -\frac{c}{b}}$$ (where $b \neq 0$)

### 3. X-Intercept

$$\boxed{\text{x-intercept} = -\frac{c}{a}}$$ (where $a \neq 0$)

---

## 📝 Special Cases

```
    Horizontal Line          Vertical Line           Through Origin
    (a = 0)                  (b = 0)                 (c = 0)
    
        Y                        Y                        Y
        ↑                        ↑                        ↑     /
        │                        │  │                     │    /
    ────●────────●──→           │  │                     │   /
        │ y = k                  │  │  x = k             │  /
        │                        │  │                     │ /
    ────O────────────→ X    ────O──│──→ X            ────O──────→ X
        │                        │  │                    /│
                                    k                   / │
        
    by + c = 0               ax + c = 0              ax + by = 0
    y = -c/b                 x = -c/a                Passes through (0,0)
```

---

## 📝 Conversion Table

| From | To General Form |
|------|-----------------|
| $y = mx + c$ | $mx - y + c = 0$ |
| $\frac{x}{a} + \frac{y}{b} = 1$ | $bx + ay - ab = 0$ |
| $x\cos\alpha + y\sin\alpha = p$ | $(\cos\alpha)x + (\sin\alpha)y - p = 0$ |
| $y - y_1 = m(x - x_1)$ | $mx - y + (y_1 - mx_1) = 0$ |

---

## ✏️ Worked Examples

### Example 1: Extracting Information

**Problem**: For the line $3x - 4y + 12 = 0$, find:
(a) Slope
(b) X-intercept
(c) Y-intercept

**Solution**:

Here $a = 3$, $b = -4$, $c = 12$

**(a) Slope:**
$$m = -\frac{a}{b} = -\frac{3}{-4} = \frac{3}{4}$$

**(b) X-intercept:**
$$-\frac{c}{a} = -\frac{12}{3} = -4$$
X-intercept is at point $(-4, 0)$

**(c) Y-intercept:**
$$-\frac{c}{b} = -\frac{12}{-4} = 3$$
Y-intercept is at point $(0, 3)$

---

### Example 2: Converting to General Form

**Problem**: Convert $y = \frac{2}{3}x - 5$ to general form.

**Solution**:

$$y = \frac{2}{3}x - 5$$

Multiply by 3:
$$3y = 2x - 15$$

Rearrange:
$$2x - 3y - 15 = 0$$

**Answer**: $2x - 3y - 15 = 0$

---

### Example 3: Line Through Two Points

**Problem**: Find the general equation of the line through (1, 2) and (4, 8).

**Solution**:

**Method 1: Using Two-Point Form**
$$\frac{y - 2}{8 - 2} = \frac{x - 1}{4 - 1}$$
$$\frac{y - 2}{6} = \frac{x - 1}{3}$$
$$3(y - 2) = 6(x - 1)$$
$$3y - 6 = 6x - 6$$
$$6x - 3y = 0$$
$$2x - y = 0$$

**Answer**: $2x - y = 0$

---

### Example 4: Finding Intersection

**Problem**: Find the point of intersection of lines $2x + 3y = 12$ and $x - y = 1$.

**Solution**:

**Equation 1**: $2x + 3y = 12$
**Equation 2**: $x - y = 1$ → $x = y + 1$

Substitute in Equation 1:
$$2(y + 1) + 3y = 12$$
$$2y + 2 + 3y = 12$$
$$5y = 10$$
$$y = 2$$

Then: $x = 2 + 1 = 3$

**Answer**: Point of intersection is **(3, 2)**

```
        Y
        ↑
    4   │       \
        │        \  2x + 3y = 12
    2   │    ●───────
        │   (3,2)   x - y = 1
        │
    ────O───┬───────→ X
            3
```

---

### Example 5: Concurrent Lines

**Problem**: Show that the lines $2x - 3y + 5 = 0$, $3x + 4y - 7 = 0$, and $9x - 5y + 8 = 0$ are concurrent.

**Solution**:

Lines are concurrent if they all pass through a common point.

**Step 1**: Find intersection of first two lines
From $2x - 3y + 5 = 0$ → $x = \frac{3y - 5}{2}$

Substitute in $3x + 4y - 7 = 0$:
$$3 \cdot \frac{3y - 5}{2} + 4y - 7 = 0$$
$$\frac{9y - 15}{2} + 4y - 7 = 0$$
$$9y - 15 + 8y - 14 = 0$$
$$17y = 29$$
$$y = \frac{29}{17}$$

$$x = \frac{3 \cdot \frac{29}{17} - 5}{2} = \frac{\frac{87 - 85}{17}}{2} = \frac{2}{34} = \frac{1}{17}$$

Point of intersection: $\left(\frac{1}{17}, \frac{29}{17}\right)$

**Step 2**: Check if this point lies on the third line
$$9 \cdot \frac{1}{17} - 5 \cdot \frac{29}{17} + 8 = \frac{9 - 145 + 136}{17} = \frac{0}{17} = 0$$ ✓

**Conclusion**: The three lines are **concurrent** at point $\left(\frac{1}{17}, \frac{29}{17}\right)$.

---

### Example 6: Determinant Condition for Concurrency

Three lines $a_1x + b_1y + c_1 = 0$, $a_2x + b_2y + c_2 = 0$, and $a_3x + b_3y + c_3 = 0$ are concurrent if:

$$\begin{vmatrix} a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \\ a_3 & b_3 & c_3 \end{vmatrix} = 0$$

**For Example 5:**
$$\begin{vmatrix} 2 & -3 & 5 \\ 3 & 4 & -7 \\ 9 & -5 & 8 \end{vmatrix}$$

$$= 2(32 - 35) - (-3)(24 + 63) + 5(-15 - 36)$$
$$= 2(-3) + 3(87) + 5(-51)$$
$$= -6 + 261 - 255 = 0$$ ✓

---

## 📝 Reduction to Other Forms

### To Slope-Intercept Form

From $ax + by + c = 0$:
$$y = -\frac{a}{b}x - \frac{c}{b}$$

### To Intercept Form

From $ax + by + c = 0$ (where $c \neq 0$):
$$\frac{x}{-c/a} + \frac{y}{-c/b} = 1$$

### To Normal Form

From $ax + by + c = 0$:
$$\frac{ax}{\pm\sqrt{a^2+b^2}} + \frac{by}{\pm\sqrt{a^2+b^2}} = \frac{-c}{\pm\sqrt{a^2+b^2}}$$

Choose sign so that RHS is positive.

---

## 📝 Position of a Point Relative to a Line

For a point $P(x_1, y_1)$ and line $ax + by + c = 0$:

Calculate: $S = ax_1 + by_1 + c$

| Value of S | Position of Point |
|------------|-------------------|
| $S = 0$ | Point lies ON the line |
| $S > 0$ | Point lies on one side |
| $S < 0$ | Point lies on the other side |

### Same Side Test

Two points $P(x_1, y_1)$ and $Q(x_2, y_2)$ are on:
- **Same side** if $S_1 \cdot S_2 > 0$
- **Opposite sides** if $S_1 \cdot S_2 < 0$
- **One on line** if $S_1 \cdot S_2 = 0$

---

## ✏️ Example: Position of Points

**Problem**: Determine on which side of the line $2x + 3y - 6 = 0$ the points (1, 1) and (3, 4) lie.

**Solution**:

For (1, 1): $S_1 = 2(1) + 3(1) - 6 = 2 + 3 - 6 = -1 < 0$

For (3, 4): $S_2 = 2(3) + 3(4) - 6 = 6 + 12 - 6 = 12 > 0$

Since $S_1 \cdot S_2 = (-1)(12) = -12 < 0$, the points are on **opposite sides** of the line.

---

## 📝 Ratio of Division by a Line

If a line $ax + by + c = 0$ divides the join of $P(x_1, y_1)$ and $Q(x_2, y_2)$ in ratio $m:n$, then:

$$\frac{m}{n} = -\frac{ax_1 + by_1 + c}{ax_2 + by_2 + c}$$

- If ratio is positive → Internal division
- If ratio is negative → External division

---

## 💡 Tips and Insights

> 💡 **Uniqueness**: The general equation is not unique — $2x + 4y - 6 = 0$ and $x + 2y - 3 = 0$ represent the same line.

> 💡 **Standard Practice**: Typically, we keep the coefficient of x positive and use integer coefficients when possible.

> ⚠️ **Zero Coefficients**: Be careful with formulas when $a = 0$ (horizontal line) or $b = 0$ (vertical line).

> 💡 **Quick Slope**: Slope = $-\frac{\text{coefficient of } x}{\text{coefficient of } y}$

---

## 🌍 Real-World Applications

1. **Linear Programming**: Constraints expressed in general form
2. **Computer Graphics**: Line representation in algorithms
3. **Economics**: Supply and demand equations
4. **Engineering**: Equation of constraint lines
5. **Navigation**: Representing boundary lines
6. **Machine Learning**: Decision boundaries

---

## 📋 Summary Table

| Property | Formula (for $ax + by + c = 0$) |
|----------|--------------------------------|
| Slope | $m = -\frac{a}{b}$ (when $b \neq 0$) |
| X-intercept | $-\frac{c}{a}$ (when $a \neq 0$) |
| Y-intercept | $-\frac{c}{b}$ (when $b \neq 0$) |
| Horizontal line | $a = 0$: $y = -\frac{c}{b}$ |
| Vertical line | $b = 0$: $x = -\frac{c}{a}$ |
| Through origin | $c = 0$ |
| Point on line | $ax_1 + by_1 + c = 0$ |
| Concurrency | Determinant = 0 |

---

## ❓ Quick Revision Questions

1. **Find the slope of the line $5x - 7y + 3 = 0$.**

2. **Convert $y = 4x - 7$ to general form.**

3. **Find the x and y intercepts of $3x + 2y - 12 = 0$.**

4. **Determine if point (2, 3) lies on the line $x + 2y - 8 = 0$.**

5. **Find the intersection of $x + y = 5$ and $2x - y = 1$.**

6. **Check if lines $x + y = 1$, $2x + 3y = 5$, and $x + 2y = 3$ are concurrent.**

<details>
<summary><b>Click to see answers</b></summary>

1. Slope = $-\frac{5}{-7} = \frac{5}{7}$

2. $4x - y - 7 = 0$

3. X-intercept: $-\frac{-12}{3} = 4$ → point (4, 0)
   Y-intercept: $-\frac{-12}{2} = 6$ → point (0, 6)

4. Substitute: $2 + 2(3) - 8 = 2 + 6 - 8 = 0$ ✓
   Yes, the point lies on the line.

5. Adding equations: $3x = 6$ → $x = 2$
   From $x + y = 5$: $y = 3$
   Intersection: **(2, 3)**

6. Determinant: $\begin{vmatrix} 1 & 1 & -1 \\ 2 & 3 & -5 \\ 1 & 2 & -3 \end{vmatrix}$
   $= 1(-9 + 10) - 1(-6 + 5) + (-1)(4 - 3)$
   $= 1 + 1 - 1 = 1 \neq 0$
   Lines are **NOT concurrent**.

</details>

---

## ⏭️ Navigation

| [← Previous: Intercept & Normal Forms](04-intercept-normal-forms.md) | [Next: Angle Between Lines →](06-angle-between-lines.md) |
|:--------------------------------------------------------------------|--------------------------------------------------------:|
