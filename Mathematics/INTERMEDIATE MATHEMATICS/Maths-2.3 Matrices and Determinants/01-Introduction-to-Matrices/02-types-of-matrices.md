# Chapter 1.2: Types of Matrices

[← Previous: Definition and Notation](01-definition-and-notation.md) | [Back to README](../README.md) | [Next: Order of Matrix →](03-order-of-matrix.md)

---

## 📚 Chapter Overview

Matrices come in various types based on their structure, elements, and special properties. Understanding these types is essential for recognizing when specific matrix operations and theorems apply.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Classify matrices based on their structure
- Identify special matrices by their properties
- Understand the relationships between different matrix types

---

## 1. Classification Overview

```
                           ┌─────────────────┐
                           │    MATRICES     │
                           └────────┬────────┘
                                    │
           ┌────────────────────────┼────────────────────────┐
           │                        │                        │
    ┌──────▼──────┐         ┌───────▼───────┐        ┌───────▼───────┐
    │   By Size   │         │ By Structure  │        │ By Properties │
    └──────┬──────┘         └───────┬───────┘        └───────┬───────┘
           │                        │                        │
    ┌──────┴──────┐         ┌───────┴───────┐        ┌───────┴───────┐
    │• Row Matrix │         │• Diagonal     │        │• Symmetric    │
    │• Column     │         │• Scalar       │        │• Skew-Symmetric│
    │• Square     │         │• Identity     │        │• Orthogonal   │
    │• Rectangular│         │• Triangular   │        │• Singular     │
    └─────────────┘         │• Null/Zero    │        │• Non-singular │
                            └───────────────┘        └───────────────┘
```

---

## 2. Row Matrix

### Definition

> A **row matrix** has only **one row** and can have any number of columns.

### General Form

$$A = \begin{bmatrix} a_1 & a_2 & a_3 & \cdots & a_n \end{bmatrix}_{1 \times n}$$

### Examples

```
        1×3 Row Matrix          1×4 Row Matrix          1×5 Row Matrix
        
        [ 2  5  7 ]            [ 1  0  3  8 ]         [ a  b  c  d  e ]
```

### Properties
- Order: $1 \times n$
- Contains $n$ elements
- Also called a **row vector**

---

## 3. Column Matrix

### Definition

> A **column matrix** has only **one column** and can have any number of rows.

### General Form

$$B = \begin{bmatrix} b_1 \\ b_2 \\ b_3 \\ \vdots \\ b_m \end{bmatrix}_{m \times 1}$$

### Examples

```
        3×1 Column Matrix       4×1 Column Matrix
        
        ┌   ┐                   ┌    ┐
        │ 2 │                   │  1 │
        │ 5 │                   │  0 │
        │ 7 │                   │  3 │
        └   ┘                   │  8 │
                                └    ┘
```

### Properties
- Order: $m \times 1$
- Contains $m$ elements
- Also called a **column vector**

---

## 4. Square Matrix

### Definition

> A **square matrix** has the **same number of rows and columns**.

### General Form

For an $n \times n$ square matrix:

$$A = \begin{bmatrix} a_{11} & a_{12} & \cdots & a_{1n} \\ a_{21} & a_{22} & \cdots & a_{2n} \\ \vdots & \vdots & \ddots & \vdots \\ a_{n1} & a_{n2} & \cdots & a_{nn} \end{bmatrix}_{n \times n}$$

### Examples

```
        2×2 Square              3×3 Square              4×4 Square
        
        ┌       ┐              ┌           ┐           ┌               ┐
        │ 1   2 │              │ 1   2   3 │           │ 1   2   3   4 │
        │ 3   4 │              │ 4   5   6 │           │ 5   6   7   8 │
        └       ┘              │ 7   8   9 │           │ 9  10  11  12 │
                               └           ┘           │13  14  15  16 │
                                                       └               ┘
```

### Special Elements

#### Main Diagonal (Principal Diagonal)

Elements where row index equals column index ($i = j$):

```
        ┌─────────────────┐
        │ [1]  2    3    4│     Main diagonal: 1, 6, 11, 16
        │  5  [6]   7    8│
        │  9  10  [11]  12│     These are: a₁₁, a₂₂, a₃₃, a₄₄
        │ 13  14   15  [16]│
        └─────────────────┘
```

#### Secondary Diagonal (Anti-diagonal)

Elements where $i + j = n + 1$:

