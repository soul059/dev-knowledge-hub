# Chapter 8: Linear Transformations

> **Unit 1 — Linear Algebra Essentials**
> Every matrix multiplication is a linear transformation. Understanding this connects algebra to geometry — and to every layer in a neural network.

---

## 8.1 What Is a Linear Transformation?

A function T: ℝⁿ → ℝᵐ is a **linear transformation** if for all vectors u, v and scalar c:

```
1. T(u + v) = T(u) + T(v)          (additivity / superposition)
2. T(cu)    = c · T(u)              (homogeneity)
```

Equivalently: `T(c₁u + c₂v) = c₁T(u) + c₂T(v)`

**Key insight:** Every linear transformation can be represented by a matrix, and every matrix represents a linear transformation.

```
T(x) = Ax     where A ∈ ℝᵐˣⁿ
```

The columns of A are the images of the standard basis vectors:

```
A = [T(e₁) | T(e₂) | ... | T(eₙ)]
```

---

## 8.2 Common 2D Transformations

### Scaling

```
S = [sₓ   0]     Stretches by sₓ along x, s_y along y
    [ 0  s_y]

Before          After (sₓ=2, s_y=1.5)
┌───┐           ┌──────┐
│   │    ──>    │      │
│   │           │      │
└───┘           │      │
                └──────┘
```

```python
import numpy as np
S = np.array([[2, 0], [0, 1.5]])
v = np.array([1, 1])
print(S @ v)  # [2.0, 1.5]
```

### Rotation (counterclockwise by θ)

```
R(θ) = [cos θ   -sin θ]
       [sin θ    cos θ]

Before          After (θ = 90°)
    ↑ y             ↑ y
    │  •(1,1)       │
    │ /         •(-1,1)
    │/              │/
    +──→ x          +──→ x

det(R) = 1 (preserves area)
```

```python
theta = np.pi / 2  # 90 degrees
R = np.array([[np.cos(theta), -np.sin(theta)],
              [np.sin(theta),  np.cos(theta)]])
print(R @ np.array([1, 1]))  # [-1, 1]
```

### Reflection

```
About x-axis:           About y-axis:          About y = x:
F_x = [1   0]          F_y = [-1  0]          F = [0  1]
      [0  -1]                [ 0  1]              [1  0]

      ↑ y                    ↑ y
    • │                      │ •
      │         →            │              det = -1 (reverses orientation)
   ───┼───              ─────┼────
      │                      │
    • │ (reflected)          │
```

### Shearing

```
Horizontal shear:        Vertical shear:
H = [1  k]              V = [1  0]
    [0  1]                   [k  1]

Before        After (k=1)
┌───┐         ╱───╱
│   │   →    ╱   ╱         Tilts the shape
│   │       ╱   ╱
└───┘      ╱───╱
```

```python
# Horizontal shear with k=0.5
H = np.array([[1, 0.5], [0, 1]])
square = np.array([[0,0],[1,0],[1,1],[0,1]]).T  # columns are vertices
print(H @ square)  # sheared vertices
```

### Projection (onto x-axis)

```
P = [1  0]      "Flattens" everything onto the x-axis
    [0  0]

(3, 4) → (3, 0)

det(P) = 0  → information is lost (not invertible)
```

---

## 8.3 Transformation Matrix Gallery

| Transformation | Matrix | det | Invertible? |
|---------------|--------|-----|-------------|
| Identity | I | 1 | ✓ |
| Scale (sₓ, s_y) | diag(sₓ, s_y) | sₓ·s_y | ✓ (if sₓ,s_y ≠ 0) |
| Rotate by θ | [cosθ, −sinθ; sinθ, cosθ] | 1 | ✓ |
| Reflect (x-axis) | diag(1, −1) | −1 | ✓ |
| Shear (horiz, k) | [1, k; 0, 1] | 1 | ✓ |
| Project (x-axis) | diag(1, 0) | 0 | ✗ |

---

## 8.4 Composition of Transformations

Applying transformation A followed by B is equivalent to the single transformation **BA** (not AB!):

```
T₂(T₁(x)) = B(Ax) = (BA)x
```

### Example: Rotate 45° then Scale 2×

```python
theta = np.pi / 4
R = np.array([[np.cos(theta), -np.sin(theta)],
              [np.sin(theta),  np.cos(theta)]])
S = np.array([[2, 0], [0, 2]])

# Scale AFTER rotate → S @ R
composed = S @ R
v = np.array([1, 0])
print(composed @ v)  # [1.414, 1.414]
```

**Order matters!** In general SR ≠ RS.

```
Geometric analogy:
- Rotate then scale: turns then stretches
- Scale then rotate: stretches then turns (different result!)
```

---

## 8.5 Linear Transformations in Higher Dimensions

### 3D Rotation Matrices

```
Rₓ(θ) = [1    0      0  ]    (rotate around x-axis)
         [0  cosθ  -sinθ ]
         [0  sinθ   cosθ ]

R_y(θ) = [ cosθ  0  sinθ]    (rotate around y-axis)
         [   0   1    0  ]
         [-sinθ  0  cosθ ]

R_z(θ) = [cosθ  -sinθ  0]    (rotate around z-axis)
         [sinθ   cosθ  0]
         [  0      0   1]
```

