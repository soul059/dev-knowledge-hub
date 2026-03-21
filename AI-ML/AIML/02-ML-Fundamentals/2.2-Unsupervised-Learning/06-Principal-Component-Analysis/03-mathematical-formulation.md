# 3. Mathematical Formulation of PCA

[← Previous: PCA Intuition](./02-pca-intuition.md) | [Next: Eigenvalue Decomposition →](./04-eigenvalue-decomposition.md)

---

## Chapter Overview

This chapter develops the **rigorous mathematical foundation** of PCA. Starting from centered data, we derive the covariance matrix, formulate the optimization problem (maximize projected variance subject to unit-norm constraint), solve it using the method of Lagrange multipliers, and arrive at the eigenvalue problem that defines the principal components.

---

## 3.1 Step 1: Center the Data

### Why Center?

```
PCA requires zero-mean data. Centering ensures that:
1. The covariance matrix correctly captures spread around the mean
2. The first PC passes through the origin
3. The projection is unbiased

Without centering:
  PCA would find the direction from the origin to the data cloud,
  which is the MEAN direction — not the variance direction!
```

### Centering Formula

```
Given data matrix X of shape (N × d):
  rows = data points, columns = features

Mean vector:
  x̄ = (1/N) Σₙ₌₁ᴺ xₙ

Centered data:
  x̃ₙ = xₙ - x̄     for each point
  X̃ = X - 1ₙx̄ᵀ    (matrix form, where 1ₙ is N×1 vector of ones)

After centering:
  (1/N) Σₙ x̃ₙ = 0   (mean of centered data = zero vector)
```

### Example

```
Original data:
  X = [ 4  6 ]     Mean: x̄ = [3, 4]
      [ 2  2 ]
      [ 3  4 ]

Centered data:
  X̃ = [ 4-3  6-4 ] = [ 1   2 ]
      [ 2-3  2-4 ]   [-1  -2 ]
      [ 3-3  4-4 ]   [ 0   0 ]

Check: mean of X̃ = [(1-1+0)/3, (2-2+0)/3] = [0, 0] ✓
```

---

## 3.2 Step 2: Compute the Covariance Matrix

### Definition

```
The covariance matrix S (or Σ) of centered data X̃:

  S = (1/N) X̃ᵀX̃    (population covariance)
  
  or
  
  S = (1/(N-1)) X̃ᵀX̃    (sample covariance, Bessel's correction)

  Size: d × d (feature × feature)
```

### What S Contains

```
For d=3 features:

  S = [ Var(x₁)      Cov(x₁,x₂)  Cov(x₁,x₃) ]
      [ Cov(x₂,x₁)  Var(x₂)      Cov(x₂,x₃) ]
      [ Cov(x₃,x₁)  Cov(x₃,x₂)  Var(x₃)     ]

Properties:
  1. S is SYMMETRIC:  S = Sᵀ (Cov(xᵢ,xⱼ) = Cov(xⱼ,xᵢ))
  2. S is POSITIVE SEMI-DEFINITE: wᵀSw ≥ 0 for all w
  3. Diagonal = variances (always ≥ 0)
  4. Off-diagonal = covariances (can be negative)
  5. trace(S) = total variance = Σ Var(xᵢ)
```

### Worked Example

```
Centered data X̃ = [ 1   2 ]
                   [-1  -2 ]
                   [ 0   0 ]

S = (1/3) X̃ᵀX̃ = (1/3) [ 1 -1  0 ] [ 1   2 ]
                         [ 2 -2  0 ] [-1  -2 ]
                                      [ 0   0 ]

  = (1/3) [ 1·1+(-1)·(-1)+0·0    1·2+(-1)·(-2)+0·0  ]
          [ 2·1+(-2)·(-1)+0·0    2·2+(-2)·(-2)+0·0  ]

  = (1/3) [ 2  4 ]
          [ 4  8 ]

  = [ 2/3   4/3 ]
    [ 4/3   8/3 ]

Check:
  Var(x₁) = (1²+(-1)²+0²)/3 = 2/3 ✓
  Var(x₂) = (2²+(-2)²+0²)/3 = 8/3 ✓
  Cov(x₁,x₂) = (1·2+(-1)·(-2)+0·0)/3 = 4/3 ✓
```

---

## 3.3 Step 3: The Optimization Problem

### What We Want

Find a unit vector **w** such that the **variance of the projected data** is maximized.

```
The projection of x̃ₙ onto w:
  zₙ = x̃ₙᵀw     (scalar)

Variance of projections:
  Var(z) = (1/N) Σₙ zₙ²
         = (1/N) Σₙ (x̃ₙᵀw)²
         = (1/N) Σₙ (wᵀx̃ₙ)(x̃ₙᵀw)
         = wᵀ [(1/N) Σₙ x̃ₙx̃ₙᵀ] w
         = wᵀ S w

  where S = (1/N) X̃ᵀX̃ is the covariance matrix
```

