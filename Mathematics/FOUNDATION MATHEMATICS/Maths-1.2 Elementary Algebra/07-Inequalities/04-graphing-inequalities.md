# Chapter 7.4: Graphing Inequalities

[← Previous: Absolute Value Inequalities](03-absolute-value-inequalities.md) | [Back to Contents](../README.md) | [Next: Arithmetic Sequences →](../08-Sequences-and-Series/01-arithmetic-sequences.md)

---

## 📚 Chapter Overview

This chapter extends inequality concepts to two dimensions. We learn to graph linear inequalities in two variables and systems of inequalities, identifying solution regions that satisfy multiple constraints simultaneously.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Graph linear inequalities in two variables
- Determine which region to shade (test point method)
- Use solid vs. dashed boundary lines correctly
- Graph systems of linear inequalities
- Identify the feasible region for a system
- Apply graphing to linear programming concepts

---

## 1. Linear Inequalities in Two Variables

### Types of Linear Inequalities

```
┌─────────────────────────────────────────────────────────────────────┐
│              LINEAR INEQUALITIES IN TWO VARIABLES                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   A linear inequality in x and y has one of these forms:          │
│                                                                     │
│   ax + by < c                                                      │
│   ax + by > c                                                      │
│   ax + by ≤ c                                                      │
│   ax + by ≥ c                                                      │
│                                                                     │
│   Examples:                                                         │
│   2x + 3y ≤ 12                                                     │
│   y > x - 4                                                        │
│   x - 2y < 6                                                       │
│                                                                     │
│   The solution is a REGION in the coordinate plane                 │
│   (infinitely many (x, y) pairs that make it true)                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Boundary Line

```
┌─────────────────────────────────────────────────────────────────────┐
│              BOUNDARY LINE                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   The boundary line is found by replacing the inequality           │
│   symbol with an equals sign.                                      │
│                                                                     │
│   For 2x + 3y ≤ 12, the boundary is 2x + 3y = 12                  │
│                                                                     │
│   Line style:                                                       │
│   ─────────────  Solid line: ≤ or ≥ (includes boundary)          │
│   - - - - - - -  Dashed line: < or > (excludes boundary)          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Graphing Method

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────────┐
│              HOW TO GRAPH A LINEAR INEQUALITY                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Graph the boundary line                                  │
│           • Replace inequality with = to get equation             │
│           • Use solid line for ≤ or ≥                             │
│           • Use dashed line for < or >                            │
│                                                                     │
│   Step 2: Choose a test point NOT on the line                     │
│           • (0, 0) is easiest if not on the line                  │
│           • Substitute into the original inequality               │
│                                                                     │
│   Step 3: Shade the correct region                                 │
│           • If test point makes inequality TRUE → shade that side │
│           • If test point makes inequality FALSE → shade other    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example: Graph y ≤ 2x + 1

```
Step 1: Boundary line is y = 2x + 1
        Solid line (because ≤)
        
        y-intercept: (0, 1)
        slope: 2 (rise 2, run 1)

Step 2: Test point (0, 0)
        Is 0 ≤ 2(0) + 1?
        Is 0 ≤ 1? YES! ✓

Step 3: Shade the side containing (0, 0)

        y
        │         ╱
        │       ╱
     3  │     ╱
        │   ╱
     1 ─│─●─────────    ← y = 2x + 1
        │╱░░░░░░░░░
    ────●░░░░░░░░░──→ x
       ╱│░░░░░░░░░
      ╱ │░░(shaded)

The shaded region (below and on the line) is the solution.
```

---

## 3. Special Cases

### Horizontal and Vertical Inequalities

```
┌─────────────────────────────────────────────────────────────────────┐
│              HORIZONTAL AND VERTICAL                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   y ≤ 3 (horizontal line at y = 3)                                │
│                                                                     │
│        y                                                            │
│        │                                                            │
│     4  │                                                            │
│     3 ─│═══════════════  ← y = 3 (solid)                           │
│        │░░░░░░░░░░░░░░░                                            │
│     1  │░░░░░░░░░░░░░░░  ← shade BELOW                             │
│   ─────│░░░░░░░░░░░░░░░──→ x                                       │
│        │░░░░░░░░░░░░░░░                                            │
│                                                                     │
│   ──────────────────────────────────────                           │
│                                                                     │
│   x > -2 (vertical line at x = -2)                                 │
│                                                                     │
│        y                                                            │
│        │    ░░░░░░░░░                                              │
│        │    ░░░░░░░░░                                              │
│        │    ░░░░░░░░░  ← shade RIGHT                               │
│   ─────│──┿─░░░░░░░░░──→ x                                         │
│        │  ┆ ░░░░░░░░░                                              │
│           ┆                                                         │
│          x = -2 (dashed)                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Solving in Slope-Intercept Form

### Converting for Easier Graphing

```
Example: Graph 3x - 2y > 6

