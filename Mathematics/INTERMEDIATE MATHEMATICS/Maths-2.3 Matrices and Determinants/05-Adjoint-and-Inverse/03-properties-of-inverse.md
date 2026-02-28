# Chapter 5.3: Properties of Inverse

[← Previous: Inverse of a Matrix](02-inverse-of-matrix.md) | [Back to README](../README.md) | [Next: Solving Equations →](04-solving-equations.md)

---

## 📚 Chapter Overview

The inverse of a matrix possesses many elegant properties that are essential for both theoretical understanding and practical computation. This chapter systematically presents these properties with proofs and examples.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State and prove key properties of matrix inverses
- Apply these properties to simplify calculations
- Understand the relationship between inverse and other operations
- Recognize patterns in inverse computations

---

## 1. Fundamental Properties

### Property 1: Uniqueness

> If A is invertible, then its inverse is **unique**.

**Proof**: Suppose B and C are both inverses of A.
$$AB = BA = I \quad \text{and} \quad AC = CA = I$$

Then:
$$B = BI = B(AC) = (BA)C = IC = C$$

Therefore $B = C$. ✓

### Property 2: Inverse of Inverse

> $(A^{-1})^{-1} = A$

**Proof**: We need to show that A is the inverse of $A^{-1}$.
$$A^{-1} \cdot A = A \cdot A^{-1} = I$$

This is exactly the definition of $A$ being the inverse of $A^{-1}$. ✓

---

## 2. Product Properties

### Property 3: Inverse of Product

> $(AB)^{-1} = B^{-1}A^{-1}$

**The order reverses!**

**Proof**:
$$(AB)(B^{-1}A^{-1}) = A(BB^{-1})A^{-1} = AIA^{-1} = AA^{-1} = I$$
$$(B^{-1}A^{-1})(AB) = B^{-1}(A^{-1}A)B = B^{-1}IB = B^{-1}B = I$$

Therefore $(AB)^{-1} = B^{-1}A^{-1}$. ✓

### Visual

```
┌─────────────────────────────────────────────────────────────────┐
│               INVERSE OF PRODUCT (ORDER REVERSES)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Think of it like putting on/taking off clothes:               │
│                                                                  │
│   Put on:    Shirt → Jacket → Coat                              │
│                  A       B       C                               │
│                                                                  │
│   Take off:  Coat → Jacket → Shirt                              │
│               C⁻¹     B⁻¹     A⁻¹                               │
│                                                                  │
│   (ABC)⁻¹ = C⁻¹ B⁻¹ A⁻¹                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Extension to Multiple Matrices

$$(A_1 A_2 \cdots A_n)^{-1} = A_n^{-1} \cdots A_2^{-1} A_1^{-1}$$

---

## 3. Transpose Properties

### Property 4: Inverse of Transpose

> $(A^T)^{-1} = (A^{-1})^T$

**Proof**:
We need to show that $(A^{-1})^T$ is the inverse of $A^T$.

$$A^T \cdot (A^{-1})^T = (A^{-1} \cdot A)^T = I^T = I$$
$$(A^{-1})^T \cdot A^T = (A \cdot A^{-1})^T = I^T = I$$

Therefore $(A^T)^{-1} = (A^{-1})^T$. ✓

### Memory Aid

> "Transpose of inverse = Inverse of transpose"

---

## 4. Scalar Properties

### Property 5: Inverse of Scalar Multiple

> $(kA)^{-1} = \frac{1}{k}A^{-1}$ (for $k \neq 0$)

**Proof**:
$$(kA)\left(\frac{1}{k}A^{-1}\right) = \frac{k}{k}(AA^{-1}) = 1 \cdot I = I$$ ✓

### Example

If $A^{-1} = \begin{bmatrix} 2 & 1 \\ 3 & 4 \end{bmatrix}$, find $(3A)^{-1}$.

$$(3A)^{-1} = \frac{1}{3}A^{-1} = \frac{1}{3}\begin{bmatrix} 2 & 1 \\ 3 & 4 \end{bmatrix} = \begin{bmatrix} \frac{2}{3} & \frac{1}{3} \\ 1 & \frac{4}{3} \end{bmatrix}$$

---

## 5. Power Properties

### Property 6: Inverse of Power

> $(A^n)^{-1} = (A^{-1})^n$

**Proof** (for positive integer n):
$$A^n \cdot (A^{-1})^n = \underbrace{A \cdot A \cdots A}_{n \text{ times}} \cdot \underbrace{A^{-1} \cdot A^{-1} \cdots A^{-1}}_{n \text{ times}} = I$$ ✓

### Notation

We often write: $A^{-n} = (A^{-1})^n = (A^n)^{-1}$

### Example

If $A^{-1} = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$, find $A^{-2}$.

$$A^{-2} = (A^{-1})^2 = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix} = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} = I$$

---

## 6. Determinant Properties

### Property 7: Determinant of Inverse

> $|A^{-1}| = \frac{1}{|A|}$

**Proof**:
$$|A \cdot A^{-1}| = |I| = 1$$
$$|A| \cdot |A^{-1}| = 1$$
$$|A^{-1}| = \frac{1}{|A|}$$ ✓

### Property 8: Determinant of Adjoint

> $|\text{adj}(A)| = |A|^{n-1}$ for n×n matrix

Since $A^{-1} = \frac{1}{|A|}\text{adj}(A)$:
$$|A^{-1}| = \frac{1}{|A|^n}|\text{adj}(A)|$$
$$\frac{1}{|A|} = \frac{|\text{adj}(A)|}{|A|^n}$$
$$|\text{adj}(A)| = |A|^{n-1}$$ ✓

---

## 7. Special Matrices

### Property 9: Inverse of Identity

> $I^{-1} = I$

**Proof**: $I \cdot I = I$ ✓

### Property 10: Inverse of Diagonal Matrix

For $D = \text{diag}(d_1, d_2, ..., d_n)$ with all $d_i \neq 0$:

> $D^{-1} = \text{diag}\left(\frac{1}{d_1}, \frac{1}{d_2}, ..., \frac{1}{d_n}\right)$

### Example

$$D = \begin{bmatrix} 2 & 0 & 0 \\ 0 & 5 & 0 \\ 0 & 0 & 3 \end{bmatrix} \Rightarrow D^{-1} = \begin{bmatrix} \frac{1}{2} & 0 & 0 \\ 0 & \frac{1}{5} & 0 \\ 0 & 0 & \frac{1}{3} \end{bmatrix}$$

### Property 11: Inverse of Triangular Matrix

> The inverse of a triangular matrix is also triangular (same type).

Upper triangular → Inverse is upper triangular
Lower triangular → Inverse is lower triangular

---

## 8. Symmetric and Orthogonal Matrices

### Property 12: Inverse of Symmetric Matrix

> If A is symmetric and invertible, then $A^{-1}$ is also symmetric.

**Proof**:
$$(A^{-1})^T = (A^T)^{-1} = A^{-1}$$

(Since $A^T = A$ for symmetric matrices) ✓

### Property 13: Orthogonal Matrices

A matrix Q is **orthogonal** if $Q^T = Q^{-1}$.

Properties of orthogonal matrices:
- $Q^T Q = QQ^T = I$
- $|Q| = \pm 1$
- Rows and columns form orthonormal sets
- Preserves lengths and angles

---

## 9. Summary of Properties

```
┌─────────────────────────────────────────────────────────────────┐
│                 ALL INVERSE PROPERTIES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Basic:                                                         │
│   • (A⁻¹)⁻¹ = A                                                 │
│   • (AB)⁻¹ = B⁻¹A⁻¹           (order reverses!)                │
│   • (Aⁿ)⁻¹ = (A⁻¹)ⁿ                                            │
│                                                                  │
│   With Transpose:                                                │
│   • (Aᵀ)⁻¹ = (A⁻¹)ᵀ                                            │
│                                                                  │
│   With Scalars:                                                  │
│   • (kA)⁻¹ = (1/k)A⁻¹                                          │
│                                                                  │
│   With Determinants:                                             │
│   • |A⁻¹| = 1/|A|                                               │
│                                                                  │
│   Special Matrices:                                              │
│   • I⁻¹ = I                                                     │
│   • (diagonal)⁻¹ = reciprocals on diagonal                      │
│   • (symmetric)⁻¹ = symmetric                                   │
│   • (orthogonal)⁻¹ = transpose                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Worked Examples

