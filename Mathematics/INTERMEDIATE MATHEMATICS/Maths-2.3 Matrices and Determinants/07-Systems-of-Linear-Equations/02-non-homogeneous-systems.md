# Chapter 7.2: Non-Homogeneous Systems of Linear Equations

[← Previous: Homogeneous Systems](01-homogeneous-systems.md) | [Back to README](./README.md) | [Next: General Solutions →](03-general-solutions.md)

---

## 📚 Chapter Overview

Non-homogeneous systems have non-zero constant terms and can have no solution, a unique solution, or infinitely many solutions. This chapter covers systematic methods to solve such systems.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify non-homogeneous systems
- Apply Gaussian elimination completely
- Use Gauss-Jordan elimination
- Solve using matrix inverse method
- Apply Cramer's rule when applicable

---

## 1. Definition

### What is a Non-Homogeneous System?

A system is **non-homogeneous** if at least one constant term is non-zero:

$$\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n = b_m
\end{cases}$$

where at least one $b_i \neq 0$.

Matrix form: $Ax = b$

```
┌─────────────────────────────────────────────────────────────────┐
│           NON-HOMOGENEOUS vs HOMOGENEOUS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Homogeneous: Ax = 0           Non-Homogeneous: Ax = b         │
│                                                                  │
│   ┌───────┐ ┌───┐   ┌───┐      ┌───────┐ ┌───┐   ┌───┐         │
│   │       │ │ x │   │ 0 │      │       │ │ x │   │ b │         │
│   │   A   │ │   │ = │ 0 │      │   A   │ │   │ = │   │         │
│   │       │ │   │   │ 0 │      │       │ │   │   │   │         │
│   └───────┘ └───┘   └───┘      └───────┘ └───┘   └───┘         │
│                                                                  │
│   Always consistent            May be inconsistent               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Solution Methods Overview

| Method | When to Use | Advantage |
|--------|-------------|-----------|
| Gaussian Elimination | Any system | Universal |
| Gauss-Jordan | Any system | Direct solution |
| Matrix Inverse | Square, invertible | Formula: x = A⁻¹b |
| Cramer's Rule | Square, invertible | Formula for each variable |

---

## 3. Gaussian Elimination

### Algorithm

1. Form augmented matrix [A|b]
2. Apply row operations to get Row Echelon Form (REF)
3. Use back substitution to solve

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────┐
│               GAUSSIAN ELIMINATION                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Step 1: Form augmented matrix [A|b]                           │
│                    ↓                                             │
│   Step 2: Identify leftmost non-zero column (pivot column)      │
│                    ↓                                             │
│   Step 3: Make pivot entry = 1 (optional, helps calculation)    │
│                    ↓                                             │
│   Step 4: Use pivot to create zeros below it                    │
│                    ↓                                             │
│   Step 5: Cover the pivot row and repeat for submatrix          │
│                    ↓                                             │
│   Step 6: Continue until REF is achieved                        │
│                    ↓                                             │
│   Step 7: Back substitution                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example 1: Complete Gaussian Elimination

Solve:
$$\begin{cases} x + y + z = 6 \\ 2x + 3y + z = 11 \\ x + 2y + 3z = 14 \end{cases}$$

**Step 1: Augmented matrix**

$$[A|b] = \left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 2 & 3 & 1 & 11 \\ 1 & 2 & 3 & 14 \end{array}\right]$$

**Step 2: Create zeros below first pivot**

$R_2 \to R_2 - 2R_1$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & -1 & -1 \\ 1 & 2 & 3 & 14 \end{array}\right]$$

$R_3 \to R_3 - R_1$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & -1 & -1 \\ 0 & 1 & 2 & 8 \end{array}\right]$$

**Step 3: Create zero below second pivot**

$R_3 \to R_3 - R_2$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & -1 & -1 \\ 0 & 0 & 3 & 9 \end{array}\right]$$

This is Row Echelon Form.

**Step 4: Back substitution**

From row 3: $3z = 9 \Rightarrow z = 3$

From row 2: $y - z = -1 \Rightarrow y = -1 + 3 = 2$

From row 1: $x + y + z = 6 \Rightarrow x = 6 - 2 - 3 = 1$

$$\boxed{x = 1, \quad y = 2, \quad z = 3}$$

**Verification**: 
- $1 + 2 + 3 = 6$ ✓
- $2(1) + 3(2) + 3 = 11$ ✓
- $1 + 2(2) + 3(3) = 14$ ✓

---

## 4. Gauss-Jordan Elimination

### Algorithm

Continue from REF to get Reduced Row Echelon Form (RREF), eliminating entries above pivots as well.

### Continuing Example 1 to RREF

From REF:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & -1 & -1 \\ 0 & 0 & 3 & 9 \end{array}\right]$$

**Make all pivots = 1**:

$R_3 \to \frac{1}{3}R_3$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & -1 & -1 \\ 0 & 0 & 1 & 3 \end{array}\right]$$

**Eliminate above pivot 3**:

$R_2 \to R_2 + R_3$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & 0 & 2 \\ 0 & 0 & 1 & 3 \end{array}\right]$$

$R_1 \to R_1 - R_3$:
$$\left[\begin{array}{ccc|c} 1 & 1 & 0 & 3 \\ 0 & 1 & 0 & 2 \\ 0 & 0 & 1 & 3 \end{array}\right]$$

**Eliminate above pivot 2**:

$R_1 \to R_1 - R_2$:
$$\left[\begin{array}{ccc|c} 1 & 0 & 0 & 1 \\ 0 & 1 & 0 & 2 \\ 0 & 0 & 1 & 3 \end{array}\right]$$

This is RREF. Solution reads directly: $x = 1, y = 2, z = 3$

---

## 5. Matrix Inverse Method

### Formula

For a square system $Ax = b$ where A is invertible:

$$x = A^{-1}b$$

### Example 2

Solve using matrix inverse:
$$\begin{cases} 2x + y = 5 \\ x + 3y = 5 \end{cases}$$

**Find A⁻¹**:

$$A = \begin{bmatrix} 2 & 1 \\ 1 & 3 \end{bmatrix}, \quad |A| = 6 - 1 = 5$$

$$A^{-1} = \frac{1}{5}\begin{bmatrix} 3 & -1 \\ -1 & 2 \end{bmatrix}$$

**Calculate x = A⁻¹b**:

$$\begin{bmatrix} x \\ y \end{bmatrix} = \frac{1}{5}\begin{bmatrix} 3 & -1 \\ -1 & 2 \end{bmatrix}\begin{bmatrix} 5 \\ 5 \end{bmatrix}$$

$$= \frac{1}{5}\begin{bmatrix} 15 - 5 \\ -5 + 10 \end{bmatrix} = \frac{1}{5}\begin{bmatrix} 10 \\ 5 \end{bmatrix} = \begin{bmatrix} 2 \\ 1 \end{bmatrix}$$

$$\boxed{x = 2, \quad y = 1}$$

---

## 6. Cramer's Rule Review

### For 3×3 Systems

$$x = \frac{|A_1|}{|A|}, \quad y = \frac{|A_2|}{|A|}, \quad z = \frac{|A_3|}{|A|}$$

where $A_i$ is A with column i replaced by b.

### Example 3

Solve:
$$\begin{cases} x + y + z = 6 \\ x - y + z = 2 \\ 2x + y - z = 1 \end{cases}$$

**Calculate |A|**:

$$|A| = \begin{vmatrix} 1 & 1 & 1 \\ 1 & -1 & 1 \\ 2 & 1 & -1 \end{vmatrix}$$

$= 1(1 - 1) - 1(-1 - 2) + 1(1 + 2) = 0 + 3 + 3 = 6$

**Calculate |A₁|** (replace column 1 with b):

$$|A_1| = \begin{vmatrix} 6 & 1 & 1 \\ 2 & -1 & 1 \\ 1 & 1 & -1 \end{vmatrix}$$

$= 6(1 - 1) - 1(-2 - 1) + 1(2 + 1) = 0 + 3 + 3 = 6$

**Calculate |A₂|**:

$$|A_2| = \begin{vmatrix} 1 & 6 & 1 \\ 1 & 2 & 1 \\ 2 & 1 & -1 \end{vmatrix}$$

$= 1(-2 - 1) - 6(-1 - 2) + 1(1 - 4) = -3 + 18 - 3 = 12$

**Calculate |A₃|**:

$$|A_3| = \begin{vmatrix} 1 & 1 & 6 \\ 1 & -1 & 2 \\ 2 & 1 & 1 \end{vmatrix}$$

$= 1(-1 - 2) - 1(1 - 4) + 6(1 + 2) = -3 + 3 + 18 = 18$

**Solution**:

$$x = \frac{6}{6} = 1, \quad y = \frac{12}{6} = 2, \quad z = \frac{18}{6} = 3$$

---

## 7. Inconsistent Systems

### Identifying Inconsistency

During row reduction, if you get a row of the form:

$$[0 \quad 0 \quad \cdots \quad 0 \quad | \quad k] \quad \text{where } k \neq 0$$

The system is **inconsistent** (no solution).

### Example 4: No Solution

Solve:
$$\begin{cases} x + y = 2 \\ 2x + 2y = 5 \end{cases}$$

**Reduce**:

$$\left[\begin{array}{cc|c} 1 & 1 & 2 \\ 2 & 2 & 5 \end{array}\right] \xrightarrow{R_2 - 2R_1} \left[\begin{array}{cc|c} 1 & 1 & 2 \\ 0 & 0 & 1 \end{array}\right]$$

Row 2 says $0 = 1$, which is impossible.

**No solution** (parallel lines)

---

## 8. Systems with Infinite Solutions

### Example 5: Parameterized Solution

Solve:
$$\begin{cases} x + 2y + z = 4 \\ 2x + 4y + 2z = 8 \\ x + y - z = 1 \end{cases}$$

**Reduce**:

$$\left[\begin{array}{ccc|c} 1 & 2 & 1 & 4 \\ 2 & 4 & 2 & 8 \\ 1 & 1 & -1 & 1 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - R_1$:

$$\left[\begin{array}{ccc|c} 1 & 2 & 1 & 4 \\ 0 & 0 & 0 & 0 \\ 0 & -1 & -2 & -3 \end{array}\right]$$

Swap $R_2 \leftrightarrow R_3$:

$$\left[\begin{array}{ccc|c} 1 & 2 & 1 & 4 \\ 0 & -1 & -2 & -3 \\ 0 & 0 & 0 & 0 \end{array}\right]$$

**Analysis**:
- rank(A) = 2
- rank([A|b]) = 2
- n = 3
- Consistent with 1 free variable

**Express solution**:

From row 2: $-y - 2z = -3 \Rightarrow y = 3 - 2z$

From row 1: $x + 2y + z = 4 \Rightarrow x = 4 - 2(3 - 2z) - z = -2 + 3z$

Let $z = t$:

$$\boxed{x = -2 + 3t, \quad y = 3 - 2t, \quad z = t}$$

**Verification** (for t = 0): $x = -2, y = 3, z = 0$
- $-2 + 6 + 0 = 4$ ✓
- $-4 + 12 + 0 = 8$ ✓
- $-2 + 3 - 0 = 1$ ✓

---

## 9. Comparison of Methods

```
┌─────────────────────────────────────────────────────────────────┐
│                METHOD SELECTION GUIDE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Is the system square (n equations, n unknowns)?               │
│           │                                                      │
│     ┌─────┴─────┐                                               │
│     │ YES       │ NO                                            │
│     ↓           ↓                                               │
│   Is |A| ≠ 0?   Use Gaussian                                    │
│     │           Elimination                                      │
│   ┌─┴──┐                                                        │
│   │YES │ NO                                                      │
│   ↓    ↓                                                        │
│  Use:  Use Gaussian                                              │
│  • Cramer's Rule (small)    Elimination                         │
│  • Matrix Inverse                                                │
│  • Gaussian Elimination                                          │
│                                                                  │
│   RECOMMENDATION: Gaussian Elimination works in ALL cases       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Solution Type Summary

