# Chapter 4.2: Determinant of Transpose

[← Previous: Product of Determinants](01-product-of-determinants.md) | [Back to README](../README.md) | [Next: Row and Column Operations →](03-row-column-operations.md)

---

## 📚 Chapter Overview

The transpose property is one of the fundamental properties of determinants. This chapter explores the relationship between a matrix and its transpose in terms of determinants, and the powerful implications this relationship has.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- State and prove the transpose property of determinants
- Apply the property to simplify calculations
- Understand the row-column duality principle
- Use the property in proofs and problem solving

---

## 1. Review: Transpose of a Matrix

### Definition

> The **transpose** of a matrix A, denoted $A^T$ (or $A'$), is obtained by interchanging rows and columns.

### Visual Representation

```
    Original Matrix A               Transpose A^T
    
    ┌───────────────┐              ┌───────────────┐
    │  a  b  c  d   │              │  a  e  i      │
    │  e  f  g  h   │      →       │  b  f  j      │
    │  i  j  k  l   │              │  c  g  k      │
    └───────────────┘              │  d  h  l      │
         3 × 4                     └───────────────┘
                                        4 × 3
```

### For Square Matrices

$$A = \begin{bmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{bmatrix} \Rightarrow A^T = \begin{bmatrix} a_{11} & a_{21} & a_{31} \\ a_{12} & a_{22} & a_{32} \\ a_{13} & a_{23} & a_{33} \end{bmatrix}$$

---

## 2. The Transpose Property

### Statement

> For any square matrix A:
> $$|A^T| = |A|$$

### In Words

The determinant of a matrix equals the determinant of its transpose.

### Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                  TRANSPOSE PROPERTY                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│          Matrix A                    Transpose A^T              │
│         ┌───────┐                    ┌───────┐                  │
│         │ rows  │                    │columns│                  │
│         │   =   │     TRANSPOSE      │   =   │                  │
│         │columns│    ═══════════>    │ rows  │                  │
│         └───────┘                    └───────┘                  │
│             │                            │                       │
│             │ det                        │ det                   │
│             ▼                            ▼                       │
│           |A|          ═══════         |A^T|                    │
│                                                                  │
│              They are EQUAL!                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Verification: 2×2 Matrix

### Example

$$A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$$

$$A^T = \begin{bmatrix} a & c \\ b & d \end{bmatrix}$$

**Determinant of A**:
$$|A| = ad - bc$$

**Determinant of A^T**:
$$|A^T| = ad - cb = ad - bc$$

Therefore: $|A^T| = |A|$ ✓

---

## 4. Verification: 3×3 Matrix

### Example

$$A = \begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$$

$$A^T = \begin{bmatrix} 1 & 4 & 7 \\ 2 & 5 & 8 \\ 3 & 6 & 9 \end{bmatrix}$$

**Determinant of A** (using Sarrus' rule):

Positive: $1(5)(9) + 2(6)(7) + 3(4)(8) = 45 + 84 + 96 = 225$
Negative: $7(5)(3) + 8(6)(1) + 9(4)(2) = 105 + 48 + 72 = 225$
$$|A| = 225 - 225 = 0$$

**Determinant of A^T** (using Sarrus' rule):

Positive: $1(5)(9) + 4(8)(3) + 7(2)(6) = 45 + 96 + 84 = 225$
Negative: $3(5)(7) + 6(8)(1) + 9(2)(4) = 105 + 48 + 72 = 225$
$$|A^T| = 225 - 225 = 0$$

Therefore: $|A^T| = |A| = 0$ ✓

---

## 5. Proof for General Case

### Proof Sketch (Cofactor Expansion)

Consider expanding $|A|$ along the i-th row and $|A^T|$ along the i-th column.

Since transposing converts rows to columns:
- Row i of A becomes Column i of $A^T$
- The cofactors correspond appropriately

Both expansions give the same value, hence $|A^T| = |A|$.

### Alternative: Using Permutations

The determinant formula using permutations is:
$$|A| = \sum_{\sigma \in S_n} \text{sgn}(\sigma) \prod_{i=1}^{n} a_{i,\sigma(i)}$$

For $A^T$: $(A^T)_{ij} = a_{ji}$

The sum over all permutations yields the same result due to the properties of permutation groups.

---

## 6. The Row-Column Duality

### Fundamental Principle

> Any property that holds for rows also holds for columns!

This is a direct consequence of $|A^T| = |A|$.

### Examples of Duality

| Row Property | Column Property |
|--------------|-----------------|
| Zero row → det = 0 | Zero column → det = 0 |
| Identical rows → det = 0 | Identical columns → det = 0 |
| Row swap changes sign | Column swap changes sign |
| Factor from row | Factor from column |
| Add multiple of row to another | Add multiple of column to another |

---

## 7. Applications of the Transpose Property

### Application 1: Expanding Along Columns

Since $|A^T| = |A|$, you can expand a determinant along any column just as you would along a row.

$$|A| = \sum_{i=1}^{n} a_{ij} \cdot C_{ij}$$ (column j expansion)

### Application 2: Simplifying by Column Operations

You can use column operations to create zeros, just like row operations.

### Example

$$\begin{vmatrix} 2 & 1 & 3 \\ 4 & 2 & 6 \\ 1 & 3 & 2 \end{vmatrix}$$

Column 3 = 3 × Column 1 - 3 × Column 2? Let's check:
$3(2) - 3(1) = 3$? No, that's 3. ✓
$3(4) - 3(2) = 6$? Yes. ✓
$3(1) - 3(3) = -6 \neq 2$

Let's use column operations:
$C_2 \to C_2 - \frac{1}{2}C_1$:

$$\begin{vmatrix} 2 & 0 & 3 \\ 4 & 0 & 6 \\ 1 & 2.5 & 2 \end{vmatrix}$$

Now Column 1 and Column 2 show that Row 2 = 2 × Row 1 in the first two elements...

Actually, notice Column 3 = (3/2) × Column 1 for the first two rows, suggesting the rows might be dependent. Let's verify the original determinant is 0:

Using row operation $R_2 - 2R_1$:
$$\begin{vmatrix} 2 & 1 & 3 \\ 0 & 0 & 0 \\ 1 & 3 & 2 \end{vmatrix} = 0$$

Indeed, Row 2 = 2 × Row 1 in the original matrix! (Check: 4=2×2, 2=2×1, 6=2×3) ✓

---

## 8. Combined Properties

### Property: $(A^T)^{-1} = (A^{-1})^T$

**Consequence for Determinants**:
$$|(A^T)^{-1}| = |(A^{-1})^T|$$
$$\frac{1}{|A^T|} = |A^{-1}|^T$$

But determinants are scalars, so:
$$\frac{1}{|A|} = \frac{1}{|A|}$$ ✓ (Consistent)

### Property: $(AB)^T = B^T A^T$

**For Determinants**:
$$|(AB)^T| = |B^T A^T| = |B^T| \cdot |A^T| = |B| \cdot |A| = |A| \cdot |B| = |AB|$$

All consistent! ✓

---

## 9. Symmetric and Skew-Symmetric Matrices

### Symmetric Matrix

A matrix is **symmetric** if $A = A^T$.

$$|A| = |A^T|$$ (Always true, trivially satisfied)

### Skew-Symmetric Matrix

A matrix is **skew-symmetric** if $A^T = -A$.

**Special Property**: For an n×n skew-symmetric matrix A:

$$|A^T| = |-A| = (-1)^n |A|$$

But also $|A^T| = |A|$.

Therefore: $|A| = (-1)^n |A|$

**For odd n**: $|A| = -|A|$, which means $2|A| = 0$, so $|A| = 0$.

> **Important Result**: Every skew-symmetric matrix of **odd order** has determinant **zero**.

### Example

$$A = \begin{bmatrix} 0 & 2 & -3 \\ -2 & 0 & 4 \\ 3 & -4 & 0 \end{bmatrix}$$ (3×3 skew-symmetric)

$|A| = 0$ (guaranteed by the property!)

---

## 10. Worked Examples

### Example 1

Given $|A| = 5$, find $|A^T|$.

**Solution**: $|A^T| = |A| = 5$

### Example 2

If $A^T = A^{-1}$ (orthogonal matrix), find $|A|$.

**Solution**:
$$|A^T| = |A^{-1}|$$
$$|A| = \frac{1}{|A|}$$
$$|A|^2 = 1$$
$$|A| = \pm 1$$

### Example 3

Prove that $|A + A^T|$ is always even if A is a 2×2 integer matrix.

**Solution**:
Let $A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$

$A + A^T = \begin{bmatrix} 2a & b+c \\ b+c & 2d \end{bmatrix}$

$|A + A^T| = 4ad - (b+c)^2$

This is $(2a)(2d) - (b+c)^2$, which has specific parity properties.

---

## 📊 Summary Table

| Property | Statement |
|----------|-----------|
| Transpose Determinant | $\|A^T\| = \|A\|$ |
| Row-Column Duality | Row properties ↔ Column properties |
| Skew-symmetric (odd n) | $\|A\| = 0$ |
| Orthogonal Matrix | $\|A\| = \pm 1$ |
| Symmetric Matrix | $A = A^T$, property trivially holds |

---

## ❓ Quick Revision Questions

1. **What is $|A^T|$ if $|A| = 7$?**
   <details>
   <summary>Click for Answer</summary>
   $|A^T| = |A| = 7$
   </details>

2. **Why can you expand a determinant along a column?**
   <details>
   <summary>Click for Answer</summary>
   Because $|A^T| = |A|$, and expanding $|A^T|$ along a row is equivalent to expanding $|A|$ along a column.
   </details>

3. **What is the determinant of a 3×3 skew-symmetric matrix?**
   <details>
   <summary>Click for Answer</summary>
   Zero. All odd-order skew-symmetric matrices have zero determinant.
   </details>

4. **If A is orthogonal (i.e., $A^T = A^{-1}$), what are the possible values of $|A|$?**
   <details>
   <summary>Click for Answer</summary>
   $|A| = \pm 1$
   </details>

5. **Does swapping two columns change the determinant?**
   <details>
   <summary>Click for Answer</summary>
   Yes, it changes the sign (multiplies by -1), just like swapping rows.
   </details>

6. **Is $|(A^T)^T| = |A|$?**
   <details>
   <summary>Click for Answer</summary>
   Yes, because $(A^T)^T = A$, so $|(A^T)^T| = |A|$.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Product of Determinants](01-product-of-determinants.md) | [Unit 4: Applications](./README.md) | [Row and Column Operations →](03-row-column-operations.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
