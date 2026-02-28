# Chapter 3.2: Determinant of 3×3 Matrix

[← Previous: Determinant of 2×2](01-determinant-2x2.md) | [Back to README](../README.md) | [Next: Properties of Determinants →](03-properties-of-determinants.md)

---

## 📚 Chapter Overview

Computing determinants of 3×3 matrices requires systematic methods. This chapter covers two main approaches: Sarrus' Rule (a quick method) and Cofactor Expansion (a general method that extends to larger matrices).

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Calculate 3×3 determinants using Sarrus' Rule
- Apply cofactor expansion method
- Choose the appropriate method for different problems
- Handle special cases efficiently

---

## 1. The 3×3 Matrix Setup

### General Form

$$A = \begin{bmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{bmatrix}$$

### The Determinant

$$|A| = \begin{vmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{vmatrix}$$

---

## 2. Method 1: Sarrus' Rule

### The Technique

Sarrus' Rule is a visual method that works **only for 3×3 matrices**.

### Step-by-Step Process

```
Step 1: Write the matrix and repeat the first two columns on the right

        | a₁₁  a₁₂  a₁₃ | a₁₁  a₁₂
        | a₂₁  a₂₂  a₂₃ | a₂₁  a₂₂
        | a₃₁  a₃₂  a₃₃ | a₃₁  a₃₂

Step 2: Draw diagonals and compute products

        Positive diagonals (↘):
        ┌─────────────────────────────┐
        │  a₁₁ ─→ a₂₂ ─→ a₃₃         │  Product: a₁₁·a₂₂·a₃₃
        │       a₁₂ ─→ a₂₃ ─→ a₃₁    │  Product: a₁₂·a₂₃·a₃₁
        │            a₁₃ ─→ a₂₁ ─→ a₃₂│  Product: a₁₃·a₂₁·a₃₂
        └─────────────────────────────┘

        Negative diagonals (↙):
        ┌─────────────────────────────┐
        │  a₁₃ ←─ a₂₂ ←─ a₃₁         │  Product: a₁₃·a₂₂·a₃₁
        │       a₁₁ ←─ a₂₃ ←─ a₃₂    │  Product: a₁₁·a₂₃·a₃₂
        │            a₁₂ ←─ a₂₁ ←─ a₃₃│  Product: a₁₂·a₂₁·a₃₃
        └─────────────────────────────┘

Step 3: Add positive products, subtract negative products
```

### Visual Diagram

```
        SARRUS' RULE - VISUAL METHOD
        
        Original Matrix    Repeated Columns
        ┌─────────────────┬─────────────┐
        │ a₁₁  a₁₂  a₁₃  │ a₁₁  a₁₂   │
        │   ╲    ╲    ╲   │   ╱    ╱    │
        │ a₂₁  a₂₂  a₂₃  │ a₂₁  a₂₂   │
        │   ╲    ╲    ╲   │   ╱    ╱    │
        │ a₃₁  a₃₂  a₃₃  │ a₃₁  a₃₂   │
        └─────────────────┴─────────────┘
           ↘    ↘    ↘       ↙    ↙
          (+)  (+)  (+)     (-)  (-)  (-)
        
        det = (a₁₁a₂₂a₃₃ + a₁₂a₂₃a₃₁ + a₁₃a₂₁a₃₂)
            - (a₁₃a₂₂a₃₁ + a₁₁a₂₃a₃₂ + a₁₂a₂₁a₃₃)
```

### Formula

$$|A| = a_{11}a_{22}a_{33} + a_{12}a_{23}a_{31} + a_{13}a_{21}a_{32}$$
$$- a_{13}a_{22}a_{31} - a_{11}a_{23}a_{32} - a_{12}a_{21}a_{33}$$

---

## 3. Sarrus' Rule Examples

### Example 1: Basic Calculation

$$\begin{vmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{vmatrix}$$

**Setup with repeated columns**:
```
        | 1  2  3 | 1  2
        | 4  5  6 | 4  5
        | 7  8  9 | 7  8
```

**Positive products** (↘):
- $1 \times 5 \times 9 = 45$
- $2 \times 6 \times 7 = 84$
- $3 \times 4 \times 8 = 96$
- Sum = 225

**Negative products** (↙):
- $3 \times 5 \times 7 = 105$
- $1 \times 6 \times 8 = 48$
- $2 \times 4 \times 9 = 72$
- Sum = 225

**Determinant**: $225 - 225 = 0$

### Example 2: Non-Zero Result

$$\begin{vmatrix} 2 & 1 & 3 \\ 1 & 0 & 2 \\ 4 & 1 & 5 \end{vmatrix}$$

**Positive products**:
- $2 \times 0 \times 5 = 0$
- $1 \times 2 \times 4 = 8$
- $3 \times 1 \times 1 = 3$
- Sum = 11

**Negative products**:
- $3 \times 0 \times 4 = 0$
- $2 \times 2 \times 1 = 4$
- $1 \times 1 \times 5 = 5$
- Sum = 9

**Determinant**: $11 - 9 = 2$

### Example 3: With Negatives

$$\begin{vmatrix} 1 & -2 & 3 \\ 2 & 1 & -1 \\ -1 & 2 & 1 \end{vmatrix}$$

**Positive products**:
- $1 \times 1 \times 1 = 1$
- $(-2) \times (-1) \times (-1) = -2$
- $3 \times 2 \times 2 = 12$
- Sum = 11

**Negative products**:
- $3 \times 1 \times (-1) = -3$
- $1 \times (-1) \times 2 = -2$
- $(-2) \times 2 \times 1 = -4$
- Sum = -9

**Determinant**: $11 - (-9) = 11 + 9 = 20$

---

## 4. Method 2: Cofactor Expansion

### The Method

Expand along any row or column using minors and cofactors.

### Expanding Along First Row

$$|A| = a_{11}C_{11} + a_{12}C_{12} + a_{13}C_{13}$$

Where $C_{ij}$ is the cofactor of element $a_{ij}$.

### What is a Cofactor?

$$C_{ij} = (-1)^{i+j} M_{ij}$$

Where $M_{ij}$ is the minor (determinant of the 2×2 matrix obtained by deleting row i and column j).

### Sign Pattern

```
        ┌───────────────┐
        │  +   -   +    │
        │  -   +   -    │        Sign = (-1)^(i+j)
        │  +   -   +    │
        └───────────────┘
        
        Checkerboard pattern starting with + at (1,1)
```

### Expansion Formula (First Row)

$$|A| = a_{11}\begin{vmatrix} a_{22} & a_{23} \\ a_{32} & a_{33} \end{vmatrix} - a_{12}\begin{vmatrix} a_{21} & a_{23} \\ a_{31} & a_{33} \end{vmatrix} + a_{13}\begin{vmatrix} a_{21} & a_{22} \\ a_{31} & a_{32} \end{vmatrix}$$

---

## 5. Cofactor Expansion Examples

### Example 1: Expand Along First Row

$$\begin{vmatrix} 2 & 1 & 3 \\ 1 & 0 & 2 \\ 4 & 1 & 5 \end{vmatrix}$$

**Step 1**: Write expansion formula
$$= 2 \cdot C_{11} + 1 \cdot C_{12} + 3 \cdot C_{13}$$

**Step 2**: Calculate each cofactor

$C_{11} = (+1)\begin{vmatrix} 0 & 2 \\ 1 & 5 \end{vmatrix} = 0 - 2 = -2$

$C_{12} = (-1)\begin{vmatrix} 1 & 2 \\ 4 & 5 \end{vmatrix} = -(5 - 8) = 3$

$C_{13} = (+1)\begin{vmatrix} 1 & 0 \\ 4 & 1 \end{vmatrix} = 1 - 0 = 1$

**Step 3**: Combine
$$|A| = 2(-2) + 1(3) + 3(1) = -4 + 3 + 3 = 2$$

### Example 2: Expand Along Column with Zeros

When a row or column has zeros, expand along it for easier calculation!

$$\begin{vmatrix} 3 & 0 & 2 \\ 1 & 0 & 4 \\ 5 & 0 & 6 \end{vmatrix}$$

**Expand along column 2** (all zeros!):

$$= 0 \cdot C_{12} + 0 \cdot C_{22} + 0 \cdot C_{32} = 0$$

The determinant is **0** (no calculation needed!)

### Example 3: Strategic Expansion

$$\begin{vmatrix} 1 & 2 & 0 \\ 3 & 4 & 0 \\ 5 & 6 & 7 \end{vmatrix}$$

**Expand along column 3** (has zeros):

$$= 0 \cdot C_{13} + 0 \cdot C_{23} + 7 \cdot C_{33}$$

$$= 7 \cdot (+1)\begin{vmatrix} 1 & 2 \\ 3 & 4 \end{vmatrix}$$

$$= 7 \cdot (4 - 6) = 7 \cdot (-2) = -14$$

---

## 6. Choosing the Best Method

```
┌─────────────────────────────────────────────────────────────────┐
│                CHOOSING A CALCULATION METHOD                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │  3×3 Matrix?    │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│                            YES                                   │
│                             │                                    │
│              ┌──────────────┴──────────────┐                    │
│              │                             │                    │
│              ▼                             ▼                    │
│    ┌─────────────────┐          ┌──────────────────┐           │
│    │ Has many zeros? │          │ No special zeros │           │
│    └────────┬────────┘          └────────┬─────────┘           │
│             │                            │                      │
│            YES                          NO                      │
│             │                            │                      │
│             ▼                            ▼                      │
│    ┌─────────────────┐          ┌──────────────────┐           │
│    │  USE COFACTOR   │          │  USE SARRUS'     │           │
│    │  EXPANSION      │          │  RULE            │           │
│    │  (along row/col │          │  (quick visual)  │           │
│    │   with zeros)   │          │                  │           │
│    └─────────────────┘          └──────────────────┘           │
│                                                                  │
│    Tip: Expand along the row or column with most zeros!        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Special Matrices

### Diagonal Matrix

$$\begin{vmatrix} a & 0 & 0 \\ 0 & b & 0 \\ 0 & 0 & c \end{vmatrix} = abc$$

**Determinant = Product of diagonal elements**

### Upper Triangular

$$\begin{vmatrix} a & b & c \\ 0 & d & e \\ 0 & 0 & f \end{vmatrix} = adf$$

### Lower Triangular

$$\begin{vmatrix} a & 0 & 0 \\ b & c & 0 \\ d & e & f \end{vmatrix} = acf$$

### Identity Matrix

$$\begin{vmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{vmatrix} = 1$$

---

## 8. Verification and Cross-Checking

### Same Answer from Different Expansions

For the matrix:
$$\begin{vmatrix} 1 & 2 & 3 \\ 0 & 4 & 5 \\ 0 & 0 & 6 \end{vmatrix}$$

**Method 1: Sarrus' Rule**
- Positive: $(1)(4)(6) + (2)(5)(0) + (3)(0)(0) = 24$
- Negative: $(3)(4)(0) + (1)(5)(0) + (2)(0)(6) = 0$
- Determinant = 24

**Method 2: Expand Row 3**
$$= 0 \cdot C_{31} + 0 \cdot C_{32} + 6 \cdot C_{33}$$
$$= 6 \cdot \begin{vmatrix} 1 & 2 \\ 0 & 4 \end{vmatrix} = 6 \cdot 4 = 24$$

**Method 3: Triangular Property**
$$= 1 \cdot 4 \cdot 6 = 24$$

All methods give **24** ✓

---

## 9. Common Mistakes to Avoid

```
┌─────────────────────────────────────────────────────────────────┐
│                    ⚠️ COMMON MISTAKES ⚠️                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. WRONG SIGN IN COFACTOR                                      │
│     Remember: Sign = (-1)^(i+j)                                 │
│     Position (1,2) has sign (-1)³ = -1                          │
│                                                                  │
│  2. WRONG DIAGONAL IN SARRUS                                    │
│     Draw carefully! Positive = ↘, Negative = ↙                 │
│                                                                  │
│  3. FORGETTING MINUS IN 2×2                                     │
│     |a b; c d| = ad - bc (not ad + bc!)                        │
│                                                                  │
│  4. USING SARRUS FOR 4×4                                        │
│     Sarrus ONLY works for 3×3!                                 │
│     For larger matrices, use cofactor expansion                 │
│                                                                  │
│  5. WRONG MINOR SELECTION                                       │
│     Delete the row AND column of the element                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Method | Best For | Formula/Approach |
|--------|----------|------------------|
| Sarrus' Rule | Quick 3×3 calculations | Diagonal products |
| Cofactor Expansion | Matrices with zeros | $\sum a_{ij}C_{ij}$ |
| Triangular | Diagonal/triangular matrices | Product of diagonal |

### Key Formulas

| Type | Determinant |
|------|-------------|
| General 3×3 | $a_{11}(a_{22}a_{33}-a_{23}a_{32}) - a_{12}(a_{21}a_{33}-a_{23}a_{31}) + a_{13}(a_{21}a_{32}-a_{22}a_{31})$ |
| Diagonal | $a_{11} \cdot a_{22} \cdot a_{33}$ |
| Triangular | Product of diagonal elements |

---

## ❓ Quick Revision Questions

1. **Calculate using Sarrus: $\begin{vmatrix} 1 & 0 & 2 \\ 3 & 1 & 0 \\ 0 & 2 & 1 \end{vmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   Positive: $(1)(1)(1) + (0)(0)(0) + (2)(3)(2) = 1 + 0 + 12 = 13$
   Negative: $(2)(1)(0) + (1)(0)(2) + (0)(3)(1) = 0 + 0 + 0 = 0$
   Determinant = $13 - 0 = 13$
   </details>

2. **What is the determinant of $\begin{bmatrix} 2 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 3 \end{bmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   It's a diagonal matrix: det = $2 \times 5 \times 3 = 30$
   </details>

3. **What is the sign of the cofactor $C_{23}$?**
   <details>
   <summary>Click for Answer</summary>
   Sign = $(-1)^{2+3} = (-1)^5 = -1$ (negative)
   </details>

4. **Which row/column would you expand along for $\begin{vmatrix} 0 & 2 & 0 \\ 1 & 0 & 3 \\ 0 & 4 & 0 \end{vmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   Column 3 or Row 1 (both have two zeros). Even better: Column 1 or Column 3 (two zeros each).
   </details>

5. **Does Sarrus' Rule work for 4×4 matrices?**
   <details>
   <summary>Click for Answer</summary>
   No! Sarrus' Rule only works for 3×3 matrices. For larger matrices, use cofactor expansion.
   </details>

6. **If the first column is all zeros, what is the determinant?**
   <details>
   <summary>Click for Answer</summary>
   Zero! Expanding along that column gives $0 \cdot C_{11} + 0 \cdot C_{21} + 0 \cdot C_{31} = 0$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Determinant of 2×2](01-determinant-2x2.md) | [Unit 3: Determinants](./README.md) | [Properties of Determinants →](03-properties-of-determinants.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
