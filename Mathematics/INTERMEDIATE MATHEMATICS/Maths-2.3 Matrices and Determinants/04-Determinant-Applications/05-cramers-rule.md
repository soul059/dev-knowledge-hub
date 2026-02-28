# Chapter 4.5: Cramer's Rule

[← Previous: Area of Triangle](04-area-of-triangle.md) | [Back to README](../README.md) | [Next: Adjoint of a Matrix →](../05-Adjoint-and-Inverse/01-adjoint-of-matrix.md)

---

## 📚 Chapter Overview

Cramer's Rule is a classical method for solving systems of linear equations using determinants. While not always the most efficient computational method, it provides elegant closed-form solutions and important theoretical insights.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State and apply Cramer's Rule for 2×2 and 3×3 systems
- Understand when Cramer's Rule can be applied
- Identify special cases (no solution, infinite solutions)
- Compare Cramer's Rule with other methods

---

## 1. Introduction to Systems of Linear Equations

### General Form

A system of n linear equations in n unknowns:

$$\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\vdots \\
a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nn}x_n = b_n
\end{cases}$$

### Matrix Form

$$AX = B$$

Where:
- A is the coefficient matrix (n×n)
- X is the column vector of unknowns
- B is the column vector of constants

---

## 2. Cramer's Rule Statement

### The Formula

> If $|A| \neq 0$, then the unique solution is:
> $$x_i = \frac{|A_i|}{|A|}$$
> 
> where $A_i$ is the matrix obtained by replacing the i-th column of A with B.

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                     CRAMER'S RULE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Original System: AX = B                                        │
│                                                                  │
│   A = | a₁  a₂  a₃ |       B = | b₁ |                          │
│       | .   .   .  |           | b₂ |                          │
│       | .   .   .  |           | b₃ |                          │
│                                                                  │
│   Replace columns with B:                                        │
│                                                                  │
│   A₁ = | b  a₂  a₃ |   A₂ = | a₁  b  a₃ |   A₃ = | a₁  a₂  b | │
│        | .  .   .  |        | .   .  .  |        | .   .   . | │
│        | .  .   .  |        | .   .  .  |        | .   .   . | │
│                                                                  │
│   x₁ = |A₁|/|A|      x₂ = |A₂|/|A|      x₃ = |A₃|/|A|          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Cramer's Rule for 2×2 Systems

### System

$$\begin{cases}
a_1x + b_1y = c_1 \\
a_2x + b_2y = c_2
\end{cases}$$

### Solution

$$D = \begin{vmatrix} a_1 & b_1 \\ a_2 & b_2 \end{vmatrix}$$

$$D_x = \begin{vmatrix} c_1 & b_1 \\ c_2 & b_2 \end{vmatrix}, \quad D_y = \begin{vmatrix} a_1 & c_1 \\ a_2 & c_2 \end{vmatrix}$$

$$x = \frac{D_x}{D}, \quad y = \frac{D_y}{D}$$

### Example 1

Solve: $\begin{cases} 2x + 3y = 7 \\ 4x - y = 1 \end{cases}$

**Step 1**: Calculate D
$$D = \begin{vmatrix} 2 & 3 \\ 4 & -1 \end{vmatrix} = -2 - 12 = -14$$

**Step 2**: Calculate $D_x$ (replace x-column with constants)
$$D_x = \begin{vmatrix} 7 & 3 \\ 1 & -1 \end{vmatrix} = -7 - 3 = -10$$

**Step 3**: Calculate $D_y$ (replace y-column with constants)
$$D_y = \begin{vmatrix} 2 & 7 \\ 4 & 1 \end{vmatrix} = 2 - 28 = -26$$

**Step 4**: Solve
$$x = \frac{D_x}{D} = \frac{-10}{-14} = \frac{5}{7}$$
$$y = \frac{D_y}{D} = \frac{-26}{-14} = \frac{13}{7}$$

**Verification**: 
$2(\frac{5}{7}) + 3(\frac{13}{7}) = \frac{10 + 39}{7} = \frac{49}{7} = 7$ ✓

---

## 4. Cramer's Rule for 3×3 Systems

### System

$$\begin{cases}
a_1x + b_1y + c_1z = d_1 \\
a_2x + b_2y + c_2z = d_2 \\
a_3x + b_3y + c_3z = d_3
\end{cases}$$

### Solution

$$D = \begin{vmatrix} a_1 & b_1 & c_1 \\ a_2 & b_2 & c_2 \\ a_3 & b_3 & c_3 \end{vmatrix}$$

