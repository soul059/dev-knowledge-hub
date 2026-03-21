# Chapter 7: Matrix Inverse

> **Unit 1 — Linear Algebra Essentials**
> The inverse "undoes" a matrix transformation. It is central to solving linear systems, computing closed-form solutions, and understanding model parameters.

---

## 7.1 Definition

For a square matrix A ∈ ℝⁿˣⁿ, its **inverse** A⁻¹ satisfies:

```
AA⁻¹ = A⁻¹A = I
```

If A⁻¹ exists, A is called **invertible** (or **non-singular**).

```python
import numpy as np
A = np.array([[2, 1], [5, 3]])
A_inv = np.linalg.inv(A)
print(A_inv)          # [[ 3, -1], [-5,  2]]
print(A @ A_inv)      # [[1, 0], [0, 1]]  (identity)
```

---

## 7.2 Conditions for Invertibility

A matrix A is invertible **if and only if** ANY of the following hold:

| Condition | Statement |
|-----------|-----------|
| 1 | det(A) ≠ 0 |
| 2 | Columns of A are linearly independent |
| 3 | Rows of A are linearly independent |
| 4 | rank(A) = n (full rank) |
| 5 | Null space = {0} (only trivial solution to Ax = 0) |
| 6 | All eigenvalues are non-zero |
| 7 | A can be reduced to I by row operations |
| 8 | Ax = b has a unique solution for every b |

These are all **equivalent statements** — if one is true, all are true.

---

## 7.3 The 2×2 Inverse Formula

```
A = [a  b]      A⁻¹ = (1/det(A)) · [ d  -b]
    [c  d]                          [-c   a]
```

### Step-by-Step Example

```
A = [4  7]
    [2  6]

det(A) = 4(6) - 7(2) = 24 - 14 = 10

A⁻¹ = (1/10) [ 6  -7] = [ 0.6  -0.7]
              [-2   4]   [-0.2   0.4]

Check: A · A⁻¹ = [4(0.6)+7(-0.2)   4(-0.7)+7(0.4)] = [1  0] ✓
                  [2(0.6)+6(-0.2)   2(-0.7)+6(0.4)]   [0  1]
```

---

## 7.4 Computing Inverse: Adjugate Method

For any n×n matrix:

```
A⁻¹ = (1/det(A)) · adj(A)
```

where adj(A) is the **adjugate** (transpose of the cofactor matrix).

### Step-by-Step for 3×3

```
A = [1  2  3]
    [0  1  4]
    [5  6  0]

Step 1: Compute all cofactors Cᵢⱼ = (-1)^(i+j) · Mᵢⱼ

C₁₁ = +|1 4| = 0-24 = -24     C₁₂ = -|0 4| = -(0-20) = 20
       |6 0|                        |5 0|
C₁₃ = +|0 1| = 0-5 = -5      C₂₁ = -|2 3| = -(0-18) = 18
       |5 6|                        |6 0|
C₂₂ = +|1 3| = 0-15 = -15    C₂₃ = -|1 2| = -(6-10) = 4
       |5 0|                        |5 6|
C₃₁ = +|2 3| = 8-3 = 5       C₃₂ = -|1 3| = -(4-0) = -4
       |1 4|                        |0 4|
C₃₃ = +|1 2| = 1-0 = 1
       |0 1|

Step 2: Cofactor matrix
Cof(A) = [-24  20  -5]
         [ 18 -15   4]
         [  5  -4   1]

Step 3: Adjugate = transpose of cofactor matrix
adj(A) = [-24  18   5]
         [ 20 -15  -4]
         [ -5   4   1]

Step 4: det(A) = 1(-24) - 2(20) + 3(-5) = -24 - 40 - 15 = ... wait
       Let me recalculate: expanding along row 1:
       det(A) = 1(0-24) - 2(0-20) + 3(0-5) = -24 + 40 - 15 = 1

A⁻¹ = (1/1) · adj(A) = [-24  18   5]
                        [ 20 -15  -4]
                        [ -5   4   1]
```

```python
A = np.array([[1,2,3],[0,1,4],[5,6,0]])
print(np.linalg.inv(A))
# [[-24.  18.   5.]
#  [ 20. -15.  -4.]
#  [ -5.   4.   1.]]
```

---

## 7.5 Computing Inverse: Gauss-Jordan Method

Augment A with I and row-reduce to get [I | A⁻¹]:

```
[A | I] → row operations → [I | A⁻¹]
```

### Example

```
A = [2  1]
    [5  3]

[2  1 | 1  0]
[5  3 | 0  1]

R₂ ← R₂ - (5/2)R₁:
[2  1   |  1    0  ]
[0  1/2 | -5/2  1  ]

R₂ ← 2R₂:
[2  1 |  1   0]
[0  1 | -5   2]

R₁ ← R₁ - R₂:
[2  0 |  6  -2]
[0  1 | -5   2]

R₁ ← R₁/2:
[1  0 |  3  -1]
[0  1 | -5   2]

A⁻¹ = [ 3  -1]
      [-5   2]
```

```python
A = np.array([[2, 1], [5, 3]])
print(np.linalg.inv(A))  # [[ 3., -1.], [-5.,  2.]]
```

---

## 7.6 Properties of the Inverse

| Property | Formula |
|----------|---------|
| Uniqueness | If A⁻¹ exists, it is unique |
| Double inverse | (A⁻¹)⁻¹ = A |
| Product | (AB)⁻¹ = B⁻¹A⁻¹ |
| Transpose | (Aᵀ)⁻¹ = (A⁻¹)ᵀ |
| Scalar | (cA)⁻¹ = (1/c)A⁻¹ |
| Determinant | det(A⁻¹) = 1/det(A) |
| Power | (Aⁿ)⁻¹ = (A⁻¹)ⁿ |
| Orthogonal | If Q is orthogonal: Q⁻¹ = Qᵀ |
| Diagonal | diag(d₁,...,dₙ)⁻¹ = diag(1/d₁,...,1/dₙ) |

