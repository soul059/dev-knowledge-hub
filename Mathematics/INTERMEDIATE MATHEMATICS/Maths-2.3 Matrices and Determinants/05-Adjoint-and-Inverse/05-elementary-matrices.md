# Chapter 5.5: Elementary Matrices

[← Previous: Solving Equations](04-solving-equations.md) | [Back to README](../README.md) | [Next: Echelon Form →](../06-Rank-of-Matrix/01-echelon-form.md)

---

## 📚 Chapter Overview

Elementary matrices correspond to elementary row operations and provide a theoretical foundation for understanding matrix inverses, LU decomposition, and the connection between row operations and matrix multiplication.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define the three types of elementary matrices
- Relate elementary matrices to row operations
- Express inverses as products of elementary matrices
- Understand the role of elementary matrices in matrix theory

---

## 1. What are Elementary Matrices?

### Definition

> An **elementary matrix** is a matrix obtained by performing exactly one elementary row operation on the identity matrix.

### The Three Types

```
┌─────────────────────────────────────────────────────────────────┐
│              THREE TYPES OF ELEMENTARY MATRICES                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Type I:   Row Interchange (Swap)                              │
│   ────────────────────────────────                              │
│   Swap two rows of I                                            │
│   Example: E swaps rows 1 and 2                                 │
│                                                                  │
│   Type II:  Row Scaling (Multiply)                              │
│   ────────────────────────────────                              │
│   Multiply a row of I by a non-zero scalar k                    │
│   Example: E multiplies row 2 by k                              │
│                                                                  │
│   Type III: Row Addition                                        │
│   ──────────────────────                                        │
│   Add k times one row to another row of I                       │
│   Example: E adds k × (row 1) to row 2                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Type I: Row Interchange Matrix

### Construction

Start with I, swap rows i and j.

### Example (3×3)

Swap rows 1 and 2:
$$E_{12} = \begin{bmatrix} 0 & 1 & 0 \\ 1 & 0 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

### Effect on Matrix A

If we multiply $E_{12} \cdot A$, it swaps rows 1 and 2 of A.

### Example

$$E_{12} \cdot \begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} 0 & 1 & 0 \\ 1 & 0 & 0 \\ 0 & 0 & 1 \end{bmatrix}\begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} c & d \\ a & b \\ e & f \end{bmatrix}$$

### Properties of Type I

- $|E| = -1$ (determinant is -1)
- $E^{-1} = E$ (self-inverse, since swapping twice returns to original)
- $E^T = E$ (symmetric)

---

## 3. Type II: Row Scaling Matrix

### Construction

Start with I, multiply row i by scalar k (k ≠ 0).

### Example (3×3)

Multiply row 2 by 5:
$$E_2(5) = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

### Effect on Matrix A

If we multiply $E_2(5) \cdot A$, it multiplies row 2 of A by 5.

### Example

$$E_2(5) \cdot \begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 1 \end{bmatrix}\begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} a & b \\ 5c & 5d \\ e & f \end{bmatrix}$$

### Properties of Type II

- $|E| = k$ (determinant equals the scalar)
- $E^{-1} = E_i(1/k)$ (inverse scales by reciprocal)
- $E$ is diagonal

---

## 4. Type III: Row Addition Matrix

### Construction

Start with I, add k times row j to row i.

### Example (3×3)

Add 3 × (row 1) to row 2:
$$E_{21}(3) = \begin{bmatrix} 1 & 0 & 0 \\ 3 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

### Effect on Matrix A

If we multiply $E_{21}(3) \cdot A$, it adds 3 times row 1 to row 2 of A.

### Example

$$E_{21}(3) \cdot \begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} 1 & 0 & 0 \\ 3 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}\begin{bmatrix} a & b \\ c & d \\ e & f \end{bmatrix} = \begin{bmatrix} a & b \\ c+3a & d+3b \\ e & f \end{bmatrix}$$

### Properties of Type III

- $|E| = 1$ (determinant is 1)
- $E^{-1} = E_{ij}(-k)$ (inverse uses negative of k)
- $E$ is triangular (lower or upper depending on i vs j)

---

## 5. Summary of Elementary Matrices

```
┌─────────────────────────────────────────────────────────────────┐
│              ELEMENTARY MATRIX PROPERTIES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Type      │ Operation      │ |E|    │ E⁻¹                     │
│   ──────────┼────────────────┼────────┼────────────────────     │
│   I (swap)  │ Rᵢ ↔ Rⱼ       │  -1    │ E (same matrix)         │
│   II (scale)│ Rᵢ → kRᵢ      │   k    │ Eᵢ(1/k)                 │
│   III (add) │ Rᵢ → Rᵢ + kRⱼ │   1    │ Eᵢⱼ(-k)                 │
│                                                                  │
│   Key Insight: Every elementary matrix is invertible!           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Row Operations as Matrix Multiplication

### The Key Connection

> Performing a row operation on matrix A is equivalent to multiplying A by the corresponding elementary matrix on the **left**.

### Example

To perform "$R_2 \to R_2 - 2R_1$" on A:

1. Create E by doing the same operation on I
2. Compute EA

$$E = \begin{bmatrix} 1 & 0 \\ -2 & 1 \end{bmatrix}$$

