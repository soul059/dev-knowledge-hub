# Chapter 2: Matrix Operations

> **Unit 1 — Linear Algebra Essentials**
> Matrices are the language of data and transformations in ML. Master the core operations here.

---

## 2.1 What Is a Matrix?

A **matrix** is a rectangular array of numbers arranged in rows and columns.

```
        n columns
       ┌─────────────┐
m rows │ a₁₁  a₁₂  a₁₃ │    A ∈ ℝᵐˣⁿ
       │ a₂₁  a₂₂  a₂₃ │    (m rows, n columns)
       └─────────────┘

Notation: A = [aᵢⱼ]   where i = row index, j = column index
```

In ML, a matrix typically represents a **dataset** (rows = samples, columns = features) or a **weight matrix** in a neural network layer.

```python
import numpy as np
# Dataset: 3 samples × 4 features
X = np.array([[5.1, 3.5, 1.4, 0.2],
              [4.9, 3.0, 1.4, 0.2],
              [7.0, 3.2, 4.7, 1.4]])
print(X.shape)  # (3, 4)
```

---

## 2.2 Matrix Addition & Subtraction

Two matrices **must have the same dimensions** (m × n) to be added or subtracted.

```
A + B = [aᵢⱼ + bᵢⱼ]    (element-wise)
A - B = [aᵢⱼ - bᵢⱼ]
```

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(A + B)   # [[ 6,  8], [10, 12]]
print(A - B)   # [[-4, -4], [-4, -4]]
```

**Properties:**
- Commutative: A + B = B + A
- Associative: (A + B) + C = A + (B + C)
- Zero matrix identity: A + **0** = A

---

## 2.3 Scalar Multiplication

Multiply every element by a scalar c ∈ ℝ:

```
cA = [c · aᵢⱼ]
```

```python
print(3 * A)   # [[ 3,  6], [ 9, 12]]
```

**Properties:**
- Distributive: c(A + B) = cA + cB
- Associative: (cd)A = c(dA)
- Identity: 1 · A = A

---

## 2.4 Matrix-Vector Multiplication

For A ∈ ℝᵐˣⁿ and x ∈ ℝⁿ, the product Ax ∈ ℝᵐ:

```
           [a₁₁  a₁₂  a₁₃] [x₁]   [a₁₁x₁ + a₁₂x₂ + a₁₃x₃]
Ax =       [a₂₁  a₂₂  a₂₃] [x₂] = [a₂₁x₁ + a₂₂x₂ + a₂₃x₃]
                              [x₃]

Each row of A is dotted with x.
```

### Step-by-Step Example

```
A = [1  2  3]    x = [2]
    [4  5  6]        [1]
                     [0]

Row 1 · x = 1(2) + 2(1) + 3(0) = 4
Row 2 · x = 4(2) + 5(1) + 6(0) = 13

Ax = [4]
     [13]
```

```python
A = np.array([[1,2,3],[4,5,6]])
x = np.array([2,1,0])
print(A @ x)  # [ 4, 13]
```

**ML Interpretation:** Matrix-vector product computes all neuron outputs in a fully-connected layer simultaneously: `z = Wx + b`.

---

## 2.5 Matrix-Matrix Multiplication

For A ∈ ℝᵐˣⁿ and B ∈ ℝⁿˣᵖ, the product C = AB ∈ ℝᵐˣᵖ:

```
cᵢⱼ = Σₖ aᵢₖ · bₖⱼ     (dot product of row i of A with column j of B)
```

**Dimension rule:**

```
(m × n) · (n × p) = (m × p)
         ↑   ↑
     must match
```

### Step-by-Step Example

```
A = [1  2]    B = [5  6]
    [3  4]        [7  8]

C₁₁ = 1(5) + 2(7) = 19    C₁₂ = 1(6) + 2(8) = 22
C₂₁ = 3(5) + 4(7) = 43    C₂₂ = 3(6) + 4(8) = 50

AB = [19  22]
     [43  50]
```

```python
A = np.array([[1,2],[3,4]])
B = np.array([[5,6],[7,8]])
print(A @ B)      # [[19, 22], [43, 50]]
print(np.matmul(A, B))  # same
```

**Key Properties:**
- **NOT commutative:** AB ≠ BA in general
- Associative: (AB)C = A(BC)
- Distributive: A(B + C) = AB + AC
- Transpose: (AB)ᵀ = BᵀAᵀ

---

## 2.6 Element-wise (Hadamard) Product

The **Hadamard product** (⊙) multiplies corresponding elements. Both matrices must have the **same dimensions**.

```
A ⊙ B = [aᵢⱼ · bᵢⱼ]
```

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(A * B)   # [[ 5, 12], [21, 32]]   <-- element-wise in NumPy
print(A @ B)   # [[19, 22], [43, 50]]   <-- matrix multiplication
```

