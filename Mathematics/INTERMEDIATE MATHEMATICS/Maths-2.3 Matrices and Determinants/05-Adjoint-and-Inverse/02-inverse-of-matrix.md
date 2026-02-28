# Chapter 5.2: Inverse of a Matrix

[← Previous: Adjoint of a Matrix](01-adjoint-of-matrix.md) | [Back to README](../README.md) | [Next: Properties of Inverse →](03-properties-of-inverse.md)

---

## 📚 Chapter Overview

The inverse of a matrix is one of the most important concepts in linear algebra. It allows us to "divide" by a matrix, solve systems of equations, and understand when transformations can be undone.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define the inverse of a matrix
- Determine when a matrix is invertible
- Calculate inverses using the adjoint method
- Apply inverses to solve equations

---

## 1. Definition of Inverse

### Formal Definition

> A square matrix A is **invertible** (or **non-singular**) if there exists a matrix $A^{-1}$ such that:
> $$A \cdot A^{-1} = A^{-1} \cdot A = I$$

### Key Points

- $A^{-1}$ is called the **inverse** of A
- Only square matrices can have inverses
- The inverse, if it exists, is unique

### Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                    MATRIX INVERSE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Matrix A         ×       Inverse A⁻¹       =       Identity   │
│                                                                  │
│   ┌───────┐              ┌───────┐              ┌───────┐       │
│   │       │              │       │              │ 1 0 0 │       │
│   │   A   │      ×       │  A⁻¹  │      =      │ 0 1 0 │       │
│   │       │              │       │              │ 0 0 1 │       │
│   └───────┘              └───────┘              └───────┘       │
│                                                                  │
│   Also: A⁻¹ × A = I (works both ways!)                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Existence of Inverse

### Condition for Invertibility

> A matrix A is invertible if and only if $|A| \neq 0$.

### Terminology

| Condition | Matrix is called |
|-----------|-----------------|
| $\|A\| \neq 0$ | Invertible, Non-singular |
| $\|A\| = 0$ | Singular, Not invertible |

### Why?

The formula for inverse involves dividing by |A|:
$$A^{-1} = \frac{1}{|A|} \text{adj}(A)$$

If $|A| = 0$, we cannot divide by zero, so no inverse exists.

---

## 3. The Inverse Formula

### Main Formula

> $$A^{-1} = \frac{1}{|A|} \text{adj}(A)$$

### Derivation

From the adjoint property:
$$A \cdot \text{adj}(A) = |A| \cdot I$$

Dividing both sides by |A| (when $|A| \neq 0$):
$$A \cdot \frac{\text{adj}(A)}{|A|} = I$$

Therefore:
$$A^{-1} = \frac{\text{adj}(A)}{|A|}$$

---

## 4. Inverse of 2×2 Matrix

### Formula

For $A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$ with $|A| = ad - bc \neq 0$:

$$A^{-1} = \frac{1}{ad-bc}\begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$$

### Step-by-Step

1. Calculate $|A| = ad - bc$
2. Check if $|A| \neq 0$
3. Swap diagonal elements (a ↔ d)
4. Negate off-diagonal elements (b → -b, c → -c)
5. Divide each element by |A|

### Example 1

Find $A^{-1}$ for $A = \begin{bmatrix} 3 & 2 \\ 5 & 4 \end{bmatrix}$

**Step 1**: $|A| = 3(4) - 2(5) = 12 - 10 = 2 \neq 0$ ✓

**Step 2**: 
$$A^{-1} = \frac{1}{2}\begin{bmatrix} 4 & -2 \\ -5 & 3 \end{bmatrix} = \begin{bmatrix} 2 & -1 \\ -\frac{5}{2} & \frac{3}{2} \end{bmatrix}$$

**Verification**:
$$AA^{-1} = \begin{bmatrix} 3 & 2 \\ 5 & 4 \end{bmatrix}\begin{bmatrix} 2 & -1 \\ -\frac{5}{2} & \frac{3}{2} \end{bmatrix} = \begin{bmatrix} 6-5 & -3+3 \\ 10-10 & -5+6 \end{bmatrix} = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$$ ✓

---

## 5. Inverse of 3×3 Matrix

### Steps

1. Calculate $|A|$ and verify $|A| \neq 0$
2. Find all 9 cofactors
3. Form the cofactor matrix
4. Transpose to get adj(A)
5. Multiply by $\frac{1}{|A|}$