### Example 1

If $A^{-1} = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}$, find $(A^T)^{-1}$.

**Solution**:
$$(A^T)^{-1} = (A^{-1})^T = \begin{bmatrix} 1 & 3 \\ 2 & 4 \end{bmatrix}$$

### Example 2

If $|A| = 5$ for a 3×3 matrix, find $|A^{-1}|$ and $|\text{adj}(A)|$.

**Solution**:
$$|A^{-1}| = \frac{1}{|A|} = \frac{1}{5}$$
$$|\text{adj}(A)| = |A|^{3-1} = 25$$

### Example 3

Simplify: $(ABC)^{-1} \cdot A$

**Solution**:
$$(ABC)^{-1} \cdot A = C^{-1}B^{-1}A^{-1} \cdot A = C^{-1}B^{-1}(A^{-1}A) = C^{-1}B^{-1}I = C^{-1}B^{-1}$$

### Example 4

If A and B are invertible and $AB = C$, express B in terms of A and C.

**Solution**:
$$AB = C$$
$$A^{-1}(AB) = A^{-1}C$$
$$B = A^{-1}C$$

---

## 📊 Summary Table

| Property | Formula |
|----------|---------|
| Inverse of inverse | $(A^{-1})^{-1} = A$ |
| Inverse of product | $(AB)^{-1} = B^{-1}A^{-1}$ |
| Inverse of transpose | $(A^T)^{-1} = (A^{-1})^T$ |
| Inverse of scalar | $(kA)^{-1} = \frac{1}{k}A^{-1}$ |
| Inverse of power | $(A^n)^{-1} = (A^{-1})^n$ |
| Determinant of inverse | $\|A^{-1}\| = 1/\|A\|$ |

---

## ❓ Quick Revision Questions

1. **What is $(A^{-1})^{-1}$?**
   <details>
   <summary>Click for Answer</summary>
   $(A^{-1})^{-1} = A$
   </details>

2. **What is $(AB)^{-1}$?**
   <details>
   <summary>Click for Answer</summary>
   $(AB)^{-1} = B^{-1}A^{-1}$ (order reverses)
   </details>

3. **If $|A| = 4$, what is $|A^{-1}|$?**
   <details>
   <summary>Click for Answer</summary>
   $|A^{-1}| = 1/4$
   </details>

4. **Is $(A^T)^{-1} = (A^{-1})^T$?**
   <details>
   <summary>Click for Answer</summary>
   Yes, the inverse of transpose equals transpose of inverse.
   </details>

5. **What is $(2A)^{-1}$ in terms of $A^{-1}$?**
   <details>
   <summary>Click for Answer</summary>
   $(2A)^{-1} = \frac{1}{2}A^{-1}$
   </details>

6. **If Q is orthogonal, what is $Q^{-1}$?**
   <details>
   <summary>Click for Answer</summary>
   $Q^{-1} = Q^T$
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Inverse of a Matrix](02-inverse-of-matrix.md) | [Unit 5: Adjoint & Inverse](./README.md) | [Solving Equations →](04-solving-equations.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
