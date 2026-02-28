# Chapter 2.1: Matrix Addition and Subtraction

[← Previous: Equality of Matrices](../01-Introduction-to-Matrices/04-equality-of-matrices.md) | [Back to README](../README.md) | [Next: Scalar Multiplication →](02-scalar-multiplication.md)

---

## 📚 Chapter Overview

Matrix addition and subtraction are fundamental operations that combine matrices element by element. These operations form the basis for more complex matrix manipulations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Add and subtract matrices correctly
- Identify when operations are possible
- Apply properties of addition
- Solve problems involving matrix addition/subtraction

---

## 1. Matrix Addition

### Definition

> **Matrix Addition**: Two matrices A and B can be added if they have the **same order**. The sum is obtained by adding corresponding elements.

### Formal Definition

If $A = [a_{ij}]_{m \times n}$ and $B = [b_{ij}]_{m \times n}$, then:

$$A + B = [a_{ij} + b_{ij}]_{m \times n}$$

### Visual Representation

```
        Matrix A              Matrix B              A + B
        
    ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
    │ a₁₁   a₁₂  │      │ b₁₁   b₁₂  │      │a₁₁+b₁₁ a₁₂+b₁₂│
    │             │  +   │             │  =   │               │
    │ a₂₁   a₂₂  │      │ b₂₁   b₂₂  │      │a₂₁+b₂₁ a₂₂+b₂₂│
    └─────────────┘      └─────────────┘      └─────────────────┘
```

### Example 1: Adding 2×2 Matrices

$$A = \begin{bmatrix} 2 & 5 \\ 3 & 8 \end{bmatrix}, \quad B = \begin{bmatrix} 1 & 4 \\ 6 & 2 \end{bmatrix}$$

**Solution**:

$$A + B = \begin{bmatrix} 2+1 & 5+4 \\ 3+6 & 8+2 \end{bmatrix} = \begin{bmatrix} 3 & 9 \\ 9 & 10 \end{bmatrix}$$

```
        Step-by-step visualization:
        
        Position (1,1): 2 + 1 = 3
        Position (1,2): 5 + 4 = 9
        Position (2,1): 3 + 6 = 9
        Position (2,2): 8 + 2 = 10
```

### Example 2: Adding 3×3 Matrices

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}, \quad B = \begin{bmatrix} 9 & 8 & 7 \\ 6 & 5 & 4 \\ 3 & 2 & 1 \end{bmatrix}$$

**Solution**:

$$A + B = \begin{bmatrix} 1+9 & 2+8 & 3+7 \\ 4+6 & 5+5 & 6+4 \\ 7+3 & 8+2 & 9+1 \end{bmatrix} = \begin{bmatrix} 10 & 10 & 10 \\ 10 & 10 & 10 \\ 10 & 10 & 10 \end{bmatrix}$$

---

## 2. Matrix Subtraction

### Definition

> **Matrix Subtraction**: Two matrices A and B can be subtracted if they have the **same order**. The difference is obtained by subtracting corresponding elements.

### Formal Definition

If $A = [a_{ij}]_{m \times n}$ and $B = [b_{ij}]_{m \times n}$, then:

$$A - B = [a_{ij} - b_{ij}]_{m \times n}$$

### Visual Representation

```
        Matrix A              Matrix B              A - B
        
    ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
    │ a₁₁   a₁₂  │      │ b₁₁   b₁₂  │      │a₁₁-b₁₁ a₁₂-b₁₂│
    │             │  -   │             │  =   │               │
    │ a₂₁   a₂₂  │      │ b₂₁   b₂₂  │      │a₂₁-b₂₁ a₂₂-b₂₂│
    └─────────────┘      └─────────────┘      └─────────────────┘
```

### Example 1: Subtracting 2×3 Matrices

$$A = \begin{bmatrix} 8 & 5 & 3 \\ 9 & 6 & 2 \end{bmatrix}, \quad B = \begin{bmatrix} 2 & 1 & 4 \\ 3 & 5 & 1 \end{bmatrix}$$

**Solution**:

$$A - B = \begin{bmatrix} 8-2 & 5-1 & 3-4 \\ 9-3 & 6-5 & 2-1 \end{bmatrix} = \begin{bmatrix} 6 & 4 & -1 \\ 6 & 1 & 1 \end{bmatrix}$$

### Example 2: Negative Results

$$P = \begin{bmatrix} 3 & 2 \\ 1 & 4 \end{bmatrix}, \quad Q = \begin{bmatrix} 5 & 7 \\ 2 & 8 \end{bmatrix}$$

$$P - Q = \begin{bmatrix} 3-5 & 2-7 \\ 1-2 & 4-8 \end{bmatrix} = \begin{bmatrix} -2 & -5 \\ -1 & -4 \end{bmatrix}$$

---

## 3. Compatibility Check

### Rule

> **Matrices can only be added or subtracted if they have the SAME ORDER**

