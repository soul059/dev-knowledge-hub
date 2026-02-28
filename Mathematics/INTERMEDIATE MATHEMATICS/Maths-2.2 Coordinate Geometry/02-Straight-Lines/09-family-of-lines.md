# Chapter 2.9: Family of Lines

## 📚 Chapter Overview

A **family of lines** is a set of lines that share a common property, typically passing through a fixed point or satisfying a common condition. This chapter explores how to represent and work with families of lines, particularly lines passing through the intersection of two given lines.

---

## 📝 Concept Introduction

### What is a Family of Lines?

A family of lines is an infinite set of lines defined by a single parameter (usually λ or k).

```
        Y
        ↑
        │     \ │ /
        │      \│/
        │   ────●────  All lines through point P
        │      /│\
        │     / │ \
        │    /  │  \
    ────│───────┼──────→ X
        │
```

**Examples**:
- All lines through a point: $y - k = m(x - h)$ where m is the parameter
- All lines with slope 2: $y = 2x + c$ where c is the parameter
- All lines parallel to x-axis: $y = k$ where k is the parameter

---

## ⚡ Family Through Intersection of Two Lines

> **Family of Lines Through Intersection**:
>
> If $L_1: a_1x + b_1y + c_1 = 0$ and $L_2: a_2x + b_2y + c_2 = 0$ are two lines,
> then the family of all lines passing through their intersection point is:
>
> $$\boxed{L_1 + \lambda L_2 = 0}$$
>
> or: $(a_1x + b_1y + c_1) + \lambda(a_2x + b_2y + c_2) = 0$

where $\lambda$ is a parameter that takes any real value.

---

## 📐 Why This Works

### Proof

Let P be the point of intersection of $L_1$ and $L_2$.

Since P lies on $L_1$: $a_1 x_P + b_1 y_P + c_1 = 0$
Since P lies on $L_2$: $a_2 x_P + b_2 y_P + c_2 = 0$

For the family $L_1 + \lambda L_2 = 0$:
$$(a_1 x_P + b_1 y_P + c_1) + \lambda(a_2 x_P + b_2 y_P + c_2) = 0$$
$$0 + \lambda \cdot 0 = 0$$ ✓

This is true for all values of λ, hence every member of the family passes through P.

```
    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │     L₁ + λL₂ = 0  represents ALL lines through     │
    │     the intersection of L₁ and L₂                  │
    │                                                    │
    │     Different values of λ give different lines:    │
    │                                                    │
    │     λ = 0   →   L₁ (the first line itself)         │
    │     λ = ∞   →   L₂ (the second line itself)        │
    │     λ = 1   →   L₁ + L₂ = 0                        │
    │     λ = -1  →   L₁ - L₂ = 0                        │
    │                                                    │
    └────────────────────────────────────────────────────┘
```

---

## ✏️ Worked Examples

### Example 1: Basic Family of Lines

**Problem**: Find the equation of the family of lines passing through the intersection of $2x + 3y - 5 = 0$ and $3x - 2y + 1 = 0$.

**Solution**:

The family is:
$$(2x + 3y - 5) + \lambda(3x - 2y + 1) = 0$$

$$2x + 3y - 5 + 3\lambda x - 2\lambda y + \lambda = 0$$

$$(2 + 3\lambda)x + (3 - 2\lambda)y + (\lambda - 5) = 0$$

**Answer**: $(2 + 3\lambda)x + (3 - 2\lambda)y + (\lambda - 5) = 0$, where $\lambda \in \mathbb{R}$

---

### Example 2: Finding a Specific Line in the Family

**Problem**: Find the line through the intersection of $x + 2y = 3$ and $2x - y = 1$ that passes through (2, 3).

**Solution**:

**Step 1**: Write the family
$$(x + 2y - 3) + \lambda(2x - y - 1) = 0$$

**Step 2**: Since it passes through (2, 3), substitute:
$$(2 + 6 - 3) + \lambda(4 - 3 - 1) = 0$$
$$5 + \lambda(0) = 0$$
$$5 = 0$$ (Contradiction!)

This means (2, 3) is NOT on any line in this family.

Let's verify: The intersection point of the two lines must be found first.

$x + 2y = 3$ ... (1)
$2x - y = 1$ ... (2)

From (2): $y = 2x - 1$
Substitute in (1): $x + 2(2x-1) = 3$ → $x + 4x - 2 = 3$ → $x = 1$
$y = 2(1) - 1 = 1$