Used extensively in **computer vision** and **3D point cloud processing**.

---

## 8.6 Connection to Neural Network Layers

A fully-connected (dense) layer computes:

```
z = Wx + b        (affine transformation)
a = σ(z)          (non-linear activation)
```

**Without the activation**, stacking layers collapses:

```
Layer 2 ∘ Layer 1 = W₂(W₁x + b₁) + b₂ = (W₂W₁)x + (W₂b₁ + b₂)
```

This is still a single linear transformation! **Non-linearity is essential** to learn complex functions.

```
Input ──→ [W₁x + b₁] ──→ σ() ──→ [W₂a₁ + b₂] ──→ σ() ──→ Output
          Linear          Non-    Linear           Non-
          Transform       linear  Transform        linear
```

### What Each Layer Does Geometrically

```
┌─────────────┐     ┌──────────────┐     ┌────────────┐
│  Input Space │ W₁  │ Hidden Space  │ W₂  │Output Space│
│              │────→│              │────→│            │
│  ○ ○         │     │    ○         │     │   ○        │
│ ○   ○        │     │  ○   ○       │     │  ○         │
│  ○ ○         │     │    ○         │     │            │
│(entangled)   │     │(rotated,     │     │(linearly   │
│              │     │ scaled)      │     │ separable) │
└─────────────┘     └──────────────┘     └────────────┘

Each weight matrix rotates, scales, and projects the data.
Activations then bend/warp the space non-linearly.
```

---

## 8.7 Kernel and Image

For T(x) = Ax:

### Kernel (Null Space)

```
ker(A) = {x ∈ ℝⁿ : Ax = 0}
```

The set of all inputs that map to zero. Dimension = n − rank(A).

### Image (Column Space / Range)

```
im(A) = {Ax : x ∈ ℝⁿ} = column space of A
```

The set of all possible outputs. Dimension = rank(A).

### Rank-Nullity Theorem

```
dim(ker(A)) + dim(im(A)) = n

nullity + rank = number of columns
```

```python
A = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(f"Rank: {np.linalg.matrix_rank(A)}")  # 2
# Nullity = 3 - 2 = 1 (there's a 1D null space)
```

---

## 8.8 Change of Basis

The same transformation has different matrix representations in different bases.

```
If B is the change-of-basis matrix (columns = new basis vectors):

A' = B⁻¹AB

A  = representation in standard basis
A' = representation in new basis B
```

**ML Application:** PCA finds a new basis (principal components) where the covariance matrix becomes diagonal — this is exactly a change of basis.

---

## 8.9 Worked Example: Composite Transformation

**Problem:** Apply a 30° rotation followed by reflection about the x-axis to the point (1, √3).

```python
import numpy as np

theta = np.pi / 6  # 30 degrees
R = np.array([[np.cos(theta), -np.sin(theta)],
              [np.sin(theta),  np.cos(theta)]])
F = np.array([[1, 0], [0, -1]])  # reflect about x-axis

# Composite: reflect AFTER rotate → F @ R
T = F @ R
v = np.array([1, np.sqrt(3)])

result = T @ v
print(f"Result: ({result[0]:.4f}, {result[1]:.4f})")
```

```
Step 1: Rotate (1, √3) by 30°
R · [1, √3]ᵀ = [cos30° - √3·sin30°, sin30° + √3·cos30°]ᵀ
             = [√3/2 - √3/2, 1/2 + 3/2]ᵀ
             = [0, 2]ᵀ

Step 2: Reflect (0, 2) about x-axis
F · [0, 2]ᵀ = [0, -2]ᵀ

Final answer: (0, -2)
```

---

## 8.10 Summary Table

| Transformation | Matrix (2D) | Effect | Invertible? |
|---------------|-------------|--------|-------------|
| Scale | diag(sₓ, s_y) | Stretch/compress | ✓ (if nonzero) |
| Rotate θ | [cosθ, −sinθ; sinθ, cosθ] | Rotate CCW | ✓ always |
| Reflect x-axis | diag(1, −1) | Mirror | ✓ |
| Shear k | [1, k; 0, 1] | Tilt | ✓ |
| Project | diag(1, 0) | Flatten | ✗ |
| NN layer | Wx + b | Affine map | Depends |
| Composition | BA (B after A) | Sequential | ✓ if both are |

---

## 8.11 Quick Revision Questions

1. **Why does stacking two linear layers without activation collapse to a single linear transformation?**
2. **If you apply rotation R then scaling S, what is the composite matrix?**
3. **What is the geometric meaning of det(A) = 0 for a transformation matrix A?**
4. **How does the rank-nullity theorem relate to information loss in a transformation?**
5. **Why are the columns of a transformation matrix the images of the standard basis vectors?**
6. **In PCA, what does "change of basis" mean geometrically?**

---

[← Previous: Chapter 7 — Matrix Inverse](./07-matrix-inverse.md) | [Next Chapter → Chapter 9: Eigenvalues and Eigenvectors](./09-eigenvalues-and-eigenvectors.md)