### Compatibility Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                 ADDITION/SUBTRACTION COMPATIBILITY               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│         A              B             Result                      │
│       (m × n)        (p × q)        (m × n)                     │
│                                                                  │
│         │              │                                         │
│         ▼              ▼                                         │
│    ┌─────────────────────────┐                                  │
│    │  Is m = p AND n = q?   │                                  │
│    └───────────┬─────────────┘                                  │
│                │                                                 │
│        ┌───────┴───────┐                                        │
│        │               │                                        │
│       YES             NO                                        │
│        │               │                                        │
│        ▼               ▼                                        │
│   ┌─────────┐    ┌──────────────────┐                          │
│   │ A ± B   │    │ NOT DEFINED      │                          │
│   │ exists  │    │ Cannot compute   │                          │
│   └─────────┘    └──────────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Examples of Incompatible Matrices

```
        A (2×3)          B (3×2)          C (2×2)
        
      ┌─────────┐      ┌───────┐        ┌───────┐
      │ 1 2 3   │      │ 1  2  │        │ 1  2  │
      │ 4 5 6   │      │ 3  4  │        │ 3  4  │
      └─────────┘      │ 5  6  │        └───────┘
                       └───────┘
        
        A + B = undefined (2×3 ≠ 3×2)
        A + C = undefined (2×3 ≠ 2×2)
        B + C = undefined (3×2 ≠ 2×2)
```

---

## 4. Properties of Matrix Addition

### Property 1: Closure

> The sum of two m×n matrices is also an m×n matrix.

$$A_{m \times n} + B_{m \times n} = C_{m \times n}$$

### Property 2: Commutative Law

> **A + B = B + A**

```
        A + B                    B + A
        
    ┌───┐   ┌───┐            ┌───┐   ┌───┐
    │ 1 │ + │ 4 │     =      │ 4 │ + │ 1 │
    │ 2 │   │ 5 │            │ 5 │   │ 2 │
    │ 3 │   │ 6 │            │ 6 │   │ 3 │
    └───┘   └───┘            └───┘   └───┘
        
        = ┌───┐              = ┌───┐
          │ 5 │                │ 5 │
          │ 7 │                │ 7 │
          │ 9 │                │ 9 │
          └───┘                └───┘
```

### Property 3: Associative Law

> **(A + B) + C = A + (B + C)**

Order of grouping doesn't matter.

### Property 4: Additive Identity

> **A + O = O + A = A**

Where O is the zero matrix of the same order.

$$\begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix} + \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix}$$

### Property 5: Additive Inverse

> **A + (-A) = O**

$$\begin{bmatrix} 2 & 3 \\ 4 & 5 \end{bmatrix} + \begin{bmatrix} -2 & -3 \\ -4 & -5 \end{bmatrix} = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$$

### Properties Summary Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│              PROPERTIES OF MATRIX ADDITION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌──────────────┐                                             │
│    │   CLOSURE    │  A + B gives another matrix of same order   │
│    └──────────────┘                                             │
│           │                                                      │
│           ▼                                                      │
│    ┌──────────────┐                                             │
│    │ COMMUTATIVE  │  A + B = B + A                              │
│    └──────────────┘                                             │
│           │                                                      │
│           ▼                                                      │
│    ┌──────────────┐                                             │
│    │ ASSOCIATIVE  │  (A + B) + C = A + (B + C)                  │
│    └──────────────┘                                             │
│           │                                                      │
│           ▼                                                      │
│    ┌──────────────┐                                             │
│    │   IDENTITY   │  A + O = A                                  │
│    └──────────────┘                                             │
│           │                                                      │
│           ▼                                                      │
│    ┌──────────────┐                                             │
│    │   INVERSE    │  A + (-A) = O                               │
│    └──────────────┘                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Worked Examples

### Example 1: Combined Operations

**Problem**: If $A = \begin{bmatrix} 3 & 1 \\ 2 & 4 \end{bmatrix}$, $B = \begin{bmatrix} 1 & 5 \\ 0 & 2 \end{bmatrix}$, $C = \begin{bmatrix} 2 & 3 \\ 1 & 1 \end{bmatrix}$

Find: A + B - C

**Solution**:

$$A + B = \begin{bmatrix} 3+1 & 1+5 \\ 2+0 & 4+2 \end{bmatrix} = \begin{bmatrix} 4 & 6 \\ 2 & 6 \end{bmatrix}$$

$$(A + B) - C = \begin{bmatrix} 4-2 & 6-3 \\ 2-1 & 6-1 \end{bmatrix} = \begin{bmatrix} 2 & 3 \\ 1 & 5 \end{bmatrix}$$

### Example 2: Finding Unknown Matrix

**Problem**: If $A + B = \begin{bmatrix} 5 & 7 \\ 3 & 9 \end{bmatrix}$ and $A = \begin{bmatrix} 2 & 4 \\ 1 & 3 \end{bmatrix}$, find B.

**Solution**:

$$B = (A + B) - A$$

$$B = \begin{bmatrix} 5-2 & 7-4 \\ 3-1 & 9-3 \end{bmatrix} = \begin{bmatrix} 3 & 3 \\ 2 & 6 \end{bmatrix}$$

### Example 3: Matrix Equation

**Problem**: Solve for X: $X + \begin{bmatrix} 2 & 3 \\ 1 & 4 \end{bmatrix} = \begin{bmatrix} 5 & 8 \\ 4 & 7 \end{bmatrix}$

**Solution**:

$$X = \begin{bmatrix} 5 & 8 \\ 4 & 7 \end{bmatrix} - \begin{bmatrix} 2 & 3 \\ 1 & 4 \end{bmatrix} = \begin{bmatrix} 3 & 5 \\ 3 & 3 \end{bmatrix}$$

### Example 4: Verify Commutative Property

**Problem**: Verify that A + B = B + A for:

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}, \quad B = \begin{bmatrix} 7 & 8 & 9 \\ 10 & 11 & 12 \end{bmatrix}$$

**Solution**:

$$A + B = \begin{bmatrix} 1+7 & 2+8 & 3+9 \\ 4+10 & 5+11 & 6+12 \end{bmatrix} = \begin{bmatrix} 8 & 10 & 12 \\ 14 & 16 & 18 \end{bmatrix}$$

$$B + A = \begin{bmatrix} 7+1 & 8+2 & 9+3 \\ 10+4 & 11+5 & 12+6 \end{bmatrix} = \begin{bmatrix} 8 & 10 & 12 \\ 14 & 16 & 18 \end{bmatrix}$$

**Conclusion**: A + B = B + A ✓

---

## 6. Real-World Applications

### Application 1: Sales Data Combination

```
        January Sales            February Sales          Total Sales
        
        ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
        │ 100  150  80│         │  90  170  95│         │ 190  320 175│
        │  75  120  60│    +    │  85  130  75│    =    │ 160  250 135│
        │  50   90  40│         │  60  100  45│         │ 110  190  85│
        └─────────────┘         └─────────────┘         └─────────────┘
        
        Rows: Stores
        Columns: Product categories
```

### Application 2: Inventory Management

```
        Current Stock          New Arrivals           Updated Stock
        
        Product A: 50    +    Product A: 30    =    Product A: 80
        Product B: 75    +    Product B: 25    =    Product B: 100
        Product C: 40    +    Product C: 60    =    Product C: 100
        
        In matrix form:
        
        ┌────┐     ┌────┐     ┌─────┐
        │ 50 │  +  │ 30 │  =  │  80 │
        │ 75 │     │ 25 │     │ 100 │
        │ 40 │     │ 60 │     │ 100 │
        └────┘     └────┘     └─────┘
```

---

## 📊 Summary Table

| Concept | Rule/Formula | Example |
|---------|--------------|---------|
| Addition | $[a_{ij}] + [b_{ij}] = [a_{ij} + b_{ij}]$ | Element-wise sum |
| Subtraction | $[a_{ij}] - [b_{ij}] = [a_{ij} - b_{ij}]$ | Element-wise difference |
| Compatibility | Same order required | 2×3 + 2×3 only |
| Commutative | A + B = B + A | Order doesn't matter |
| Associative | (A + B) + C = A + (B + C) | Grouping doesn't matter |
| Identity | A + O = A | Zero matrix |
| Inverse | A + (-A) = O | Negation |

---

## ❓ Quick Revision Questions

1. **Can we add a 3×2 matrix to a 2×3 matrix?**
   <details>
   <summary>Click for Answer</summary>
   No. Addition requires both matrices to have the same order. 3×2 ≠ 2×3.
   </details>

2. **Find A + B if $A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$ and $B = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $A + B = \begin{bmatrix} 1+5 & 2+6 \\ 3+7 & 4+8 \end{bmatrix} = \begin{bmatrix} 6 & 8 \\ 10 & 12 \end{bmatrix}$
   </details>

3. **What is the additive identity for a 3×3 matrix?**
   <details>
   <summary>Click for Answer</summary>
   The 3×3 zero matrix: $O = \begin{bmatrix} 0 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0 \end{bmatrix}$
   </details>

4. **If A - B = C, express B in terms of A and C.**
   <details>
   <summary>Click for Answer</summary>
   B = A - C
   </details>

5. **Is matrix subtraction commutative?**
   <details>
   <summary>Click for Answer</summary>
   No! A - B ≠ B - A in general. In fact, A - B = -(B - A).
   </details>

6. **Find X if $X + \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} = \begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $X = \begin{bmatrix} -1 & -2 \\ -3 & -4 \end{bmatrix}$ (the additive inverse)
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Equality of Matrices](../01-Introduction-to-Matrices/04-equality-of-matrices.md) | [Unit 2: Operations](./README.md) | [Scalar Multiplication →](02-scalar-multiplication.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