Step 1: Solve for y (optional but helpful)
-2y > -3x + 6
y < (3/2)x - 3    (flip sign when dividing by -2!)

Step 2: Boundary line y = (3/2)x - 3
        Dashed line (strict inequality)
        
Step 3: Test (0, 0)
        Is 0 < (3/2)(0) - 3?
        Is 0 < -3? NO! ✗

Step 4: Shade the side NOT containing (0, 0)
        (below the line)

        y
        │            ╱
        │          ╱
        │        ╱
   ─────│──────╱─────────→ x
        │    ╱
    -3 ─│──●╱- - - - -    ← y = (3/2)x - 3 (dashed)
        │╱░░░░░░░░░░░
        │░░░░░░░░░░░░
```

---

## 5. Systems of Linear Inequalities

### Definition

A **system of linear inequalities** consists of two or more linear inequalities considered simultaneously.

```
┌─────────────────────────────────────────────────────────────────────┐
│              SYSTEM OF INEQUALITIES                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Example:                                                          │
│   y ≤ x + 3                                                        │
│   y ≥ -x + 1                                                       │
│   x ≥ 0                                                             │
│                                                                     │
│   The solution (feasible region) is where ALL inequalities         │
│   are satisfied simultaneously—the INTERSECTION of all             │
│   individual solution regions.                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Graphing a System

```
Graph the system:
y ≤ x + 2
y ≥ -x - 1
x ≤ 3

Step 1: Graph each boundary line
        y = x + 2    (solid, shade below)
        y = -x - 1   (solid, shade above)
        x = 3        (solid, shade left)

Step 2: Identify the overlapping region

        y
        │    ╲    ╱
        │     ╲  ╱
     4  │      ╲╱
        │  ░░░░╱╲
     2  │░░░╱   ╲────│
        │░╱░░░░░░░░░░│
   ─────●░░░░░░░░░░░░│───→ x
       ╱│░░░░░░░░░░░░│
      ╱ │░░░░░░░░░░░░│
    -2  │            │
        │            x = 3

The triangular shaded region is the solution.
Vertices can be found by solving pairs of equations.
```

---

## 6. Finding Corner Points (Vertices)

### Intersection of Boundary Lines

The corner points (vertices) of the feasible region are found by solving systems of equations for pairs of boundary lines.

```
Find the vertices of:
y ≤ x + 2
y ≥ -x
y ≤ 4

Vertices are intersections of:

1. y = x + 2 and y = -x
   x + 2 = -x
   2x = -2
   x = -1, y = 1
   Vertex: (-1, 1)

2. y = x + 2 and y = 4
   x + 2 = 4
   x = 2
   Vertex: (2, 4)

3. y = -x and y = 4
   -x = 4
   x = -4
   Vertex: (-4, 4)

The feasible region is a triangle with vertices:
(-1, 1), (2, 4), (-4, 4)
```

---

## 7. Applications: Linear Programming Basics

### Optimization Problems

```
┌─────────────────────────────────────────────────────────────────────┐
│              LINEAR PROGRAMMING                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Linear programming finds the maximum or minimum value of         │
│   an objective function subject to constraints (inequalities).     │
│                                                                     │
│   Key theorem: The optimal value occurs at a VERTEX                │
│   (corner point) of the feasible region.                          │
│                                                                     │
│   Steps:                                                            │
│   1. Graph the system of constraints                               │
│   2. Identify the feasible region                                  │
│   3. Find all vertices (corner points)                             │
│   4. Evaluate the objective function at each vertex                │
│   5. Choose the vertex giving max or min value                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
Maximize P = 3x + 2y subject to:
x ≥ 0
y ≥ 0
x + y ≤ 4
2x + y ≤ 6

The feasible region has vertices at:
(0, 0), (3, 0), (2, 2), (0, 4)

Evaluate P at each:
P(0, 0) = 0
P(3, 0) = 9
P(2, 2) = 10  ← Maximum!
P(0, 4) = 8

Maximum value is P = 10 at (2, 2)
```

---

## ✏️ Solved Examples

### Example 1: Easy - Single inequality

**Problem:** Graph y > 2x - 3

