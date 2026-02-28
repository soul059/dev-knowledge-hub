# Chapter 3.1: Gaussian Elimination

[← Previous: Convergence Analysis](../02-Solution-of-Equations/06-convergence-analysis.md) | [Next: Gauss-Jordan Method →](02-gauss-jordan.md)

---

## Overview

**Gaussian Elimination** is the fundamental direct method for solving systems of linear equations $Ax = b$. It transforms the augmented matrix $[A|b]$ into **upper triangular form** using elementary row operations, then solves by **back substitution**. It is the backbone of linear algebra computation.

---

## 1. The Problem

Solve the system of $n$ linear equations in $n$ unknowns:

$$a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1$$
$$a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2$$
$$\vdots$$
$$a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nn}x_n = b_n$$

In matrix form: $Ax = b$

---

## 2. Elementary Row Operations

| Operation | Notation | Effect |
|-----------|----------|--------|
| Swap two rows | $R_i \leftrightarrow R_j$ | Reorder equations |
| Multiply a row by nonzero constant | $R_i \to cR_i$ | Scale an equation |
| Add a multiple of one row to another | $R_i \to R_i + cR_j$ | Eliminate a variable |

These operations do NOT change the solution of the system.

---

## 3. Forward Elimination

Transform $[A|b]$ into upper triangular form:

```
  BEFORE (Augmented Matrix):         AFTER (Upper Triangular):

  ┌                          ┐      ┌                          ┐
  │ a₁₁  a₁₂  a₁₃ │ b₁     │      │ a₁₁' a₁₂' a₁₃'│ b₁'    │
  │ a₂₁  a₂₂  a₂₃ │ b₂     │  ──► │  0   a₂₂' a₂₃'│ b₂'    │
  │ a₃₁  a₃₂  a₃₃ │ b₃     │      │  0    0   a₃₃'│ b₃'    │
  └                          ┘      └                          ┘

  Step 1: Eliminate x₁ from rows 2,3 using row 1 (pivot row)
  Step 2: Eliminate x₂ from row 3 using row 2 (pivot row)
```

### Multiplier Formula

To eliminate element $a_{ij}$ using pivot $a_{jj}$:

$$m_{ij} = \frac{a_{ij}}{a_{jj}} \quad \text{(multiplier)}$$

$$R_i \to R_i - m_{ij} \cdot R_j$$

---

## 4. Back Substitution

Once in upper triangular form:

$$x_n = \frac{b_n'}{a_{nn}'}$$

$$x_i = \frac{1}{a_{ii}'}\left(b_i' - \sum_{j=i+1}^{n} a_{ij}' x_j\right), \quad i = n-1, n-2, \ldots, 1$$

---

## 5. Worked Example

Solve:
$$2x_1 + x_2 + x_3 = 5$$
$$4x_1 + 3x_2 + 3x_3 = 11$$
$$2x_1 + x_2 + 2x_3 = 6$$

### Step 1: Set up augmented matrix

$$\left[\begin{array}{ccc|c} 2 & 1 & 1 & 5 \\ 4 & 3 & 3 & 11 \\ 2 & 1 & 2 & 6 \end{array}\right]$$

### Step 2: Eliminate $x_1$ from rows 2 and 3

$m_{21} = 4/2 = 2$: $R_2 \to R_2 - 2R_1$
$m_{31} = 2/2 = 1$: $R_3 \to R_3 - 1R_1$

$$\left[\begin{array}{ccc|c} 2 & 1 & 1 & 5 \\ 0 & 1 & 1 & 1 \\ 0 & 0 & 1 & 1 \end{array}\right]$$

### Step 3: Already upper triangular! Back substitute.

$$x_3 = 1$$
$$x_2 = 1 - 1(1) = 0$$
$$x_1 = \frac{5 - 1(0) - 1(1)}{2} = 2$$

**Solution**: $x_1 = 2, x_2 = 0, x_3 = 1$

---

## 6. Pivoting Strategies

### Why Pivot?

If a pivot $a_{jj} = 0$ (or very small), division fails or amplifies round-off errors.

### Partial Pivoting (Most Common)

At step $k$, find the row with the **largest** $|a_{ik}|$ for $i \geq k$, and swap it into the pivot position.

```
  Before pivoting (step 2):     After partial pivoting:

  ┌               ┐              ┌               ┐
  │ * * * │ *    │              │ * * * │ *    │
  │ 0 ε * │ *    │    swap     │ 0 8 * │ *    │  ← largest
  │ 0 8 * │ *    │   R₂↔R₃    │ 0 ε * │ *    │
  └               ┘              └               ┘

  Using ε as pivot → multiplier ≈ 8/ε → HUGE → BAD!
  Using 8 as pivot → multiplier ≈ ε/8 → small → GOOD!
```

### Scaled Partial Pivoting

Take into account the scale of each row when choosing the pivot. Choose the row $i$ that maximizes $|a_{ik}|/s_i$ where $s_i = \max_j |a_{ij}|$.

---

## 7. Operation Count

| Phase | Multiplications | Additions |
|-------|:--------------:|:---------:|
| Forward elimination | $\frac{n^3}{3} + O(n^2)$ | $\frac{n^3}{3} + O(n^2)$ |
| Back substitution | $\frac{n^2}{2} + O(n)$ | $\frac{n^2}{2} + O(n)$ |
| **Total** | $\approx \frac{n^3}{3}$ | $\approx \frac{n^3}{3}$ |

For large $n$, the cost is **$O(n^3)$**.

---

## 8. Special Cases and Pitfalls

| Situation | What Happens | Solution |
|-----------|-------------|----------|
| $\det(A) = 0$ | No unique solution | System is singular |
| $a_{kk} = 0$ during elimination | Division by zero | Use pivoting |
| $A$ nearly singular | Large round-off errors | Use pivoting + iterative refinement |
| $A$ is tridiagonal | Many zeros to exploit | Use Thomas algorithm: $O(n)$ |

---

## 9. Real-World Applications

| Application | System Size |
|-------------|:-----------:|
| Circuit analysis (Kirchhoff's laws) | 10-1000 |
| Structural engineering (FEA) | $10^3 - 10^6$ |
| Chemical equilibrium | 10-100 |
| Network flow optimization | $10^3 - 10^5$ |
| Computer graphics (transformations) | 3-4 |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Direct method |
| **Idea** | Row operations → upper triangular → back substitution |
| **Complexity** | $O(n^3/3)$ operations |
| **Pivoting** | Essential for numerical stability |
| **Storage** | Can be done in-place (overwrite $A$) |
| **Multiple RHS** | Efficient: elimination once, back-sub per RHS |
| **Determinant** | $\det(A) = \prod a_{ii}'$ (product of pivots) |

---

## Quick Revision Questions

1. **Solve using Gaussian elimination: $x + y + z = 6$, $2x + 3y + z = 10$, $x + 2y + 4z = 16$.**

2. **Why is partial pivoting necessary? Give an example where its absence causes problems.**

3. **What is the operation count for Gaussian elimination on an $n \times n$ system?**

4. **How can you compute $\det(A)$ as a by-product of Gaussian elimination?**

5. **Compare Gaussian elimination with Cramer's rule for an $n \times n$ system in terms of operation count.**

6. **What modifications are needed to handle a system with more equations than unknowns?**

---

[← Previous: Convergence Analysis](../02-Solution-of-Equations/06-convergence-analysis.md) | [Next: Gauss-Jordan Method →](02-gauss-jordan.md)
