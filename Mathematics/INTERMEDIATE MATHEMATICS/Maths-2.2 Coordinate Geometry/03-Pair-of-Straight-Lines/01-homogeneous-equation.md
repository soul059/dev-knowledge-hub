# Chapter 3.1: Homogeneous Equation of Pair of Lines

## 📚 Chapter Overview

A **homogeneous equation** of the second degree always represents a pair of straight lines passing through the origin. This chapter introduces this fundamental concept and explores how to extract individual line equations from the combined form.

---

## 📝 What is a Homogeneous Equation?

### Definition

An equation is **homogeneous of degree n** if every term has the same total degree n.

**Examples**:
- $3x^2 + 5xy - 2y^2 = 0$ (degree 2, homogeneous)
- $x^2 + y^2 + 1 = 0$ (NOT homogeneous - constant term has degree 0)
- $x^3 - 3x^2y + xy^2 = 0$ (degree 3, homogeneous)

---

## ⚡ Homogeneous Equation of Second Degree

> **Standard Form**:
>
> $$\boxed{ax^2 + 2hxy + by^2 = 0}$$
>
> This equation **always represents a pair of straight lines passing through the origin**.

**Why 2h?** The coefficient of xy is written as 2h for mathematical convenience in formulas.

---

## 📐 Why It Represents Two Lines Through Origin?

### Proof

Consider $ax^2 + 2hxy + by^2 = 0$

**Case 1**: If $b \neq 0$, divide by $x^2$:
$$a + 2h\left(\frac{y}{x}\right) + b\left(\frac{y}{x}\right)^2 = 0$$

Let $m = \frac{y}{x}$ (slope of line through origin):
$$bm^2 + 2hm + a = 0$$

This is a quadratic in m, giving two values $m_1$ and $m_2$.

These correspond to lines $y = m_1 x$ and $y = m_2 x$ — both through origin!

**Case 2**: If $b = 0$, then $ax^2 + 2hxy = 0$ → $x(ax + 2hy) = 0$
This gives $x = 0$ and $ax + 2hy = 0$, both lines through origin.

```
        Y
        ↑
        │     /  y = m₁x
        │    /
        │   /
        │  /
        │ /
    ────O───────────→ X
        │\
        │ \
        │  \
        │   \
        │    \  y = m₂x
```

---

## 📝 Relationship Between Coefficients and Slopes

From the quadratic $bm^2 + 2hm + a = 0$:

> **Sum of Slopes**:
> $$\boxed{m_1 + m_2 = -\frac{2h}{b}}$$
>
> **Product of Slopes**:
> $$\boxed{m_1 \cdot m_2 = \frac{a}{b}}$$

---

## ✏️ Worked Examples

### Example 1: Identifying the Lines

**Problem**: Find the separate equations of lines represented by $x^2 - 5xy + 6y^2 = 0$.

**Solution**:

**Method 1**: Factorization
$$x^2 - 5xy + 6y^2 = 0$$

Find factors of 6 that add to -5: -2 and -3

$$x^2 - 2xy - 3xy + 6y^2 = 0$$
$$x(x - 2y) - 3y(x - 2y) = 0$$
$$(x - 2y)(x - 3y) = 0$$

**Answer**: Lines are $x - 2y = 0$ and $x - 3y = 0$

**Method 2**: Using slope formula
Divide by $y^2$: $\left(\frac{x}{y}\right)^2 - 5\left(\frac{x}{y}\right) + 6 = 0$

Let $t = \frac{x}{y}$: $t^2 - 5t + 6 = 0$ → $(t-2)(t-3) = 0$

$t = 2$ or $t = 3$, giving $x = 2y$ and $x = 3y$

---

### Example 2: Finding Slopes

**Problem**: Find the slopes of lines given by $3x^2 + 10xy + 3y^2 = 0$.

**Solution**:

Comparing with $ax^2 + 2hxy + by^2 = 0$:
- $a = 3$, $2h = 10$ → $h = 5$, $b = 3$

Sum of slopes: $m_1 + m_2 = -\frac{2h}{b} = -\frac{10}{3}$

Product of slopes: $m_1 \cdot m_2 = \frac{a}{b} = \frac{3}{3} = 1$

Solving: $m_1$ and $m_2$ are roots of $m^2 + \frac{10}{3}m + 1 = 0$
$$3m^2 + 10m + 3 = 0$$
$$(3m + 1)(m + 3) = 0$$

**Answer**: $m_1 = -\frac{1}{3}$ and $m_2 = -3$

---

### Example 3: Forming the Joint Equation

**Problem**: Find the joint equation of lines $3x + 2y = 0$ and $x - 4y = 0$.

**Solution**:

**Method**: Multiply the individual equations