```
        ┌─────────────────┐
        │  1   2   3  [4] │     Secondary diagonal: 4, 7, 10, 13
        │  5   6  [7]  8  │
        │  9 [10] 11  12  │
        │[13] 14  15  16  │
        └─────────────────┘
```

### Trace of a Square Matrix

> **Trace** = Sum of all diagonal elements

$$\text{Trace}(A) = \text{tr}(A) = \sum_{i=1}^{n} a_{ii} = a_{11} + a_{22} + \cdots + a_{nn}$$

**Example**: For $A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$

$\text{tr}(A) = 1 + 5 + 9 = 15$

---

## 5. Diagonal Matrix

### Definition

> A **diagonal matrix** is a square matrix where all non-diagonal elements are zero.

### General Form

$$D = \begin{bmatrix} d_1 & 0 & 0 & \cdots & 0 \\ 0 & d_2 & 0 & \cdots & 0 \\ 0 & 0 & d_3 & \cdots & 0 \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & 0 & \cdots & d_n \end{bmatrix}$$

### Condition

$a_{ij} = 0$ when $i \neq j$

### Notation

$$D = \text{diag}(d_1, d_2, d_3, \ldots, d_n)$$

### Examples

```
        2×2 Diagonal            3×3 Diagonal
        
        ┌       ┐              ┌           ┐
        │ 5   0 │              │ 2   0   0 │
        │ 0   3 │              │ 0   7   0 │
        └       ┘              │ 0   0   4 │
                               └           ┘
        
        diag(5, 3)              diag(2, 7, 4)
```

---

## 6. Scalar Matrix

### Definition

> A **scalar matrix** is a diagonal matrix where all diagonal elements are equal.

### General Form

$$S = \begin{bmatrix} k & 0 & 0 & \cdots & 0 \\ 0 & k & 0 & \cdots & 0 \\ 0 & 0 & k & \cdots & 0 \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & 0 & \cdots & k \end{bmatrix} = kI_n$$

### Condition

$a_{ij} = \begin{cases} k & \text{if } i = j \\ 0 & \text{if } i \neq j \end{cases}$

### Examples

```
        2×2 Scalar              3×3 Scalar
        
        ┌       ┐              ┌           ┐
        │ 5   0 │              │ 3   0   0 │
        │ 0   5 │              │ 0   3   0 │
        └       ┘              │ 0   0   3 │
                               └           ┘
        
           5I₂                     3I₃
```

---

## 7. Identity Matrix (Unit Matrix)

### Definition

> An **identity matrix** is a scalar matrix where the diagonal elements are all 1.

### General Form

$$I_n = \begin{bmatrix} 1 & 0 & 0 & \cdots & 0 \\ 0 & 1 & 0 & \cdots & 0 \\ 0 & 0 & 1 & \cdots & 0 \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & 0 & \cdots & 1 \end{bmatrix}$$

### Condition

$a_{ij} = \delta_{ij} = \begin{cases} 1 & \text{if } i = j \\ 0 & \text{if } i \neq j \end{cases}$

(This is the **Kronecker delta**)

### Examples

```
        I₂                      I₃                      I₄
        
        ┌     ┐                ┌         ┐             ┌             ┐
        │ 1 0 │                │ 1  0  0 │             │ 1  0  0  0  │
        │ 0 1 │                │ 0  1  0 │             │ 0  1  0  0  │
        └     ┘                │ 0  0  1 │             │ 0  0  1  0  │
                               └         ┘             │ 0  0  0  1  │
                                                       └             ┘
```

### Key Property

$$AI = IA = A$$ (for compatible dimensions)

The identity matrix is the **multiplicative identity** for matrix multiplication.

---

## 8. Null Matrix (Zero Matrix)

### Definition

> A **null matrix** (or **zero matrix**) has all elements equal to zero.

### General Form

$$O_{m \times n} = \begin{bmatrix} 0 & 0 & \cdots & 0 \\ 0 & 0 & \cdots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \cdots & 0 \end{bmatrix}$$

### Examples

```
        O₂ₓ₂                    O₂ₓ₃                    O₃ₓ₂
        
        ┌     ┐                ┌         ┐             ┌     ┐
        │ 0 0 │                │ 0  0  0 │             │ 0 0 │
        │ 0 0 │                │ 0  0  0 │             │ 0 0 │
        └     ┘                └         ┘             │ 0 0 │
                                                       └     ┘
```

### Key Property

$$A + O = O + A = A$$ (for same dimensions)

The zero matrix is the **additive identity** for matrix addition.

---

## 9. Triangular Matrices

