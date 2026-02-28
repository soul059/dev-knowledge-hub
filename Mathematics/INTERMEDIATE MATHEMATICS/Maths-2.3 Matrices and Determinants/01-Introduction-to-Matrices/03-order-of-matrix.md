# Chapter 1.3: Order of a Matrix

[← Previous: Types of Matrices](02-types-of-matrices.md) | [Back to README](../README.md) | [Next: Equality of Matrices →](04-equality-of-matrices.md)

---

## 📚 Chapter Overview

The **order** (or **dimension**) of a matrix is a fundamental concept that determines what operations can be performed on matrices. Understanding order is essential for matrix arithmetic and applications.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Determine the order of any matrix
- Count rows and columns correctly
- Understand how order affects matrix operations
- Calculate the number of elements from order

---

## 1. Definition of Order

### Formal Definition

> The **order** of a matrix is written as **m × n** (read as "m by n"), where:
> - **m** = number of rows
> - **n** = number of columns

### General Notation

$$A_{m \times n} = \begin{bmatrix} a_{11} & a_{12} & \cdots & a_{1n} \\ a_{21} & a_{22} & \cdots & a_{2n} \\ \vdots & \vdots & \ddots & \vdots \\ a_{m1} & a_{m2} & \cdots & a_{mn} \end{bmatrix}$$

### Memory Trick

```
         ORDER = ROWS × COLUMNS
         
              m  ×  n
              ↓     ↓
           Rows   Columns
           
         "RC" = "Rows Come first"
         Think: "RC Cola" or "Remote Control"
```

---

## 2. Counting Rows and Columns

### Visual Guide

```
                    ← ─ ─ ─ n columns ─ ─ ─ →
                    
         ┌    ┌─────────────────────────────┐
         │    │  a₁₁   a₁₂   a₁₃   ...  a₁ₙ │  ← Row 1
         │    │                              │
         │    │  a₂₁   a₂₂   a₂₃   ...  a₂ₙ │  ← Row 2
         m    │                              │
       rows   │  a₃₁   a₃₂   a₃₃   ...  a₃ₙ │  ← Row 3
         │    │   ⋮     ⋮     ⋮     ⋱    ⋮   │
         │    │                              │
         │    │  aₘ₁   aₘ₂   aₘ₃   ...  aₘₙ │  ← Row m
         ↓    └─────────────────────────────┘
                 ↑     ↑     ↑          ↑
                 C₁    C₂    C₃         Cₙ
```

### Examples

**Example 1**: Find the order of matrix A

$$A = \begin{bmatrix} 2 & 5 & 7 \\ 3 & 8 & 1 \end{bmatrix}$$

```
        ┌─────────────┐
        │  2   5   7  │  ← Row 1
        │  3   8   1  │  ← Row 2
        └─────────────┘
           ↑   ↑   ↑
          C1  C2  C3

        Rows = 2, Columns = 3
        Order = 2 × 3
```

**Example 2**: Find the order of matrix B

$$B = \begin{bmatrix} 1 \\ 4 \\ 7 \\ 2 \end{bmatrix}$$

```
        ┌───┐
        │ 1 │  ← Row 1
        │ 4 │  ← Row 2
        │ 7 │  ← Row 3
        │ 2 │  ← Row 4
        └───┘
          ↑
         C1

        Rows = 4, Columns = 1
        Order = 4 × 1 (Column matrix)
```

**Example 3**: Find the order of matrix C

$$C = \begin{bmatrix} 5 & 2 & 8 & 1 & 9 \end{bmatrix}$$

```
        ┌─────────────────┐
        │  5  2  8  1  9  │  ← Row 1
        └─────────────────┘
           ↑  ↑  ↑  ↑  ↑
          C1 C2 C3 C4 C5

        Rows = 1, Columns = 5
        Order = 1 × 5 (Row matrix)
```

---

## 3. Number of Elements

### Formula

> **Total elements = m × n** (rows times columns)

### Element Count Table

| Order | Rows (m) | Columns (n) | Total Elements |
|-------|----------|-------------|----------------|
| 2×2 | 2 | 2 | 4 |
| 2×3 | 2 | 3 | 6 |
| 3×2 | 3 | 2 | 6 |
| 3×3 | 3 | 3 | 9 |
| 3×4 | 3 | 4 | 12 |
| 4×4 | 4 | 4 | 16 |
| m×n | m | n | m·n |

### Visual Representation

