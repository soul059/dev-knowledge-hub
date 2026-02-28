# Chapter 3.3: Properties of Determinants

[← Previous: Determinant of 3×3](02-determinant-3x3.md) | [Back to README](../README.md) | [Next: Minor and Cofactor →](04-minor-and-cofactor.md)

---

## 📚 Chapter Overview

Determinants have many powerful properties that simplify calculations and reveal important relationships. Understanding these properties is crucial for efficient computation and deeper understanding of linear algebra.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply properties to simplify determinant calculations
- Predict determinant values without full calculation
- Use properties to prove mathematical statements
- Recognize when determinants equal zero

---

## 1. Property List Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                 PROPERTIES OF DETERMINANTS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Transpose Property      |A^T| = |A|                         │
│  2. Row/Column Swap         Changes sign                        │
│  3. Scalar Multiple         Factor out from row/column          │
│  4. Zero Row/Column         Determinant = 0                     │
│  5. Identical Rows/Columns  Determinant = 0                     │
│  6. Proportional Rows/Cols  Determinant = 0                     │
│  7. Row/Column Addition     Determinant unchanged               │
│  8. Product Property        |AB| = |A||B|                       │
│  9. Triangular Matrix       Product of diagonal                 │
│  10. Scalar Matrix          |kA| = k^n|A| (n×n matrix)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Property 1: Transpose Property

### Statement

> $|A^T| = |A|$

The determinant of a matrix equals the determinant of its transpose.

### Implication

**Any property that holds for rows also holds for columns!**

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad A^T = \begin{bmatrix} 1 & 3 \\ 2 & 4 \end{bmatrix}$$

$$|A| = (1)(4) - (2)(3) = -2$$
$$|A^T| = (1)(4) - (3)(2) = -2$$

Both equal -2 ✓

---

## 3. Property 2: Row/Column Interchange

### Statement

> If two rows (or columns) are interchanged, the determinant changes sign.

### Visual

```
        Original                After Row Swap
        
        | a  b |                | c  d |
        | c  d |        →       | a  b |
        
        det = ad - bc           det = cb - da = -(ad - bc)
```

### Example

$$\begin{vmatrix} 1 & 2 \\ 3 & 4 \end{vmatrix} = -2$$

Swap rows:

$$\begin{vmatrix} 3 & 4 \\ 1 & 2 \end{vmatrix} = 6 - 4 = 2 = -(-2)$$

Sign changed! ✓

### Corollary

If a matrix is obtained from another by an **odd** number of row swaps, determinants are negatives.
If by an **even** number of swaps, determinants are equal.

---

## 4. Property 3: Scalar Multiple of Row/Column

### Statement

> If all elements of a row (or column) are multiplied by a scalar k, the determinant is multiplied by k.

### Formula

$$\begin{vmatrix} ka & kb \\ c & d \end{vmatrix} = k\begin{vmatrix} a & b \\ c & d \end{vmatrix}$$

### Example

$$\begin{vmatrix} 6 & 9 \\ 2 & 5 \end{vmatrix} = \begin{vmatrix} 3 \times 2 & 3 \times 3 \\ 2 & 5 \end{vmatrix} = 3\begin{vmatrix} 2 & 3 \\ 2 & 5 \end{vmatrix}$$

$$= 3(10 - 6) = 3(4) = 12$$

Verify: $6(5) - 9(2) = 30 - 18 = 12$ ✓

### Corollary: Scalar Multiple of Entire Matrix

For an n×n matrix:
$$|kA| = k^n|A|$$

For 2×2: $|kA| = k^2|A|$
For 3×3: $|kA| = k^3|A|$

---

## 5. Property 4: Zero Row or Column

### Statement

> If any row or column consists entirely of zeros, the determinant equals zero.

### Visual

```
        | a  b  c |
        | 0  0  0 |  ← Zero row
        | g  h  i |
        
        Determinant = 0
```

### Proof (Cofactor Expansion)

Expand along the zero row:
$$|A| = 0 \cdot C_{21} + 0 \cdot C_{22} + 0 \cdot C_{23} = 0$$

---

## 6. Property 5: Identical Rows or Columns

### Statement

> If two rows (or columns) are identical, the determinant equals zero.

### Example

