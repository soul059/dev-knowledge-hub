# Chapter 5.1: Adjoint of a Matrix

[← Previous: Cramer's Rule](../04-Determinant-Applications/05-cramers-rule.md) | [Back to README](../README.md) | [Next: Inverse of a Matrix →](02-inverse-of-matrix.md)

---

## 📚 Chapter Overview

The adjoint (or adjugate) of a matrix is a fundamental concept that provides a pathway to finding the inverse of a matrix. Understanding the adjoint is essential for both theoretical and practical applications in linear algebra.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define the adjoint of a matrix
- Construct the cofactor matrix and adjoint
- Calculate adjoints for 2×2 and 3×3 matrices
- Apply properties of the adjoint

---

## 1. Definition of Adjoint

### Formal Definition

> The **adjoint** (or **adjugate**) of a square matrix A, denoted adj(A), is the **transpose of the cofactor matrix** of A.

$$\text{adj}(A) = C^T$$

where C is the matrix of cofactors of A.

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONSTRUCTING THE ADJOINT                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Step 1: For each element a_ij, find its minor M_ij            │
│                                                                  │
│   Step 2: Calculate cofactor C_ij = (-1)^(i+j) × M_ij           │
│                                                                  │
│   Step 3: Form the cofactor matrix C                            │
│                                                                  │
│   Step 4: Take the transpose: adj(A) = C^T                      │
│                                                                  │
│   Key: The (i,j) element of adj(A) is C_ji (not C_ij!)         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Visual Representation

```
    Original Matrix A              Cofactor Matrix C           Adjoint = C^T
    
    ┌─────────────────┐           ┌─────────────────┐       ┌─────────────────┐
    │ a₁₁  a₁₂  a₁₃  │           │ C₁₁  C₁₂  C₁₃  │       │ C₁₁  C₂₁  C₃₁  │
    │ a₂₁  a₂₂  a₂₃  │    →      │ C₂₁  C₂₂  C₂₃  │   →   │ C₁₂  C₂₂  C₃₂  │
    │ a₃₁  a₃₂  a₃₃  │           │ C₃₁  C₃₂  C₃₃  │       │ C₁₃  C₂₃  C₃₃  │
    └─────────────────┘           └─────────────────┘       └─────────────────┘
```

---

## 3. Adjoint of a 2×2 Matrix

### Formula

For $A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$:

$$\text{adj}(A) = \begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$$

### Derivation

Cofactors:
- $C_{11} = (+1) \cdot d = d$
- $C_{12} = (-1) \cdot c = -c$
- $C_{21} = (-1) \cdot b = -b$
- $C_{22} = (+1) \cdot a = a$

Cofactor matrix: $C = \begin{bmatrix} d & -c \\ -b & a \end{bmatrix}$

Transpose: $\text{adj}(A) = C^T = \begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$

### Memory Aid

```
    Original              Adjoint
    
    | a   b |            | d  -b |
    | c   d |     →      |-c   a |
    
    Rule: Swap diagonal, negate off-diagonal
```

### Example 1

Find adj(A) for $A = \begin{bmatrix} 3 & 2 \\ 1 & 4 \end{bmatrix}$

$$\text{adj}(A) = \begin{bmatrix} 4 & -2 \\ -1 & 3 \end{bmatrix}$$

---

## 4. Adjoint of a 3×3 Matrix

### Example 2

Find adj(A) for $A = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 4 & 5 \\ 1 & 0 & 6 \end{bmatrix}$

**Step 1: Calculate all cofactors**

$C_{11} = (+)\begin{vmatrix} 4 & 5 \\ 0 & 6 \end{vmatrix} = 24 - 0 = 24$

$C_{12} = (-)\begin{vmatrix} 0 & 5 \\ 1 & 6 \end{vmatrix} = -(0 - 5) = 5$

$C_{13} = (+)\begin{vmatrix} 0 & 4 \\ 1 & 0 \end{vmatrix} = 0 - 4 = -4$

$C_{21} = (-)\begin{vmatrix} 2 & 3 \\ 0 & 6 \end{vmatrix} = -(12 - 0) = -12$

$C_{22} = (+)\begin{vmatrix} 1 & 3 \\ 1 & 6 \end{vmatrix} = 6 - 3 = 3$

$C_{23} = (-)\begin{vmatrix} 1 & 2 \\ 1 & 0 \end{vmatrix} = -(0 - 2) = 2$

$C_{31} = (+)\begin{vmatrix} 2 & 3 \\ 4 & 5 \end{vmatrix} = 10 - 12 = -2$

$C_{32} = (-)\begin{vmatrix} 1 & 3 \\ 0 & 5 \end{vmatrix} = -(5 - 0) = -5$

$C_{33} = (+)\begin{vmatrix} 1 & 2 \\ 0 & 4 \end{vmatrix} = 4 - 0 = 4$

**Step 2: Form cofactor matrix**

$$C = \begin{bmatrix} 24 & 5 & -4 \\ -12 & 3 & 2 \\ -2 & -5 & 4 \end{bmatrix}$$

**Step 3: Transpose to get adjoint**

$$\text{adj}(A) = C^T = \begin{bmatrix} 24 & -12 & -2 \\ 5 & 3 & -5 \\ -4 & 2 & 4 \end{bmatrix}$$

---

## 5. Key Property: A · adj(A)

### The Fundamental Identity

> $$A \cdot \text{adj}(A) = \text{adj}(A) \cdot A = |A| \cdot I$$

### Verification for 2×2

$$A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}, \quad \text{adj}(A) = \begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$$

