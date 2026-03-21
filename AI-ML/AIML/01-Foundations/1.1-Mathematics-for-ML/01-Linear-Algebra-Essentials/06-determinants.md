# Chapter 6: Determinants

> **Unit 1 — Linear Algebra Essentials**
> The determinant tells you whether a matrix is invertible, how it scales space, and whether your linear system has a unique solution.

---

## 6.1 What Is a Determinant?

The **determinant** is a scalar value computed from a square matrix that encodes fundamental geometric and algebraic information.

```
det(A) ≠ 0  →  A is invertible, columns are linearly independent
det(A) = 0  →  A is singular, columns are linearly dependent
```

---

## 6.2 The 2×2 Determinant

```
A = [a  b]      det(A) = ad - bc
    [c  d]
```

### Example

```
A = [3  1]      det(A) = 3(4) - 1(2) = 12 - 2 = 10
    [2  4]
```

### Geometric Interpretation

```
         d
    ┌────/──────┐
    │   /       │     Area of parallelogram
    │  / v₂    │     formed by column vectors
    │ /        │     = |det(A)|
    │/─────────┘
    v₁

det(A) = signed area of the parallelogram spanned by columns of A
```

If det(A) > 0, the transformation preserves orientation. If det(A) < 0, it reverses orientation.

```python
import numpy as np
A = np.array([[3, 1], [2, 4]])
print(np.linalg.det(A))  # 10.0
```

---

## 6.3 The 3×3 Determinant

### Sarrus' Rule (Shortcut for 3×3)

```
A = [a₁  a₂  a₃]
    [b₁  b₂  b₃]
    [c₁  c₂  c₃]

det(A) = a₁b₂c₃ + a₂b₃c₁ + a₃b₁c₂ - a₃b₂c₁ - a₂b₁c₃ - a₁b₃c₂
```

### Cofactor Expansion (General Method)

Expand along **row 1**:

```
det(A) = a₁₁ · C₁₁ + a₁₂ · C₁₂ + a₁₃ · C₁₃
```

where Cᵢⱼ = (-1)^(i+j) · Mᵢⱼ is the **cofactor** and Mᵢⱼ is the **minor** (determinant of the submatrix obtained by deleting row i and column j).

### Step-by-Step Example

```
A = [2   1  -1]
    [3   0   2]
    [1   4   3]

Expand along row 1:

det(A) = 2 · |0  2| - 1 · |3  2| + (-1) · |3  0|
             |4  3|       |1  3|            |1  4|

= 2(0·3 - 2·4) - 1(3·3 - 2·1) + (-1)(3·4 - 0·1)
= 2(0 - 8) - 1(9 - 2) - 1(12 - 0)
= 2(-8) - 1(7) - 1(12)
= -16 - 7 - 12
= -35
```

```python
A = np.array([[2,1,-1],[3,0,2],[1,4,3]])
print(np.linalg.det(A))  # -35.0
```

**Geometric meaning:** det(A) = ±(volume of parallelepiped formed by column vectors). Here |−35| = 35 is the volume, and the negative sign indicates orientation reversal.

---

## 6.4 Properties of Determinants

| Property | Statement | Example |
|----------|-----------|---------|
| **Row swap** | Swapping two rows negates the determinant | det(swap) = −det(A) |
| **Scalar row** | Multiplying a row by c scales det by c | det(cR₁, R₂) = c·det(A) |
| **Row addition** | Adding a multiple of one row to another doesn't change det | det unchanged |
| **Triangular** | det = product of diagonal entries | det(U) = u₁₁·u₂₂·...·uₙₙ |
| **Product** | det(AB) = det(A)·det(B) | Multiplicative |
| **Inverse** | det(A⁻¹) = 1/det(A) | Reciprocal |
| **Transpose** | det(Aᵀ) = det(A) | Same determinant |
| **Scalar matrix** | det(cA) = cⁿ·det(A) for n×n matrix | Scales by cⁿ |
| **Singular** | det(A) = 0 ⟺ A is not invertible | Columns are dependent |

### Key Proof: det(AB) = det(A)·det(B)

```
This is proven using the fact that elementary row operations have
known effects on determinants, and any matrix can be reduced to
row echelon form using these operations.

Quick verification:
A = [1 2; 3 4], det(A) = -2
B = [5 6; 7 8], det(B) = -2
AB = [19 22; 43 50], det(AB) = 19·50 - 22·43 = 950 - 946 = 4 = (-2)(-2) ✓
```

---

## 6.5 Computing Determinants Efficiently

### For Small Matrices

- **1×1:** det([a]) = a
- **2×2:** det = ad - bc
- **3×3:** Sarrus' rule or cofactor expansion

### For Large Matrices: LU Decomposition

