# Chapter 1.1: Definition and Notation of Matrices

[← Back to README](../README.md) | [Next: Types of Matrices →](02-types-of-matrices.md)

---

## 📚 Chapter Overview

This chapter introduces the fundamental concept of a matrix, its definition, elements, and the standard notation used throughout linear algebra. Understanding this foundation is crucial for all subsequent topics.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define what a matrix is
- Identify the elements of a matrix
- Use proper notation to represent matrices
- Understand row and column indexing

---

## 1. What is a Matrix?

### Definition

> **A matrix is a rectangular arrangement of numbers (or expressions) arranged in rows and columns, enclosed within brackets.**

A matrix is essentially an ordered collection of numbers organized in a specific two-dimensional structure. The numbers in the matrix are called **elements** or **entries**.

### Visual Representation

```
                    Columns
                 1    2    3
               ┌              ┐
        Row 1  │  2   5   7   │
               │              │
        Row 2  │  3   8   1   │
               │              │
        Row 3  │  4   6   9   │
               └              ┘
```

This is a **3×3 matrix** with 3 rows and 3 columns.

---

## 2. Matrix Notation

### General Form

A matrix is typically denoted by a **capital letter** (A, B, C, etc.) and its elements by the corresponding **lowercase letter** with subscripts.

$$A = \begin{bmatrix} a_{11} & a_{12} & a_{13} & \cdots & a_{1n} \\ a_{21} & a_{22} & a_{23} & \cdots & a_{2n} \\ a_{31} & a_{32} & a_{33} & \cdots & a_{3n} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ a_{m1} & a_{m2} & a_{m3} & \cdots & a_{mn} \end{bmatrix}$$

### Compact Notation

$$A = [a_{ij}]_{m \times n}$$

Where:
- $a_{ij}$ = element in the $i^{th}$ row and $j^{th}$ column
- $m$ = number of rows
- $n$ = number of columns

### ASCII Representation

```
        A = [ a₁₁  a₁₂  a₁₃  ...  a₁ₙ ]
            [ a₂₁  a₂₂  a₂₃  ...  a₂ₙ ]
            [ a₃₁  a₃₂  a₃₃  ...  a₃ₙ ]
            [  ⋮    ⋮    ⋮    ⋱    ⋮  ]
            [ aₘ₁  aₘ₂  aₘ₃  ...  aₘₙ ]
```

---

## 3. Understanding Element Notation

### The Double Subscript System

For any element $a_{ij}$:

```
         a  ᵢⱼ
         ↑  ↑↑
         │  │└── Column number (j)
         │  └─── Row number (i)
         └────── Element symbol
```

### Example: Identifying Elements

Given matrix A:

$$A = \begin{bmatrix} 2 & 5 & 7 \\ 3 & 8 & 1 \\ 4 & 6 & 9 \end{bmatrix}$$

| Element | Position | Value |
|---------|----------|-------|
| $a_{11}$ | Row 1, Column 1 | 2 |
| $a_{12}$ | Row 1, Column 2 | 5 |
| $a_{13}$ | Row 1, Column 3 | 7 |
| $a_{21}$ | Row 2, Column 1 | 3 |
| $a_{22}$ | Row 2, Column 2 | 8 |
| $a_{23}$ | Row 2, Column 3 | 1 |
| $a_{31}$ | Row 3, Column 1 | 4 |
| $a_{32}$ | Row 3, Column 2 | 6 |
| $a_{33}$ | Row 3, Column 3 | 9 |

### Visual Element Map

```
        ┌─────────────────────────────┐
        │                             │
        │    a₁₁    a₁₂    a₁₃        │
        │    (2)    (5)    (7)        │
        │                             │
        │    a₂₁    a₂₂    a₂₃        │
        │    (3)    (8)    (1)        │
        │                             │
        │    a₃₁    a₃₂    a₃₃        │
        │    (4)    (6)    (9)        │
        │                             │
        └─────────────────────────────┘
```

---

## 4. Different Bracket Notations

Matrices can be written using different types of brackets:

### Square Brackets (Most Common)
$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$$

### Parentheses
$$A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$$

### Double Vertical Lines (for Determinants)
$$|A| = \begin{vmatrix} 1 & 2 \\ 3 & 4 \end{vmatrix}$$

> **Note**: Double vertical lines typically represent the **determinant** of a matrix, not the matrix itself.

---

## 5. Rows and Columns

### Rows (Horizontal)

```
        ┌                     ┐
        │  1   2   3   4   5  │  ← Row 1
        │                     │
        │  6   7   8   9  10  │  ← Row 2
        │                     │
        │ 11  12  13  14  15  │  ← Row 3
        └                     ┘
```

### Columns (Vertical)

```
        ┌                     ┐
        │  1   2   3   4   5  │
        │  ↓   ↓   ↓   ↓   ↓  │
        │  6   7   8   9  10  │
        │  ↓   ↓   ↓   ↓   ↓  │
        │ 11  12  13  14  15  │
        └                     ┘
           │   │   │   │   │
           C₁  C₂  C₃  C₄  C₅
```

---

## 6. Practical Examples

### Example 1: Write the Matrix

**Problem**: Write a 2×3 matrix A where $a_{ij} = i + 2j$

**Solution**:
- $a_{11} = 1 + 2(1) = 3$
- $a_{12} = 1 + 2(2) = 5$
- $a_{13} = 1 + 2(3) = 7$
- $a_{21} = 2 + 2(1) = 4$
- $a_{22} = 2 + 2(2) = 6$
- $a_{23} = 2 + 2(3) = 8$