$$A \cdot \text{adj}(A) = \begin{bmatrix} a & b \\ c & d \end{bmatrix}\begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$$

$$= \begin{bmatrix} ad - bc & -ab + ab \\ cd - cd & -bc + ad \end{bmatrix} = \begin{bmatrix} ad-bc & 0 \\ 0 & ad-bc \end{bmatrix}$$

$$= (ad - bc)\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} = |A| \cdot I$$ ✓

---

## 6. Properties of Adjoint

### Property 1: Product with Original Matrix
$$A \cdot \text{adj}(A) = \text{adj}(A) \cdot A = |A| \cdot I$$

### Property 2: Adjoint of Identity
$$\text{adj}(I) = I$$

### Property 3: Adjoint of Scalar Multiple
$$\text{adj}(kA) = k^{n-1} \cdot \text{adj}(A)$$ for n×n matrix

### Property 4: Adjoint of Transpose
$$\text{adj}(A^T) = (\text{adj}(A))^T$$

### Property 5: Adjoint of Product
$$\text{adj}(AB) = \text{adj}(B) \cdot \text{adj}(A)$$

Note: Order reverses!

### Property 6: Determinant of Adjoint
$$|\text{adj}(A)| = |A|^{n-1}$$ for n×n matrix

For 2×2: $|\text{adj}(A)| = |A|$
For 3×3: $|\text{adj}(A)| = |A|^2$

### Property 7: Adjoint of Adjoint
$$\text{adj}(\text{adj}(A)) = |A|^{n-2} \cdot A$$ for n×n matrix (n ≥ 2)

---

## 7. Properties Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                 ADJOINT PROPERTIES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. A · adj(A) = |A| · I                                       │
│                                                                  │
│   2. adj(I) = I                                                 │
│                                                                  │
│   3. adj(kA) = k^(n-1) · adj(A)                                │
│                                                                  │
│   4. adj(A^T) = (adj(A))^T                                      │
│                                                                  │
│   5. adj(AB) = adj(B) · adj(A)                                  │
│                                                                  │
│   6. |adj(A)| = |A|^(n-1)                                       │
│                                                                  │
│   7. adj(adj(A)) = |A|^(n-2) · A                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Special Cases

### Diagonal Matrix

For a diagonal matrix $D = \text{diag}(d_1, d_2, d_3)$:

$$\text{adj}(D) = \begin{bmatrix} d_2 d_3 & 0 & 0 \\ 0 & d_1 d_3 & 0 \\ 0 & 0 & d_1 d_2 \end{bmatrix}$$