> ⚠️ In NumPy, `*` is element-wise and `@` is matrix multiplication. Don't confuse them!

### ML Applications of Hadamard Product

| Application | Usage |
|-------------|-------|
| Attention mechanisms | Element-wise masking of attention scores |
| Gating in LSTMs/GRUs | Gate vector ⊙ candidate vector |
| Feature-wise scaling | Batch normalization: γ ⊙ x̂ + β |
| Dropout | Mask ⊙ activations |

---

## 2.7 Broadcasting (NumPy Concept)

NumPy automatically **broadcasts** smaller arrays to match dimensions:

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])       # shape (2, 3)
b = np.array([10, 20, 30])     # shape (3,)

# b is broadcast across rows
print(A + b)
# [[11, 22, 33],
#  [14, 25, 36]]
```

**Broadcasting rules:**
1. Dimensions are compared from the trailing end
2. Dimensions must be equal OR one of them must be 1
3. Missing dimensions are treated as 1

This is how `z = Wx + b` works in practice — `b` is broadcast across all samples.

---

## 2.8 Outer Product

The **outer product** of u ∈ ℝᵐ and v ∈ ℝⁿ produces an m × n matrix:

```
u ⊗ v = u · vᵀ

[u₁]              [u₁v₁  u₁v₂  u₁v₃]
[u₂] [v₁ v₂ v₃] = [u₂v₁  u₂v₂  u₂v₃]
[u₃]              [u₃v₁  u₃v₂  u₃v₃]
```

```python
u = np.array([1, 2, 3])
v = np.array([4, 5])
print(np.outer(u, v))
# [[ 4,  5],
#  [ 8, 10],
#  [12, 15]]
```

Used in gradient computation and rank-1 updates in optimization.

---

## 2.9 Common Pitfalls

```
❌  A * B       →  element-wise (Hadamard) in NumPy
✅  A @ B       →  matrix multiplication in NumPy
✅  np.dot(A,B) →  matrix multiplication (for 2D arrays)

❌  Assuming AB = BA
✅  Always check dimensions: (m×n)(n×p) = (m×p)

❌  Forgetting that matrix multiplication requires inner dimensions to match
```

---

## 2.10 Worked Example: Forward Pass of a Neural Layer

```
Input:  x = [0.5, 0.3, 0.8]ᵀ     (3 features)
Weights: W = [0.2  0.4  0.1]      (2 neurons × 3 inputs)
             [0.6  0.3  0.5]
Bias:    b = [0.1, 0.2]ᵀ

z = Wx + b

z₁ = 0.2(0.5) + 0.4(0.3) + 0.1(0.8) + 0.1 = 0.10 + 0.12 + 0.08 + 0.1 = 0.40
z₂ = 0.6(0.5) + 0.3(0.3) + 0.5(0.8) + 0.2 = 0.30 + 0.09 + 0.40 + 0.2 = 0.99

z = [0.40, 0.99]ᵀ
```

```python
x = np.array([0.5, 0.3, 0.8])
W = np.array([[0.2, 0.4, 0.1],
              [0.6, 0.3, 0.5]])
b = np.array([0.1, 0.2])
z = W @ x + b
print(z)  # [0.40, 0.99]
```

---

## 2.11 Summary Table

| Operation | Symbol | Dimensions | NumPy |
|-----------|--------|------------|-------|
| Addition | A + B | Same shape | `A + B` |
| Scalar multiply | cA | Same as A | `c * A` |
| Matrix multiply | AB | (m×n)(n×p)→(m×p) | `A @ B` |
| Hadamard product | A ⊙ B | Same shape | `A * B` |
| Outer product | u ⊗ v | (m,) × (n,) → (m×n) | `np.outer(u,v)` |
| Transpose | Aᵀ | (m×n) → (n×m) | `A.T` |

---

## 2.12 Quick Revision Questions

1. **What dimensions must A and B have for AB to be valid? What is the resulting shape?**
2. **Why is matrix multiplication not commutative? Give a 2×2 counter-example.**
3. **In NumPy, what is the difference between `A * B` and `A @ B`?**
4. **How does broadcasting allow `z = Wx + b` to work for a batch of inputs?**
5. **Where does the Hadamard product appear in neural networks?**
6. **What is the outer product, and how is it used in gradient computation?**

---

[← Previous: Chapter 1 — Vectors and Vector Spaces](./01-vectors-and-vector-spaces.md) | [Next Chapter → Chapter 3: Matrix Types](./03-matrix-types.md)
