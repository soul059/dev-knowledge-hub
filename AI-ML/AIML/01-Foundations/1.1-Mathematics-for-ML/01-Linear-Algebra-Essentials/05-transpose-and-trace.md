# Chapter 5: Transpose and Trace

> **Unit 1 — Linear Algebra Essentials**
> Two deceptively simple operations — transpose and trace — appear everywhere in ML derivations. Know their properties cold.

---

## 5.1 Transpose: Definition

The **transpose** of a matrix A ∈ ℝᵐˣⁿ is Aᵀ ∈ ℝⁿˣᵐ, obtained by flipping rows and columns:

```
(Aᵀ)ᵢⱼ = Aⱼᵢ
```

```
A = [1  2  3]       Aᵀ = [1  4]
    [4  5  6]             [2  5]
                          [3  6]

(2×3) → (3×2)
```

```python
import numpy as np
A = np.array([[1,2,3],[4,5,6]])
print(A.T)
# [[1, 4],
#  [2, 5],
#  [3, 6]]
```

---

## 5.2 Transpose Properties

| Property | Formula | Proof Sketch |
|----------|---------|--------------|
| Double transpose | (Aᵀ)ᵀ = A | Flip twice = original |
| Sum | (A + B)ᵀ = Aᵀ + Bᵀ | (aᵢⱼ + bᵢⱼ) flipped |
| Scalar | (cA)ᵀ = cAᵀ | Scalar commutes |
| Product | **(AB)ᵀ = BᵀAᵀ** | See proof below |
| Inverse | (A⁻¹)ᵀ = (Aᵀ)⁻¹ | From (AB)ᵀ rule |
| Dot product | u · v = uᵀv | Converts dot to matrix form |
| Symmetric | A = Aᵀ | Definition of symmetry |

### Proof: (AB)ᵀ = BᵀAᵀ

```
Let C = AB.   Then cᵢⱼ = Σₖ aᵢₖ bₖⱼ

(Cᵀ)ⱼᵢ = cᵢⱼ = Σₖ aᵢₖ bₖⱼ

Now consider (BᵀAᵀ)ⱼᵢ = Σₖ (Bᵀ)ⱼₖ (Aᵀ)ₖᵢ = Σₖ bₖⱼ aᵢₖ = Σₖ aᵢₖ bₖⱼ  ✓

The order reverses because rows become columns and vice versa.
```

### Extension to Multiple Matrices

```
(ABC)ᵀ = CᵀBᵀAᵀ     ("reverse the order")
```

This is critical in backpropagation where gradient expressions involve transposed weight matrices.

---

## 5.3 Transpose in ML

### Writing the Dot Product in Matrix Form

```
u · v = uᵀv = [u₁ u₂ u₃] [v₁]  = u₁v₁ + u₂v₂ + u₃v₃
                            [v₂]
                            [v₃]
```

### Linear Regression Normal Equation

```
ŵ = (XᵀX)⁻¹ Xᵀy
     ↑          ↑
   Gram matrix  projection
```

XᵀX is always symmetric: `(XᵀX)ᵀ = Xᵀ(Xᵀ)ᵀ = XᵀX  ✓`

### Covariance Matrix

```
Σ = (1/n) XᵀX    (assuming X is mean-centered)
```

Always symmetric because XᵀX is symmetric.

---

## 5.4 Trace: Definition

The **trace** of a square matrix A ∈ ℝⁿˣⁿ is the sum of its diagonal entries:

```
tr(A) = a₁₁ + a₂₂ + ... + aₙₙ = Σᵢ aᵢᵢ
```

```
A = [1  2  3]
    [4  5  6]     tr(A) = 1 + 5 + 9 = 15
    [7  8  9]
```

```python
A = np.array([[1,2,3],[4,5,6],[7,8,9]])
print(np.trace(A))  # 15
```

---

## 5.5 Trace Properties

| Property | Formula |
|----------|---------|
| Linearity | tr(A + B) = tr(A) + tr(B) |
| Scalar | tr(cA) = c · tr(A) |
| Transpose | tr(Aᵀ) = tr(A) |
| **Cyclic** | **tr(ABC) = tr(BCA) = tr(CAB)** |
| Eigenvalue sum | tr(A) = λ₁ + λ₂ + ... + λₙ |
| Frobenius norm | ‖A‖²_F = tr(AᵀA) |
| Scalar value | tr(a) = a, for scalar a |

### Proof: Cyclic Property — tr(AB) = tr(BA)

```
tr(AB) = Σᵢ (AB)ᵢᵢ = Σᵢ Σⱼ aᵢⱼ bⱼᵢ

tr(BA) = Σⱼ (BA)ⱼⱼ = Σⱼ Σᵢ bⱼᵢ aᵢⱼ = Σᵢ Σⱼ aᵢⱼ bⱼᵢ  ✓

Both sums equal Σᵢ Σⱼ aᵢⱼ bⱼᵢ, just with summation order swapped.
```