$$D_x = \begin{vmatrix} d_1 & b_1 & c_1 \\ d_2 & b_2 & c_2 \\ d_3 & b_3 & c_3 \end{vmatrix}, \quad D_y = \begin{vmatrix} a_1 & d_1 & c_1 \\ a_2 & d_2 & c_2 \\ a_3 & d_3 & c_3 \end{vmatrix}, \quad D_z = \begin{vmatrix} a_1 & b_1 & d_1 \\ a_2 & b_2 & d_2 \\ a_3 & b_3 & d_3 \end{vmatrix}$$

$$x = \frac{D_x}{D}, \quad y = \frac{D_y}{D}, \quad z = \frac{D_z}{D}$$

### Example 2

Solve: $\begin{cases} x + y + z = 6 \\ 2x - y + z = 3 \\ x + 2y - z = 2 \end{cases}$

**Step 1**: Calculate D
$$D = \begin{vmatrix} 1 & 1 & 1 \\ 2 & -1 & 1 \\ 1 & 2 & -1 \end{vmatrix}$$

Using Sarrus' Rule:
Positive: $1(-1)(-1) + 1(1)(1) + 1(2)(2) = 1 + 1 + 4 = 6$
Negative: $1(-1)(1) + 2(1)(1) + (-1)(2)(1) = -1 + 2 - 2 = -1$
$$D = 6 - (-1) = 7$$

**Step 2**: Calculate $D_x$
$$D_x = \begin{vmatrix} 6 & 1 & 1 \\ 3 & -1 & 1 \\ 2 & 2 & -1 \end{vmatrix}$$

Positive: $6(-1)(-1) + 1(1)(2) + 1(3)(2) = 6 + 2 + 6 = 14$
Negative: $2(-1)(1) + 2(1)(6) + (-1)(3)(1) = -2 + 12 - 3 = 7$
$$D_x = 14 - 7 = 7$$

**Step 3**: Calculate $D_y$
$$D_y = \begin{vmatrix} 1 & 6 & 1 \\ 2 & 3 & 1 \\ 1 & 2 & -1 \end{vmatrix}$$

Positive: $1(3)(-1) + 6(1)(1) + 1(2)(2) = -3 + 6 + 4 = 7$
Negative: $1(3)(1) + 2(1)(1) + (-1)(2)(6) = 3 + 2 - 12 = -7$
$$D_y = 7 - (-7) = 14$$

**Step 4**: Calculate $D_z$
$$D_z = \begin{vmatrix} 1 & 1 & 6 \\ 2 & -1 & 3 \\ 1 & 2 & 2 \end{vmatrix}$$

Positive: $1(-1)(2) + 1(3)(1) + 6(2)(2) = -2 + 3 + 24 = 25$
Negative: $1(-1)(6) + 2(3)(1) + 2(2)(1) = -6 + 6 + 4 = 4$
$$D_z = 25 - 4 = 21$$

**Step 5**: Solve
$$x = \frac{D_x}{D} = \frac{7}{7} = 1$$
$$y = \frac{D_y}{D} = \frac{14}{7} = 2$$
$$z = \frac{D_z}{D} = \frac{21}{7} = 3$$

**Solution**: $(x, y, z) = (1, 2, 3)$

**Verification**: $1 + 2 + 3 = 6$ ✓, $2(1) - 2 + 3 = 3$ ✓, $1 + 2(2) - 3 = 2$ ✓

---

## 5. Special Cases

### Case 1: D = 0 (Singular Matrix)

When $D = |A| = 0$, Cramer's Rule cannot be applied directly.

