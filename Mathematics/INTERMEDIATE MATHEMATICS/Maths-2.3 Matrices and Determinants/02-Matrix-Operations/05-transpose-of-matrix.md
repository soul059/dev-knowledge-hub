# Chapter 2.5: Transpose of a Matrix

[← Previous: Properties of Operations](04-properties-of-operations.md) | [Back to README](../README.md) | [Next: Unit 3 - Determinant 2×2 →](../03-Determinants/01-determinant-2x2.md)

---

## 📚 Chapter Overview

The transpose operation flips a matrix over its main diagonal, turning rows into columns and vice versa. This operation is fundamental in many matrix applications and leads to important matrix types like symmetric and skew-symmetric matrices.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Compute the transpose of any matrix
- Apply properties of transpose
- Identify symmetric and skew-symmetric matrices
- Use transpose in matrix equations

---

## 1. Definition of Transpose

### Formal Definition

> The **transpose** of a matrix A, denoted $A^T$ or $A'$, is obtained by interchanging its rows and columns.

### Mathematical Definition

If $A = [a_{ij}]_{m \times n}$, then $A^T = [a_{ji}]_{n \times m}$

The element in row i, column j of A becomes the element in row j, column i of $A^T$.

### Visual Representation

```
        Original Matrix A              Transpose A^T
            (m × n)                      (n × m)
        
        ┌─────────────────┐           ┌───────────────┐
        │ a₁₁  a₁₂  a₁₃  │           │ a₁₁  a₂₁  a₃₁ │
        │ a₂₁  a₂₂  a₂₃  │    →      │ a₁₂  a₂₂  a₃₂ │
        │ a₃₁  a₃₂  a₃₃  │           │ a₁₃  a₂₃  a₃₃ │
        └─────────────────┘           └───────────────┘
        
               ↓ Flip over main diagonal ↓
        
        Row 1 of A → Column 1 of A^T
        Row 2 of A → Column 2 of A^T
        Row 3 of A → Column 3 of A^T
```

---

## 2. Basic Examples

### Example 1: 2×3 Matrix

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}_{2 \times 3}$$

$$A^T = \begin{bmatrix} 1 & 4 \\ 2 & 5 \\ 3 & 6 \end{bmatrix}_{3 \times 2}$$

```
        A (2×3)                    A^T (3×2)
        
        ┌─────────────┐           ┌───────┐
        │  1   2   3  │           │ 1   4 │
        │  4   5   6  │    →      │ 2   5 │
        └─────────────┘           │ 3   6 │
                                  └───────┘
```

### Example 2: 3×3 Matrix

$$B = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$

$$B^T = \begin{bmatrix} 1 & 4 & 7 \\ 2 & 5 & 8 \\ 3 & 6 & 9 \end{bmatrix}$$

```
        Original B                 Transpose B^T
        
        ┌─────────────┐           ┌─────────────┐
        │  1   2   3  │           │  1   4   7  │
        │  4   5   6  │    →      │  2   5   8  │
        │  7   8   9  │           │  3   6   9  │
        └─────────────┘           └─────────────┘
        
        Notice: Main diagonal (1, 5, 9) stays the same!
```

### Example 3: Row and Column Matrices

**Row matrix**:
$$R = \begin{bmatrix} 1 & 2 & 3 & 4 \end{bmatrix}_{1 \times 4}$$

$$R^T = \begin{bmatrix} 1 \\ 2 \\ 3 \\ 4 \end{bmatrix}_{4 \times 1}$$

**Column matrix**:
$$C = \begin{bmatrix} a \\ b \\ c \end{bmatrix}_{3 \times 1}$$

$$C^T = \begin{bmatrix} a & b & c \end{bmatrix}_{1 \times 3}$$

---

## 3. Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRANSPOSE ALGORITHM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Given: Matrix A of order m × n                                │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 1: Create new matrix of order n × m               │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 2: For each element a_ij in A:                    │   │
│   │          Place it at position (j, i) in A^T             │   │
│   │          i.e., swap row and column indices              │   │
│   └────────────────────────┬────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Step 3: Result is A^T                                  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   Shortcut: Write each row of A as a column in A^T              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Properties of Transpose

### Property 1: Double Transpose

> $(A^T)^T = A$

Taking transpose twice returns the original matrix.

### Property 2: Transpose of Sum

> $(A + B)^T = A^T + B^T$

### Property 3: Transpose of Scalar Multiple

> $(kA)^T = kA^T$

### Property 4: Transpose of Product (IMPORTANT!)

> $(AB)^T = B^T A^T$

**Note**: The order reverses!

### Property 5: Transpose of Identity

> $I^T = I$

### Property 6: Order of Transpose

> If A is $m \times n$, then $A^T$ is $n \times m$

### Properties Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                  TRANSPOSE PROPERTIES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    Property              Formula                                 │
│    ────────────────────────────────────────────────────         │
│    Double transpose      (A^T)^T = A                            │
│    Sum                   (A + B)^T = A^T + B^T                  │
│    Scalar multiple       (kA)^T = kA^T                          │
│    Product               (AB)^T = B^T A^T    ← ORDER REVERSES!  │
│    Triple product        (ABC)^T = C^T B^T A^T                  │
│    Identity              I^T = I                                 │
│    Zero matrix           O^T = O                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Proof: $(AB)^T = B^T A^T$