### The Constrained Optimization

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  MAXIMIZE:   wᵀSw       (projected variance)               │
│                                                              │
│  SUBJECT TO: wᵀw = 1    (unit vector constraint)           │
│                                                              │
│  Why the constraint?                                         │
│  Without ||w|| = 1, we could make wᵀSw arbitrarily large   │
│  by scaling w → ∞. The unit norm constraint makes the       │
│  problem well-defined.                                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 Step 4: Lagrange Multiplier Solution

### Setting Up the Lagrangian

```
L(w, λ) = wᵀSw - λ(wᵀw - 1)

where λ is the Lagrange multiplier for the constraint wᵀw = 1.
```

### Taking the Derivative and Setting to Zero

```
∂L/∂w = 2Sw - 2λw = 0

Simplify:
  Sw = λw

This is an EIGENVALUE EQUATION!

  S · w = λ · w
  ↑   ↑   ↑   ↑
  cov  eigen  eigen  eigen
  matrix vector value vector
```

### The Key Result

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  The principal components are the EIGENVECTORS of the            │
│  covariance matrix S.                                            │
│                                                                  │
│  The variance along each PC is the corresponding EIGENVALUE.     │
│                                                                  │
│  Sw = λw                                                         │
│                                                                  │
│  Substituting back:                                              │
│    wᵀSw = wᵀ(λw) = λ(wᵀw) = λ · 1 = λ                        │
│                                                                  │
│  So the projected variance = eigenvalue λ                        │
│                                                                  │
│  To MAXIMIZE variance → choose the LARGEST eigenvalue            │
│  → its eigenvector is PC1!                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Why This Makes Sense

```
The eigenvectors of S are the "natural" axes of the data ellipsoid.
The eigenvalues are the squared "radii" along those axes.

  Original axes:        Eigenvector axes:
  
  y │  ╱──╮              λ₂ │
    │╱    │                  │ ╭────╮
    │     │  ──→             │ │    │
    │╲    │              ────┼─┤    ├────  λ₁
    │  ╲──╯                  │ │    │
    └───────── x             │ ╰────╯
                             └──────────
  
  Data is tilted            Data aligned with axes
  (off-diagonal covariance)  (diagonal in PC space)
```

---

## 3.5 All Principal Components

### For the k-th PC

```
PC1: maximize wᵀSw s.t. ||w||=1
     → w₁ = eigenvector with LARGEST eigenvalue λ₁

PC2: maximize wᵀSw s.t. ||w||=1 AND wᵀw₁=0 (orthogonal to PC1)
     → w₂ = eigenvector with SECOND LARGEST eigenvalue λ₂

PC3: maximize wᵀSw s.t. ||w||=1 AND wᵀw₁=0 AND wᵀw₂=0
     → w₃ = eigenvector with THIRD LARGEST eigenvalue λ₃

...continuing for all d components.

This works because:
  • Eigenvectors of a symmetric matrix are AUTOMATICALLY orthogonal
  • S is symmetric (covariance matrix)
  • So we get all PCs from one eigendecomposition!
```

### Matrix Form

```
S = W Λ Wᵀ    (eigendecomposition)

where:
  W = [w₁ | w₂ | ... | w_d]   d×d matrix of eigenvectors (columns)
  Λ = diag(λ₁, λ₂, ..., λ_d)  d×d diagonal matrix of eigenvalues

  with: λ₁ ≥ λ₂ ≥ ... ≥ λ_d ≥ 0  (sorted in decreasing order)

Projection to k dimensions:
  Z = X̃ · W_k    where W_k = first k columns of W

  Z is N×k: the data in the new k-dimensional PC space
```

---

## 3.6 Complete Worked Example

### Data

```
5 points in 2D:
  x₁ = (4, 11)
  x₂ = (8, 4)
  x₃ = (13, 5)
  x₄ = (7, 14)
  x₅ = (3, 6)
```

### Step 1: Center

```
x̄ = [(4+8+13+7+3)/5, (11+4+5+14+6)/5] = [7, 8]

Centered:
  x̃₁ = (4-7, 11-8)  = (-3, 3)
  x̃₂ = (8-7, 4-8)   = (1, -4)
  x̃₃ = (13-7, 5-8)  = (6, -3)
  x̃₄ = (7-7, 14-8)  = (0, 6)
  x̃₅ = (3-7, 6-8)   = (-4, -2)
```

### Step 2: Covariance Matrix