```
A = LU   (where L is lower triangular, U is upper triangular)

det(A) = det(L) · det(U)
       = (product of L diagonal) × (product of U diagonal)

With partial pivoting: A = PLU
det(A) = det(P) · det(L) · det(U) = (-1)^s · 1 · (product of U diagonal)
         where s = number of row swaps
```

This reduces the complexity from O(n!) to **O(n³)**.

```python
from scipy import linalg
A = np.random.randn(5, 5)
P, L, U = linalg.lu(A)
det_via_lu = np.linalg.det(P) * np.prod(np.diag(U))
det_direct = np.linalg.det(A)
print(f"LU method: {det_via_lu:.4f}")
print(f"Direct:    {det_direct:.4f}")
```

---

## 6.6 Singular Matrices (det = 0)

A matrix with det(A) = 0 is called **singular**. This means:

```
1. A is NOT invertible
2. Columns are linearly dependent
3. Ax = 0 has non-trivial solutions
4. Ax = b may have no solution or infinitely many
5. At least one eigenvalue is 0
6. The transformation "collapses" space by at least one dimension
```

```
Example:     A = [1  2]     det = 1(4) - 2(2) = 0
                 [2  4]

Column 2 = 2 × Column 1 → linearly dependent!

Geometrically: maps all of ℝ² onto a line (dimension collapse).
```

---

## 6.7 Determinants and Volume Scaling

```
Unit square         After transformation A = [2  1]
(area = 1)                                   [0  3]
                          det(A) = 6
┌───┐              ╱╲
│   │    ──A──>   ╱  ╲        Area = |det(A)| = 6
│   │            ╱    ╲
└───┘           ╱──────╲

The determinant tells you the scaling factor of areas (2D) or volumes (3D).
```

**ML Application:** In probability, when you transform random variables `y = f(x)`, the probability density transforms as:

```
p(y) = p(x) · |det(∂x/∂y)|  =  p(x) / |det(J)|
```

This is the **change of variables formula**, fundamental to **Normalizing Flows** in deep generative models.

---

## 6.8 Cramer's Rule

For a system Ax = b where det(A) ≠ 0:

```
xᵢ = det(Aᵢ) / det(A)
```

where Aᵢ is A with column i replaced by b.

### Example

```
2x + y = 5        A = [2  1]    b = [5]
3x + 4y = 6           [3  4]        [6]

det(A) = 8 - 3 = 5

x = det([5  1]) / 5 = (20 - 6) / 5 = 14/5 = 2.8
        [6  4]

y = det([2  5]) / 5 = (12 - 15) / 5 = -3/5 = -0.6
        [3  6]
```

> Note: Cramer's rule is elegant but **impractical** for large systems (O(n!) without optimization). Use LU decomposition or iterative methods instead.

---

## 6.9 Worked Example: Full 3×3 Determinant Calculation

```
B = [1   3  -2]
    [0   1   4]
    [5   2   1]

Method: Cofactor expansion along column 1 (has a zero!)

det(B) = 1·|1  4| - 0·|3  -2| + 5·|3  -2|
           |2  1|     |2   1|     |1   4|

= 1(1 - 8) - 0 + 5(12 - (-2))
= 1(-7) + 5(14)
= -7 + 70
= 63
```

**Tip:** Expand along the row or column with the most zeros to save computation.

```python
B = np.array([[1,3,-2],[0,1,4],[5,2,1]])
print(np.linalg.det(B))  # 63.0
```

---

## 6.10 Summary Table

| Concept | Key Fact |
|---------|----------|
| det(2×2) | ad - bc |
| det(3×3) | Cofactor expansion or Sarrus' rule |
| det(AB) | det(A) · det(B) |
| det(Aᵀ) | det(A) |
| det(cA) | cⁿ · det(A) for n×n |
| det(A⁻¹) | 1 / det(A) |
| Singular | det = 0 → not invertible |
| Geometry | \|det\| = volume scaling factor |
| Eigenvalue link | det(A) = λ₁ · λ₂ · ... · λₙ |
| ML application | Normalizing flows, Gaussian distributions |

---

## 6.11 Quick Revision Questions

1. **What does det(A) = 0 tell you about the system Ax = b?**
2. **If you swap two rows of a matrix, what happens to the determinant?**
3. **Why is expanding along a row/column with zeros computationally advantageous?**
4. **How is the determinant used in the change-of-variables formula for probability distributions?**
5. **What is the determinant of an orthogonal matrix? Why?**
6. **For a 4×4 matrix, would you use cofactor expansion or LU decomposition? Why?**

---

[← Previous: Chapter 5 — Transpose and Trace](./05-transpose-and-trace.md) | [Next Chapter → Chapter 7: Matrix Inverse](./07-matrix-inverse.md)
