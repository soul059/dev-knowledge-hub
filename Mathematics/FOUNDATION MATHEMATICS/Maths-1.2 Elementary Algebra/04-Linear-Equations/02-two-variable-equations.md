# Chapter 4.2: Two Variable Linear Equations

[← Previous: One Variable Linear Equations](01-one-variable-equations.md) | [Back to Contents](../README.md) | [Next: Word Problems with Linear Equations →](03-word-problems.md)

---

## 📚 Chapter Overview

A **linear equation in two variables** has the form $ax + by = c$, where both $x$ and $y$ appear to the first power. Unlike one-variable equations with a single solution, these equations have infinitely many solutions that form a straight line when graphed.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand linear equations in two variables
- Express equations in various forms (standard, slope-intercept, point-slope)
- Find solutions to two-variable equations
- Understand the concept of slope
- Graph linear equations
- Find intercepts and use them for graphing

---

## 1. Understanding Two-Variable Equations

### Definition

A **linear equation in two variables** is an equation that can be written as:

$$ax + by = c$$

where $a$, $b$, and $c$ are constants, and $a$ and $b$ are not both zero.

```
┌─────────────────────────────────────────────────────────────────────┐
│           TWO-VARIABLE LINEAR EQUATIONS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Standard Form: ax + by = c                                       │
│                                                                     │
│   Examples:                                                         │
│   • 2x + 3y = 12                                                   │
│   • x - y = 5                                                      │
│   • y = 2x + 1  (can be rewritten as 2x - y = -1)                │
│                                                                     │
│   Solutions:                                                        │
│   A solution is an ordered pair (x, y) that makes the equation    │
│   true.                                                             │
│                                                                     │
│   For 2x + 3y = 12:                                                │
│   • (0, 4) is a solution: 2(0) + 3(4) = 12 ✓                      │
│   • (6, 0) is a solution: 2(6) + 3(0) = 12 ✓                      │
│   • (3, 2) is a solution: 2(3) + 3(2) = 12 ✓                      │
│                                                                     │
│   There are INFINITELY MANY solutions!                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Finding Solutions

To find solutions, substitute a value for one variable and solve for the other.

```
Equation: x + 2y = 8

Finding solutions:

If x = 0:  0 + 2y = 8 → y = 4    Solution: (0, 4)
If x = 2:  2 + 2y = 8 → y = 3    Solution: (2, 3)
If x = 4:  4 + 2y = 8 → y = 2    Solution: (4, 2)
If y = 0:  x + 0 = 8 → x = 8     Solution: (8, 0)

Table of solutions:
┌─────┬─────┐
│  x  │  y  │
├─────┼─────┤
│  0  │  4  │
│  2  │  3  │
│  4  │  2  │
│  6  │  1  │
│  8  │  0  │
└─────┴─────┘
```

---

## 2. Forms of Linear Equations

### Standard Form

$$ax + by = c$$

- $a$, $b$, $c$ are integers (usually)
- $a$ is typically positive
- Used for many algebraic manipulations

### Slope-Intercept Form

$$y = mx + b$$

```
┌─────────────────────────────────────────────────────────────────────┐
│              SLOPE-INTERCEPT FORM                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                    y = mx + b                                       │
│                        ↑    ↑                                       │
│                        │    │                                       │
│                        │    └── y-intercept (where line crosses    │
│                        │        the y-axis)                        │
│                        │                                            │
│                        └─────── slope (steepness of the line)      │
│                                                                     │
│   Example: y = 2x + 3                                              │
│   • Slope m = 2 (rises 2 for every 1 unit right)                  │
│   • y-intercept b = 3 (crosses y-axis at (0, 3))                  │
│                                                                     │
│          y                                                          │
│          │          /                                               │
│        5 ┤        /                                                 │
│          │      /                                                   │
│        3 ┤    ● (0,3)                                              │
│          │  /                                                       │
│        1 ┤/                                                         │
│          │                                                          │
│    ──────┼──────── x                                               │
│        -1│   1   2                                                  │
│          │                                                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Point-Slope Form

$$y - y_1 = m(x - x_1)$$

Where $(x_1, y_1)$ is a point on the line and $m$ is the slope.

### Converting Between Forms