$$A = \begin{bmatrix} 3 & 5 & 7 \\ 4 & 6 & 8 \end{bmatrix}$$

### Example 2: Identify Elements

**Problem**: For matrix B, find $b_{23}$ and $b_{31}$

$$B = \begin{bmatrix} 1 & 0 & 5 \\ 2 & 7 & 9 \\ 3 & 4 & 8 \end{bmatrix}$$

**Solution**:
- $b_{23}$ = element in row 2, column 3 = **9**
- $b_{31}$ = element in row 3, column 1 = **3**

### Example 3: Construct from Rule

**Problem**: Construct a 3×3 matrix where $a_{ij} = i^2 - j$

**Solution**:

| Position | Formula | Value |
|----------|---------|-------|
| $a_{11}$ | $1^2 - 1$ | 0 |
| $a_{12}$ | $1^2 - 2$ | -1 |
| $a_{13}$ | $1^2 - 3$ | -2 |
| $a_{21}$ | $2^2 - 1$ | 3 |
| $a_{22}$ | $2^2 - 2$ | 2 |
| $a_{23}$ | $2^2 - 3$ | 1 |
| $a_{31}$ | $3^2 - 1$ | 8 |
| $a_{32}$ | $3^2 - 2$ | 7 |
| $a_{33}$ | $3^2 - 3$ | 6 |

$$A = \begin{bmatrix} 0 & -1 & -2 \\ 3 & 2 & 1 \\ 8 & 7 & 6 \end{bmatrix}$$

---

## 7. Real-World Applications

### Where Matrices Are Used

```
┌─────────────────────────────────────────────────────────────┐
│                    MATRIX APPLICATIONS                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Computer   │  │   Physics    │  │  Economics   │       │
│  │   Graphics   │  │ & Engineering│  │  & Finance   │       │
│  │              │  │              │  │              │       │
│  │ • 3D rotations│ │ • Stress     │  │ • Input-output│      │
│  │ • Transformations│ analysis   │  │   models     │       │
│  │ • Image processing│ • Circuits│  │ • Portfolio  │       │
│  │ • Game engines│  │ • Quantum   │  │   optimization│      │
│  └──────────────┘  │   mechanics │  └──────────────┘       │
│                    └──────────────┘                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Data       │  │   Machine    │  │   Network    │       │
│  │   Science    │  │   Learning   │  │   Analysis   │       │
│  │              │  │              │  │              │       │
│  │ • Statistics │  │ • Neural     │  │ • Adjacency  │       │
│  │ • Regression │  │   networks   │  │   matrices   │       │
│  │ • PCA        │  │ • Deep       │  │ • Graph      │       │
│  │              │  │   learning   │  │   theory     │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Example: Data Representation

A company tracks sales across 3 products over 4 quarters:

```
                    Q1    Q2    Q3    Q4
         Product A [ 150  200  180  220 ]
         Product B [ 300  280  350  400 ]
         Product C [ 175  190  210  240 ]
```

This is a **3×4 matrix** representing sales data!

---

## 📊 Summary Table

| Concept | Description | Example |
|---------|-------------|---------|
| Matrix | Rectangular array of numbers | $\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ |
| Element | Individual entry in a matrix | $a_{ij}$ |
| Row | Horizontal line of elements | Elements with same first subscript |
| Column | Vertical line of elements | Elements with same second subscript |
| Order | Dimensions (rows × columns) | 3×4 matrix |
| Notation | Capital letter for matrix | A, B, C |

---

## ❓ Quick Revision Questions

1. **What is a matrix?**
   <details>
   <summary>Click for Answer</summary>
   A matrix is a rectangular arrangement of numbers organized in rows and columns, enclosed within brackets.
   </details>

2. **In the notation $a_{34}$, what do the subscripts represent?**
   <details>
   <summary>Click for Answer</summary>
   The first subscript (3) represents the row number, and the second subscript (4) represents the column number. So $a_{34}$ is the element in row 3, column 4.
   </details>

3. **Construct a 2×2 matrix where $a_{ij} = i \times j$**
   <details>
   <summary>Click for Answer</summary>
   
   $$A = \begin{bmatrix} 1 & 2 \\ 2 & 4 \end{bmatrix}$$
   
   ($a_{11}=1×1=1$, $a_{12}=1×2=2$, $a_{21}=2×1=2$, $a_{22}=2×2=4$)
   </details>

4. **How many elements are in a 4×5 matrix?**
   <details>
   <summary>Click for Answer</summary>
   4 × 5 = 20 elements
   </details>

5. **What is the difference between square brackets and double vertical lines in matrix notation?**
   <details>
   <summary>Click for Answer</summary>
   Square brackets [ ] denote the matrix itself, while double vertical lines | | denote the determinant of the matrix (a scalar value).
   </details>

6. **For matrix $B = \begin{bmatrix} 7 & 3 & 9 \\ 2 & 5 & 1 \end{bmatrix}$, find $b_{12}$ and $b_{23}$**
   <details>
   <summary>Click for Answer</summary>
   $b_{12} = 3$ (row 1, column 2) and $b_{23} = 1$ (row 2, column 3)
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← README](../README.md) | [Unit 1: Introduction](./README.md) | [Types of Matrices →](02-types-of-matrices.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