### Upper Triangular Matrix

> All elements **below** the main diagonal are zero.

$$U = \begin{bmatrix} a_{11} & a_{12} & a_{13} \\ 0 & a_{22} & a_{23} \\ 0 & 0 & a_{33} \end{bmatrix}$$

**Condition**: $a_{ij} = 0$ when $i > j$

```
        ┌─────────────┐
        │ 1    2    3 │     Non-zero region
        │ 0    4    5 │     ═══════════════
        │ 0    0    6 │           ▲
        └─────────────┘           │
                               Upper triangle
```

### Lower Triangular Matrix

> All elements **above** the main diagonal are zero.

$$L = \begin{bmatrix} a_{11} & 0 & 0 \\ a_{21} & a_{22} & 0 \\ a_{31} & a_{32} & a_{33} \end{bmatrix}$$

**Condition**: $a_{ij} = 0$ when $i < j$

```
        ┌─────────────┐
        │ 1    0    0 │     Non-zero region
        │ 2    4    0 │     ═══════════════
        │ 3    5    6 │           ▼
        └─────────────┘           │
                               Lower triangle
```

### Visual Comparison

```
        Upper Triangular               Lower Triangular
        
        ┌───────────────┐              ┌───────────────┐
        │ × × × × × × × │              │ ×             │
        │   × × × × × × │              │ × ×           │
        │     × × × × × │              │ × × ×         │
        │       × × × × │              │ × × × ×       │
        │         × × × │              │ × × × × ×     │
        │           × × │              │ × × × × × ×   │
        │             × │              │ × × × × × × × │
        └───────────────┘              └───────────────┘
        
        × = possible non-zero          (blank) = always zero
```

---

## 10. Symmetric Matrix

### Definition

> A **symmetric matrix** equals its own transpose: $A = A^T$

### Condition

$a_{ij} = a_{ji}$ for all $i, j$

### Examples

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 5 \\ 3 & 5 & 6 \end{bmatrix}$$

Notice: The matrix is "mirrored" across the main diagonal.

```
        ┌─────────────────┐
        │  1  │  2  │  3  │
        │─────┼─────┼─────│
        │  2  │  4  │  5  │    Elements reflect across
        │─────┼─────┼─────│    the main diagonal
        │  3  │  5  │  6  │
        └─────────────────┘
            ↖     ↗
              ↘ ↙
         Symmetric!
```

### Visual Pattern

```
        ┌───────────────┐
        │ a   b   c   d │
        │ b   e   f   g │     Mirror image across
        │ c   f   h   i │     the main diagonal
        │ d   g   i   j │
        └───────────────┘
```

---

## 11. Skew-Symmetric Matrix (Anti-Symmetric)

### Definition

> A **skew-symmetric matrix** satisfies: $A^T = -A$

### Conditions

1. $a_{ij} = -a_{ji}$ for all $i, j$
2. All diagonal elements are **zero** (since $a_{ii} = -a_{ii} \Rightarrow a_{ii} = 0$)

### Example

$$B = \begin{bmatrix} 0 & 2 & -3 \\ -2 & 0 & 5 \\ 3 & -5 & 0 \end{bmatrix}$$

Notice: Diagonal is all zeros, and $b_{ij} = -b_{ji}$

```
        ┌─────────────────────┐
        │  0  │  2  │ -3  │
        │─────┼─────┼─────│
        │ -2  │  0  │  5  │    Elements are negatives
        │─────┼─────┼─────│    across the diagonal
        │  3  │ -5  │  0  │
        └─────────────────────┘
        
        b₁₂ = 2  and  b₂₁ = -2  ✓
        b₁₃ = -3 and  b₃₁ = 3   ✓
```

### Important Properties

1. Diagonal elements are always **zero**
2. $\text{tr}(A) = 0$ for any skew-symmetric matrix
3. Every square matrix can be written as: $A = \frac{1}{2}(A + A^T) + \frac{1}{2}(A - A^T)$
   - Symmetric part + Skew-symmetric part

---