```
        2×3 Matrix (6 elements)        3×2 Matrix (6 elements)
        
        ┌───────────────┐              ┌─────────┐
        │ [1] [2] [3]   │              │ [1] [2] │
        │ [4] [5] [6]   │              │ [3] [4] │
        └───────────────┘              │ [5] [6] │
                                       └─────────┘
        
        Same number of elements, different orders!
```

---

## 4. Order and Matrix Operations

### Compatibility Rules

The order of matrices determines which operations are possible:

```
┌─────────────────────────────────────────────────────────────────┐
│                    OPERATION COMPATIBILITY                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADDITION / SUBTRACTION:                                        │
│  ─────────────────────────                                      │
│  A ± B is possible only if A and B have SAME ORDER              │
│                                                                  │
│      A         B          A + B                                 │
│    (m×n)  +  (m×n)   =   (m×n)    ✓                            │
│    (2×3)  +  (2×3)   =   (2×3)    ✓                            │
│    (2×3)  +  (3×2)   =    ???     ✗ Not possible               │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MULTIPLICATION:                                                 │
│  ─────────────────                                              │
│  A × B is possible only if:                                     │
│  Columns of A = Rows of B                                       │
│                                                                  │
│      A         B          A × B                                 │
│    (m×n)  ×  (n×p)   =   (m×p)                                 │
│    (2×3)  ×  (3×4)   =   (2×4)    ✓                            │
│    (2×3)  ×  (2×3)   =    ???     ✗ Not possible               │
│          ↑       ↑                                              │
│          3   ≠   2                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multiplication Order Rule

```
        Matrix A        Matrix B         Result
        (m × n)    ×    (n × p)    =    (m × p)
             ↑          ↑
             └──────────┘
              Must match!
              
        "Inner dimensions must match"
        "Outer dimensions give result"