$$EA = \begin{bmatrix} 1 & 0 \\ -2 & 1 \end{bmatrix}\begin{bmatrix} a & b \\ c & d \end{bmatrix} = \begin{bmatrix} a & b \\ c-2a & d-2b \end{bmatrix}$$

---

## 7. Column Operations

### Right Multiplication

> Performing a column operation on A is equivalent to multiplying A by an elementary matrix on the **right**.

### Example

To swap columns 1 and 2 of A:
$$AE_{12} = \begin{bmatrix} a & b \\ c & d \end{bmatrix}\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix} = \begin{bmatrix} b & a \\ d & c \end{bmatrix}$$

---

## 8. Expressing Inverse Using Elementary Matrices

### Theorem

> If A is invertible, then A can be written as a product of elementary matrices:
> $$A = E_1 E_2 \cdots E_k$$

### Finding $A^{-1}$

Since each $E_i$ is invertible:
$$A^{-1} = E_k^{-1} \cdots E_2^{-1} E_1^{-1}$$

### The Gauss-Jordan Process

$$[A | I] \xrightarrow{E_k \cdots E_2 E_1} [I | A^{-1}]$$

The sequence of elementary matrices that transforms A to I, when applied to I, gives $A^{-1}$.

---

## 9. Example: Finding Inverse via Elementary Matrices

### Problem

Find $A^{-1}$ for $A = \begin{bmatrix} 1 & 2 \\ 3 & 7 \end{bmatrix}$ using elementary matrices.

### Solution

**Operation 1**: $R_2 \to R_2 - 3R_1$
$$E_1 = \begin{bmatrix} 1 & 0 \\ -3 & 1 \end{bmatrix}$$

$$E_1 A = \begin{bmatrix} 1 & 2 \\ 0 & 1 \end{bmatrix}$$

**Operation 2**: $R_1 \to R_1 - 2R_2$
$$E_2 = \begin{bmatrix} 1 & -2 \\ 0 & 1 \end{bmatrix}$$

$$E_2 E_1 A = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} = I$$

**Finding $A^{-1}$**:
$$E_2 E_1 A = I$$
$$A^{-1} = E_2 E_1 = \begin{bmatrix} 1 & -2 \\ 0 & 1 \end{bmatrix}\begin{bmatrix} 1 & 0 \\ -3 & 1 \end{bmatrix} = \begin{bmatrix} 7 & -2 \\ -3 & 1 \end{bmatrix}$$

**Verify**: $|A| = 7 - 6 = 1$, and $A^{-1} = \frac{1}{1}\begin{bmatrix} 7 & -2 \\ -3 & 1 \end{bmatrix}$ ✓

---

## 10. LU Decomposition (Introduction)

### Concept

Any invertible matrix can be decomposed as:
$$A = LU$$

Where:
- L = Lower triangular matrix (product of Type III elementary matrices)
- U = Upper triangular matrix (row echelon form)

### Example

$$A = \begin{bmatrix} 2 & 1 \\ 6 & 4 \end{bmatrix}$$

Row operation: $R_2 \to R_2 - 3R_1$
$$U = \begin{bmatrix} 2 & 1 \\ 0 & 1 \end{bmatrix}$$

The L matrix stores the multipliers:
$$L = \begin{bmatrix} 1 & 0 \\ 3 & 1 \end{bmatrix}$$

Verify: $LU = \begin{bmatrix} 1 & 0 \\ 3 & 1 \end{bmatrix}\begin{bmatrix} 2 & 1 \\ 0 & 1 \end{bmatrix} = \begin{bmatrix} 2 & 1 \\ 6 & 4 \end{bmatrix} = A$ ✓

---

## 📊 Summary Table

| Type | Row Operation | Elementary Matrix | Determinant | Inverse |
|------|--------------|-------------------|-------------|---------|
| I | $R_i \leftrightarrow R_j$ | Swap rows in I | -1 | E itself |
| II | $R_i \to kR_i$ | Scale diagonal entry | k | Scale by 1/k |
| III | $R_i \to R_i + kR_j$ | Add k in position (i,j) | 1 | Add -k |

---

## ❓ Quick Revision Questions

1. **What is an elementary matrix?**
   <details>
   <summary>Click for Answer</summary>
   A matrix obtained by performing one elementary row operation on the identity matrix.
   </details>

2. **How do you perform a row operation using matrices?**
   <details>
   <summary>Click for Answer</summary>
   Multiply the matrix A on the left by the corresponding elementary matrix E: EA
   </details>

3. **What is the determinant of a row-swap elementary matrix?**
   <details>
   <summary>Click for Answer</summary>
   -1
   </details>

4. **What is the inverse of a Type I (swap) elementary matrix?**
   <details>
   <summary>Click for Answer</summary>
   The matrix itself (swapping twice returns to original)
   </details>

5. **If E adds 5 times row 1 to row 2, what does $E^{-1}$ do?**
   <details>
   <summary>Click for Answer</summary>
   $E^{-1}$ subtracts 5 times row 1 from row 2 (uses -5 instead of 5)
   </details>

6. **How are A and $A^{-1}$ related through elementary matrices?**
   <details>
   <summary>Click for Answer</summary>
   If $E_k \cdots E_1 A = I$, then $A^{-1} = E_k \cdots E_1$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Solving Equations](04-solving-equations.md) | [Unit 5: Adjoint & Inverse](./README.md) | [Echelon Form →](../06-Rank-of-Matrix/01-echelon-form.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