## 12. Matrix Type Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ALL MATRICES                                  │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    SQUARE MATRICES                          │    │
│  │                                                              │    │
│  │  ┌─────────────────────────────────────────────────────┐    │    │
│  │  │              DIAGONAL MATRICES                       │    │    │
│  │  │                                                      │    │    │
│  │  │  ┌─────────────────────────────────────────────┐    │    │    │
│  │  │  │           SCALAR MATRICES                   │    │    │    │
│  │  │  │                                             │    │    │    │
│  │  │  │  ┌─────────────────────────────────────┐   │    │    │    │
│  │  │  │  │       IDENTITY MATRIX               │   │    │    │    │
│  │  │  │  │            I = 1 · I                │   │    │    │    │
│  │  │  │  └─────────────────────────────────────┘   │    │    │    │
│  │  │  │                                             │    │    │    │
│  │  │  └─────────────────────────────────────────────┘    │    │    │
│  │  │                                                      │    │    │
│  │  └─────────────────────────────────────────────────────┘    │    │
│  │                                                              │    │
│  │        Symmetric    Skew-Symmetric    Triangular            │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│      Row Matrices    Column Matrices    Rectangular Matrices        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Matrix Type | Definition | Condition | Example |
|------------|------------|-----------|---------|
| **Row Matrix** | Single row | Order: 1×n | $[1 \quad 2 \quad 3]$ |
| **Column Matrix** | Single column | Order: m×1 | $\begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}$ |
| **Square Matrix** | Equal rows and columns | m = n | $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ |
| **Diagonal Matrix** | Non-diagonal = 0 | $a_{ij}=0$ if $i≠j$ | $\begin{bmatrix} 2 & 0 \\ 0 & 5 \end{bmatrix}$ |
| **Scalar Matrix** | Diagonal with equal elements | $a_{ii}=k$ | $\begin{bmatrix} 3 & 0 \\ 0 & 3 \end{bmatrix}$ |
| **Identity Matrix** | Diagonal all 1s | $a_{ij}=δ_{ij}$ | $\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$ |
| **Null Matrix** | All elements 0 | $a_{ij}=0$ | $\begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$ |
| **Symmetric** | Equals transpose | $A = A^T$ | $\begin{bmatrix} 1 & 2 \\ 2 & 3 \end{bmatrix}$ |
| **Skew-Symmetric** | Negative of transpose | $A^T = -A$ | $\begin{bmatrix} 0 & 2 \\ -2 & 0 \end{bmatrix}$ |
| **Upper Triangular** | Lower part = 0 | $a_{ij}=0$ if $i>j$ | $\begin{bmatrix} 1 & 2 \\ 0 & 3 \end{bmatrix}$ |
| **Lower Triangular** | Upper part = 0 | $a_{ij}=0$ if $i<j$ | $\begin{bmatrix} 1 & 0 \\ 2 & 3 \end{bmatrix}$ |

---

## ❓ Quick Revision Questions

1. **What is the main difference between a diagonal matrix and a scalar matrix?**
   <details>
   <summary>Click for Answer</summary>
   In a diagonal matrix, the diagonal elements can be different. In a scalar matrix, all diagonal elements must be equal to the same value k.
   </details>

2. **Why are the diagonal elements of a skew-symmetric matrix always zero?**
   <details>
   <summary>Click for Answer</summary>
   For a skew-symmetric matrix, $a_{ij} = -a_{ji}$. For diagonal elements, $i = j$, so $a_{ii} = -a_{ii}$, which means $2a_{ii} = 0$, therefore $a_{ii} = 0$.
   </details>

3. **Is every identity matrix also a scalar matrix? Is every scalar matrix also a diagonal matrix?**
   <details>
   <summary>Click for Answer</summary>
   Yes to both! Identity matrix is a scalar matrix with k=1. Every scalar matrix is a diagonal matrix where all diagonal elements happen to be equal.
   </details>

4. **Can a 3×4 matrix be symmetric?**
   <details>
   <summary>Click for Answer</summary>
   No. Symmetric matrices must be square (same number of rows and columns). A 3×4 matrix cannot be symmetric because its transpose would be 4×3.
   </details>

5. **Write an example of a 3×3 matrix that is both upper triangular and lower triangular.**
   <details>
   <summary>Click for Answer</summary>
   A diagonal matrix! For example:
   $$\begin{bmatrix} 2 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 3 \end{bmatrix}$$
   It satisfies both conditions since all non-diagonal elements are zero.
   </details>

6. **If A is symmetric and B is skew-symmetric, what type of matrix is A + B?**
   <details>
   <summary>Click for Answer</summary>
   A + B is neither symmetric nor skew-symmetric (in general). However, any square matrix C can be uniquely written as A + B where A is symmetric and B is skew-symmetric.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Definition and Notation](01-definition-and-notation.md) | [Unit 1: Introduction](./README.md) | [Order of Matrix →](03-order-of-matrix.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