Joint equation = $(3x + 2y)(x - 4y) = 0$

$$3x^2 - 12xy + 2xy - 8y^2 = 0$$
$$3x^2 - 10xy - 8y^2 = 0$$

**Answer**: $3x^2 - 10xy - 8y^2 = 0$

---

### Example 4: Condition for Real and Distinct Lines

**Problem**: For what values of k does $x^2 + 2kxy + 4y^2 = 0$ represent two distinct lines?

**Solution**:

The equation represents two distinct real lines if the quadratic in slope has real distinct roots.

From $bm^2 + 2hm + a = 0$, discriminant > 0:
$$4h^2 - 4ab > 0$$
$$h^2 > ab$$

Here: $a = 1$, $2h = 2k$ → $h = k$, $b = 4$

$$k^2 > 1 \times 4$$
$$k^2 > 4$$
$$|k| > 2$$

**Answer**: $k < -2$ or $k > 2$

---

### Example 5: Perpendicular Lines Through Origin

**Problem**: Find the value of k if lines $2x^2 + 5xy + ky^2 = 0$ are perpendicular.

**Solution**:

For perpendicular lines: $m_1 \cdot m_2 = -1$

Product of slopes: $m_1 \cdot m_2 = \frac{a}{b} = \frac{2}{k}$

$$\frac{2}{k} = -1$$
$$k = -2$$

**Verification**: $2x^2 + 5xy - 2y^2 = 0$
Factoring: $(2x - y)(x + 2y) = 0$
Lines: $y = 2x$ (slope = 2) and $y = -\frac{x}{2}$ (slope = $-\frac{1}{2}$)
Product: $2 \times (-\frac{1}{2}) = -1$ ✓

**Answer**: $k = -2$

---

### Example 6: Lines Making Given Angle with X-axis

**Problem**: Find the equation of pair of lines through origin making angle 30° and 60° with x-axis.

**Solution**:

Slopes: $m_1 = \tan 30° = \frac{1}{\sqrt{3}}$, $m_2 = \tan 60° = \sqrt{3}$

Sum: $m_1 + m_2 = \frac{1}{\sqrt{3}} + \sqrt{3} = \frac{1 + 3}{\sqrt{3}} = \frac{4}{\sqrt{3}}$

Product: $m_1 \cdot m_2 = \frac{1}{\sqrt{3}} \cdot \sqrt{3} = 1$

The slopes are roots of: $m^2 - \frac{4}{\sqrt{3}}m + 1 = 0$

Multiply by $\sqrt{3}$: $\sqrt{3}m^2 - 4m + \sqrt{3} = 0$

Since $m = \frac{y}{x}$:
$$\sqrt{3}\left(\frac{y}{x}\right)^2 - 4\left(\frac{y}{x}\right) + \sqrt{3} = 0$$

Multiply by $x^2$:
$$\sqrt{3}y^2 - 4xy + \sqrt{3}x^2 = 0$$

**Answer**: $\sqrt{3}x^2 - 4xy + \sqrt{3}y^2 = 0$ or $x^2 - \frac{4}{\sqrt{3}}xy + y^2 = 0$

---

### Example 7: Coincident Lines

**Problem**: For what value of λ does $x^2 - 2\lambda xy + 4y^2 = 0$ represent coincident lines?

**Solution**:

Coincident lines occur when discriminant = 0:
$$h^2 = ab$$

Here: $h = -\lambda$, $a = 1$, $b = 4$
$$(-\lambda)^2 = 1 \times 4$$
$$\lambda^2 = 4$$
$$\lambda = \pm 2$$

**Answer**: $\lambda = 2$ or $\lambda = -2$

---

## 📝 Types of Line Pairs

| Condition | Nature of Lines |
|-----------|-----------------|
| $h^2 > ab$ | Two real and distinct lines |
| $h^2 = ab$ | Two coincident (same) lines |
| $h^2 < ab$ | Imaginary lines (no real representation) |

---

## 📝 Special Cases

### 1. Perpendicular Lines Through Origin

Condition: $a + b = 0$

This is equivalent to $m_1 \cdot m_2 = -1$

### 2. One Line Along X-axis

If one line is $y = 0$, then $a = 0$ (no $x^2$ term after factoring)

### 3. One Line Along Y-axis

If one line is $x = 0$, then $b = 0$ (no $y^2$ term after factoring)

### 4. Lines Equally Inclined to Axes

If $m_1 = -m_2$, then $m_1 + m_2 = 0$, hence $h = 0$ (no xy term)

---

## 📝 Homogenization Technique

### Concept

If a line $lx + my + n = 0$ intersects a curve, we can find the joint equation of lines joining the origin to the intersection points by **homogenizing** the curve equation.

