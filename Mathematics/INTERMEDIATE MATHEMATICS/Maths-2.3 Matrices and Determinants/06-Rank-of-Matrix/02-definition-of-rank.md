# Chapter 6.2: Definition and Calculation of Rank

[← Previous: Echelon Form](01-echelon-form.md) | [Back to README](../README.md) | [Next: Properties of Rank →](03-properties-of-rank.md)

---

## 📚 Chapter Overview

The rank of a matrix is a fundamental concept that captures the "essential size" of a matrix—how many dimensions the matrix actually uses. This chapter covers multiple equivalent definitions and methods for calculating rank.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define rank using multiple equivalent methods
- Calculate rank using row reduction
- Calculate rank using minors
- Understand the geometric meaning of rank

---

## 1. Definition of Rank

### Primary Definition

> The **rank** of a matrix A, denoted rank(A) or r(A) or ρ(A), is the number of non-zero rows in its row echelon form.

Equivalently:
- The number of pivot positions
- The number of leading 1s in RREF

### Alternative Definitions

> **Definition 2**: The rank equals the maximum number of linearly independent rows.

> **Definition 3**: The rank equals the maximum number of linearly independent columns.

> **Definition 4**: The rank equals the order of the largest non-zero minor (sub-determinant).

All four definitions give the same number!

---

## 2. Visual Understanding

```
┌─────────────────────────────────────────────────────────────────┐
│                    RANK = NUMBER OF PIVOTS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Original Matrix          Row Echelon Form                     │
│                                                                  │
│   ┌─────────────┐          ┌─────────────┐                      │
│   │ * * * * * * │          │ ■ * * * * * │ ← pivot              │
│   │ * * * * * * │    →     │ 0 ■ * * * * │ ← pivot              │
│   │ * * * * * * │          │ 0 0 0 ■ * * │ ← pivot              │
│   │ * * * * * * │          │ 0 0 0 0 0 0 │ ← zero row           │
│   └─────────────┘          └─────────────┘                      │
│                                                                  │
│   Rank = 3 (three non-zero rows = three pivots)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Method 1: Row Reduction

### Steps

1. Convert matrix to row echelon form (REF)
2. Count the number of non-zero rows

### Example 1

Find the rank of $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \\ 3 & 5 & 7 \end{bmatrix}$

**Row reduction**:

$R_2 \to R_2 - 2R_1$: $[0, 0, 0]$
$R_3 \to R_3 - 3R_1$: $[0, -1, -2]$

$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 0 & 0 \\ 0 & -1 & -2 \end{bmatrix}$$

$R_2 \leftrightarrow R_3$:

$$\begin{bmatrix} 1 & 2 & 3 \\ 0 & -1 & -2 \\ 0 & 0 & 0 \end{bmatrix}$$

**Rank = 2** (two non-zero rows)

### Example 2

Find the rank of $A = \begin{bmatrix} 1 & 2 & 1 \\ 2 & 1 & -1 \\ 1 & -1 & -2 \end{bmatrix}$

$R_2 \to R_2 - 2R_1$: $[0, -3, -3]$
$R_3 \to R_3 - R_1$: $[0, -3, -3]$

$$\begin{bmatrix} 1 & 2 & 1 \\ 0 & -3 & -3 \\ 0 & -3 & -3 \end{bmatrix}$$

$R_3 \to R_3 - R_2$: $[0, 0, 0]$

$$\begin{bmatrix} 1 & 2 & 1 \\ 0 & -3 & -3 \\ 0 & 0 & 0 \end{bmatrix}$$

**Rank = 2**

---

## 4. Method 2: Minor Method

### Definition of Minors

A **minor of order k** is the determinant of a k×k submatrix.

### Steps

1. Check if there exists a non-zero minor of order n (full matrix)
2. If yes, rank = n
3. If no, check minors of order n-1
4. Continue until you find a non-zero minor
5. Rank = order of largest non-zero minor

### Example 3

Find the rank of $A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$

**Check 3×3 minor (determinant of A)**:
Using Sarrus: $|A| = 45 + 84 + 96 - 105 - 48 - 72 = 225 - 225 = 0$

Since $|A| = 0$, rank < 3.

**Check 2×2 minors**:
$\begin{vmatrix} 1 & 2 \\ 4 & 5 \end{vmatrix} = 5 - 8 = -3 \neq 0$

Found a non-zero 2×2 minor!

**Rank = 2**

---

## 5. Comparison of Methods

```
┌─────────────────────────────────────────────────────────────────┐
│                 METHODS TO FIND RANK                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Method 1: Row Reduction                                        │
│   ────────────────────────                                      │
│   ✓ Systematic and reliable                                     │
│   ✓ Works for any size matrix                                   │
│   ✓ Also gives REF for other uses                               │
│   ○ More computation for large matrices                         │
│                                                                  │
│   Method 2: Minor/Determinant                                    │
│   ───────────────────────────                                   │
│   ✓ Good for small matrices                                     │
│   ✓ Sometimes faster if rank is obviously full                  │
│   ○ Must check many minors if rank is low                       │
│   ○ Less efficient for large matrices                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Special Cases

