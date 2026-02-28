# Chapter 3.2: Gauss-Jordan Method

[← Previous: Gaussian Elimination](01-gaussian-elimination.md) | [Next: LU Decomposition →](03-lu-decomposition.md)

---

## Overview

The **Gauss-Jordan method** extends Gaussian elimination by reducing the augmented matrix to **reduced row echelon form** (RREF) — a diagonal or identity matrix — eliminating the need for back substitution. It is also the standard method for computing matrix inverses.

---

## 1. Key Idea

Instead of just making zeros below each pivot (upper triangular), also make zeros **above** each pivot, and scale each pivot to 1.

```
  Gaussian Elimination:           Gauss-Jordan:

  ┌           ┐                   ┌           ┐
  │ * * * │ * │                   │ 1 0 0 │ * │
  │ 0 * * │ * │     ──────►      │ 0 1 0 │ * │
  │ 0 0 * │ * │                   │ 0 0 1 │ * │
  └           ┘                   └           ┘
  Upper Triangular                Identity (RREF)
  (needs back-sub)               (solution is read off!)
```

---

## 2. Algorithm

1. Form augmented matrix $[A|b]$
2. For each column $k = 1, 2, \ldots, n$:
   - **Pivot**: Find largest $|a_{ik}|$ for $i \geq k$, swap rows
   - **Scale**: Divide row $k$ by $a_{kk}$ to make pivot = 1
   - **Eliminate**: For ALL other rows $i \neq k$: $R_i \to R_i - a_{ik} R_k$
3. Read solution directly: $x_i = b_i'$

---

## 3. Worked Example

Solve:
$$x + y + z = 9$$
$$2x - 3y + 4z = 13$$
$$3x + 4y + 5z = 40$$

### Augmented Matrix

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 9 \\ 2 & -3 & 4 & 13 \\ 3 & 4 & 5 & 40 \end{array}\right]$$

### Step 1: Pivot on column 1x (already 1)

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - 3R_1$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 9 \\ 0 & -5 & 2 & -5 \\ 0 & 1 & 2 & 13 \end{array}\right]$$

### Step 2: Pivot on column 2

$R_2 \to R_2 / (-5)$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 9 \\ 0 & 1 & -0.4 & 1 \\ 0 & 1 & 2 & 13 \end{array}\right]$$

$R_1 \to R_1 - R_2$, $R_3 \to R_3 - R_2$:

$$\left[\begin{array}{ccc|c} 1 & 0 & 1.4 & 8 \\ 0 & 1 & -0.4 & 1 \\ 0 & 0 & 2.4 & 12 \end{array}\right]$$

### Step 3: Pivot on column 3

$R_3 \to R_3 / 2.4$:

$$\left[\begin{array}{ccc|c} 1 & 0 & 1.4 & 8 \\ 0 & 1 & -0.4 & 1 \\ 0 & 0 & 1 & 5 \end{array}\right]$$

$R_1 \to R_1 - 1.4R_3$, $R_2 \to R_2 + 0.4R_3$:

$$\left[\begin{array}{ccc|c} 1 & 0 & 0 & 1 \\ 0 & 1 & 0 & 3 \\ 0 & 0 & 1 & 5 \end{array}\right]$$

**Solution**: $x = 1, y = 3, z = 5$

---

## 4. Finding the Inverse of a Matrix

To find $A^{-1}$, augment $A$ with the identity matrix and reduce:

$$[A | I] \xrightarrow{\text{Gauss-Jordan}} [I | A^{-1}]$$

### Example: Find inverse of $A = \begin{bmatrix} 2 & 1 \\ 5 & 3 \end{bmatrix}$

$$\left[\begin{array}{cc|cc} 2 & 1 & 1 & 0 \\ 5 & 3 & 0 & 1 \end{array}\right]$$

$R_1 \to R_1/2$:
$$\left[\begin{array}{cc|cc} 1 & 0.5 & 0.5 & 0 \\ 5 & 3 & 0 & 1 \end{array}\right]$$

$R_2 \to R_2 - 5R_1$:
$$\left[\begin{array}{cc|cc} 1 & 0.5 & 0.5 & 0 \\ 0 & 0.5 & -2.5 & 1 \end{array}\right]$$

$R_2 \to 2R_2$:
$$\left[\begin{array}{cc|cc} 1 & 0.5 & 0.5 & 0 \\ 0 & 1 & -5 & 2 \end{array}\right]$$

$R_1 \to R_1 - 0.5R_2$:
$$\left[\begin{array}{cc|cc} 1 & 0 & 3 & -1 \\ 0 & 1 & -5 & 2 \end{array}\right]$$

$$A^{-1} = \begin{bmatrix} 3 & -1 \\ -5 & 2 \end{bmatrix}$$

---

## 5. Gauss-Jordan vs. Gaussian Elimination

| Feature | Gaussian Elimination | Gauss-Jordan |
|---------|:-------------------:|:------------:|
| Result | Upper triangular | RREF (identity) |
| Back substitution | Required | Not needed |
| Operation count | $\frac{n^3}{3}$ | $\frac{n^3}{2}$ |
| Matrix inverse | Not directly | Yes ($[A|I] \to [I|A^{-1}]$) |
| Multiple RHS | Less efficient | More efficient |

**Note**: Gauss-Jordan requires ~50% more operations than Gaussian elimination for solving a single system. It is preferred when finding the inverse or solving for many right-hand sides simultaneously.

---

## 6. Operation Count

| Phase | Operations |
|-------|:----------:|
| Forward + Backward elimination | $\frac{n^3}{2} + O(n^2)$ |
| Compared to Gauss + back-sub | $\frac{n^3}{3} + \frac{n^2}{2}$ |

For matrix inversion ($n$ right-hand sides): $O(n^3)$ for both methods.

---

## 7. Real-World Applications

| Application | Use Case |
|-------------|----------|
| Matrix inversion | Control theory, signal processing |
| Solving multiple systems | Same $A$, different $b$ vectors |
| Linear programming | Simplex method uses row reduction |
| Cryptography | Inverting encoding matrices |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Direct method |
| **Result** | Reduced Row Echelon Form (RREF) |
| **Complexity** | $O(n^3/2)$ for single system |
| **No back substitution** | Solution read directly |
| **Matrix inverse** | Augment with $I$, reduce to RREF |
| **Pivoting** | Essential (same as Gaussian elimination) |

---

## Quick Revision Questions

1. **How does Gauss-Jordan differ from Gaussian elimination? When is each preferred?**

2. **Solve using Gauss-Jordan: $2x + y = 5$, $x + 3y = 10$.**

3. **Find the inverse of $A = \begin{bmatrix} 1 & 2 \\ 3 & 5 \end{bmatrix}$ using Gauss-Jordan.**

4. **What is the operation count for Gauss-Jordan compared to Gaussian elimination?**

5. **Can Gauss-Jordan detect a singular matrix? How?**

6. **Explain why Gauss-Jordan is preferred for computing matrix inverses over using Cramer's rule.**

---

[← Previous: Gaussian Elimination](01-gaussian-elimination.md) | [Next: LU Decomposition →](03-lu-decomposition.md)