**Method**: Replace every constant with $\left(\frac{lx + my}{-n}\right)$ appropriately.

```
        Y
        ↑
        │    ● P
        │   /│
        │  / │  Curve
        │ /  │
        │/   ● Q
    ────O────│──────→ X
        │    │
        │    Line: lx + my + n = 0
```

---

## 💡 Tips and Insights

> 💡 **Quick Factorization**: Look for factors by trial: coefficients of $x^2$ × $y^2$ should give the middle coefficient when combined correctly.

> 💡 **Always Through Origin**: Any homogeneous equation passes through origin since (0, 0) satisfies it.

> ⚠️ **Check Reality**: Before solving, verify $h^2 \geq ab$ for real lines.

> 💡 **Symmetric Coefficients**: If $a = b$, the lines are equally inclined to the axes.

---

## 🌍 Real-World Applications

1. **Optics**: Light rays reflecting through a point
2. **Structural Engineering**: Force vectors through a joint
3. **Perspective Drawing**: Vanishing point constructions
4. **Physics**: Velocity components in two directions
5. **Navigation**: Course lines from a starting point

---

## 📋 Summary Table

| Property | Formula/Condition |
|----------|-------------------|
| Standard form | $ax^2 + 2hxy + by^2 = 0$ |
| Sum of slopes | $m_1 + m_2 = -\frac{2h}{b}$ |
| Product of slopes | $m_1 \cdot m_2 = \frac{a}{b}$ |
| Real distinct lines | $h^2 > ab$ |
| Coincident lines | $h^2 = ab$ |
| Perpendicular lines | $a + b = 0$ |
| Slopes equal magnitude, opposite sign | $h = 0$ |

---

## ❓ Quick Revision Questions

1. **Find the separate equations of lines given by $2x^2 - 7xy + 3y^2 = 0$.**

2. **Find the joint equation of lines $y = 3x$ and $y = -2x$.**

3. **If $x^2 + kxy - 6y^2 = 0$ represents perpendicular lines, find k.**

4. **Find the slopes of lines represented by $6x^2 + 5xy - 6y^2 = 0$.**

5. **For what value of λ are the lines $x^2 + 2\lambda xy + 2y^2 = 0$ imaginary?**

6. **Find the equation of pair of lines through origin, one having slope 2 and other perpendicular to $3x + y = 5$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $2x^2 - 7xy + 3y^2 = (2x - y)(x - 3y) = 0$
   **Lines**: $2x - y = 0$ and $x - 3y = 0$

2. Lines: $y - 3x = 0$ and $y + 2x = 0$
   Joint equation: $(y - 3x)(y + 2x) = 0$
   $y^2 - xy - 6x^2 = 0$ or **$6x^2 + xy - y^2 = 0$**

3. For perpendicular lines: $a + b = 0$
   Here: $1 + (-6) = -5 \neq 0$
   This condition is already satisfied regardless of k!
   Product of slopes = $\frac{a}{b} = \frac{1}{-6} \neq -1$
   Actually for perpendicular: $m_1 \cdot m_2 = -1$, so $\frac{a}{b} = -1$
   This gives $a = -b$, i.e., $1 = 6$ (false)
   **No value of k makes these lines perpendicular** (the condition is $a + b = 0$, which gives $1 - 6 \neq 0$)
   Wait - let me reconsider: Perpendicular condition is $a + b = 0$
   Here $a = 1$, $b = -6$, so $a + b = -5 \neq 0$
   **The pair cannot be perpendicular for any k** since k only affects h.

4. $6m^2 + 5m - 6 = 0$ (where $m = y/x$)
   $(3m - 2)(2m + 3) = 0$
   **Slopes**: $m = \frac{2}{3}$ and $m = -\frac{3}{2}$

5. Imaginary when $h^2 < ab$
   $\lambda^2 < 1 \times 2 = 2$
   **$-\sqrt{2} < \lambda < \sqrt{2}$**

6. Slope 1: $m_1 = 2$
   Slope 2: Perpendicular to $3x + y = 5$ (slope = -3), so $m_2 = \frac{1}{3}$
   Sum: $2 + \frac{1}{3} = \frac{7}{3} = -\frac{2h}{b}$
   Product: $2 \times \frac{1}{3} = \frac{2}{3} = \frac{a}{b}$
   Taking $b = 3$: $a = 2$, $h = -\frac{7}{2}$
   **$2x^2 - 7xy + 3y^2 = 0$**

</details>

---

## ⏭️ Navigation

| [← Back to Unit 3 Overview](README.md) | [Next: General Second Degree Equation →](02-general-second-degree.md) |
|:--------------------------------------|---------------------------------------------------------------------:|