### Triangular Matrix

The adjoint of a triangular matrix is also triangular (with same orientation).

### Singular Matrix

If $|A| = 0$, then:
$$A \cdot \text{adj}(A) = 0 \cdot I = O$$

The adjoint still exists, but $A \cdot \text{adj}(A) = O$ (zero matrix).

---

## 9. Connection to Inverse

### The Key Formula

If $|A| \neq 0$:
$$A^{-1} = \frac{1}{|A|} \text{adj}(A)$$

This is derived from:
$$A \cdot \text{adj}(A) = |A| \cdot I$$
$$A \cdot \frac{\text{adj}(A)}{|A|} = I$$
$$\therefore A^{-1} = \frac{\text{adj}(A)}{|A|}$$

---

## 10. Worked Examples

### Example 3

If $A = \begin{bmatrix} 2 & 3 \\ 1 & 4 \end{bmatrix}$, verify that $A \cdot \text{adj}(A) = |A| \cdot I$.

**Solution**:

$|A| = 8 - 3 = 5$

$\text{adj}(A) = \begin{bmatrix} 4 & -3 \\ -1 & 2 \end{bmatrix}$

$A \cdot \text{adj}(A) = \begin{bmatrix} 2 & 3 \\ 1 & 4 \end{bmatrix}\begin{bmatrix} 4 & -3 \\ -1 & 2 \end{bmatrix}$

$= \begin{bmatrix} 8-3 & -6+6 \\ 4-4 & -3+8 \end{bmatrix} = \begin{bmatrix} 5 & 0 \\ 0 & 5 \end{bmatrix} = 5I$ ✓

### Example 4

If $|A| = 3$ for a 3×3 matrix A, find $|\text{adj}(A)|$.

**Solution**:
$$|\text{adj}(A)| = |A|^{n-1} = 3^{3-1} = 3^2 = 9$$

---

## 📊 Summary Table

| Concept | 2×2 Matrix | n×n Matrix |
|---------|------------|------------|
| adj(A) definition | Swap diag, negate off-diag | Transpose of cofactor matrix |
| A · adj(A) | $\|A\| \cdot I$ | $\|A\| \cdot I$ |
| \|adj(A)\| | $\|A\|$ | $\|A\|^{n-1}$ |
| adj(kA) | $k \cdot \text{adj}(A)$ | $k^{n-1} \cdot \text{adj}(A)$ |

---

## ❓ Quick Revision Questions

1. **What is the adjoint of $\begin{bmatrix} 5 & 3 \\ 2 & 4 \end{bmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   $\text{adj}(A) = \begin{bmatrix} 4 & -3 \\ -2 & 5 \end{bmatrix}$
   </details>

2. **What is $A \cdot \text{adj}(A)$ equal to?**
   <details>
   <summary>Click for Answer</summary>
   $|A| \cdot I$ (determinant of A times the identity matrix)
   </details>

3. **If A is 3×3 and $|A| = 2$, what is $|\text{adj}(A)|$?**
   <details>
   <summary>Click for Answer</summary>
   $|\text{adj}(A)| = |A|^{3-1} = 2^2 = 4$
   </details>

4. **What is the adjoint of the identity matrix?**
   <details>
   <summary>Click for Answer</summary>
   adj(I) = I
   </details>

5. **Is adj(AB) = adj(A) · adj(B)?**
   <details>
   <summary>Click for Answer</summary>
   No! adj(AB) = adj(B) · adj(A). The order reverses.
   </details>

6. **If $|A| = 0$, what is $A \cdot \text{adj}(A)$?**
   <details>
   <summary>Click for Answer</summary>
   $A \cdot \text{adj}(A) = 0 \cdot I = O$ (zero matrix)
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Cramer's Rule](../04-Determinant-Applications/05-cramers-rule.md) | [Unit 5: Adjoint & Inverse](./README.md) | [Inverse of a Matrix →](02-inverse-of-matrix.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