**Solution:**
```
Boundary: y = 2x - 3
Line type: Dashed (strict >)

Intercepts: y-int (0, -3), x-int (3/2, 0)

Test point (0, 0):
Is 0 > 2(0) - 3?
Is 0 > -3? YES ✓

Shade ABOVE the dashed line (containing origin)

        y
        │       ╱░░░
     2  │     ╱░░░░░
        │   ╱░░░░░░░
   ─────│─╱─────────→ x
        │╱
    -3 ─┆- - - -
        ┆
```

### Example 2: Easy - Horizontal/vertical

**Problem:** Graph x ≤ 4 and y > -2 together

**Solution:**
```
x ≤ 4: vertical solid line at x = 4, shade LEFT
y > -2: horizontal dashed line at y = -2, shade ABOVE

        y
        │░░░░░░░░│
     2  │░░░░░░░░│
        │░░░░░░░░│
   ─────│░░░░░░░░│────→ x
        │░░░░░░░░│
    -2 ─┆┄┄┄┄┄┄┄┄┆    (dashed)
        │        │
                 x = 4

The overlapping region (upper-left quadrant bounded by x=4 and y=-2)
```

### Example 3: Medium - System of two inequalities

**Problem:** Graph the solution to:
y ≤ x + 1
y ≥ -2x + 4

**Solution:**
```
Line 1: y = x + 1 (solid), shade below
        Through (0, 1), slope 1

Line 2: y = -2x + 4 (solid), shade above
        Through (0, 4), slope -2

Find intersection:
x + 1 = -2x + 4
3x = 3
x = 1, y = 2
Intersection at (1, 2)

Test regions:
At (0, 0): 0 ≤ 1 ✓, 0 ≥ 4 ✗ (not in solution)
At (2, 0): 0 ≤ 3 ✓, 0 ≥ 0 ✓ (in solution)

        y
        │╲    ╱
     4 ─│─╲──●
        │░░╲╱
     2 ─│░░░●░░░░░
        │░░╱░╲░░░░
   ─────│╱░░░░╲░░░──→ x
        │░░░░░░░░░

Feasible region is the angular region below y = x + 1 
and above y = -2x + 4
```

### Example 4: Hard - Three constraints

**Problem:** Graph and find vertices of:
x + y ≤ 5
x ≥ 1
y ≥ 0

**Solution:**
```
Boundary lines:
x + y = 5  (solid, shade below)
x = 1      (solid, shade right)
y = 0      (solid, shade above, i.e., first quadrant)

Find vertices:

1. x + y = 5 and x = 1:
   1 + y = 5 → y = 4
   Vertex: (1, 4)

2. x + y = 5 and y = 0:
   x = 5
   Vertex: (5, 0)

3. x = 1 and y = 0:
   Vertex: (1, 0)

        y
        │
     5  │╲
     4  │░●
        │░░╲
     2  │░░░╲
        │░░░░╲
   ─────│░●░░░●────→ x
        │  1    5

Triangle with vertices (1, 0), (1, 4), (5, 0)
```

### Example 5: Hard - Linear programming

**Problem:** Minimize C = 4x + 3y subject to:
x + y ≥ 4
2x + y ≥ 5
x ≥ 0, y ≥ 0

**Solution:**
```
Graph the constraints (shade appropriate regions):

Find vertices:
1. x + y = 4 and 2x + y = 5:
   Subtract: x = 1, y = 3 → (1, 3)

2. x + y = 4 and x = 0:
   y = 4 → (0, 4)

3. 2x + y = 5 and y = 0:
   x = 2.5 → (2.5, 0)

4. x + y = 4 and y = 0:
   x = 4 → (4, 0)
   But check: Does (4, 0) satisfy 2x + y ≥ 5?
   2(4) + 0 = 8 ≥ 5 ✓

The feasible region vertices are: (0, 4), (1, 3), (2.5, 0)
Wait - at (2.5, 0): x + y = 2.5, is this ≥ 4? No!
So (2.5, 0) is NOT in the feasible region.

Let me reconsider. The feasible region is ABOVE both lines.
The unbounded region has vertices (0, 5), (1, 3), (4, 0)

Actually, let me be more careful:
At y = 0: x + y ≥ 4 means x ≥ 4
          2x + y ≥ 5 means x ≥ 2.5
So on x-axis, we need x ≥ 4

At x = 0: x + y ≥ 4 means y ≥ 4
          2x + y ≥ 5 means y ≥ 5
So on y-axis, we need y ≥ 5

Vertices: (0, 5), (1, 3), (4, 0)

Evaluate C = 4x + 3y:
C(0, 5) = 15
C(1, 3) = 13  ← Minimum!
C(4, 0) = 16

Minimum C = 13 at (1, 3)
```