> ⚠️ The cyclic property means you can **cycle** the matrices, NOT arbitrarily reorder them. `tr(ABC) = tr(CAB) ≠ tr(ACB)` in general.

---

## 5.6 The Trace Trick

The **trace trick** converts scalar expressions into trace form and vice versa, enabling easier differentiation.

### Key Identity: xᵀAx = tr(Axxᵀ)

**Proof:**

```
xᵀAx is a scalar (1×1 matrix).
tr(xᵀAx) = xᵀAx          (trace of a scalar = the scalar)

Now apply cyclic property:
tr(xᵀAx) = tr(Axxᵀ) = tr(xxᵀA)
```

### Why Is This Useful?

When differentiating with respect to A:

```
∂/∂A tr(Axxᵀ) = xxᵀ     ← simple!
```

This is much easier than directly differentiating xᵀAx using index notation.

### Common Trace Derivatives

| Expression | Derivative w.r.t. A |
|-----------|-------------------|
| tr(A) | I |
| tr(AB) | Bᵀ |
| tr(AᵀB) | B |
| tr(ABAᵀC) | CAB + CᵀABᵀ |

These are essential for deriving the gradient of loss functions in ML.

---

## 5.7 Connection to Eigenvalues

For any square matrix A with eigenvalues λ₁, λ₂, ..., λₙ:

```
tr(A) = λ₁ + λ₂ + ... + λₙ        (sum of eigenvalues)
det(A) = λ₁ · λ₂ · ... · λₙ        (product of eigenvalues)
```

### Proof (Trace = Sum of Eigenvalues)

```
If A = PΛP⁻¹ (eigendecomposition), then:
tr(A) = tr(PΛP⁻¹) = tr(P⁻¹PΛ)   (cyclic property)
      = tr(Λ) = λ₁ + λ₂ + ... + λₙ  ✓
```

### Example

```python
A = np.array([[4, 2], [1, 3]])
print("Trace:", np.trace(A))                 # 7
eigenvalues = np.linalg.eigvals(A)
print("Sum of eigenvalues:", sum(eigenvalues))  # 7.0
print("Det:", np.linalg.det(A))              # 10
print("Product of eigenvalues:", np.prod(eigenvalues))  # 10.0
```

---

## 5.8 Frobenius Norm via Trace

The **Frobenius norm** (matrix equivalent of L2 norm for vectors):

```
‖A‖_F = √(Σᵢ Σⱼ aᵢⱼ²) = √(tr(AᵀA))
```

```python
A = np.array([[1, 2], [3, 4]])
frob1 = np.linalg.norm(A, 'fro')
frob2 = np.sqrt(np.trace(A.T @ A))
print(f"Frobenius: {frob1:.4f} = {frob2:.4f}")  # 5.4772
```

**ML Application:** Weight decay regularization penalizes ‖W‖²_F = tr(WᵀW), keeping weights small.

---

## 5.9 Worked Example: Deriving the OLS Gradient

**Objective:** Minimize L(w) = ‖y - Xw‖² = (y - Xw)ᵀ(y - Xw)

```
L = yᵀy - 2wᵀXᵀy + wᵀXᵀXw

Using trace trick: wᵀXᵀXw = tr(XᵀXwwᵀ), but let's use direct method:

∂L/∂w = -2Xᵀy + 2XᵀXw

Set to zero:
2XᵀXw = 2Xᵀy
w* = (XᵀX)⁻¹Xᵀy     ← Normal equation!
```

Here the transpose appears naturally in forming XᵀX and Xᵀy.

---

## 5.10 Summary Table

| Concept | Key Formula | Remember |
|---------|------------|----------|
| Transpose | (Aᵀ)ᵢⱼ = Aⱼᵢ | Flip rows ↔ columns |
| Product transpose | (AB)ᵀ = BᵀAᵀ | Reverse order |
| Trace definition | tr(A) = Σ aᵢᵢ | Sum of diagonal |
| Cyclic property | tr(ABC) = tr(CAB) | Cycle, don't reorder |
| Trace = eigenvalue sum | tr(A) = Σ λᵢ | Invariant under similarity |
| Trace trick | xᵀAx = tr(Axxᵀ) | Enables matrix calculus |
| Frobenius norm | ‖A‖²_F = tr(AᵀA) | Matrix "size" measure |

---

## 5.11 Quick Revision Questions

1. **Why does (AB)ᵀ = BᵀAᵀ and not AᵀBᵀ? Explain intuitively.**
2. **What is the cyclic property of trace? Give an example where changing order (not cycling) gives a different result.**
3. **Prove that tr(A) = tr(Aᵀ).**
4. **How does the trace trick simplify differentiation in matrix calculus?**
5. **If A has eigenvalues 2, 3, 5, what is tr(A) and det(A)?**
6. **Why is ‖W‖²_F = tr(WᵀW) a natural choice for regularization?**

---

[← Previous: Chapter 4 — Matrix Multiplication Interpretations](./04-matrix-multiplication-interpretations.md) | [Next Chapter → Chapter 6: Determinants](./06-determinants.md)