Intersection point: (1, 1)

But we want line through (2, 3). Let me recalculate:

**Alternative**: Let's try again with the family equation through (2, 3):
$$(2 + 2(3) - 3) + \lambda(2(2) - 3 - 1) = 0$$
$$5 + \lambda(0) = 0$$

Since coefficient of λ is 0, this means (2, 3) lies on $L_2: 2x - y - 1 = 0$ 
Check: $2(2) - 3 - 1 = 0$ ✓

**Answer**: The required line is $2x - y = 1$ (which is $L_2$ itself, i.e., $\lambda \to \infty$)

---

### Example 3: Line Through Intersection with Given Slope

**Problem**: Find the line through intersection of $3x - 4y + 1 = 0$ and $5x + y - 1 = 0$ having slope 2.

**Solution**:

**Step 1**: Family of lines:
$$(3x - 4y + 1) + \lambda(5x + y - 1) = 0$$
$$(3 + 5\lambda)x + (-4 + \lambda)y + (1 - \lambda) = 0$$

**Step 2**: Slope of this line:
$$m = -\frac{(3 + 5\lambda)}{(-4 + \lambda)} = \frac{3 + 5\lambda}{4 - \lambda}$$

**Step 3**: Set slope = 2:
$$\frac{3 + 5\lambda}{4 - \lambda} = 2$$
$$3 + 5\lambda = 8 - 2\lambda$$
$$7\lambda = 5$$
$$\lambda = \frac{5}{7}$$

**Step 4**: Substitute λ:
$$(3x - 4y + 1) + \frac{5}{7}(5x + y - 1) = 0$$

Multiply by 7:
$$7(3x - 4y + 1) + 5(5x + y - 1) = 0$$
$$21x - 28y + 7 + 25x + 5y - 5 = 0$$
$$46x - 23y + 2 = 0$$

**Answer**: $46x - 23y + 2 = 0$ or $2x - y + \frac{2}{23} = 0$

---

### Example 4: Line Parallel to an Axis

**Problem**: Find the line through intersection of $2x + 3y = 6$ and $x - y = 2$ that is parallel to the y-axis.

**Solution**:

**Step 1**: Family:
$$(2x + 3y - 6) + \lambda(x - y - 2) = 0$$
$$(2 + \lambda)x + (3 - \lambda)y + (-6 - 2\lambda) = 0$$

**Step 2**: For line parallel to y-axis, coefficient of y = 0:
$$3 - \lambda = 0$$
$$\lambda = 3$$

**Step 3**: Substitute:
$$(2 + 3)x + 0 \cdot y + (-6 - 6) = 0$$
$$5x - 12 = 0$$
$$x = \frac{12}{5}$$

**Answer**: $x = \frac{12}{5}$ or $5x = 12$

---

### Example 5: Line Perpendicular to a Given Line

**Problem**: Find the line through intersection of $x + y = 4$ and $2x - y = 5$ that is perpendicular to $3x + 4y = 7$.

**Solution**:

**Step 1**: Family:
$$(x + y - 4) + \lambda(2x - y - 5) = 0$$
$$(1 + 2\lambda)x + (1 - \lambda)y + (-4 - 5\lambda) = 0$$

**Step 2**: Slope of this line:
$$m_1 = -\frac{1 + 2\lambda}{1 - \lambda}$$

Slope of $3x + 4y = 7$: $m_2 = -\frac{3}{4}$

**Step 3**: For perpendicular lines: $m_1 \cdot m_2 = -1$
$$-\frac{1 + 2\lambda}{1 - \lambda} \times \left(-\frac{3}{4}\right) = -1$$

$$\frac{3(1 + 2\lambda)}{4(1 - \lambda)} = -1$$

$$3 + 6\lambda = -4 + 4\lambda$$
$$2\lambda = -7$$
$$\lambda = -\frac{7}{2}$$

**Step 4**: Substitute:
$$(x + y - 4) - \frac{7}{2}(2x - y - 5) = 0$$

Multiply by 2:
$$2(x + y - 4) - 7(2x - y - 5) = 0$$
$$2x + 2y - 8 - 14x + 7y + 35 = 0$$
$$-12x + 9y + 27 = 0$$
$$4x - 3y - 9 = 0$$

**Answer**: $4x - 3y = 9$

---

### Example 6: Equidistant Line

**Problem**: Find the line through intersection of $x - 3y + 1 = 0$ and $2x + 5y - 9 = 0$ equidistant from points (5, 1) and (-1, 5).