| Condition | Solution Type |
|-----------|---------------|
| rank(A) = rank([A\|b]) = n | Unique |
| rank(A) = rank([A\|b]) < n | Infinite |
| rank(A) < rank([A\|b]) | None |

---

## ❓ Quick Revision Questions

1. **What makes a system non-homogeneous?**
   <details>
   <summary>Click for Answer</summary>
   At least one constant term is non-zero (b ≠ 0).
   </details>

2. **What row indicates inconsistency?**
   <details>
   <summary>Click for Answer</summary>
   [0 0 ... 0 | k] where k ≠ 0 (e.g., 0 = 5)
   </details>

3. **When can you use Cramer's rule?**
   <details>
   <summary>Click for Answer</summary>
   Square system with |A| ≠ 0 (unique solution exists).
   </details>

4. **What is the formula for x = A⁻¹b valid?**
   <details>
   <summary>Click for Answer</summary>
   When A is square and invertible (|A| ≠ 0).
   </details>

5. **3 equations, 3 unknowns, rank(A) = 2, rank([A|b]) = 2. Solution type?**
   <details>
   <summary>Click for Answer</summary>
   Infinite solutions (ranks equal but less than n).
   </details>

6. **Which method works for ALL systems?**
   <details>
   <summary>Click for Answer</summary>
   Gaussian elimination.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Homogeneous Systems](01-homogeneous-systems.md) | [Unit 7: Systems](./README.md) | [General Solutions →](03-general-solutions.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
