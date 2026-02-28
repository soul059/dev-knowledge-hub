# Chapter 3.1: Determinant of 2×2 Matrix

[← Previous: Transpose of Matrix](../02-Matrix-Operations/05-transpose-of-matrix.md) | [Back to README](../README.md) | [Next: Determinant of 3×3 →](02-determinant-3x3.md)

---

## 📚 Chapter Overview

The determinant is a scalar value associated with square matrices that provides crucial information about the matrix, including whether it's invertible, the scaling factor for area/volume, and solutions to linear equations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Calculate the determinant of a 2×2 matrix
- Understand the geometric meaning of determinants
- Apply determinants to check invertibility
- Solve simple problems using 2×2 determinants

---

## 1. Definition of Determinant

### What is a Determinant?

> A **determinant** is a scalar value that can be computed from a square matrix and encodes certain properties of the matrix.

### Notation

For a matrix A, the determinant is written as:
- $\det(A)$
- $|A|$
- $\begin{vmatrix} \text{elements} \end{vmatrix}$

### Important Distinction

```
┌─────────────────────────────────────────────────────────────────┐
│                MATRIX vs DETERMINANT                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    MATRIX (an array)              DETERMINANT (a number)        │
│                                                                  │
│    A = [ a  b ]                   |A| = | a  b | = ad - bc      │
│        [ c  d ]                         | c  d |                │
│                                                                  │
│    Square brackets [ ]            Vertical bars | |             │
│    Represents an array            Represents a scalar value     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Formula for 2×2 Determinant

### The Formula

For a 2×2 matrix:

$$A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$$

The determinant is:

$$\det(A) = |A| = \begin{vmatrix} a & b \\ c & d \end{vmatrix} = ad - bc$$

### Visual Memory Aid

```
        ┌─────────────┐
        │  a       b  │
        │    ╲   ╱    │
        │      ╳      │         det = (a × d) - (b × c)
        │    ╱   ╲    │              = ad - bc
        │  c       d  │
        └─────────────┘
        
        Main diagonal: a × d  (positive)
        Anti-diagonal: b × c  (negative)
        
        ╲                     ╱
          ╲ (ad: positive)  ╱ (bc: negative)
            ╲             ╱
              ╲         ╱
                ╳
              ╱         ╲
            ╱             ╲
          ╱                 ╲
```

### Pattern Recognition

```
        | a  b |
        | c  d |
        
        Step 1: Multiply main diagonal (↘): a × d
        Step 2: Multiply anti-diagonal (↙): b × c
        Step 3: Subtract: ad - bc