Let A be $m \times n$ and B be $n \times p$.

- AB is $m \times p$
- $(AB)^T$ is $p \times m$
- $A^T$ is $n \times m$
- $B^T$ is $p \times n$
- $B^T A^T$ is $p \times m$ ✓ (dimensions match)

The (i,j) element of $(AB)^T$ equals the (j,i) element of AB:
$$[(AB)^T]_{ij} = [AB]_{ji} = \sum_{k=1}^n a_{jk}b_{ki}$$

The (i,j) element of $B^TA^T$:
$$[B^TA^T]_{ij} = \sum_{k=1}^n b^T_{ik}a^T_{kj} = \sum_{k=1}^n b_{ki}a_{jk}$$

These are equal! ✓

---

## 5. Symmetric Matrices

### Definition

> A square matrix A is **symmetric** if $A^T = A$

### Condition

$a_{ij} = a_{ji}$ for all i, j

The matrix is "mirrored" across the main diagonal.

### Examples

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 5 \\ 3 & 5 & 6 \end{bmatrix}$$

Check: $a_{12} = 2 = a_{21}$, $a_{13} = 3 = a_{31}$, $a_{23} = 5 = a_{32}$ ✓

```
        ┌─────────────────┐
        │  1  │  2  │  3  │      Mirror across
        │─────┼─────┼─────│      the main diagonal
        │  2  │  4  │  5  │      
        │─────┼─────┼─────│      Elements reflect:
        │  3  │  5  │  6  │      a₁₂ ↔ a₂₁
        └─────────────────┘      a₁₃ ↔ a₃₁
                                 a₂₃ ↔ a₃₂
```

### Special Symmetric Matrices

- **Identity matrix**: $I^T = I$ ✓
- **Diagonal matrices**: All diagonal matrices are symmetric
- **Scalar matrices**: Symmetric
- **Zero matrix**: Symmetric

---

## 6. Skew-Symmetric Matrices

### Definition

> A square matrix A is **skew-symmetric** if $A^T = -A$

### Conditions

1. $a_{ij} = -a_{ji}$ for all i, j
2. All diagonal elements are zero ($a_{ii} = 0$)

### Proof of Diagonal Being Zero

For a skew-symmetric matrix: $a_{ii} = -a_{ii}$
Therefore: $2a_{ii} = 0$, so $a_{ii} = 0$

### Example

$$B = \begin{bmatrix} 0 & 2 & -3 \\ -2 & 0 & 4 \\ 3 & -4 & 0 \end{bmatrix}$$

Check:
- Diagonal: 0, 0, 0 ✓
- $b_{12} = 2 = -b_{21} = -(-2) = 2$ ✓
- $b_{13} = -3 = -b_{31} = -(3) = -3$ ✓
- $b_{23} = 4 = -b_{32} = -(-4) = 4$ ✓

```
        ┌─────────────────────┐
        │  0  │  2  │ -3  │        All diagonal = 0
        │─────┼─────┼─────│        
        │ -2  │  0  │  4  │        Off-diagonal elements
        │─────┼─────┼─────│        are negatives of each other
        │  3  │ -4  │  0  │        
        └─────────────────────┘
```

---

## 7. Decomposition Theorem

### Every Square Matrix = Symmetric + Skew-Symmetric

> Any square matrix A can be written as:
> $$A = \frac{1}{2}(A + A^T) + \frac{1}{2}(A - A^T)$$
> Where:
> - $\frac{1}{2}(A + A^T)$ is symmetric
> - $\frac{1}{2}(A - A^T)$ is skew-symmetric

### Proof

Let $P = \frac{1}{2}(A + A^T)$ and $Q = \frac{1}{2}(A - A^T)$

**P is symmetric**:
$$P^T = \frac{1}{2}(A + A^T)^T = \frac{1}{2}(A^T + A) = P$$ ✓

**Q is skew-symmetric**:
$$Q^T = \frac{1}{2}(A - A^T)^T = \frac{1}{2}(A^T - A) = -\frac{1}{2}(A - A^T) = -Q$$ ✓

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$$

$$A^T = \begin{bmatrix} 1 & 3 \\ 2 & 4 \end{bmatrix}$$

**Symmetric part**:
$$P = \frac{1}{2}\begin{bmatrix} 1+1 & 2+3 \\ 3+2 & 4+4 \end{bmatrix} = \begin{bmatrix} 1 & 2.5 \\ 2.5 & 4 \end{bmatrix}$$

**Skew-symmetric part**:
$$Q = \frac{1}{2}\begin{bmatrix} 1-1 & 2-3 \\ 3-2 & 4-4 \end{bmatrix} = \begin{bmatrix} 0 & -0.5 \\ 0.5 & 0 \end{bmatrix}$$

**Verification**: $P + Q = A$ ✓

---

## 8. Important Results

### For Symmetric Matrices