```
S = (1/5) Σ x̃ₙx̃ₙᵀ

  = (1/5) [(-3)²+1²+6²+0²+(-4)²    (-3)(3)+1(-4)+6(-3)+0(6)+(-4)(-2)]
          [(-3)(3)+1(-4)+6(-3)+0(6)+(-4)(-2)    3²+(-4)²+(-3)²+6²+(-2)²]

  = (1/5) [9+1+36+0+16    -9-4-18+0+8]
          [-9-4-18+0+8     9+16+9+36+4]

  = (1/5) [62   -23]
          [-23   74]

  = [12.4   -4.6]
    [-4.6   14.8]
```

### Step 3: Eigenvalues

```
det(S - λI) = 0

| 12.4-λ    -4.6  |
| -4.6    14.8-λ  | = 0

(12.4-λ)(14.8-λ) - (-4.6)(-4.6) = 0
183.52 - 12.4λ - 14.8λ + λ² - 21.16 = 0
λ² - 27.2λ + 162.36 = 0

λ = (27.2 ± √(739.84 - 649.44)) / 2
  = (27.2 ± √90.4) / 2
  = (27.2 ± 9.508) / 2

λ₁ = (27.2 + 9.508) / 2 = 18.354
λ₂ = (27.2 - 9.508) / 2 = 8.846

Check: λ₁ + λ₂ = 18.354 + 8.846 = 27.2 = trace(S) = 12.4 + 14.8 ✓
```

### Step 4: Eigenvectors

```
For λ₁ = 18.354:
  (S - 18.354·I)w = 0

  [12.4-18.354    -4.6    ] [w₁]   [0]
  [-4.6       14.8-18.354] [w₂] = [0]

  [-5.954   -4.6 ] [w₁]   [0]
  [-4.6    -3.554] [w₂] = [0]

  -5.954·w₁ - 4.6·w₂ = 0
  w₁ = -4.6/5.954 · w₂ = -0.773·w₂

  Take w₂ = 1: w₁ = -0.773
  Normalize: ||w|| = √(0.773² + 1²) = √(1.597) = 1.264
  w₁ = [-0.773/1.264, 1/1.264] = [-0.611, 0.791]

For λ₂ = 8.846:
  Similarly: w₂ = [0.791, 0.611]  (perpendicular to w₁)

Verify orthogonality:
  w₁ · w₂ = (-0.611)(0.791) + (0.791)(0.611) = -0.483 + 0.483 = 0 ✓
```

### Step 5: Results

```
PC1: w₁ = [-0.611, 0.791],  λ₁ = 18.354  (67.5% of variance)
PC2: w₂ = [0.791, 0.611],   λ₂ = 8.846   (32.5% of variance)

Total variance = 18.354 + 8.846 = 27.2

If we keep only PC1 (2D → 1D):
  We preserve 67.5% of the variance
  We lose 32.5% (the variance along PC2)
```

---

## 3.7 Summary Table

| Step | Operation | Formula | Output |
|------|-----------|---------|--------|
| **1. Center** | Subtract mean | X̃ = X - 1x̄ᵀ | Centered data |
| **2. Covariance** | Compute scatter | S = (1/N)X̃ᵀX̃ | d×d symmetric matrix |
| **3. Optimize** | Max wᵀSw, \|\|w\|\|=1 | Lagrangian → Sw = λw | Eigenvalue problem |
| **4. Solve** | Eigendecompose S | S = WΛWᵀ | Eigenvectors + values |
| **5. Sort** | By eigenvalue (desc) | λ₁ ≥ λ₂ ≥ ... ≥ λ_d | Ordered PCs |
| **6. Project** | Keep top k | Z = X̃·W_k | k-dimensional data |

---

## 3.8 Revision Questions

1. **Why must data be centered** before PCA? What would happen if we applied PCA to uncentered data?

2. **Derive the optimization problem** for PC1. Start from the variance of projections and arrive at the eigenvalue equation Sw = λw.

3. **Prove that the projected variance** equals the eigenvalue. That is, show wᵀSw = λ when Sw = λw and wᵀw = 1.

4. **Why does the Lagrange multiplier λ** have a physical interpretation (variance)? How does this connect the optimization solution to the amount of information captured?

5. **For the covariance matrix** S = [[4, 2], [2, 3]], find the eigenvalues and eigenvectors. Which direction is PC1? What percentage of variance does it capture?

6. **Explain why eigenvectors of a symmetric matrix** are orthogonal. Why is this property essential for PCA?

---

[← Previous: PCA Intuition](./02-pca-intuition.md) | [Next: Eigenvalue Decomposition →](./04-eigenvalue-decomposition.md)