```

---

## 3. Worked Examples

### Example 1: Basic Calculation

$$\begin{vmatrix} 3 & 2 \\ 1 & 4 \end{vmatrix}$$

**Solution**:
$$= (3)(4) - (2)(1) = 12 - 2 = 10$$

### Example 2: With Negative Numbers

$$\begin{vmatrix} 5 & -3 \\ 2 & 4 \end{vmatrix}$$

**Solution**:
$$= (5)(4) - (-3)(2) = 20 - (-6) = 20 + 6 = 26$$

### Example 3: Zero Determinant

$$\begin{vmatrix} 2 & 4 \\ 1 & 2 \end{vmatrix}$$

**Solution**:
$$= (2)(2) - (4)(1) = 4 - 4 = 0$$

### Example 4: Negative Determinant

$$\begin{vmatrix} 1 & 5 \\ 3 & 2 \end{vmatrix}$$

**Solution**:
$$= (1)(2) - (5)(3) = 2 - 15 = -13$$

### Example 5: With Fractions

$$\begin{vmatrix} \frac{1}{2} & \frac{1}{3} \\ \frac{2}{3} & \frac{1}{4} \end{vmatrix}$$

**Solution**:
$$= \left(\frac{1}{2}\right)\left(\frac{1}{4}\right) - \left(\frac{1}{3}\right)\left(\frac{2}{3}\right)$$
$$= \frac{1}{8} - \frac{2}{9} = \frac{9}{72} - \frac{16}{72} = -\frac{7}{72}$$

### Example 6: With Variables

$$\begin{vmatrix} x & 3 \\ 2 & x \end{vmatrix} = 0$$

**Solution**:
$$x^2 - 6 = 0$$
$$x^2 = 6$$
$$x = \pm\sqrt{6}$$

---

## 4. Special Cases

### Identity Matrix

$$\begin{vmatrix} 1 & 0 \\ 0 & 1 \end{vmatrix} = (1)(1) - (0)(0) = 1$$

### Diagonal Matrix

$$\begin{vmatrix} a & 0 \\ 0 & d \end{vmatrix} = ad$$

**The determinant of a diagonal matrix = product of diagonal elements**

### Upper Triangular

$$\begin{vmatrix} a & b \\ 0 & d \end{vmatrix} = ad$$

### Lower Triangular

$$\begin{vmatrix} a & 0 \\ c & d \end{vmatrix} = ad$$

### Zero Row/Column

$$\begin{vmatrix} 0 & 0 \\ c & d \end{vmatrix} = 0$$

**If any row or column is all zeros, determinant = 0**

---

## 5. Geometric Interpretation

### Area of Parallelogram

The absolute value of the determinant equals the area of the parallelogram formed by the column vectors.

```
        For A = [ a  b ]
                [ c  d ]
        
        Vectors: v₁ = (a, c) and v₂ = (b, d)
        
                    d │      •(b,d)
                      │     /│
                      │    / │
                      │   /  │         Area = |ad - bc|
                      │  /   │
                    c │ •────┼────────
                      │(a,c) │
                      │      │
                      └──────┴──────────
                             a    b
```

### Example: Area Calculation

Find the area of parallelogram with sides represented by vectors (3, 1) and (2, 4):

$$\text{Area} = \left|\begin{vmatrix} 3 & 2 \\ 1 & 4 \end{vmatrix}\right| = |12 - 2| = |10| = 10 \text{ square units}$$

---

## 6. Determinant and Invertibility

### Key Theorem

> **A matrix A is invertible (non-singular) if and only if $\det(A) \neq 0$**

### Classification

```
┌─────────────────────────────────────────────────────────────────┐
│                    DETERMINANT VALUES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    det(A) ≠ 0                    det(A) = 0                     │
│    ──────────                    ──────────                      │
│                                                                  │
│    • Matrix is INVERTIBLE        • Matrix is SINGULAR           │
│    • Matrix is NON-SINGULAR      • Matrix is NOT invertible     │
│    • A⁻¹ exists                  • A⁻¹ does NOT exist           │
│    • Columns are linearly        • Columns are linearly         │
│      independent                   dependent                     │
│    • Unique solution to Ax = b   • No unique solution           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example

$$A = \begin{bmatrix} 2 & 4 \\ 1 & 2 \end{bmatrix}$$

$\det(A) = 4 - 4 = 0$

Matrix A is **singular** (not invertible).

Notice: Row 1 = 2 × Row 2 (linearly dependent)

---

## 7. Quick Calculation Patterns

### Pattern 1: Swap Rows/Columns

If we swap rows (or columns), the determinant changes sign:

$$\begin{vmatrix} a & b \\ c & d \end{vmatrix} = -\begin{vmatrix} c & d \\ a & b \end{vmatrix}$$

### Pattern 2: Scalar Multiple of Row

If one row is multiplied by k, determinant is multiplied by k:

$$\begin{vmatrix} ka & kb \\ c & d \end{vmatrix} = k\begin{vmatrix} a & b \\ c & d \end{vmatrix}$$

### Pattern 3: Same Rows

If two rows are identical, determinant = 0:

$$\begin{vmatrix} a & b \\ a & b \end{vmatrix} = ab - ba = 0$$

### Pattern 4: Proportional Rows

If rows are proportional, determinant = 0:

$$\begin{vmatrix} a & b \\ ka & kb \end{vmatrix} = akb - bka = 0$$

