# Chapter 3.4: Minor and Cofactor

[← Previous: Properties of Determinants](03-properties-of-determinants.md) | [Back to README](../README.md) | [Next: Expansion Methods →](05-expansion-methods.md)

---

## 📚 Chapter Overview

Minors and cofactors are fundamental building blocks for computing determinants of larger matrices. They provide a systematic way to reduce an n×n determinant to smaller determinants, ultimately leading to the cofactor expansion method.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and calculate minors of matrix elements
- Calculate cofactors using minors
- Understand the relationship between minors and cofactors
- Apply minors and cofactors in determinant computation

---

## 1. Introduction to Minors

### Definition

> The **minor** of an element $a_{ij}$ in a matrix A, denoted $M_{ij}$, is the determinant of the submatrix obtained by deleting the $i$-th row and $j$-th column.

### Visual Representation

```
         Original Matrix                    Minor M₁₂
         
      ┌─────────────────┐             Delete Row 1
      │  a₁₁  a₁₂  a₁₃  │                   ↓
      │  a₂₁  a₂₂  a₂₃  │    →        | a₂₁  a₂₃ |
      │  a₃₁  a₃₂  a₃₃  │             | a₃₁  a₃₃ |
      └─────────────────┘                   ↑
              ↑                       Delete Col 2
         Delete Col 2
```

---

## 2. Computing Minors: Step-by-Step

### For a 3×3 Matrix