**Solution**:

**Step 1**: Family:
$$(x - 3y + 1) + \lambda(2x + 5y - 9) = 0$$
$$(1 + 2\lambda)x + (-3 + 5\lambda)y + (1 - 9\lambda) = 0$$

**Step 2**: Distance from (5, 1) = Distance from (-1, 5)

$$\frac{|(1+2\lambda)(5) + (-3+5\lambda)(1) + (1-9\lambda)|}{\sqrt{(1+2\lambda)^2 + (-3+5\lambda)^2}}$$
$$= \frac{|(1+2\lambda)(-1) + (-3+5\lambda)(5) + (1-9\lambda)|}{\sqrt{(1+2\lambda)^2 + (-3+5\lambda)^2}}$$

Since denominators are same:
$$|5 + 10\lambda - 3 + 5\lambda + 1 - 9\lambda| = |-1 - 2\lambda - 15 + 25\lambda + 1 - 9\lambda|$$
$$|3 + 6\lambda| = |-15 + 14\lambda|$$

**Case 1**: $3 + 6\lambda = -15 + 14\lambda$
$18 = 8\lambda$ → $\lambda = \frac{9}{4}$

**Case 2**: $3 + 6\lambda = 15 - 14\lambda$
$20\lambda = 12$ → $\lambda = \frac{3}{5}$

For $\lambda = \frac{9}{4}$:
$$(x - 3y + 1) + \frac{9}{4}(2x + 5y - 9) = 0$$
$$4(x - 3y + 1) + 9(2x + 5y - 9) = 0$$
$$4x - 12y + 4 + 18x + 45y - 81 = 0$$
$$22x + 33y - 77 = 0$$
$$2x + 3y - 7 = 0$$

**Answer**: One such line is $2x + 3y = 7$ (the other can be found using $\lambda = \frac{3}{5}$)

---

## 📝 Special Families of Lines

### 1. Lines Through a Point

All lines through $(h, k)$: $y - k = m(x - h)$

where m is the parameter.

### 2. Lines with Given Inclination

All lines with slope m: $y = mx + c$

where c is the parameter.

### 3. Lines at Fixed Distance from Origin

All lines at distance p from origin: $x\cos\alpha + y\sin\alpha = p$

where α is the parameter.

---

## 📝 Angle Bisectors as Family Members

The angle bisectors of two lines $L_1 = 0$ and $L_2 = 0$ are special members of the family where the line is equidistant from both original lines.

```
        Y
        ↑
        │    L₁/    \
        │   /    \   
        │  /      \ L₂
        │ /   B₁   \
        │/    /     \    B₁, B₂ are angle bisectors
    ────●────/───────\───→ X
       /│   / B₂      \
      / │  /           \
```

The angle bisectors are:

$$\frac{a_1 x + b_1 y + c_1}{\sqrt{a_1^2 + b_1^2}} = \pm \frac{a_2 x + b_2 y + c_2}{\sqrt{a_2^2 + b_2^2}}$$

---

## 📝 Important Note

> **Caution**: The family $L_1 + \lambda L_2 = 0$ represents all lines through the intersection **except** $L_2$ itself.
>
> To get $L_2$, we need $\lambda \to \infty$, which is not a finite value.
>
> Alternative form: $\mu L_1 + \nu L_2 = 0$ includes both lines for suitable $(\mu, \nu)$.

---

## 💡 Tips and Insights

> 💡 **No Need to Find Intersection**: The family method lets you find lines through intersection without actually computing the intersection point!

> 💡 **One Equation, Infinite Lines**: Different λ values give different lines, all passing through the same point.

> ⚠️ **λ = 0 gives L₁**: Remember that setting λ = 0 recovers the first line.

> 💡 **Useful for Complex Conditions**: When finding a line with multiple conditions (through intersection + another property), this method is very efficient.

---

## 🌍 Real-World Applications

1. **Optics**: Family of light rays through a lens point
2. **Surveying**: Lines through a reference point
3. **Computer Graphics**: Pencil of lines for transformations
4. **Robotics**: Path options from a junction point
5. **Architecture**: Beam arrangements from a node
6. **Traffic Design**: Road options from an intersection

---

## 📋 Summary Table

| Family Type | Equation | Parameter |
|-------------|----------|-----------|
| Through intersection of L₁ and L₂ | $L_1 + \lambda L_2 = 0$ | λ ∈ ℝ |
| Through point (h, k) | $y - k = m(x - h)$ | m (slope) |
| With slope m | $y = mx + c$ | c (intercept) |
| Parallel to ax + by = 0 | $ax + by = k$ | k (constant) |
| Perpendicular to ax + by = 0 | $bx - ay = k$ | k (constant) |