```
Standard to Slope-Intercept:
2x + 3y = 12
3y = -2x + 12
y = -2/3 x + 4

Slope-Intercept to Standard:
y = 3x - 5
-3x + y = -5
3x - y = 5
```

---

## 3. Understanding Slope

### Definition of Slope

The **slope** measures the steepness and direction of a line.

$$m = \frac{\text{rise}}{\text{run}} = \frac{y_2 - y_1}{x_2 - x_1} = \frac{\Delta y}{\Delta x}$$

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SLOPE CONCEPT                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Given two points: (x₁, y₁) and (x₂, y₂)                         │
│                                                                     │
│             (x₂, y₂)                                               │
│              ●                                                      │
│             /│                                                      │
│            / │ rise = y₂ - y₁                                      │
│           /  │                                                      │
│          ●───┘                                                      │
│       (x₁, y₁)                                                     │
│          └─────┘                                                    │
│          run = x₂ - x₁                                             │
│                                                                     │
│   Slope m = rise/run = (y₂ - y₁)/(x₂ - x₁)                        │
│                                                                     │
│   Example: Points (1, 2) and (4, 8)                                │
│   m = (8 - 2)/(4 - 1) = 6/3 = 2                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Types of Slopes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TYPES OF SLOPES                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   POSITIVE SLOPE (m > 0)       NEGATIVE SLOPE (m < 0)              │
│   Line goes up ↗               Line goes down ↘                    │
│                                                                     │
│      y    /                       y   \                             │
│      │  /                         │    \                            │
│      │/                           │     \                           │
│      └──── x                      └───── x                          │
│                                                                     │
│   ZERO SLOPE (m = 0)           UNDEFINED SLOPE                     │
│   Horizontal line               Vertical line                       │
│                                                                     │
│      y                            y   │                             │
│      │────────                    │   │                             │
│      │                            │   │                             │
│      └──── x                      └───┼── x                         │
│                                                                     │
│   y = k (constant)             x = k (not a function)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Slope Values and Steepness

| Slope | Meaning |
|-------|---------|
| m = 1 | 45° angle, rises 1 for every 1 right |
| m = 2 | Steeper, rises 2 for every 1 right |
| m = 1/2 | Gentler, rises 1 for every 2 right |
| m = -1 | Falls 1 for every 1 right |
| m = 0 | Horizontal |
| undefined | Vertical |

---

## 4. Intercepts

### Definitions

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERCEPTS                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   x-intercept: Where the line crosses the x-axis                  │
│                Point has form (a, 0)                               │
│                To find: Set y = 0 and solve for x                 │
│                                                                     │
│   y-intercept: Where the line crosses the y-axis                  │
│                Point has form (0, b)                               │
│                To find: Set x = 0 and solve for y                 │
│                                                                     │
│          y                                                          │
│          │        /                                                 │
│          │      /                                                   │
│    (0,b) ●    /                                                    │
│          │  /                                                       │
│          │/                                                         │
│    ──────●──────── x                                               │
│        (a,0)                                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Finding Intercepts

```
Equation: 3x + 4y = 12

x-intercept (set y = 0):
3x + 4(0) = 12
3x = 12
x = 4
x-intercept: (4, 0)

y-intercept (set x = 0):
3(0) + 4y = 12
4y = 12
y = 3
y-intercept: (0, 3)
```

---

## 5. Graphing Linear Equations

### Method 1: Using Slope and y-intercept

```
Graph: y = 2x - 1

Step 1: Identify slope and y-intercept
        m = 2, b = -1

Step 2: Plot y-intercept
        Point: (0, -1)

Step 3: Use slope to find another point
        slope = 2 = 2/1 (rise 2, run 1)
        From (0, -1): go right 1, up 2 → (1, 1)

Step 4: Draw the line through the points

        y
        │      /
      2 ┤    ●(1,1)
        │  /
      0 ┼────────── x
        │/
     -1 ●(0,-1)
        │
```

### Method 2: Using Two Points (Intercept Method)