```

### Example: Checking Compatibility

Given:
- A is 3×4
- B is 4×2
- C is 3×2
- D is 2×4

| Operation | Dimensions Check | Result |
|-----------|------------------|--------|
| A + B | 3×4 ≠ 4×2 | ✗ Not possible |
| A + C | 3×4 ≠ 3×2 | ✗ Not possible |
| A × B | 3×**4** × **4**×2 = 3×2 | ✓ Possible |
| B × A | 4×**2** × **3**×4 | ✗ Not possible (2≠3) |
| C + A×B | 3×2 + 3×2 = 3×2 | ✓ Possible |
| D × A | 2×**4** × **3**×4 | ✗ Not possible (4≠3) |
| A × D | 3×**4** × **2**×4 | ✗ Not possible (4≠2) |

---

## 5. Finding Possible Orders

### Problem Type: Given Total Elements

**Question**: What are the possible orders of a matrix with 12 elements?

**Solution**: Find all factor pairs of 12

| Factors | Order | Type |
|---------|-------|------|
| 1 × 12 | 1×12 | Row matrix |
| 2 × 6 | 2×6 | Rectangular |
| 3 × 4 | 3×4 | Rectangular |
| 4 × 3 | 4×3 | Rectangular |
| 6 × 2 | 6×2 | Rectangular |
| 12 × 1 | 12×1 | Column matrix |

### Visualization

```
        12 elements can be arranged as:
        
        1×12: [ • • • • • • • • • • • • ]
        
        2×6:  [ • • • • • • ]
              [ • • • • • • ]
        
        3×4:  [ • • • • ]
              [ • • • • ]
              [ • • • • ]
        
        4×3:  [ • • • ]
              [ • • • ]
              [ • • • ]
              [ • • • ]
        
        6×2:  [ • • ]
              [ • • ]
              [ • • ]
              [ • • ]
              [ • • ]
              [ • • ]
        
        12×1: [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
              [ • ]
```

---

## 6. Special Order Cases

### Square Matrices

When m = n, the matrix is square:

```
        1×1     2×2         3×3             4×4
        
        [a]   [a  b]    [a  b  c]    [a  b  c  d]
              [c  d]    [d  e  f]    [e  f  g  h]
                        [g  h  i]    [i  j  k  l]
                                     [m  n  o  p]
```

### Row and Column Matrices

```
        Row Matrix (1×n)              Column Matrix (m×1)
        
        [a₁  a₂  a₃  ...  aₙ]         [a₁]
                                      [a₂]
                                      [a₃]
                                       ⋮
                                      [aₘ]
```

### Singleton Matrix

A 1×1 matrix contains only one element:

$$A_{1×1} = [a_{11}]$$

This is essentially a scalar wrapped in matrix notation.

---

## 7. Practical Applications

### Data Organization Example

**Scenario**: A store tracks sales of 4 products across 3 months.

```
        Products → 4 columns
        Months ↓  3 rows
        
                     Prod1  Prod2  Prod3  Prod4
                   ┌─────────────────────────────┐
        January    │  150    200    180    220   │
        February   │  175    210    195    240   │
        March      │  160    230    185    250   │
                   └─────────────────────────────┘
        
        Order: 3×4 (3 rows, 4 columns)
        Total data points: 12
```

### Image Representation

```
        A grayscale image can be represented as a matrix:
        
        Resolution: 1920 × 1080 pixels
        Matrix order: 1080 × 1920
        Total pixels: 2,073,600
        
        Each element = pixel intensity (0-255)
```

---

## 8. Order Determination Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│              HOW TO DETERMINE MATRIX ORDER                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │   Given Matrix  │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│                             ▼                                    │
│              ┌──────────────────────────────┐                   │
│              │  Count horizontal lines of   │                   │
│              │  elements (ROWS)             │                   │
│              └──────────────┬───────────────┘                   │
│                             │                                    │
│                             ▼                                    │
│              ┌──────────────────────────────┐                   │
│              │  Count vertical lines of     │                   │
│              │  elements (COLUMNS)          │                   │
│              └──────────────┬───────────────┘                   │
│                             │                                    │
│                             ▼                                    │
│              ┌──────────────────────────────┐                   │
│              │  Order = Rows × Columns      │                   │
│              │         = m × n              │                   │
│              └──────────────┬───────────────┘                   │
│                             │                                    │
│                             ▼                                    │
│         ┌───────────────────┼───────────────────┐               │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│    ┌─────────┐        ┌──────────┐        ┌──────────┐         │
│    │  m = n  │        │  m = 1   │        │  n = 1   │         │
│    │ Square  │        │   Row    │        │  Column  │         │
│    │ Matrix  │        │  Matrix  │        │  Matrix  │         │
│    └─────────┘        └──────────┘        └──────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Formula/Rule | Example |
|---------|--------------|---------|
| Order notation | m × n | 3 × 4 |
| m represents | Number of rows | 3 rows |
| n represents | Number of columns | 4 columns |
| Total elements | m × n | 3 × 4 = 12 |
| Square matrix | m = n | 3 × 3 |
| Row matrix | m = 1 | 1 × 5 |
| Column matrix | n = 1 | 5 × 1 |
| Addition compatible | Same order | (2×3) + (2×3) |
| Multiplication compatible | Inner dimensions match | (2×3) × (3×4) |
| Product order | Outer dimensions | (2×3) × (3×4) = (2×4) |

---

## ❓ Quick Revision Questions

1. **What is the order of $\begin{bmatrix} 1 & 2 & 3 & 4 \\ 5 & 6 & 7 & 8 \\ 9 & 10 & 11 & 12 \end{bmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   Order is 3 × 4 (3 rows, 4 columns). Total elements = 12.
   </details>

2. **If A is 3×5 and B is 5×2, what is the order of AB?**
   <details>
   <summary>Click for Answer</summary>
   AB is 3×2. The inner dimensions (5 and 5) match and cancel, leaving outer dimensions (3 and 2).
   </details>

3. **Can we add a 2×3 matrix to a 3×2 matrix?**
   <details>
   <summary>Click for Answer</summary>
   No. Matrix addition requires both matrices to have the same order. 2×3 ≠ 3×2.
   </details>

4. **How many matrices of different orders can have exactly 6 elements?**
   <details>
   <summary>Click for Answer</summary>
   4 different orders: 1×6, 2×3, 3×2, and 6×1. (Factor pairs of 6)
   </details>

5. **If matrix P has order 4×3 and matrix Q has order 3×4, find the orders of PQ and QP.**
   <details>
   <summary>Click for Answer</summary>
   PQ: (4×3)(3×4) = 4×4 (square matrix)
   QP: (3×4)(4×3) = 3×3 (square matrix)
   Both products exist but have different orders!
   </details>

6. **A matrix has 24 elements. List all possible orders if it must be a square matrix.**
   <details>
   <summary>Click for Answer</summary>
   For a square matrix, we need m = n, so m² = 24. Since 24 is not a perfect square, there is NO possible square matrix with exactly 24 elements.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Types of Matrices](02-types-of-matrices.md) | [Unit 1: Introduction](./README.md) | [Equality of Matrices →](04-equality-of-matrices.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