### Example 2

Find $A^{-1}$ for $A = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 1 & 4 \\ 5 & 6 & 0 \end{bmatrix}$

**Step 1**: Calculate |A|

Using expansion along row 2:
$$|A| = -0 + 1\begin{vmatrix} 1 & 3 \\ 5 & 0 \end{vmatrix} - 4\begin{vmatrix} 1 & 2 \\ 5 & 6 \end{vmatrix}$$
$$= 1(0-15) - 4(6-10) = -15 - 4(-4) = -15 + 16 = 1$$

$|A| = 1 \neq 0$ ✓

**Step 2**: Find all cofactors

$C_{11} = \begin{vmatrix} 1 & 4 \\ 6 & 0 \end{vmatrix} = -24$

$C_{12} = -\begin{vmatrix} 0 & 4 \\ 5 & 0 \end{vmatrix} = -(-20) = 20$

$C_{13} = \begin{vmatrix} 0 & 1 \\ 5 & 6 \end{vmatrix} = -5$

$C_{21} = -\begin{vmatrix} 2 & 3 \\ 6 & 0 \end{vmatrix} = -(-18) = 18$

$C_{22} = \begin{vmatrix} 1 & 3 \\ 5 & 0 \end{vmatrix} = -15$

$C_{23} = -\begin{vmatrix} 1 & 2 \\ 5 & 6 \end{vmatrix} = -(6-10) = 4$

$C_{31} = \begin{vmatrix} 2 & 3 \\ 1 & 4 \end{vmatrix} = 8-3 = 5$

$C_{32} = -\begin{vmatrix} 1 & 3 \\ 0 & 4 \end{vmatrix} = -4$

$C_{33} = \begin{vmatrix} 1 & 2 \\ 0 & 1 \end{vmatrix} = 1$

**Step 3**: Form cofactor matrix and transpose

$$\text{Cofactor matrix} = \begin{bmatrix} -24 & 20 & -5 \\ 18 & -15 & 4 \\ 5 & -4 & 1 \end{bmatrix}$$

$$\text{adj}(A) = \begin{bmatrix} -24 & 18 & 5 \\ 20 & -15 & -4 \\ -5 & 4 & 1 \end{bmatrix}$$

**Step 4**: Calculate inverse

$$A^{-1} = \frac{1}{1}\begin{bmatrix} -24 & 18 & 5 \\ 20 & -15 & -4 \\ -5 & 4 & 1 \end{bmatrix} = \begin{bmatrix} -24 & 18 & 5 \\ 20 & -15 & -4 \\ -5 & 4 & 1 \end{bmatrix}$$

---

## 6. Solving Equations Using Inverse

### System AX = B

If A is invertible:
$$X = A^{-1}B$$

### Example 3

Solve: $\begin{cases} 3x + 2y = 11 \\ 5x + 4y = 19 \end{cases}$

**Matrix form**: $\begin{bmatrix} 3 & 2 \\ 5 & 4 \end{bmatrix}\begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} 11 \\ 19 \end{bmatrix}$

**From Example 1**: $A^{-1} = \begin{bmatrix} 2 & -1 \\ -\frac{5}{2} & \frac{3}{2} \end{bmatrix}$

**Solution**:
$$\begin{bmatrix} x \\ y \end{bmatrix} = \begin{bmatrix} 2 & -1 \\ -\frac{5}{2} & \frac{3}{2} \end{bmatrix}\begin{bmatrix} 11 \\ 19 \end{bmatrix} = \begin{bmatrix} 22-19 \\ -\frac{55}{2} + \frac{57}{2} \end{bmatrix} = \begin{bmatrix} 3 \\ 1 \end{bmatrix}$$

**Answer**: $x = 3, y = 1$

**Verification**: $3(3) + 2(1) = 11$ ✓, $5(3) + 4(1) = 19$ ✓

---

## 7. Alternative Method: Row Reduction

### Augmented Matrix Method

To find $A^{-1}$:
1. Form augmented matrix $[A | I]$
2. Use row operations to transform A to I
3. The right side becomes $A^{-1}$: $[I | A^{-1}]$

### Example 4

Find inverse using row reduction: $A = \begin{bmatrix} 1 & 2 \\ 3 & 7 \end{bmatrix}$

Start: $\begin{bmatrix} 1 & 2 & | & 1 & 0 \\ 3 & 7 & | & 0 & 1 \end{bmatrix}$