```
Graph: 2x + 3y = 6

Step 1: Find x-intercept (set y = 0)
        2x = 6, x = 3
        Point: (3, 0)

Step 2: Find y-intercept (set x = 0)
        3y = 6, y = 2
        Point: (0, 2)

Step 3: Plot points and draw line

        y
        │
      2 ●(0,2)
        │ \
      1 ┤  \
        │   \
      0 ┼────●──── x
             (3,0)
```

### Method 3: Using a Table of Values

```
Graph: y = x + 2

Create a table:
┌─────┬─────────────┬─────┐
│  x  │ y = x + 2   │  y  │
├─────┼─────────────┼─────┤
│ -2  │ -2 + 2      │  0  │
│ -1  │ -1 + 2      │  1  │
│  0  │  0 + 2      │  2  │
│  1  │  1 + 2      │  3  │
│  2  │  2 + 2      │  4  │
└─────┴─────────────┴─────┘

Plot points: (-2,0), (-1,1), (0,2), (1,3), (2,4)
Draw line through points
```

---

## 6. Special Lines

### Horizontal Lines

$$y = k \quad \text{(where k is a constant)}$$

- Slope = 0
- Parallel to x-axis
- Every point has y-coordinate = k

```
y = 3

    y
    │
  3 │──────────────
    │
  0 ┼────────────── x
    │
```

### Vertical Lines

$$x = k \quad \text{(where k is a constant)}$$

- Slope is undefined
- Parallel to y-axis
- Every point has x-coordinate = k
- NOT a function!

```
x = 2

    y
    │   │
  3 ┤   │
    │   │
  0 ┼───┼────────── x
    │   │
        2
```

---

## 7. Parallel and Perpendicular Lines

### Parallel Lines

Two lines are **parallel** if they have the **same slope**.

$$m_1 = m_2$$

```
y = 2x + 3  and  y = 2x - 1  are parallel (both have slope 2)

    y       /  /
    │      /  /
  3 │     /  /
    │    /  /
  0 ┼───/──/─── x
    │  /  /
   -1│ /  /
```

### Perpendicular Lines

Two lines are **perpendicular** if their slopes are **negative reciprocals**.

$$m_1 \cdot m_2 = -1 \quad \text{or} \quad m_2 = -\frac{1}{m_1}$$

```
y = 2x + 1  and  y = -½x + 3  are perpendicular

Slopes: 2 and -½
Check: 2 × (-½) = -1 ✓

The lines meet at 90°
```

---

## ✏️ Solved Examples

### Example 1: Easy - Find Solutions

**Problem:** Find three solutions to $2x - y = 4$

**Solution:**
```
If x = 0: 2(0) - y = 4 → y = -4    Solution: (0, -4)
If x = 2: 2(2) - y = 4 → y = 0     Solution: (2, 0)
If x = 3: 2(3) - y = 4 → y = 2     Solution: (3, 2)

Answers: (0, -4), (2, 0), (3, 2)
```

### Example 2: Easy - Convert to Slope-Intercept Form

**Problem:** Write $3x + 2y = 8$ in slope-intercept form

**Solution:**
```
3x + 2y = 8
2y = -3x + 8
y = -3/2 x + 4

Slope-intercept form: y = -3/2 x + 4
Slope m = -3/2, y-intercept b = 4
```

### Example 3: Medium - Find Slope from Two Points

**Problem:** Find the slope of the line through (2, 3) and (5, 9)

**Solution:**
```
m = (y₂ - y₁)/(x₂ - x₁)
m = (9 - 3)/(5 - 2)
m = 6/3
m = 2

Answer: slope = 2
```

### Example 4: Medium - Find Intercepts

**Problem:** Find the x- and y-intercepts of $4x - 5y = 20$

**Solution:**
```
x-intercept (y = 0):
4x - 5(0) = 20
4x = 20
x = 5
x-intercept: (5, 0)

y-intercept (x = 0):
4(0) - 5y = 20
-5y = 20
y = -4
y-intercept: (0, -4)
```

### Example 5: Hard - Write Equation from Point and Slope

**Problem:** Write the equation of a line with slope 3 passing through (2, 5)

**Solution:**
```
Use point-slope form:
y - y₁ = m(x - x₁)
y - 5 = 3(x - 2)
y - 5 = 3x - 6
y = 3x - 1

Answer: y = 3x - 1 (slope-intercept form)
     or 3x - y = 1 (standard form)
```