---

## 8. Solving Equations with Determinants

### Finding Unknown Values

**Problem**: Find x if $\begin{vmatrix} x+1 & 2 \\ 3 & x-1 \end{vmatrix} = 5$

**Solution**:
$$(x+1)(x-1) - (2)(3) = 5$$
$$x^2 - 1 - 6 = 5$$
$$x^2 = 12$$
$$x = \pm 2\sqrt{3}$$

### Finding When Determinant is Zero

**Problem**: For what value of k is $\begin{vmatrix} 2 & k \\ 3 & 6 \end{vmatrix} = 0$?

**Solution**:
$$2(6) - k(3) = 0$$
$$12 - 3k = 0$$
$$k = 4$$

---

## 9. Calculation Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│              2×2 DETERMINANT CALCULATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │   Given 2×2     │                          │
│                    │   Matrix A      │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│                             ▼                                    │
│                    ┌─────────────────┐                          │
│                    │ Identify:       │                          │
│                    │ a, b, c, d      │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│              ┌──────────────┼──────────────┐                    │
│              │              │              │                    │
│              ▼              ▼              ▼                    │
│       ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│       │ Main diag│   │Anti diag │   │ Subtract │               │
│       │  a × d   │   │  b × c   │   │ ad - bc  │               │
│       └──────────┘   └──────────┘   └──────────┘               │
│              │              │              │                    │
│              └──────────────┴──────────────┘                    │
│                             │                                    │
│                             ▼                                    │
│                    ┌─────────────────┐                          │
│                    │   det(A) =      │                          │
│                    │   ad - bc       │                          │
│                    └─────────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Formula/Rule | Example |
|---------|--------------|---------|
| 2×2 Determinant | $ad - bc$ | $\begin{vmatrix} 3 & 2 \\ 1 & 4 \end{vmatrix} = 10$ |
| Geometric meaning | Area of parallelogram | $\|ad - bc\|$ = area |
| det(I) | 1 | Identity determinant |
| det(diagonal) | Product of diagonal | $\begin{vmatrix} 3 & 0 \\ 0 & 5 \end{vmatrix} = 15$ |
| Invertible | det ≠ 0 | Non-zero determinant |
| Singular | det = 0 | Zero determinant |

---

## ❓ Quick Revision Questions

1. **Calculate $\begin{vmatrix} 4 & 3 \\ 2 & 5 \end{vmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $(4)(5) - (3)(2) = 20 - 6 = 14$
   </details>

2. **Is the matrix $\begin{bmatrix} 6 & 3 \\ 2 & 1 \end{bmatrix}$ invertible?**
   <details>
   <summary>Click for Answer</summary>
   det = $6(1) - 3(2) = 6 - 6 = 0$
   No, the matrix is NOT invertible (singular).
   </details>

3. **Find x if $\begin{vmatrix} x & 4 \\ 3 & 6 \end{vmatrix} = 0$**
   <details>
   <summary>Click for Answer</summary>
   $6x - 12 = 0 \Rightarrow x = 2$
   </details>

4. **What is the determinant of $\begin{bmatrix} 5 & 0 \\ 0 & 7 \end{bmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   $5 \times 7 - 0 \times 0 = 35$ (product of diagonal elements)
   </details>

5. **If det(A) = 3, what is det(2A) for a 2×2 matrix?**
   <details>
   <summary>Click for Answer</summary>
   det(2A) = $2^2 \times$ det(A) = $4 \times 3 = 12$
   (For n×n matrix, det(kA) = $k^n$ det(A))
   </details>

6. **Calculate $\begin{vmatrix} -2 & 5 \\ 3 & -4 \end{vmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $(-2)(-4) - (5)(3) = 8 - 15 = -7$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Transpose of Matrix](../02-Matrix-Operations/05-transpose-of-matrix.md) | [Unit 3: Determinants](./README.md) | [Determinant of 3×3 →](02-determinant-3x3.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
