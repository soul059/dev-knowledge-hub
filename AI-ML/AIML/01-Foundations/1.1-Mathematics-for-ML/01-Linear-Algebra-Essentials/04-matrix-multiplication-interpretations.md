# Chapter 4: Matrix Multiplication — Four Interpretations

> **Unit 1 — Linear Algebra Essentials**
> Understanding matrix multiplication from four different perspectives deepens intuition and unlocks new problem-solving approaches.

---

## 4.1 Why Four Views?

Matrix multiplication `C = AB` can be understood in four equivalent but conceptually different ways. Each view shines in different contexts.

We will use this running example throughout:

```
A = [1  2]    B = [5  6]    C = AB = [19  22]
    [3  4]        [7  8]             [43  50]
```

```python
import numpy as np
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
C = A @ B
print(C)  # [[19, 22], [43, 50]]
```

---

## 4.2 View 1: Row-Column Dot Product (Standard View)

**Each entry cᵢⱼ is the dot product of row i of A with column j of B.**

```
cᵢⱼ = (row i of A) · (column j of B) = Σₖ aᵢₖ · bₖⱼ
```

### Visualization

```
         B
      [5  6]
      [7  8]
       ↓  ↓
A     col1 col2
[1 2]→ •         c₁₁ = 1·5 + 2·7 = 19     c₁₂ = 1·6 + 2·8 = 22
[3 4]→            c₂₁ = 3·5 + 4·7 = 43     c₂₂ = 3·6 + 4·8 = 50
```

### When This View Is Useful

- Computing individual entries of C
- Understanding complexity: each entry requires n multiplications → total O(m·n·p)
- This is how we teach matrix multiplication initially

---

## 4.3 View 2: Linear Combination of Columns

**Each column of C is a linear combination of the columns of A, with coefficients from the corresponding column of B.**

```
c_j = A · b_j = b₁ⱼ · (col 1 of A) + b₂ⱼ · (col 2 of A) + ...
```

### Worked Out

```
Column 1 of C:                    Column 2 of C:
c₁ = A · [5] = 5·[1] + 7·[2]     c₂ = A · [6] = 6·[1] + 8·[2]
         [7]     [3]    [4]              [8]     [3]    [4]

   = [5]  + [14] = [19]              = [6]  + [16] = [22]
     [15]   [28]   [43]                [18]   [32]   [50]
```

```python
# Column 1 of C = A @ column 1 of B
col1_C = A @ B[:, 0]
print(col1_C)  # [19, 43]
```

### When This View Is Useful

- **Neural networks:** Ax means "combine the columns of A (basis vectors) using weights x"
- Understanding range/column space: columns of C live in the column space of A
- Interpreting what a weight matrix does to a batch of inputs

---

## 4.4 View 3: Linear Combination of Rows

**Each row of C is a linear combination of the rows of B, with coefficients from the corresponding row of A.**

```
rowᵢ(C) = aᵢ₁ · (row 1 of B) + aᵢ₂ · (row 2 of B) + ...
```

### Worked Out

```
Row 1 of C:                        Row 2 of C:
= 1·[5 6] + 2·[7 8]                = 3·[5 6] + 4·[7 8]
= [5 6] + [14 16]                  = [15 18] + [28 32]
= [19 22]                          = [43 50]
```

```python
# Row 0 of C = linear combination of rows of B, weighted by row 0 of A
row0_C = A[0, 0] * B[0, :] + A[0, 1] * B[1, :]
print(row0_C)  # [19, 22]
```

### When This View Is Useful

- Understanding row space: rows of C live in the row space of B
- Backward pass in neural networks (gradient flows through rows)
- Database/query operations: selecting and mixing rows

---

## 4.5 View 4: Sum of Outer Products

**C is the sum of outer products of corresponding columns of A and rows of B.**

```
C = Σₖ (column k of A) ⊗ (row k of B)
  = a₁ · b₁ᵀ + a₂ · b₂ᵀ + ... + aₙ · bₙᵀ
```

Each outer product produces a **rank-1 matrix**, and C is their sum.

### Worked Out

```
Outer product 1:                  Outer product 2:
col 1 of A ⊗ row 1 of B          col 2 of A ⊗ row 2 of B

[1] · [5  6] = [5   6]           [2] · [7  8] = [14  16]
[3]            [15  18]           [4]            [28  32]

C = [5   6]  + [14  16] = [19  22]
    [15  18]   [28  32]   [43  50]  ✓
```

```python
# Sum of outer products
C_check = np.outer(A[:,0], B[0,:]) + np.outer(A[:,1], B[1,:])
print(C_check)  # [[19, 22], [43, 50]]
```

### When This View Is Useful