$$\begin{vmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 1 & 2 & 3 \end{vmatrix} = 0$$

Row 1 = Row 3

### Proof

If we swap the two identical rows, we get the same matrix.
But swapping rows changes the sign of the determinant.
Therefore: $|A| = -|A|$, which means $2|A| = 0$, so $|A| = 0$.

---

## 7. Property 6: Proportional Rows or Columns

### Statement

> If one row (or column) is a scalar multiple of another, the determinant equals zero.

### Example

$$\begin{vmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \\ 4 & 5 & 6 \end{vmatrix} = 0$$

Row 2 = 2 × Row 1

### Proof

Factor out the scalar:
$$\begin{vmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \\ 4 & 5 & 6 \end{vmatrix} = 2\begin{vmatrix} 1 & 2 & 3 \\ 1 & 2 & 3 \\ 4 & 5 & 6 \end{vmatrix} = 2 \times 0 = 0$$

---

## 8. Property 7: Row/Column Operations (Addition)

### Statement

> If a multiple of one row is added to another row, the determinant remains unchanged.

### Formula

$$\begin{vmatrix} a & b \\ c & d \end{vmatrix} = \begin{vmatrix} a & b \\ c + ka & d + kb \end{vmatrix}$$

### Example

$$\begin{vmatrix} 2 & 3 \\ 4 & 5 \end{vmatrix} = 10 - 12 = -2$$

Add 2×(Row 1) to Row 2:

$$\begin{vmatrix} 2 & 3 \\ 4+4 & 5+6 \end{vmatrix} = \begin{vmatrix} 2 & 3 \\ 8 & 11 \end{vmatrix} = 22 - 24 = -2$$

Same determinant! ✓

### This is the Key Property for Simplification!

```
┌─────────────────────────────────────────────────────────────────┐
│                 SIMPLIFICATION STRATEGY                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Use Property 7 to create zeros, then:                         │
│   - Expand along row/column with zeros                          │
│   - Or transform to triangular form                             │
│                                                                  │
│   Example:                                                       │
│                                                                  │
│   | 2  3  1 |     R₂ - R₁      | 2  3  1 |                     │
│   | 2  1  4 |   ─────────→     | 0 -2  3 |                     │
│   | 3  2  5 |     R₃ - R₁      | 1 -1  4 |                     │
│                                                                  │
│   Now expand along column 1 (has a zero)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Property 8: Product of Determinants

### Statement

> $|AB| = |A| \cdot |B|$

The determinant of a product equals the product of determinants.

### Example

$$A = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad B = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

$$|A| = -2, \quad |B| = -2$$

$$AB = \begin{bmatrix} 19 & 22 \\ 43 & 50 \end{bmatrix}$$

$$|AB| = 950 - 946 = 4 = (-2)(-2) = |A||B|$$ ✓

### Corollaries

| Result | Formula |
|--------|---------|
| Power | $|A^n| = |A|^n$ |
| Inverse | $|A^{-1}| = \frac{1}{|A|}$ (if $|A| \neq 0$) |
| Product of inverses | $|(AB)^{-1}| = |A^{-1}||B^{-1}|$ |

---

## 10. Property 9: Triangular Matrices

### Statement

> The determinant of a triangular matrix equals the product of its diagonal elements.

### Upper Triangular

$$\begin{vmatrix} a & b & c \\ 0 & d & e \\ 0 & 0 & f \end{vmatrix} = adf$$

### Lower Triangular

$$\begin{vmatrix} a & 0 & 0 \\ b & c & 0 \\ d & e & f \end{vmatrix} = acf$$

### Diagonal Matrix

$$\begin{vmatrix} a & 0 & 0 \\ 0 & b & 0 \\ 0 & 0 & c \end{vmatrix} = abc$$

---

## 11. Summary of Zero Determinant Conditions

```
┌─────────────────────────────────────────────────────────────────┐
│              WHEN IS DETERMINANT = 0?                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   The determinant |A| = 0 when:                                 │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  1. Any row or column is all zeros                      │   │
│   │                                                          │   │
│   │  2. Two rows (or columns) are identical                 │   │
│   │                                                          │   │
│   │  3. One row is a scalar multiple of another             │   │
│   │                                                          │   │
│   │  4. Rows (or columns) are linearly dependent            │   │
│   │                                                          │   │
│   │  5. A diagonal element is zero in triangular form       │   │
│   │                                                          │   │
│   │  6. Matrix is singular (not invertible)                 │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12. Working Examples

### Example 1: Using Multiple Properties

Evaluate: $\begin{vmatrix} 2 & 4 & 6 \\ 1 & 2 & 3 \\ 3 & 5 & 7 \end{vmatrix}$

**Solution**:
Row 1 = 2 × Row 2 (proportional rows)

Therefore, determinant = **0**

### Example 2: Factoring Out Scalars

Evaluate: $\begin{vmatrix} 3 & 6 & 9 \\ 1 & 4 & 2 \\ 2 & 5 & 8 \end{vmatrix}$

**Solution**:
Factor 3 from Row 1:
$$= 3\begin{vmatrix} 1 & 2 & 3 \\ 1 & 4 & 2 \\ 2 & 5 & 8 \end{vmatrix}$$

Using Sarrus or expansion:
$$= 3[1(32-10) - 2(8-4) + 3(5-8)]$$
$$= 3[22 - 8 - 9] = 3(5) = 15$$

### Example 3: Using Row Operations

Evaluate: $\begin{vmatrix} 1 & 2 & 3 \\ 2 & 5 & 7 \\ 3 & 8 & 12 \end{vmatrix}$

**Solution**:
Apply $R_2 \to R_2 - 2R_1$ and $R_3 \to R_3 - 3R_1$:

$$= \begin{vmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 2 & 3 \end{vmatrix}$$

Apply $R_3 \to R_3 - 2R_2$:

$$= \begin{vmatrix} 1 & 2 & 3 \\ 0 & 1 & 1 \\ 0 & 0 & 1 \end{vmatrix}$$

Triangular matrix: det = $1 \times 1 \times 1 = 1$

---

## 13. Properties Summary Table

| # | Property | Effect on Determinant |
|---|----------|----------------------|
| 1 | Transpose | $\|A^T\| = \|A\|$ |
| 2 | Row swap | Changes sign |
| 3 | Scalar × row | Multiplies det by scalar |
| 4 | Zero row | det = 0 |
| 5 | Identical rows | det = 0 |
| 6 | Proportional rows | det = 0 |
| 7 | Add multiple of row | No change |
| 8 | Product | $\|AB\| = \|A\|\|B\|$ |
| 9 | Triangular | Product of diagonal |
| 10 | kA (n×n) | $k^n\|A\|$ |

---

## 📊 Summary Table

| Situation | Determinant Result |
|-----------|-------------------|
| Zero row/column | 0 |
| Identical rows/columns | 0 |
| Proportional rows/columns | 0 |
| Swap two rows | Sign changes |
| Multiply row by k | Multiply det by k |
| Add row to another | Unchanged |
| Triangular matrix | Product of diagonal |
| $\|A^T\|$ | $= \|A\|$ |
| $\|AB\|$ | $= \|A\| \cdot \|B\|$ |

---

## ❓ Quick Revision Questions

1. **What is $|A^T|$ if $|A| = 5$?**
   <details>
   <summary>Click for Answer</summary>
   $|A^T| = |A| = 5$
   </details>

2. **What happens to the determinant when two rows are swapped?**
   <details>
   <summary>Click for Answer</summary>
   The determinant changes sign (multiplied by -1).
   </details>

3. **If $\begin{vmatrix} 2 & 3 \\ 4 & 5 \end{vmatrix} = -2$, find $\begin{vmatrix} 6 & 9 \\ 4 & 5 \end{vmatrix}$**
   <details>
   <summary>Click for Answer</summary>
   Row 1 is multiplied by 3, so det = $3 \times (-2) = -6$
   </details>

4. **What is $|A| \cdot |A^{-1}|$?**
   <details>
   <summary>Click for Answer</summary>
   $|A| \cdot |A^{-1}| = |AA^{-1}| = |I| = 1$
   </details>

5. **If A is 3×3 and $|A| = 2$, what is $|3A|$?**
   <details>
   <summary>Click for Answer</summary>
   $|3A| = 3^3 \cdot |A| = 27 \times 2 = 54$
   </details>

6. **Without calculating, what is $\begin{vmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 2 & 4 & 6 \end{vmatrix}$?**
   <details>
   <summary>Click for Answer</summary>
   Row 3 = 2 × Row 1 (proportional rows), so determinant = 0
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Determinant of 3×3](02-determinant-3x3.md) | [Unit 3: Determinants](./README.md) | [Minor and Cofactor →](04-minor-and-cofactor.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*