$$A = \begin{bmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{bmatrix}$$

### All Nine Minors

| Minor | Delete | Submatrix | Formula |
|-------|--------|-----------|---------|
| $M_{11}$ | Row 1, Col 1 | $\begin{vmatrix} a_{22} & a_{23} \\ a_{32} & a_{33} \end{vmatrix}$ | $a_{22}a_{33} - a_{23}a_{32}$ |
| $M_{12}$ | Row 1, Col 2 | $\begin{vmatrix} a_{21} & a_{23} \\ a_{31} & a_{33} \end{vmatrix}$ | $a_{21}a_{33} - a_{23}a_{31}$ |
| $M_{13}$ | Row 1, Col 3 | $\begin{vmatrix} a_{21} & a_{22} \\ a_{31} & a_{32} \end{vmatrix}$ | $a_{21}a_{32} - a_{22}a_{31}$ |
| $M_{21}$ | Row 2, Col 1 | $\begin{vmatrix} a_{12} & a_{13} \\ a_{32} & a_{33} \end{vmatrix}$ | $a_{12}a_{33} - a_{13}a_{32}$ |
| $M_{22}$ | Row 2, Col 2 | $\begin{vmatrix} a_{11} & a_{13} \\ a_{31} & a_{33} \end{vmatrix}$ | $a_{11}a_{33} - a_{13}a_{31}$ |
| $M_{23}$ | Row 2, Col 3 | $\begin{vmatrix} a_{11} & a_{12} \\ a_{31} & a_{32} \end{vmatrix}$ | $a_{11}a_{32} - a_{12}a_{31}$ |
| $M_{31}$ | Row 3, Col 1 | $\begin{vmatrix} a_{12} & a_{13} \\ a_{22} & a_{23} \end{vmatrix}$ | $a_{12}a_{23} - a_{13}a_{22}$ |
| $M_{32}$ | Row 3, Col 2 | $\begin{vmatrix} a_{11} & a_{13} \\ a_{21} & a_{23} \end{vmatrix}$ | $a_{11}a_{23} - a_{13}a_{21}$ |
| $M_{33}$ | Row 3, Col 3 | $\begin{vmatrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{vmatrix}$ | $a_{11}a_{22} - a_{12}a_{21}$ |

---

## 3. Numerical Example: Finding Minors

### Given Matrix

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$

### Step-by-Step Calculation

**Minor $M_{11}$** (delete row 1, column 1):
$$M_{11} = \begin{vmatrix} 5 & 6 \\ 8 & 9 \end{vmatrix} = 45 - 48 = -3$$

**Minor $M_{12}$** (delete row 1, column 2):
$$M_{12} = \begin{vmatrix} 4 & 6 \\ 7 & 9 \end{vmatrix} = 36 - 42 = -6$$

**Minor $M_{13}$** (delete row 1, column 3):
$$M_{13} = \begin{vmatrix} 4 & 5 \\ 7 & 8 \end{vmatrix} = 32 - 35 = -3$$

**Minor $M_{21}$** (delete row 2, column 1):
$$M_{21} = \begin{vmatrix} 2 & 3 \\ 8 & 9 \end{vmatrix} = 18 - 24 = -6$$

**Minor $M_{22}$** (delete row 2, column 2):
$$M_{22} = \begin{vmatrix} 1 & 3 \\ 7 & 9 \end{vmatrix} = 9 - 21 = -12$$

**Minor $M_{23}$** (delete row 2, column 3):
$$M_{23} = \begin{vmatrix} 1 & 2 \\ 7 & 8 \end{vmatrix} = 8 - 14 = -6$$

**Minor $M_{31}$** (delete row 3, column 1):
$$M_{31} = \begin{vmatrix} 2 & 3 \\ 5 & 6 \end{vmatrix} = 12 - 15 = -3$$

**Minor $M_{32}$** (delete row 3, column 2):
$$M_{32} = \begin{vmatrix} 1 & 3 \\ 4 & 6 \end{vmatrix} = 6 - 12 = -6$$

**Minor $M_{33}$** (delete row 3, column 3):
$$M_{33} = \begin{vmatrix} 1 & 2 \\ 4 & 5 \end{vmatrix} = 5 - 8 = -3$$

### Matrix of Minors

$$\text{Matrix of Minors} = \begin{bmatrix} -3 & -6 & -3 \\ -6 & -12 & -6 \\ -3 & -6 & -3 \end{bmatrix}$$

---

## 4. Introduction to Cofactors

### Definition

> The **cofactor** of an element $a_{ij}$, denoted $C_{ij}$ (or $A_{ij}$), is the minor multiplied by a sign factor:
> $$C_{ij} = (-1)^{i+j} \cdot M_{ij}$$

### The Sign Pattern (Checkerboard Pattern)

```
        ┌─────────────────────────────────────┐
        │                                      │
        │    (+)   (-)   (+)   (-)   (+)      │
        │                                      │
        │    (-)   (+)   (-)   (+)   (-)      │
        │                                      │
        │    (+)   (-)   (+)   (-)   (+)      │
        │                                      │
        │    (-)   (+)   (-)   (+)   (-)      │
        │                                      │
        └─────────────────────────────────────┘
        
        Sign of position (i,j) = (-1)^(i+j)
```

### 3×3 Sign Matrix

$$\text{Signs} = \begin{bmatrix} + & - & + \\ - & + & - \\ + & - & + \end{bmatrix} = \begin{bmatrix} +1 & -1 & +1 \\ -1 & +1 & -1 \\ +1 & -1 & +1 \end{bmatrix}$$

### Quick Rule

```
┌───────────────────────────────────────────────────────────┐
│                    SIGN DETERMINATION                      │
├───────────────────────────────────────────────────────────┤
│                                                            │
│   Position (i, j)     i + j      (-1)^(i+j)    Sign       │
│   ─────────────────────────────────────────────────────   │
│   (1,1)               2          +1            +          │
│   (1,2)               3          -1            -          │
│   (1,3)               4          +1            +          │
│   (2,1)               3          -1            -          │
│   (2,2)               4          +1            +          │
│   (2,3)               5          -1            -          │
│   (3,1)               4          +1            +          │
│   (3,2)               5          -1            -          │
│   (3,3)               6          +1            +          │
│                                                            │
│   Pattern: Even sum → +, Odd sum → -                      │
│                                                            │
└───────────────────────────────────────────────────────────┘
```

---

## 5. Computing Cofactors: Example

Using the same matrix:
$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$

### All Cofactors

| Element | Minor $M_{ij}$ | Sign $(-1)^{i+j}$ | Cofactor $C_{ij}$ |
|---------|----------------|-------------------|-------------------|
| $a_{11} = 1$ | $-3$ | $+1$ | $-3$ |
| $a_{12} = 2$ | $-6$ | $-1$ | $+6$ |
| $a_{13} = 3$ | $-3$ | $+1$ | $-3$ |
| $a_{21} = 4$ | $-6$ | $-1$ | $+6$ |
| $a_{22} = 5$ | $-12$ | $+1$ | $-12$ |
| $a_{23} = 6$ | $-6$ | $-1$ | $+6$ |
| $a_{31} = 7$ | $-3$ | $+1$ | $-3$ |
| $a_{32} = 8$ | $-6$ | $-1$ | $+6$ |
| $a_{33} = 9$ | $-3$ | $+1$ | $-3$ |

### Cofactor Matrix

$$\text{Cofactor Matrix} = \begin{bmatrix} -3 & 6 & -3 \\ 6 & -12 & 6 \\ -3 & 6 & -3 \end{bmatrix}$$

---

## 6. Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                    MATRIX A (n × n)                             │
│                          │                                       │
│                          ▼                                       │
│    ┌─────────────────────────────────────────────────────┐      │
│    │        For each element a_ij                         │      │
│    └─────────────────────────────────────────────────────┘      │
│                          │                                       │
│            ┌─────────────┴─────────────┐                        │
│            ▼                           ▼                        │
│    ┌───────────────┐           ┌───────────────┐                │
│    │ Delete row i  │           │               │                │
│    │ Delete col j  │           │ Calculate     │                │
│    │               │           │ (-1)^(i+j)    │                │
│    └───────┬───────┘           └───────┬───────┘                │
│            │                           │                        │
│            ▼                           │                        │
│    ┌───────────────┐                   │                        │
│    │   MINOR M_ij  │                   │                        │
│    │ (n-1)×(n-1)   │                   │                        │
│    │ determinant   │                   │                        │
│    └───────┬───────┘                   │                        │
│            │                           │                        │
│            └───────────┬───────────────┘                        │
│                        │                                         │
│                        ▼                                         │
│             ┌─────────────────────┐                             │
│             │   COFACTOR C_ij    │                              │
│             │                     │                              │
│             │  C_ij = (-1)^(i+j) │                              │
│             │         × M_ij     │                              │
│             └─────────────────────┘                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. For 2×2 Matrices

### Matrix

$$A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$$

### Minors and Cofactors

| Element | Minor | Sign | Cofactor |
|---------|-------|------|----------|
| $a = a_{11}$ | $M_{11} = d$ | $+$ | $C_{11} = d$ |
| $b = a_{12}$ | $M_{12} = c$ | $-$ | $C_{12} = -c$ |
| $c = a_{21}$ | $M_{21} = b$ | $-$ | $C_{21} = -b$ |
| $d = a_{22}$ | $M_{22} = a$ | $+$ | $C_{22} = a$ |

### Cofactor Matrix of 2×2

$$\text{Cofactor Matrix} = \begin{bmatrix} d & -c \\ -b & a \end{bmatrix}$$

---

## 8. Using Cofactors for Determinant

### The Cofactor Expansion Formula

The determinant can be calculated by expanding along any row or column:

**Row i expansion:**
$$|A| = \sum_{j=1}^{n} a_{ij} \cdot C_{ij} = a_{i1}C_{i1} + a_{i2}C_{i2} + \cdots + a_{in}C_{in}$$

**Column j expansion:**
$$|A| = \sum_{i=1}^{n} a_{ij} \cdot C_{ij} = a_{1j}C_{1j} + a_{2j}C_{2j} + \cdots + a_{nj}C_{nj}$$

### Example: Expand Along Row 1

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$

Using row 1 expansion:
$$|A| = a_{11}C_{11} + a_{12}C_{12} + a_{13}C_{13}$$
$$= 1(-3) + 2(6) + 3(-3)$$
$$= -3 + 12 - 9$$
$$= 0$$

---

## 9. Important Properties

### Property 1: Sum of Products

The sum of products of elements of any row (or column) with their corresponding cofactors equals the determinant.

$$\sum_{j=1}^{n} a_{ij} \cdot C_{ij} = |A|$$

### Property 2: Alien Cofactors

The sum of products of elements of any row (or column) with the cofactors of a **different** row (or column) equals zero.

$$\sum_{j=1}^{n} a_{ij} \cdot C_{kj} = 0 \quad \text{for } i \neq k$$

### Example of Alien Cofactor Property

Using the same matrix, multiply row 1 elements with row 2 cofactors:

$$a_{11}C_{21} + a_{12}C_{22} + a_{13}C_{23}$$
$$= 1(6) + 2(-12) + 3(6)$$
$$= 6 - 24 + 18 = 0$$ ✓

---

## 10. Worked Problems

### Problem 1

Find the minors and cofactors of all elements of:
$$A = \begin{bmatrix} 2 & -1 \\ 3 & 4 \end{bmatrix}$$

**Solution:**

| Element | Position | Minor | Sign | Cofactor |
|---------|----------|-------|------|----------|
| 2 | (1,1) | $M_{11} = 4$ | + | $C_{11} = 4$ |
| -1 | (1,2) | $M_{12} = 3$ | - | $C_{12} = -3$ |
| 3 | (2,1) | $M_{21} = -1$ | - | $C_{21} = 1$ |
| 4 | (2,2) | $M_{22} = 2$ | + | $C_{22} = 2$ |

### Problem 2

Find $C_{23}$ for:
$$A = \begin{bmatrix} 1 & 0 & 2 \\ 3 & 1 & -1 \\ 2 & 4 & 5 \end{bmatrix}$$

**Solution:**

Step 1: Find $M_{23}$ (delete row 2, column 3):
$$M_{23} = \begin{vmatrix} 1 & 0 \\ 2 & 4 \end{vmatrix} = 4 - 0 = 4$$

Step 2: Apply sign:
$$C_{23} = (-1)^{2+3} \cdot M_{23} = (-1)^5 \cdot 4 = -4$$

---

## 📊 Summary Table

| Concept | Definition | Formula |
|---------|------------|---------|
| Minor $M_{ij}$ | Determinant after deleting row i, col j | det of $(n-1) \times (n-1)$ submatrix |
| Cofactor $C_{ij}$ | Signed minor | $C_{ij} = (-1)^{i+j} \cdot M_{ij}$ |
| Sign Pattern | Checkerboard | $+$ if $i+j$ is even, $-$ if odd |
| Determinant | Sum of element × cofactor | $\|A\| = \sum a_{ij}C_{ij}$ |
| Alien Cofactors | Elements × wrong row cofactors | $= 0$ |

---

## ❓ Quick Revision Questions

1. **What is the minor of element $a_{12}$ in a 3×3 matrix?**
   <details>
   <summary>Click for Answer</summary>
   The minor $M_{12}$ is the determinant of the 2×2 matrix obtained by deleting row 1 and column 2.
   </details>

2. **What sign is associated with position (2, 3)?**
   <details>
   <summary>Click for Answer</summary>
   $(-1)^{2+3} = (-1)^5 = -1$, so the sign is negative.
   </details>

3. **If $M_{11} = 5$, what is $C_{11}$?**
   <details>
   <summary>Click for Answer</summary>
   $C_{11} = (-1)^{1+1} \cdot 5 = (+1) \cdot 5 = 5$
   </details>

4. **Find $C_{12}$ for $\begin{bmatrix} 3 & 4 \\ 5 & 6 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $M_{12} = 5$, $C_{12} = (-1)^{1+2} \cdot 5 = -5$
   </details>

5. **Why is $a_{11}C_{21} + a_{12}C_{22} + a_{13}C_{23} = 0$?**
   <details>
   <summary>Click for Answer</summary>
   This is the alien cofactor property: multiplying elements of one row with cofactors of a different row always gives zero.
   </details>

6. **For a 4×4 matrix, how many minors exist?**
   <details>
   <summary>Click for Answer</summary>
   16 minors (one for each element), and each minor is a 3×3 determinant.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Properties of Determinants](03-properties-of-determinants.md) | [Unit 3: Determinants](./README.md) | [Expansion Methods →](05-expansion-methods.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