### Example 6: Hard - Parallel and Perpendicular

**Problem:** Find equations for lines through (1, 4) that are:
(a) parallel to y = 2x + 3
(b) perpendicular to y = 2x + 3

**Solution:**
```
(a) Parallel: Same slope m = 2
    y - 4 = 2(x - 1)
    y - 4 = 2x - 2
    y = 2x + 2

(b) Perpendicular: slope = -1/2 (negative reciprocal)
    y - 4 = -½(x - 1)
    y - 4 = -½x + ½
    y = -½x + 4½

Answers: (a) y = 2x + 2  (b) y = -½x + 9/2
```

---

## ❓ Practice Problems

### Easy Level

1. Find two solutions to $x + y = 7$

2. Write $x - 3y = 9$ in slope-intercept form

3. Find the slope of the line through (1, 2) and (4, 11)

### Medium Level

4. Find the x- and y-intercepts of $2x + 5y = 10$

5. Write the equation of a line with slope -2 and y-intercept 5

6. Graph $y = -x + 3$ (describe key points)

### Hard Level

7. Write the equation of the line through (3, -1) and (6, 5)

8. Are the lines $3x + 4y = 12$ and $6x + 8y = 20$ parallel, perpendicular, or neither?

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. When x=0: y=7 → (0,7); When x=3: y=4 → (3,4)
   **$(0, 7)$ and $(3, 4)$** (or any valid pairs)

2. x - 3y = 9 → -3y = -x + 9 → y = (1/3)x - 3
   **$y = \frac{1}{3}x - 3$**

3. m = (11-2)/(4-1) = 9/3 = 3
   **$m = 3$**

4. x-int: 2x = 10 → (5, 0); y-int: 5y = 10 → (0, 2)
   **x-intercept: (5, 0), y-intercept: (0, 2)**

5. Using y = mx + b: **$y = -2x + 5$**

6. y-intercept: (0, 3); slope = -1
   Points: (0,3), (1,2), (2,1), (3,0)
   **Line passes through (0,3) and (3,0)**

7. m = (5-(-1))/(6-3) = 6/3 = 2
   y - 5 = 2(x - 6) → y = 2x - 7
   **$y = 2x - 7$**

8. Line 1: y = -3/4 x + 3 → slope = -3/4
   Line 2: y = -3/4 x + 5/2 → slope = -3/4
   Same slope → **Parallel**

</details>

---

## 📋 Summary Table

| Form | Equation | Key Information |
|------|----------|-----------------|
| Standard | $ax + by = c$ | Good for finding intercepts |
| Slope-Intercept | $y = mx + b$ | Shows slope (m) and y-int (b) |
| Point-Slope | $y - y_1 = m(x - x_1)$ | Uses point and slope |

---

## 🔄 Quick Revision Questions

1. **What form is $y = 3x + 7$?**

2. **What is the slope of the line $y = -5x + 2$?**

3. **What is the y-intercept of $2x + y = 6$?**

4. **What is the slope of a horizontal line?**

5. **If one line has slope 4, what is the slope of a perpendicular line?**

6. **How do you find the x-intercept?**

<details>
<summary>Quick Answers</summary>

1. Slope-intercept form
2. m = -5
3. Set x = 0: y = 6, so (0, 6)
4. m = 0
5. m = -1/4
6. Set y = 0 and solve for x

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Two-variable linear equations have infinitely many solutions  │
│                                                                     │
│   ★ Slope m = rise/run = (y₂-y₁)/(x₂-x₁)                          │
│                                                                     │
│   ★ Slope-intercept form y = mx + b shows slope and y-intercept  │
│                                                                     │
│   ★ To find x-intercept: set y = 0                                │
│     To find y-intercept: set x = 0                                │
│                                                                     │
│   ★ Horizontal lines: y = k (slope = 0)                           │
│     Vertical lines: x = k (undefined slope)                       │
│                                                                     │
│   ★ Parallel lines: same slope (m₁ = m₂)                          │
│     Perpendicular lines: m₁ · m₂ = -1                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: One Variable Linear Equations](01-one-variable-equations.md) | [Back to Contents](../README.md) | [Next: Word Problems with Linear Equations →](03-word-problems.md)