- **Low-rank approximations:** truncated SVD keeps only top-k outer products
- **Gradient updates:** weight update `ΔW = η · δ ⊗ x` is a rank-1 outer product
- Understanding matrix rank: rank(C) ≤ min(rank(A), rank(B))
- Matrix completion / recommender systems

---

## 4.6 Side-by-Side Comparison

```
┌─────────────────────┬──────────────────────────────────────┬─────────────────────┐
│   View              │   Formula                            │   Key Insight       │
├─────────────────────┼──────────────────────────────────────┼─────────────────────┤
│ 1. Dot Product      │ cᵢⱼ = rowᵢ(A) · colⱼ(B)            │ Entry-by-entry      │
│ 2. Column Combo     │ colⱼ(C) = A · colⱼ(B)               │ Columns of A mixed  │
│ 3. Row Combo        │ rowᵢ(C) = Σₖ aᵢₖ · rowₖ(B)          │ Rows of B mixed     │
│ 4. Outer Products   │ C = Σₖ colₖ(A) · rowₖ(B)ᵀ           │ Sum of rank-1 mats  │
└─────────────────────┴──────────────────────────────────────┴─────────────────────┘
```

---

## 4.7 Larger Example: 3×2 times 2×3

```
A = [1  4]      B = [2  0  1]
    [2  5]          [3  1  0]
    [3  6]
```

### View 1 — Dot Product

```
C₁₁ = 1·2 + 4·3 = 14    C₁₂ = 1·0 + 4·1 = 4    C₁₃ = 1·1 + 4·0 = 1
C₂₁ = 2·2 + 5·3 = 19    C₂₂ = 2·0 + 5·1 = 5    C₂₃ = 2·1 + 5·0 = 2
C₃₁ = 3·2 + 6·3 = 24    C₃₂ = 3·0 + 6·1 = 6    C₃₃ = 3·1 + 6·0 = 3

C = [14  4  1]
    [19  5  2]
    [24  6  3]
```

### View 4 — Outer Products

```
[1]              [4]
[2] · [2 0 1] + [5] · [3 1 0]
[3]              [6]

= [2  0  1]   [12  4  0]   [14  4  1]
  [4  0  2] + [15  5  0] = [19  5  2]
  [6  0  3]   [18  6  0]   [24  6  3]  ✓
```

```python
A = np.array([[1,4],[2,5],[3,6]])
B = np.array([[2,0,1],[3,1,0]])
print(A @ B)
# [[14  4  1]
#  [19  5  2]
#  [24  6  3]]
```

---

## 4.8 ML Applications of Each View

| View | ML Application |
|------|---------------|
| **Dot product** | Computing similarity scores, attention weights |
| **Column combination** | Forward pass: output = weighted sum of learned features |
| **Row combination** | Backpropagation: gradient = weighted sum of downstream gradients |
| **Outer product** | Weight gradient `∂L/∂W = δ · xᵀ`, low-rank factorization in recommender systems |

### Example: Attention Mechanism

```
Q, K, V are matrices.
Attention = softmax(QKᵀ/√d) V

- QKᵀ → dot product view (similarity between queries and keys)
- Result × V → column combination view (weighted sum of value vectors)
```

---

## 4.9 Computational Perspective

```
Standard matrix multiplication: O(n³)

Outer product view → easily parallelizable on GPUs
Column view → natural for column-major storage (Fortran, MATLAB)
Row view → natural for row-major storage (C, NumPy)
```

The four views also connect to **blocked/tiled** matrix multiplication strategies used in high-performance computing (BLAS libraries).

---

## 4.10 Summary Table

| View | What You Compute | Result Unit | Best For |
|------|-----------------|-------------|----------|
| Dot product | Single entry cᵢⱼ | Scalar | Manual calculation |
| Column combo | One column of C | Vector | Understanding Ax |
| Row combo | One row of C | Vector | Understanding xᵀA |
| Outer products | Full C as rank-1 sum | Matrix | Rank, SVD, gradients |

---

## 4.11 Quick Revision Questions

1. **In the column combination view, what determines the coefficients for combining columns of A?**
2. **Why does the outer product view reveal the rank of the product matrix?**
3. **Which view best explains why columns of AB lie in the column space of A?**
4. **In backpropagation, `∂L/∂W = δᵀ · x`. Which view of matrix multiplication does this correspond to?**
5. **If A is m×n and B is n×p, how many rank-1 outer products are summed?**
6. **Which view is most natural for understanding the attention mechanism in Transformers?**

---

[← Previous: Chapter 3 — Matrix Types](./03-matrix-types.md) | [Next Chapter → Chapter 5: Transpose and Trace](./05-transpose-and-trace.md)