```
┌─────────────────────────────────────────────────────────────────┐
│                 WHEN D = 0                                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Two possibilities:                                             │
│                                                                  │
│   1. If D = 0 AND at least one D_i ≠ 0:                         │
│      → NO SOLUTION (inconsistent system)                        │
│                                                                  │
│   2. If D = 0 AND all D_i = 0:                                  │
│      → INFINITELY MANY SOLUTIONS (dependent system)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Case 2: Homogeneous System (all constants = 0)

$$AX = 0$$

- If $D \neq 0$: Only the trivial solution $(x_1, x_2, ..., x_n) = (0, 0, ..., 0)$
- If $D = 0$: Infinitely many solutions (non-trivial solutions exist)

---

## 6. Example: No Solution Case

Solve: $\begin{cases} x + y = 1 \\ 2x + 2y = 5 \end{cases}$

$$D = \begin{vmatrix} 1 & 1 \\ 2 & 2 \end{vmatrix} = 2 - 2 = 0$$

$$D_x = \begin{vmatrix} 1 & 1 \\ 5 & 2 \end{vmatrix} = 2 - 5 = -3 \neq 0$$

Since $D = 0$ but $D_x \neq 0$: **No solution exists**.

(The lines are parallel!)

---

## 7. Example: Infinite Solutions Case

Solve: $\begin{cases} x + y = 3 \\ 2x + 2y = 6 \end{cases}$

$$D = \begin{vmatrix} 1 & 1 \\ 2 & 2 \end{vmatrix} = 0$$

$$D_x = \begin{vmatrix} 3 & 1 \\ 6 & 2 \end{vmatrix} = 6 - 6 = 0$$

$$D_y = \begin{vmatrix} 1 & 3 \\ 2 & 6 \end{vmatrix} = 6 - 6 = 0$$

Since $D = 0$ and all $D_i = 0$: **Infinitely many solutions**.

General solution: $y = 3 - x$ (same line!)

---

## 8. Comparison with Other Methods

| Method | Advantages | Disadvantages |
|--------|------------|---------------|
| Cramer's Rule | Closed-form solution, theoretical value | Slow for large systems (O(n! × n)) |
| Gaussian Elimination | Efficient (O(n³)), handles all cases | No closed-form expression |
| Matrix Inverse | Direct ($X = A^{-1}B$) | Expensive to find inverse |

### When to Use Cramer's Rule

✅ Small systems (2×2 or 3×3)
✅ When a closed-form solution is needed
✅ Theoretical analysis
✅ Systems with symbolic coefficients

❌ Large systems (n > 4)
❌ When D = 0

---

## 9. Cramer's Rule and Matrix Inverse

### Connection

If $AX = B$ and $|A| \neq 0$:
$$X = A^{-1}B$$

The formula $x_i = \frac{|A_i|}{|A|}$ is equivalent to the i-th component of $A^{-1}B$.

### Relationship with Adjoint

$$A^{-1} = \frac{1}{|A|} \text{adj}(A)$$

Cramer's Rule extracts individual solutions without computing the full inverse.

---

## 10. Flowchart for Solving Systems

```
┌─────────────────────────────────────────────────────────────────┐
│                 SOLVING LINEAR SYSTEMS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                   Calculate D = |A|                              │
│                          │                                       │
│            ┌─────────────┴─────────────┐                        │
│            │                           │                        │
│            ▼                           ▼                        │
│        D ≠ 0?                       D = 0?                      │
│            │                           │                        │
│            ▼                           ▼                        │
│    UNIQUE SOLUTION            Calculate all D_i                 │
│    x_i = D_i/D                        │                        │
│                           ┌───────────┴───────────┐             │
│                           │                       │             │
│                           ▼                       ▼             │
│                    Any D_i ≠ 0?           All D_i = 0?         │
│                           │                       │             │
│                           ▼                       ▼             │
│                     NO SOLUTION         INFINITE SOLUTIONS      │
│                    (Inconsistent)         (Dependent)           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Condition | Result |
|-----------|--------|
| $D \neq 0$ | Unique solution: $x_i = D_i/D$ |
| $D = 0$, some $D_i \neq 0$ | No solution |
| $D = 0$, all $D_i = 0$ | Infinitely many solutions |
| Homogeneous, $D \neq 0$ | Only trivial solution |
| Homogeneous, $D = 0$ | Non-trivial solutions exist |

---

## ❓ Quick Revision Questions

1. **What is the formula for $x$ in a 2×2 system using Cramer's Rule?**
   <details>
   <summary>Click for Answer</summary>
   $x = \frac{D_x}{D}$ where $D_x$ is the determinant with the x-column replaced by constants.
   </details>

2. **When can Cramer's Rule NOT be applied?**
   <details>
   <summary>Click for Answer</summary>
   When $D = |A| = 0$ (the coefficient matrix is singular).
   </details>

3. **If D = 0 and $D_x \neq 0$, what does this mean?**
   <details>
   <summary>Click for Answer</summary>
   The system has no solution (inconsistent).
   </details>

4. **For a homogeneous system with D ≠ 0, what is the solution?**
   <details>
   <summary>Click for Answer</summary>
   The trivial solution: all variables equal zero.
   </details>

5. **Solve: 3x + 2y = 12, x - y = 1**
   <details>
   <summary>Click for Answer</summary>
   $D = -3-2 = -5$, $D_x = -12-2 = -14$, $D_y = 3-12 = -9$. So $x = 14/5$, $y = 9/5$.
   </details>

6. **Why is Cramer's Rule inefficient for large systems?**
   <details>
   <summary>Click for Answer</summary>
   Computing each determinant requires O(n!) operations, and we need n+1 determinants. Gaussian elimination is O(n³).
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Area of Triangle](04-area-of-triangle.md) | [Unit 4: Applications](./README.md) | [Adjoint of a Matrix →](../05-Adjoint-and-Inverse/01-adjoint-of-matrix.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
