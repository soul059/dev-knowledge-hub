# Chapter 1: Vectors and Vector Spaces

> **Unit 1 — Linear Algebra Essentials**
> Master the building blocks of every ML algorithm: vectors, their operations, and the spaces they inhabit.

---

## 1.1 What Is a Vector?

A **vector** is an ordered list of numbers that represents both magnitude and direction.

```
Geometric view          Algebraic view

    ↑ y                 v = [v₁]   (column vector)
    |   /v                  [v₂]
    |  /                    [v₃]
    | /
    +———→ x             v ∈ ℝⁿ  (n-dimensional real vector)
```

In ML, a vector typically represents a **data point** — each element is a feature value.

```python
import numpy as np
# A data point: [age, income, credit_score]
x = np.array([25, 50000, 720])
```

---

## 1.2 Vector Operations

### Addition & Subtraction

```
u + v = [u₁ + v₁,  u₂ + v₂,  ...,  uₙ + vₙ]
```

```python
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])
print(u + v)  # [5, 7, 9]
print(u - v)  # [-3, -3, -3]
```

### Scalar Multiplication

```
c · v = [c·v₁,  c·v₂,  ...,  c·vₙ]
```

Scaling a vector stretches or shrinks it without changing direction (unless c < 0).

### Dot Product (Inner Product)

```
u · v = u₁v₁ + u₂v₂ + ... + uₙvₙ = Σ uᵢvᵢ = |u| |v| cos θ
```

**Properties:**
- Commutative: `u · v = v · u`
- Distributive: `u · (v + w) = u · v + u · w`
- If `u · v = 0`, vectors are **orthogonal** (perpendicular)

```python
dot = np.dot(u, v)        # 1*4 + 2*5 + 3*6 = 32
dot_alt = u @ v            # same result
```

**ML Application:** The dot product is the core of linear regression — `ŷ = w · x + b`.

### Cross Product (3D only)

```
u × v = | i   j   k  |
        | u₁  u₂  u₃ |  = (u₂v₃ - u₃v₂)i - (u₁v₃ - u₃v₁)j + (u₁v₂ - u₂v₁)k
        | v₁  v₂  v₃ |
```

The result is a vector **perpendicular** to both `u` and `v`. Used mainly in computer graphics, not common in standard ML.

---

## 1.3 Vector Norms

A **norm** measures the "size" or "length" of a vector.

| Norm | Formula | Also Called |
|------|---------|-------------|
| L1 | `‖v‖₁ = Σ|vᵢ|` | Manhattan norm |
| L2 | `‖v‖₂ = √(Σ vᵢ²)` | Euclidean norm |
| L∞ | `‖v‖∞ = max(|vᵢ|)` | Max / Chebyshev norm |
| Lp | `‖v‖ₚ = (Σ|vᵢ|ᵖ)^(1/p)` | General p-norm |

```python
v = np.array([3, -4])
print(np.linalg.norm(v, 1))    # L1 = 7
print(np.linalg.norm(v, 2))    # L2 = 5.0
print(np.linalg.norm(v, np.inf))  # Linf = 4
```

### Unit Vectors

A **unit vector** has norm = 1. To normalize any vector:

```
û = v / ‖v‖
```

```python
v_hat = v / np.linalg.norm(v)  # [0.6, -0.8]
```

### ML Relevance of Norms

- **L1 norm** → Lasso regularization (promotes sparsity)
- **L2 norm** → Ridge regularization (prevents large weights)
- **L2 distance** → k-NN, k-Means clustering

---

## 1.4 Vector Spaces

A **vector space** V over ℝ is a set of vectors closed under addition and scalar multiplication, satisfying 8 axioms (associativity, commutativity, identity, inverse, distributivity, etc.).

**Common examples:** ℝ², ℝ³, ℝⁿ, space of all polynomials of degree ≤ n.

### Subspaces

A **subspace** W ⊆ V must satisfy:
1. Contains the zero vector: **0** ∈ W
2. Closed under addition: if **u**, **v** ∈ W, then **u + v** ∈ W
3. Closed under scalar multiplication: if **v** ∈ W and c ∈ ℝ, then c**v** ∈ W

```
Example: In ℝ³, any plane through the origin is a subspace.
A plane NOT through the origin is NOT a subspace (fails condition 1).
```

---

## 1.5 Linear Independence, Span, and Basis

### Linear Independence