---

## ❓ Quick Revision Questions

1. **Write the family of lines through intersection of $x + y = 1$ and $2x - y = 3$.**

2. **Find the line through intersection of $3x + 2y = 5$ and $x - y = 2$ that passes through origin.**

3. **Find the line through intersection of $2x + y = 4$ and $x + 3y = 7$ that is parallel to x-axis.**

4. **Find the line through intersection of $x + 2y = 5$ and $3x - y = 1$ having slope 3.**

5. **Find the line through intersection of $x - y = 0$ and $x + y = 2$ perpendicular to $2x + y = 5$.**

6. **Find the equation of a line passing through intersection of $2x + 3y = 1$ and $3x - 5y = 7$ and having equal intercepts on both axes.**

<details>
<summary><b>Click to see answers</b></summary>

1. $(x + y - 1) + \lambda(2x - y - 3) = 0$
   or $(1 + 2\lambda)x + (1 - \lambda)y - (1 + 3\lambda) = 0$

2. Family: $(3x + 2y - 5) + \lambda(x - y - 2) = 0$
   Through (0,0): $-5 + \lambda(-2) = 0$ → $\lambda = -\frac{5}{2}$
   Line: $3x + 2y - 5 - \frac{5}{2}(x - y - 2) = 0$
   → $6x + 4y - 10 - 5x + 5y + 10 = 0$
   → **$x + 9y = 0$**

3. Family: $(2x + y - 4) + \lambda(x + 3y - 7) = 0$
   For parallel to x-axis: coefficient of x = 0
   $2 + \lambda = 0$ → $\lambda = -2$
   Line: $(2x + y - 4) - 2(x + 3y - 7) = 0$
   → $-5y + 10 = 0$ → **$y = 2$**

4. Family: $(x + 2y - 5) + \lambda(3x - y - 1) = 0$
   Slope = $-\frac{1 + 3\lambda}{2 - \lambda} = 3$
   $1 + 3\lambda = -6 + 3\lambda$ → Contradiction!
   Recalculating: $-(1 + 3\lambda) = 3(2 - \lambda)$
   $-1 - 3\lambda = 6 - 3\lambda$ → No solution with finite λ
   **No such line exists in this family with slope 3**

5. Family: $(x - y) + \lambda(x + y - 2) = 0$
   Slope of family = $-\frac{1 + \lambda}{-1 + \lambda} = \frac{1 + \lambda}{1 - \lambda}$
   For ⊥ to $2x + y = 5$ (slope = -2): $\frac{1 + \lambda}{1 - \lambda} = \frac{1}{2}$
   $2 + 2\lambda = 1 - \lambda$ → $3\lambda = -1$ → $\lambda = -\frac{1}{3}$
   Line: $(x - y) - \frac{1}{3}(x + y - 2) = 0$
   → $3x - 3y - x - y + 2 = 0$ → **$2x - 4y + 2 = 0$ or $x - 2y + 1 = 0$**

6. Family: $(2x + 3y - 1) + \lambda(3x - 5y - 7) = 0$
   Line: $(2 + 3\lambda)x + (3 - 5\lambda)y - (1 + 7\lambda) = 0$
   Intercept form: $\frac{x}{a} + \frac{y}{b} = 1$
   For equal intercepts: a = b (excluding a = -b for now)
   $\frac{1 + 7\lambda}{2 + 3\lambda} = \frac{1 + 7\lambda}{3 - 5\lambda}$
   $2 + 3\lambda = 3 - 5\lambda$ → $8\lambda = 1$ → $\lambda = \frac{1}{8}$
   Line: **$19x + 19y = 15$** or **$x + y = \frac{15}{19}$**

</details>

---

## 🎉 Unit 2 Complete!

Congratulations on completing Unit 2: Straight Lines! You have learned:
- Different forms of line equations
- Conditions for parallel and perpendicular lines
- Distance formulas
- Family of lines

---

## ⏭️ Navigation

| [← Previous: Distance of Point from Line](08-distance-point-line.md) | [Back to Unit 2 Overview](README.md) |
|:--------------------------------------------------------------------|-------------------------------------:|
| | [Next Unit: Pair of Straight Lines →](../03-Pair-of-Straight-Lines/README.md) |