### Full Rank (Maximum Rank)

For an m×n matrix:
- Maximum possible rank = min(m, n)
- Matrix has **full row rank** if rank = m
- Matrix has **full column rank** if rank = n
- Square matrix is **full rank** if rank = n (equivalent to being invertible)

### Rank of Special Matrices

| Matrix Type | Rank |
|-------------|------|
| Zero matrix | 0 |
| n×n Identity matrix | n |
| n×n Invertible matrix | n |
| n×n Singular matrix | < n |
| Any matrix with all rows identical | 1 |
| Outer product $uv^T$ (u, v non-zero) | 1 |

---

## 7. Examples of Various Ranks

### Rank 0
$$A = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$$
Only the zero matrix has rank 0.

### Rank 1
$$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \\ 3 & 6 & 9 \end{bmatrix}$$
All rows are multiples of [1, 2, 3].

### Rank 2
$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$
$|A| = 0$, but has non-zero 2×2 minor.

### Full Rank (Rank 3)
$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 10 \end{bmatrix}$$
$|A| = 45 + 84 + 96 - 105 - 48 - 80 = 225 - 233 = -8 \neq 0$

---

## 8. Row Rank = Column Rank

### Important Theorem

> For any matrix A, the row rank equals the column rank.

This is why we simply call it "the rank" without specifying row or column.

### Intuition

- Row operations don't change the column space (span of columns)
- The number of pivot columns equals the number of pivot rows

---

## 9. Rank and Linear Dependence

### Connection

- If rank < number of rows → rows are linearly dependent
- If rank < number of columns → columns are linearly dependent
- If rank = n for n×n matrix → rows (and columns) are linearly independent

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 6 \end{bmatrix}$$

Rank = 1 (since row 2 = 3 × row 1)

The rows are linearly dependent: $R_2 = 3R_1$.

---

## 10. Rank and Determinant

### For Square Matrices

| Condition | Implication |
|-----------|-------------|
| $\|A\| \neq 0$ | rank(A) = n (full rank) |
| $\|A\| = 0$ | rank(A) < n |

### Quick Test

For a square matrix, first calculate the determinant:
- Non-zero determinant → full rank
- Zero determinant → need further analysis to find exact rank

---

## 📊 Summary Table

| Method | Description | Best For |
|--------|-------------|----------|
| Row Reduction | Count non-zero rows in REF | All cases |
| Minor Method | Find largest non-zero minor | Small matrices |
| Determinant | Non-zero det ⟹ full rank | Square matrices |

---

## ❓ Quick Revision Questions

1. **What is the rank of a matrix?**
   <details>
   <summary>Click for Answer</summary>
   The number of non-zero rows in its row echelon form, or equivalently, the number of linearly independent rows (or columns).
   </details>

2. **What is the maximum rank of a 3×5 matrix?**
   <details>
   <summary>Click for Answer</summary>
   3 (minimum of 3 and 5)
   </details>

3. **If a 4×4 matrix has |A| = 0, what can you say about its rank?**
   <details>
   <summary>Click for Answer</summary>
   rank(A) < 4 (less than full rank)
   </details>

4. **What is the rank of the identity matrix $I_n$?**
   <details>
   <summary>Click for Answer</summary>
   n
   </details>

5. **Find rank of $\begin{bmatrix} 1 & 2 \\ 2 & 4 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   Rank = 1 (row 2 = 2 × row 1)
   </details>

6. **Does row rank equal column rank?**
   <details>
   <summary>Click for Answer</summary>
   Yes, always.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Echelon Form](01-echelon-form.md) | [Unit 6: Rank](./README.md) | [Properties of Rank →](03-properties-of-rank.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