Vectors {v₁, v₂, ..., vₖ} are **linearly independent** if the only solution to:

```
c₁v₁ + c₂v₂ + ... + cₖvₖ = 0
```

is c₁ = c₂ = ... = cₖ = 0. Otherwise, they are **linearly dependent** (one can be written as a combination of others).

### Span

The **span** of a set of vectors is the set of all possible linear combinations:

```
span{v₁, v₂, ..., vₖ} = {c₁v₁ + c₂v₂ + ... + cₖvₖ | cᵢ ∈ ℝ}
```

### Basis

A **basis** of a vector space V is a set of vectors that is:
1. **Linearly independent**
2. **Spans** the entire space V

The number of vectors in any basis = **dimension** of the space.

```
Standard basis for ℝ³:
e₁ = [1, 0, 0]    e₂ = [0, 1, 0]    e₃ = [0, 0, 1]
dim(ℝ³) = 3
```

```python
# Check linear independence via rank
A = np.array([[1, 0, 2],
              [0, 1, 1],
              [1, 1, 3]])
print(np.linalg.matrix_rank(A))  # 2 → columns are dependent
```

---

## 1.6 Vectors as Data in ML

```
Feature space (ℝ³ example):

    height ↑
           |    • patient₁ = [170, 65, 120/80]
           |  •   patient₂ = [160, 55, 110/70]
           | •
           +————————→ weight
          /
         ↙ blood_pressure

Each patient = a vector in feature space.
ML models learn decision boundaries in this space.
```

| ML Concept | Vector Role |
|------------|------------|
| Feature vector | Data point `x = [x₁, x₂, ..., xₙ]` |
| Weight vector | Model parameters `w` in linear models |
| Word embedding | Dense vector representing a word (Word2Vec, GloVe) |
| Gradient | Direction of steepest ascent in loss landscape |
| Image pixel row | Flattened image as a vector in ℝᵈ |

---

## 1.7 Worked Example

**Problem:** Given u = [1, 2, 3] and v = [4, -1, 2], find:
(a) u + v, (b) 3u, (c) u · v, (d) ‖u‖₂, (e) angle between u and v

**Solution:**

```
(a) u + v = [1+4, 2+(-1), 3+2] = [5, 1, 5]

(b) 3u = [3, 6, 9]

(c) u · v = 1(4) + 2(-1) + 3(2) = 4 - 2 + 6 = 8

(d) ‖u‖₂ = √(1² + 2² + 3²) = √14 ≈ 3.742

(e) cos θ = (u · v) / (‖u‖ · ‖v‖)
    ‖v‖ = √(16 + 1 + 4) = √21 ≈ 4.583
    cos θ = 8 / (3.742 × 4.583) ≈ 0.4665
    θ = arccos(0.4665) ≈ 62.2°
```

```python
u, v = np.array([1,2,3]), np.array([4,-1,2])
angle = np.arccos(np.dot(u,v) / (np.linalg.norm(u)*np.linalg.norm(v)))
print(f"Angle: {np.degrees(angle):.1f}°")  # 62.2°
```

---

## 1.8 Summary Table

| Concept | Key Idea | Formula / Rule |
|---------|----------|----------------|
| Vector | Ordered list of numbers | v ∈ ℝⁿ |
| Dot product | Measures alignment | u · v = Σ uᵢvᵢ |
| Cross product | Perpendicular vector (3D) | u × v |
| L1 norm | Sum of absolutes | Σ\|vᵢ\| |
| L2 norm | Euclidean length | √(Σ vᵢ²) |
| Unit vector | Norm = 1 | û = v/‖v‖ |
| Linear independence | No redundant vectors | Only trivial solution to Σ cᵢvᵢ = 0 |
| Basis | Independent + spans space | Number of basis vectors = dimension |
| Subspace | Subset closed under +, · | Must contain **0** |

---

## 1.9 Quick Revision Questions

1. **What does a dot product of zero between two vectors signify geometrically?**
2. **If you have 4 vectors in ℝ³, can they be linearly independent? Why or why not?**
3. **Why does L1 regularization promote sparsity while L2 does not?**
4. **What is the dimension of the span of {[1,0,0], [0,1,0], [1,1,0]}?**
5. **How is the gradient vector related to the direction of steepest descent in ML?**
6. **Give an example of a subset of ℝ² that is NOT a subspace.**

---

[Next Chapter → Chapter 2: Matrix Operations](./02-matrix-operations.md)