### Why (AB)⁻¹ = B⁻¹A⁻¹ (Not A⁻¹B⁻¹)?

```
Verify: (AB)(B⁻¹A⁻¹) = A(BB⁻¹)A⁻¹ = AIA⁻¹ = AA⁻¹ = I  ✓

Think of it as removing layers: put on socks then shoes →
take off shoes first, then socks. The order reverses!
```

---

## 7.7 Solving Linear Systems with the Inverse

The system Ax = b has solution:

```
x = A⁻¹b    (when A is invertible)
```

### Example: 3 Equations, 3 Unknowns

```
x₁ + 2x₂ + 3x₃ = 1
      x₂ + 4x₃ = 2
5x₁ + 6x₂      = 3

A = [1 2 3]    b = [1]
    [0 1 4]        [2]
    [5 6 0]        [3]

x = A⁻¹b = [-24  18   5] [1]   [-24+36+15]   [27]
            [ 20 -15  -4] [2] = [ 20-30-12] = [-22]
            [ -5   4   1] [3]   [ -5+8+3  ]   [ 6]

x₁ = 27, x₂ = -22, x₃ = 6
```

> ⚠️ **In practice, never compute A⁻¹ explicitly** to solve Ax = b. Instead, use LU decomposition or `np.linalg.solve(A, b)` — it's faster and more numerically stable.

```python
A = np.array([[1,2,3],[0,1,4],[5,6,0]])
b = np.array([1, 2, 3])
x = np.linalg.solve(A, b)  # preferred over inv(A) @ b
print(x)  # [27, -22, 6]
```

---

## 7.8 Pseudo-Inverse (Moore-Penrose)

When A is **not square** or **not invertible**, the pseudo-inverse A⁺ is the best substitute:

```
For A ∈ ℝᵐˣⁿ:

If m > n (overdetermined, more equations than unknowns):
    A⁺ = (AᵀA)⁻¹Aᵀ     (left pseudo-inverse)
    Gives least-squares solution: x = A⁺b

If m < n (underdetermined, more unknowns than equations):
    A⁺ = Aᵀ(AAᵀ)⁻¹     (right pseudo-inverse)
    Gives minimum-norm solution

General (via SVD):
    A = UΣVᵀ  →  A⁺ = VΣ⁺Uᵀ
    where Σ⁺ inverts only the non-zero singular values
```

```python
A = np.array([[1, 2], [3, 4], [5, 6]])  # 3×2 (overdetermined)
A_pinv = np.linalg.pinv(A)
print(A_pinv.shape)  # (2, 3)

# Solve overdetermined system (least-squares)
b = np.array([1, 2, 3])
x_ls = A_pinv @ b
print(x_ls)
```

**ML Application:** The normal equation for linear regression `w = (XᵀX)⁻¹Xᵀy` is exactly `w = X⁺y`.

---

## 7.9 When Not to Invert

| Situation | Instead of Inverse, Use |
|-----------|------------------------|
| Solving Ax = b | `np.linalg.solve(A, b)` (LU decomposition) |
| Solving AX = B | `np.linalg.solve(A, B)` |
| Least squares | `np.linalg.lstsq(A, b)` |
| Ill-conditioned A | Regularization or SVD-based methods |

**Why?**
1. Computing A⁻¹ is O(n³) AND then multiplying is another O(n²)
2. `solve` uses LU which is O(n³) total — faster
3. Floating-point errors accumulate more with explicit inverse
4. Ill-conditioned matrices → inversion amplifies numerical errors

---

## 7.10 Condition Number

The **condition number** κ(A) measures how sensitive A⁻¹b is to perturbations:

```
κ(A) = ‖A‖ · ‖A⁻¹‖ = σ_max / σ_min
```

| κ(A) | Interpretation |
|------|---------------|
| ≈ 1 | Well-conditioned |
| 10²–10⁴ | Moderate |
| > 10⁶ | Ill-conditioned — beware of numerical errors |
| ∞ | Singular (not invertible) |

```python
A = np.array([[1, 1], [1, 1.0001]])
print(f"Condition number: {np.linalg.cond(A):.0f}")  # ~40000 (ill-conditioned)
```

---

## 7.11 Summary Table

| Concept | Key Formula / Rule |
|---------|-------------------|
| Definition | AA⁻¹ = A⁻¹A = I |
| 2×2 formula | (1/det) · [d, −b; −c, a] |
| General formula | A⁻¹ = (1/det) · adj(A) |
| Product inverse | (AB)⁻¹ = B⁻¹A⁻¹ |
| Exists iff | det(A) ≠ 0, full rank, etc. |
| Pseudo-inverse | A⁺ = VΣ⁺Uᵀ (via SVD) |
| Don't invert | Use `np.linalg.solve` instead |
| Condition number | κ = σ_max/σ_min |

---

## 7.12 Quick Revision Questions

1. **List three equivalent conditions for a matrix to be invertible.**
2. **Why is (AB)⁻¹ = B⁻¹A⁻¹ and not A⁻¹B⁻¹?**
3. **When should you use the pseudo-inverse instead of the regular inverse?**
4. **Why is `np.linalg.solve(A, b)` preferred over `np.linalg.inv(A) @ b`?**
5. **What does a large condition number imply about solving Ax = b?**
6. **In the normal equation w = (XᵀX)⁻¹Xᵀy, what could go wrong if XᵀX is singular?**

---

[← Previous: Chapter 6 — Determinants](./06-determinants.md) | [Next Chapter → Chapter 8: Linear Transformations](./08-linear-transformations.md)