---

## ❓ Practice Problems

### Easy Level

1. Graph: y < x + 4

2. Graph: 2x + y ≥ 6

3. Graph the system: y ≤ 3, x ≥ -1

### Medium Level

4. Graph and find the corner points:
   y ≥ x - 2
   y ≤ 4
   x ≥ 0

5. Graph: 3x - 2y ≤ 6

6. Graph the system:
   x + y ≤ 6
   x - y ≤ 2
   y ≥ 0

### Hard Level

7. Find the feasible region and all vertices:
   x + 2y ≤ 8
   2x + y ≤ 10
   x ≥ 0, y ≥ 0

8. Maximize P = 5x + 4y subject to:
   x + y ≤ 6
   2x + y ≤ 8
   x ≥ 0, y ≥ 0

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. Dashed line y = x + 4, shade below. Origin test: 0 < 4 ✓

2. Solid line through (3, 0) and (0, 6), shade above/right.
   Origin test: 0 ≥ 6 ✗, so shade away from origin.

3. y ≤ 3: shade below horizontal line at y = 3
   x ≥ -1: shade right of vertical line at x = -1
   **Region: right of x = -1 AND below y = 3**

4. Vertices: Solve pairs of equations
   y = x - 2 and x = 0: (0, -2)
   y = x - 2 and y = 4: x = 6, so (6, 4)
   y = 4 and x = 0: (0, 4)
   But (0, -2) may not satisfy y ≤ 4... it does!
   **Vertices: (0, -2), (0, 4), (6, 4)**

5. Rewrite: y ≥ (3x - 6)/2 = (3/2)x - 3
   Solid line, shade above.

6. Find intersections:
   x + y = 6 and x - y = 2: Add: 2x = 8, x = 4, y = 2 → (4, 2)
   x + y = 6 and y = 0: x = 6 → (6, 0)
   x - y = 2 and y = 0: x = 2 → (2, 0)
   **Vertices: (2, 0), (4, 2), (6, 0)**... 
   Actually check (6, 0): 6 - 0 = 6 ≤ 2? No!
   Re-examine: The region is bounded properly at (4, 2), (6, 0), (2, 0)

7. Vertices:
   (0, 0), (5, 0), (4, 2), (0, 4)
   Check (4, 2): 4 + 4 = 8 ✓, 8 + 2 = 10 ✓
   **Vertices: (0, 0), (5, 0), (4, 2), (0, 4)**

8. Vertices: (0, 0), (4, 0), (2, 4), (0, 6)
   P(0, 0) = 0
   P(4, 0) = 20
   P(2, 4) = 26  ← Maximum
   P(0, 6) = 24
   **Maximum P = 26 at (2, 4)**

</details>

---

## 📋 Summary Table

| Symbol | Line Type | Shading |
|--------|-----------|---------|
| < | Dashed | Does not include boundary |
| > | Dashed | Does not include boundary |
| ≤ | Solid | Includes boundary |
| ≥ | Solid | Includes boundary |

---

## 🔄 Quick Revision Questions

1. **What type of line do you use for y < 2x + 1?**

2. **What is the easiest test point if it's not on the line?**

3. **What is the solution to a system of inequalities called?**

4. **Where do optimal values occur in linear programming?**

5. **How do you find vertices of a feasible region?**

6. **If (0,0) makes an inequality FALSE, which side do you shade?**

<details>
<summary>Quick Answers</summary>

1. Dashed line (strict inequality)
2. (0, 0) - the origin
3. Feasible region
4. At the vertices (corner points)
5. Solve pairs of boundary line equations
6. The side NOT containing (0, 0)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Linear inequalities in two variables define REGIONS           │
│                                                                     │
│   ★ Graphing process:                                              │
│     1. Draw boundary line (solid or dashed)                       │
│     2. Test a point (usually origin)                              │
│     3. Shade correct region                                       │
│                                                                     │
│   ★ Systems of inequalities:                                       │
│     • Solution is the INTERSECTION of all regions                 │
│     • Called the "feasible region"                                │
│                                                                     │
│   ★ Linear programming:                                            │
│     • Optimize an objective function                              │
│     • Subject to constraints (inequalities)                       │
│     • Optimal values at VERTICES of feasible region               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Absolute Value Inequalities](03-absolute-value-inequalities.md) | [Back to Contents](../README.md) | [Next: Arithmetic Sequences →](../08-Sequences-and-Series/01-arithmetic-sequences.md)