| Result | Proof/Explanation |
|--------|-------------------|
| $A + B$ is symmetric | $(A+B)^T = A^T + B^T = A + B$ |
| $kA$ is symmetric | $(kA)^T = kA^T = kA$ |
| $A^n$ is symmetric | $(A^n)^T = (A^T)^n = A^n$ |
| $A^{-1}$ is symmetric (if exists) | $(A^{-1})^T = (A^T)^{-1} = A^{-1}$ |

### For Skew-Symmetric Matrices

| Result | Proof/Explanation |
|--------|-------------------|
| $A + B$ is skew-symmetric | $(A+B)^T = -(A+B)$ |
| $kA$ is skew-symmetric | $(kA)^T = -kA$ |
| Diagonal elements are 0 | $a_{ii} = -a_{ii}$ |
| $A^2$ is symmetric | $(A^2)^T = (A^T)^2 = (-A)^2 = A^2$ |

### Mixed Results

| Matrices | Product Type |
|----------|--------------|
| Symmetric × Symmetric | Not necessarily symmetric |
| Skew × Skew | Not necessarily skew |
| $A^TA$ always | Symmetric |
| $AA^T$ always | Symmetric |

---

## 9. Applications

### Application 1: Dot Product using Transpose

For column vectors $\mathbf{u}$ and $\mathbf{v}$:

$$\mathbf{u} \cdot \mathbf{v} = \mathbf{u}^T \mathbf{v}$$

$$\begin{bmatrix} u_1 \\ u_2 \\ u_3 \end{bmatrix} \cdot \begin{bmatrix} v_1 \\ v_2 \\ v_3 \end{bmatrix} = \begin{bmatrix} u_1 & u_2 & u_3 \end{bmatrix} \begin{bmatrix} v_1 \\ v_2 \\ v_3 \end{bmatrix} = u_1v_1 + u_2v_2 + u_3v_3$$

### Application 2: Matrix Equations

To solve $Ax = b$ using least squares:

$$A^TAx = A^Tb$$

### Application 3: Covariance Matrix

In statistics, the covariance matrix is always symmetric:

$$\Sigma = \frac{1}{n}X^TX$$

---

## 📊 Summary Table

| Concept | Definition | Example |
|---------|------------|---------|
| Transpose | Rows ↔ Columns | $(A_{m×n})^T = A^T_{n×m}$ |
| $(A^T)^T$ | Double transpose | Returns A |
| $(AB)^T$ | Product transpose | $= B^TA^T$ (reverses!) |
| Symmetric | $A^T = A$ | $\begin{bmatrix} 1 & 2 \\ 2 & 3 \end{bmatrix}$ |
| Skew-Symmetric | $A^T = -A$ | $\begin{bmatrix} 0 & 2 \\ -2 & 0 \end{bmatrix}$ |
| Decomposition | Any square matrix | $A = P + Q$ where P is symmetric, Q is skew-symmetric |

---

## ❓ Quick Revision Questions

1. **Find the transpose of $\begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $\begin{bmatrix} 1 & 4 \\ 2 & 5 \\ 3 & 6 \end{bmatrix}$
   </details>

2. **If A is 3×5, what is the order of $A^T$?**
   <details>
   <summary>Click for Answer</summary>
   5×3 (rows and columns are swapped)
   </details>

3. **Is $\begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 5 \\ 3 & 5 & 6 \end{bmatrix}$ symmetric?**
   <details>
   <summary>Click for Answer</summary>
   Yes! $a_{12}=a_{21}=2$, $a_{13}=a_{31}=3$, $a_{23}=a_{32}=5$
   </details>

4. **What is $(AB)^T$ in terms of $A^T$ and $B^T$?**
   <details>
   <summary>Click for Answer</summary>
   $(AB)^T = B^TA^T$ (order reverses!)
   </details>

5. **Why are all diagonal elements of a skew-symmetric matrix zero?**
   <details>
   <summary>Click for Answer</summary>
   For skew-symmetric: $a_{ij} = -a_{ji}$
   For diagonal: $a_{ii} = -a_{ii}$
   This means $2a_{ii} = 0$, so $a_{ii} = 0$
   </details>

6. **Express $A = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}$ as sum of symmetric and skew-symmetric matrices.**
   <details>
   <summary>Click for Answer</summary>
   Symmetric: $\frac{1}{2}(A + A^T) = \begin{bmatrix} 2 & 5 \\ 5 & 8 \end{bmatrix}$
   Skew-symmetric: $\frac{1}{2}(A - A^T) = \begin{bmatrix} 0 & -1 \\ 1 & 0 \end{bmatrix}$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Properties of Operations](04-properties-of-operations.md) | [Unit 2: Operations](./README.md) | [Unit 3: Determinant 2×2 →](../03-Determinants/01-determinant-2x2.md) |

---

**🎉 Congratulations!** You have completed **Unit 2: Matrix Operations**

### Unit 2 Checklist:
- [x] Addition and Subtraction
- [x] Scalar Multiplication
- [x] Matrix Multiplication
- [x] Properties of Operations
- [x] Transpose of Matrix

**[Continue to Unit 3: Determinants →](../03-Determinants/01-determinant-2x2.md)**

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