$R_2 \to R_2 - 3R_1$:
$\begin{bmatrix} 1 & 2 & | & 1 & 0 \\ 0 & 1 & | & -3 & 1 \end{bmatrix}$

$R_1 \to R_1 - 2R_2$:
$\begin{bmatrix} 1 & 0 & | & 7 & -2 \\ 0 & 1 & | & -3 & 1 \end{bmatrix}$

Therefore: $A^{-1} = \begin{bmatrix} 7 & -2 \\ -3 & 1 \end{bmatrix}$

**Verify**: $|A| = 7 - 6 = 1$, Using formula: $A^{-1} = \frac{1}{1}\begin{bmatrix} 7 & -2 \\ -3 & 1 \end{bmatrix}$ ✓

---

## 8. When Inverse Doesn't Exist

### Example: Singular Matrix

$A = \begin{bmatrix} 2 & 4 \\ 1 & 2 \end{bmatrix}$

$|A| = 4 - 4 = 0$

**No inverse exists!**

### What This Means

- The rows (or columns) are linearly dependent
- The matrix "collapses" dimensions
- Cannot undo the transformation
- System AX = B has either no solution or infinitely many

---

## 9. Method Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                 METHODS TO FIND INVERSE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Method 1: Adjoint Formula                                      │
│   ─────────────────────────                                     │
│   A⁻¹ = (1/|A|) × adj(A)                                        │
│   • Good for theoretical work                                   │
│   • Better for small matrices                                   │
│   • Direct formula for 2×2                                      │
│                                                                  │
│   Method 2: Row Reduction (Gauss-Jordan)                        │
│   ────────────────────────────────────────                      │
│   [A | I] → [I | A⁻¹]                                           │
│   • More efficient for larger matrices                          │
│   • Shows if inverse doesn't exist (row of zeros)              │
│   • Good for numerical computation                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Important Formulas

| Matrix Type | Inverse |
|-------------|---------|
| Diagonal $D = \text{diag}(d_1, d_2, ...)$ | $D^{-1} = \text{diag}(1/d_1, 1/d_2, ...)$ |
| Identity $I$ | $I^{-1} = I$ |
| Scalar $kI$ | $(kI)^{-1} = \frac{1}{k}I$ |
| 2×2 $\begin{bmatrix} a & b \\ c & d \end{bmatrix}$ | $\frac{1}{ad-bc}\begin{bmatrix} d & -b \\ -c & a \end{bmatrix}$ |

---

## 📊 Summary Table

| Condition | Matrix Status | Inverse |
|-----------|---------------|---------|
| $\|A\| \neq 0$ | Invertible | $A^{-1} = \frac{1}{\|A\|}\text{adj}(A)$ |
| $\|A\| = 0$ | Singular | Does not exist |
| $A = I$ | Identity | $I^{-1} = I$ |
| $A = kI$ | Scalar matrix | $(kI)^{-1} = \frac{1}{k}I$ |

---

## ❓ Quick Revision Questions

1. **When does a matrix have an inverse?**
   <details>
   <summary>Click for Answer</summary>
   When its determinant is non-zero ($|A| \neq 0$).
   </details>

2. **What is the formula for inverse?**
   <details>
   <summary>Click for Answer</summary>
   $A^{-1} = \frac{1}{|A|}\text{adj}(A)$
   </details>

3. **Find the inverse of $\begin{bmatrix} 2 & 1 \\ 5 & 3 \end{bmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   $|A| = 6-5 = 1$, so $A^{-1} = \begin{bmatrix} 3 & -1 \\ -5 & 2 \end{bmatrix}$
   </details>

4. **What is $A \cdot A^{-1}$?**
   <details>
   <summary>Click for Answer</summary>
   The identity matrix I.
   </details>

5. **How do you solve AX = B using inverse?**
   <details>
   <summary>Click for Answer</summary>
   $X = A^{-1}B$
   </details>

6. **Is $\begin{bmatrix} 1 & 2 \\ 2 & 4 \end{bmatrix}$ invertible?**
   <details>
   <summary>Click for Answer</summary>
   No. $|A| = 4 - 4 = 0$, so it's singular.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Adjoint of a Matrix](01-adjoint-of-matrix.md) | [Unit 5: Adjoint & Inverse](./README.md) | [Properties of Inverse →](03-properties-of-inverse.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
