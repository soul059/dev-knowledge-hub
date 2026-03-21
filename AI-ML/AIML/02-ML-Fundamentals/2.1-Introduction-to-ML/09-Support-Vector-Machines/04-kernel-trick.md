# 🔮 The Kernel Trick

> **Unit 9, Chapter 4 — Mapping to Higher Dimensions, Kernel Functions & Mercer's Theorem**

---

## 📋 Table of Contents

1. [Chapter Overview](#1-chapter-overview)
2. [The Need for Higher Dimensions](#2-the-need-for-higher-dimensions)
3. [Feature Mapping φ](#3-feature-mapping-φ)
4. [The Kernel Function](#4-the-kernel-function)
5. [Why the Kernel Trick is Efficient](#5-why-the-kernel-trick-is-efficient)
6. [Kernel Matrix (Gram Matrix)](#6-kernel-matrix-gram-matrix)
7. [Mercer's Theorem — Valid Kernels](#7-mercers-theorem--valid-kernels)
8. [Intuition: 2D → 3D Example](#8-intuition-2d--3d-example)
9. [ASCII Diagram of Kernel Mapping](#9-ascii-diagram-of-kernel-mapping)
10. [Python Implementation](#10-python-implementation)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Chapter Overview

The kernel trick is arguably the most brilliant idea in SVM theory. It allows SVMs to learn **nonlinear decision boundaries** by implicitly mapping data to higher-dimensional spaces — without ever computing the mapping explicitly.

```
The Big Picture:

Data not linearly           Map to higher          Now linearly
separable in                dimensional            separable!
original space              space via φ

  ○ ○ ● ●                    ○      ○              │ ○    ○
  ○ ● ● ○    ──→ φ ──→      ○  ●  ●    ○    ──→   │  ○  ●  ●    ○
  ● ● ○ ○                  ●  ●                   │ ●  ●
  ○ ● ○ ●                ↗    ↗                   │      (hyperplane
                       z-axis added                │       in new space)
 (can't separate       (φ adds new features)
  with a line!)

BUT: φ can be very high dimensional (even ∞)!
SOLUTION: Kernel trick — compute dot products in high-D space
          without ever going there!
```

---

## 2. The Need for Higher Dimensions

### Linearly Non-Separable Data

Many real-world datasets cannot be separated by a linear boundary:

```
Example 1: XOR Problem            Example 2: Circular Data

  + | -                               - - - - -
  ──┼──                             -   + + +   -
  - | +                           -   + + + + +   -
                                  -   + + + + +   -
  No line can separate!           -   + + + + +   -
                                    -   + + +   -
                                      - - - - -
                                  
                                  No line can separate!
                                  (Need a circle)
```

### Cover's Theorem (1965)

> A complex pattern classification problem, cast in a high-dimensional space nonlinearly, is more likely to be linearly separable than in a low-dimensional space.

**Intuition**: As we add more dimensions, there's "more room" for a hyperplane to fit between classes.

```
1D: Two intervals → cannot separate [+, -, +]
    + + - - - + +
    ──●───●───●──→ x₁
    Can't separate with a single point (0D hyperplane)

2D: Same data with φ(x) = (x, x²) → NOW separable!
    x₂ = x₁²  ↑
               │  +           +
               │    ─────────
               │      -   -
               │ ────────────
               └──────────────→ x₁
    A line can now separate!
```

---

## 3. Feature Mapping φ

### Definition

A **feature mapping** (or feature map) is a function φ that transforms input vectors from the original space to a higher-dimensional **feature space**:

```
φ: ℝⁿ → ℝᵐ    where m > n (typically m >> n or even m = ∞)

x = (x₁, x₂) ──φ──→ φ(x) = (x₁², √2·x₁x₂, x₂², √2·x₁, √2·x₂, 1)
   ↑ 2D input          ↑ 6D feature space
```

### Example: Polynomial Feature Map (Degree 2)

For input x = (x₁, x₂):

```
φ(x) = (x₁², √2·x₁x₂, x₂², √2·x₁, √2·x₂, 1)

This maps from ℝ² → ℝ⁶
```

### The Problem with Explicit Mapping

| Original Dimension n | Degree d | Feature Space Dimension | Computation |
|---------------------|----------|------------------------|-------------|
| 2 | 2 | 6 | Manageable |
| 10 | 2 | 66 | OK |
| 100 | 2 | 5,151 | Getting big |
| 1,000 | 2 | 501,501 | Expensive |
| 100 | 5 | 96,560,646 | Impractical |
| n | d | C(n+d, d) | Exponential growth |
| Any | RBF | ∞ | IMPOSSIBLE to compute explicitly! |

**The kernel trick solves this**: We never compute φ(x) explicitly!

---

## 4. The Kernel Function

### Definition

A **kernel function** K(x, y) computes the dot product of two vectors in the feature space **without explicitly computing the mapping**:

```
K(x, y) = φ(x) · φ(y)
          ──────────────
          Dot product in feature space
          computed directly from x and y!
```

### The Magic

```
EXPLICIT APPROACH (Expensive):              KERNEL TRICK (Efficient):

Step 1: Compute φ(x) in ℝᵐ                 Step 1: Compute K(x,y)
Step 2: Compute φ(y) in ℝᵐ                          directly from x, y
Step 3: Compute φ(x) · φ(y) in ℝᵐ                   in ℝⁿ

Cost: O(m) where m can be huge or ∞         Cost: O(n) — always in original space!
Memory: Store m-dimensional vectors         Memory: Store n-dimensional vectors
```

### Example: Polynomial Kernel Verification

Let x = (x₁, x₂) and y = (y₁, y₂).

**Kernel**: K(x, y) = (x · y + 1)² = (x₁y₁ + x₂y₂ + 1)²

**Expanding**:
```
K(x, y) = (x₁y₁ + x₂y₂ + 1)²
        = x₁²y₁² + x₂²y₂² + 1 + 2x₁x₂y₁y₂ + 2x₁y₁ + 2x₂y₂
        = (x₁²)(y₁²) + (√2·x₁x₂)(√2·y₁y₂) + (x₂²)(y₂²) 
          + (√2·x₁)(√2·y₁) + (√2·x₂)(√2·y₂) + (1)(1)
```

This is exactly φ(x) · φ(y) where:
```
φ(x) = (x₁², √2·x₁x₂, x₂², √2·x₁, √2·x₂, 1)
φ(y) = (y₁², √2·y₁y₂, y₂², √2·y₁, √2·y₂, 1)
```

**Verification**: K(x,y) = φ(x) · φ(y) ✓

**Cost comparison**:
```
Explicit: Map to 6D, compute 6-element dot product  → 6 multiplications + 5 additions
Kernel:   Compute x·y (2D), add 1, square           → 2 multiplications + 1 addition + 1 square

Savings grow dramatically with dimension!
```

---

## 5. Why the Kernel Trick is Efficient

### Recall: SVM Dual Only Uses Dot Products

The SVM dual problem is:

```
maximize  W(α) = Σαᵢ - ½ Σᵢ Σⱼ αᵢαⱼyᵢyⱼ (xᵢ · xⱼ)
                                           ──────────
                                           Only dot products!
```

And the prediction function is:

```
f(x) = Σᵢ αᵢyᵢ (xᵢ · x) + b
                 ─────────
                 Only dot products!
```

### The Kernel Substitution

Replace every dot product with a kernel:

```
DUAL:       maximize  W(α) = Σαᵢ - ½ Σᵢ Σⱼ αᵢαⱼyᵢyⱼ K(xᵢ, xⱼ)
                                                      ───────────
                                                      Kernel!

PREDICTION: f(x) = Σᵢ αᵢyᵢ K(xᵢ, x) + b
                             ──────────
                             Kernel!
```

This is equivalent to:
1. Mapping all data to high-dimensional space via φ
2. Running linear SVM in that space
3. But **without ever computing φ**!

### Computational Comparison

```
Approach 1: Explicit Mapping
├── Map N points to ℝᵐ:  O(N · m)  ← Can be impossible if m = ∞
├── Compute N² dot products in ℝᵐ:  O(N² · m)
└── Total: O(N² · m)

Approach 2: Kernel Trick
├── Compute N² kernel values:  O(N² · n)  ← Always in original space
└── Total: O(N² · n)

Speedup: m/n  (can be infinite!)
```

### The Beauty

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  The kernel trick lets us:                               │
│                                                          │
│  ✅ Work in infinite-dimensional spaces (RBF kernel)     │
│  ✅ Never explicitly compute the mapping φ               │
│  ✅ Keep computation in original space O(n)              │
│  ✅ Find nonlinear decision boundaries                   │
│  ✅ Maintain all SVM theoretical guarantees              │
│                                                          │
│  All by simply replacing x·y with K(x,y) everywhere!    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 6. Kernel Matrix (Gram Matrix)

### Definition

The **kernel matrix** (or Gram matrix) K is an N × N matrix where:

```
Kᵢⱼ = K(xᵢ, xⱼ) = φ(xᵢ) · φ(xⱼ)
```

### Properties

```
┌────────────────────────────────────────────────────┐
│  K is:                                             │
│  ├── Symmetric:  Kᵢⱼ = Kⱼᵢ                       │
│  │   (because K(x,y) = φ(x)·φ(y) = φ(y)·φ(x))   │
│  │                                                 │
│  ├── Positive semi-definite (PSD):                 │
│  │   zᵀKz ≥ 0 for all vectors z                   │
│  │   (because zᵀKz = ‖Σᵢ zᵢφ(xᵢ)‖² ≥ 0)        │
│  │                                                 │
│  └── Size N × N (independent of feature dim!)      │
│       Even if φ maps to ℝ∞, K is still N × N      │
└────────────────────────────────────────────────────┘
```

### Example

For 3 data points with linear kernel:

```
          ┌ x₁·x₁  x₁·x₂  x₁·x₃ ┐
K =       │ x₂·x₁  x₂·x₂  x₂·x₃ │    (3 × 3 matrix)
          └ x₃·x₁  x₃·x₂  x₃·x₃ ┘

With x₁=(1,0), x₂=(0,1), x₃=(1,1):

          ┌ 1  0  1 ┐
K =       │ 0  1  1 │
          └ 1  1  2 ┘

Eigenvalues: 3.414, 0.586, 0  (all ≥ 0 → PSD ✓)
```

### Computing the Kernel Matrix in Practice

```python
import numpy as np

def compute_kernel_matrix(X, kernel='linear', **kwargs):
    """Compute the kernel (Gram) matrix."""
    N = X.shape[0]
    K = np.zeros((N, N))
    
    for i in range(N):
        for j in range(i, N):
            if kernel == 'linear':
                K[i, j] = np.dot(X[i], X[j])
            elif kernel == 'polynomial':
                d = kwargs.get('degree', 3)
                gamma = kwargs.get('gamma', 1.0)
                r = kwargs.get('coef0', 1.0)
                K[i, j] = (gamma * np.dot(X[i], X[j]) + r) ** d
            elif kernel == 'rbf':
                gamma = kwargs.get('gamma', 1.0)
                K[i, j] = np.exp(-gamma * np.linalg.norm(X[i] - X[j])**2)
            K[j, i] = K[i, j]  # Symmetric
    
    return K
```

---

## 7. Mercer's Theorem — Valid Kernels

### The Question

Not every function K(x, y) is a valid kernel. A valid kernel must correspond to some feature mapping φ such that K(x, y) = φ(x) · φ(y).

### Mercer's Theorem (1909)

> A function K(x, y) is a valid kernel if and only if the kernel matrix K formed from any finite set of points {x₁, ..., xₙ} is **positive semi-definite (PSD)**.

```
K is a valid kernel  ⟺  For ANY set of points {x₁,...,xₙ}:
                         the matrix Kᵢⱼ = K(xᵢ,xⱼ) is PSD
                         
                    ⟺  All eigenvalues of K are ≥ 0
                    
                    ⟺  zᵀKz ≥ 0 for all z ∈ ℝⁿ
```

### Why Mercer's Theorem Matters

```
Can I use this function as a kernel?

K(x,y) = ???
    │
    ▼
Check: Is K(x,y) PSD for ALL finite point sets?
    │
    ├── YES → Valid kernel! ∃ feature map φ such that K(x,y) = φ(x)·φ(y)
    │         (φ may be implicit/unknown but guaranteed to exist)
    │
    └── NO  → NOT a valid kernel. Using it may give:
              ├── Non-convex optimization (no global optimum guarantee)
              ├── Negative eigenvalues in K → imaginary distances
              └── Meaningless "distances" in feature space
```

### Rules for Building Valid Kernels

If K₁ and K₂ are valid kernels, then:

```
┌──────────────────────────────────────────────────────────┐
│  KERNEL CLOSURE PROPERTIES                               │
│                                                          │
│  1. K(x,y) = c · K₁(x,y)           (c > 0)            │
│  2. K(x,y) = K₁(x,y) + K₂(x,y)                        │
│  3. K(x,y) = K₁(x,y) · K₂(x,y)                        │
│  4. K(x,y) = f(x) · f(y)           (any function f)    │
│  5. K(x,y) = K₃(φ(x), φ(y))       (K₃ valid kernel)   │
│  6. K(x,y) = exp(K₁(x,y))                              │
│  7. K(x,y) = xᵀAy                  (A is PSD matrix)   │
│  8. K(x,y) = p(K₁(x,y))           (p polynomial with  │
│                                      non-negative coeff) │
│                                                          │
│  All of these produce VALID kernels!                     │
└──────────────────────────────────────────────────────────┘
```

### Example: Proving RBF is a Valid Kernel

```
K_RBF(x,y) = exp(-γ‖x-y‖²)

Step 1: ‖x-y‖² = ‖x‖² - 2x·y + ‖y‖²

Step 2: K_RBF = exp(-γ‖x‖²) · exp(2γ x·y) · exp(-γ‖y‖²)
              = f(x) · exp(2γ x·y) · f(y)
              
Step 3: exp(2γ x·y) = Σ (2γ)ᵏ(x·y)ᵏ / k!
                       k=0
        Each term (x·y)ᵏ is a valid kernel (polynomial)
        Non-negative coefficients → sum is valid kernel
        
Step 4: f(x)·f(y) is a valid kernel (Rule 4)

Step 5: Product of valid kernels → valid kernel (Rule 3)

→ RBF is a valid kernel! ✓
  (It corresponds to an ∞-dimensional feature map)
```

---

## 8. Intuition: 2D → 3D Example

### The XOR / Circular Problem

Consider data that forms two concentric circles:

```
Original 2D space (NOT linearly separable):

    y ↑
      │       - - -
      │     -       -
      │   -   + + +   -
      │  -   +     +   -
      │ -   +       +   -
      │  -   +     +   -
      │   -   + + +   -
      │     -       -
      │       - - -
      └──────────────────→ x
      
Inner circle: class +1
Outer circle: class -1
No line can separate!
```

### The Mapping

Define: φ(x₁, x₂) = (x₁, x₂, x₁² + x₂²)

This adds a third dimension z = x₁² + x₂² (distance from origin squared):

```
After mapping to 3D:

    z = x₁² + x₂² ↑
                   │    - - - - - - -        (outer ring → high z)
                   │
                   │   + + + + + + +         (inner ring → low z)
                   │
                   └──────────────────→ x₁ (or x₂)

A horizontal PLANE can now separate them!
z = threshold separates inner (+) from outer (-)
```

### Step-by-Step Numerical Example

Original 2D points:

| Point | x₁ | x₂ | Class | x₁²+x₂² (= z) |
|-------|----|----|-------|-----------------|
| A | 0 | 0 | +1 | 0 |
| B | 1 | 0 | +1 | 1 |
| C | 0 | 1 | +1 | 1 |
| D | 2 | 0 | -1 | 4 |
| E | 0 | 2 | -1 | 4 |
| F | 2 | 2 | -1 | 8 |

```
After φ mapping to 3D:

    z ↑
   8  │                               F(2,2,8)  ← Class -1
      │
   4  │         D(2,0,4)   E(0,2,4)             ← Class -1
      │
   2  │═════════════════════════════  ← Separating plane z = 2
      │
   1  │    B(1,0,1)   C(0,1,1)                  ← Class +1
      │
   0  │  A(0,0,0)                                ← Class +1
      └──────────────────────────→ x₁

LINEARLY SEPARABLE! The plane z = 2 separates the classes.
In original space, this corresponds to: x₁² + x₂² = 2 (a CIRCLE!)
```

### The Kernel Version

Instead of computing φ explicitly, use a polynomial kernel:

```
K(x, y) = (x · y + 1)²

This implicitly maps to a space where x₁² + x₂² is a feature,
achieving the same separation without computing φ!
```

---

## 9. ASCII Diagram of Kernel Mapping

### The Complete Picture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         THE KERNEL TRICK                                │
│                                                                         │
│   ORIGINAL SPACE (ℝ²)              FEATURE SPACE (ℝᵐ, m possibly ∞)   │
│                                                                         │
│       ● ● ○ ○                           ●                              │
│     ● ● ○ ○ ○                           ● ●                            │
│     ○ ○ ● ● ○         ─── φ ──→          ○ ○ ○                         │
│     ○ ● ● ○ ○                           ○ ● ● ═══ Hyperplane          │
│       ○ ○ ● ●                             ○ ○                          │
│                                           ● ●                           │
│   NOT linearly                          LINEARLY                       │
│   separable!                            SEPARABLE!                     │
│                                                                         │
│                    ┌─────────────────────┐                              │
│                    │   KERNEL TRICK:     │                              │
│                    │                     │                              │
│   Instead of       │  K(xᵢ, xⱼ)        │   Compute dot product       │
│   computing φ(x)   │  = φ(xᵢ) · φ(xⱼ) │   in feature space          │
│   explicitly       │                     │   DIRECTLY from             │
│                    │  Cost: O(n)         │   original vectors!         │
│                    │  Not: O(m)          │                              │
│                    └─────────────────────┘                              │
│                                                                         │
│   DECISION BOUNDARY:                                                    │
│                                                                         │
│   In feature space: Linear hyperplane w·φ(x) + b = 0                  │
│   In original space: Nonlinear boundary (circle, curve, etc.)          │
│                                                                         │
│       ● ● ○ ○                                                           │
│     ● ● ╱○ ○ ○       The hyperplane in feature space                   │
│     ○ ○╱● ● ○        maps to a CURVED boundary                        │
│     ○ ●╲● ○ ○        in the original space!                            │
│       ○ ○╲● ●                                                           │
│          curved boundary                                                │
│                                                                         │
└──────────────────────────────────────────────────────────────────────────┘
```

### Kernel Trick Flow Diagram

```
INPUT DATA (ℝⁿ)
    │
    ├── Option A: Explicit mapping (EXPENSIVE)
    │   │
    │   ├── Compute φ(xᵢ) for all i         O(N × m)
    │   ├── Compute all φ(xᵢ)·φ(xⱼ)        O(N² × m)
    │   ├── Solve QP with these products     O(N³)
    │   └── Total: O(N² × m + N³)           ← m can be ∞!
    │
    └── Option B: Kernel trick (EFFICIENT)
        │
        ├── Compute K(xᵢ, xⱼ) for all pairs  O(N² × n)
        ├── Solve QP with kernel matrix         O(N³)
        └── Total: O(N² × n + N³)              ← Always O(n)!
```

---

## 10. Python Implementation

### Demonstrating the Kernel Trick

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_circles, make_moons

# ── Generate nonlinearly separable data ──
X_circles, y_circles = make_circles(n_samples=300, noise=0.1,
                                     factor=0.3, random_state=42)

X_moons, y_moons = make_moons(n_samples=300, noise=0.15, random_state=42)

# ── Compare Linear vs Kernel SVM ──
fig, axes = plt.subplots(2, 3, figsize=(18, 10))

datasets = [
    (X_circles, y_circles, "Circles"),
    (X_moons, y_moons, "Moons")
]

kernels = [
    ('linear', {}),
    ('poly', {'degree': 3}),
    ('rbf', {'gamma': 'scale'})
]

for row, (X, y, name) in enumerate(datasets):
    for col, (kernel, params) in enumerate(kernels):
        ax = axes[row, col]
        
        svm = SVC(kernel=kernel, C=1.0, **params)
        svm.fit(X, y)
        
        # Decision boundary
        h = 0.02
        x_min, x_max = X[:, 0].min()-0.5, X[:, 0].max()+0.5
        y_min, y_max = X[:, 1].min()-0.5, X[:, 1].max()+0.5
        xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                             np.arange(y_min, y_max, h))
        Z = svm.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
        
        ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
        ax.scatter(X[y==0, 0], X[y==0, 1], c='red', s=20, edgecolors='k')
        ax.scatter(X[y==1, 0], X[y==1, 1], c='blue', s=20, edgecolors='k')
        
        acc = svm.score(X, y)
        ax.set_title(f'{name} - {kernel}\nAcc: {acc:.2f}, SVs: {sum(svm.n_support_)}')

plt.suptitle('Linear vs Kernel SVM', fontsize=16)
plt.tight_layout()
plt.savefig('kernel_trick_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

### Visualizing the Feature Map

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# ── Create circular data ──
np.random.seed(42)
n = 100

# Inner circle (class +1)
theta_inner = np.random.uniform(0, 2*np.pi, n)
r_inner = np.random.uniform(0, 1, n)
X_inner = np.column_stack([r_inner * np.cos(theta_inner),
                            r_inner * np.sin(theta_inner)])
y_inner = np.ones(n)

# Outer ring (class -1)
theta_outer = np.random.uniform(0, 2*np.pi, n)
r_outer = np.random.uniform(1.5, 2.5, n)
X_outer = np.column_stack([r_outer * np.cos(theta_outer),
                            r_outer * np.sin(theta_outer)])
y_outer = -np.ones(n)

X = np.vstack([X_inner, X_outer])
y = np.concatenate([y_inner, y_outer])

# ── Feature mapping: φ(x₁, x₂) = (x₁, x₂, x₁² + x₂²) ──
X_mapped = np.column_stack([X, X[:, 0]**2 + X[:, 1]**2])

fig = plt.figure(figsize=(16, 6))

# Original 2D space
ax1 = fig.add_subplot(121)
ax1.scatter(X[y==1, 0], X[y==1, 1], c='blue', label='+1', alpha=0.6)
ax1.scatter(X[y==-1, 0], X[y==-1, 1], c='red', label='-1', alpha=0.6)
ax1.set_title('Original 2D Space\n(Not linearly separable)', fontsize=14)
ax1.legend()
ax1.set_aspect('equal')
ax1.grid(True, alpha=0.3)

# Mapped 3D space
ax2 = fig.add_subplot(122, projection='3d')
ax2.scatter(X_mapped[y==1, 0], X_mapped[y==1, 1], X_mapped[y==1, 2],
            c='blue', label='+1', alpha=0.6)
ax2.scatter(X_mapped[y==-1, 0], X_mapped[y==-1, 1], X_mapped[y==-1, 2],
            c='red', label='-1', alpha=0.6)

# Separating plane
xx_plane, yy_plane = np.meshgrid(np.linspace(-3, 3, 10), np.linspace(-3, 3, 10))
zz_plane = np.ones_like(xx_plane) * 2.0
ax2.plot_surface(xx_plane, yy_plane, zz_plane, alpha=0.2, color='green')

ax2.set_title('Mapped 3D Space: φ(x₁,x₂) = (x₁, x₂, x₁²+x₂²)\n(Linearly separable!)', fontsize=12)
ax2.set_xlabel('x₁')
ax2.set_ylabel('x₂')
ax2.set_zlabel('x₁² + x₂²')
ax2.legend()

plt.tight_layout()
plt.savefig('feature_map_visualization.png', dpi=150, bbox_inches='tight')
plt.show()
```

### Building a Custom Kernel SVM

```python
from sklearn.svm import SVC
from sklearn.metrics.pairwise import rbf_kernel
import numpy as np

# Custom kernel function
def my_custom_kernel(X, Y):
    """
    Custom kernel: combination of linear and RBF.
    K(x,y) = x·y + exp(-0.5 ||x-y||²)
    """
    linear_part = X @ Y.T
    rbf_part = rbf_kernel(X, Y, gamma=0.5)
    return linear_part + rbf_part

# Use custom kernel in SVM
X, y = make_circles(n_samples=200, noise=0.1, factor=0.3, random_state=42)

svm_custom = SVC(kernel=my_custom_kernel)
svm_custom.fit(X, y)

print(f"Custom kernel SVM accuracy: {svm_custom.score(X, y):.4f}")
print(f"Number of support vectors: {sum(svm_custom.n_support_)}")
```

### Computing and Visualizing the Kernel Matrix

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.metrics.pairwise import rbf_kernel, polynomial_kernel

X = np.array([[0, 0], [1, 0], [0, 1], [1, 1], [2, 2], [3, 3]])

# Compute different kernel matrices
K_linear = X @ X.T
K_poly = polynomial_kernel(X, degree=2, gamma=1, coef0=1)
K_rbf = rbf_kernel(X, gamma=0.5)

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for ax, K, name in zip(axes, [K_linear, K_poly, K_rbf],
                        ['Linear', 'Polynomial (d=2)', 'RBF (γ=0.5)']):
    im = ax.imshow(K, cmap='viridis', aspect='auto')
    ax.set_title(f'{name} Kernel Matrix')
    plt.colorbar(im, ax=ax, shrink=0.8)
    
    # Check PSD (all eigenvalues ≥ 0)
    eigenvalues = np.linalg.eigvalsh(K)
    is_psd = np.all(eigenvalues >= -1e-10)
    ax.set_xlabel(f'PSD: {is_psd}, min eig: {eigenvalues.min():.4f}')

plt.tight_layout()
plt.savefig('kernel_matrices.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 11. Applications

### Where Kernel Trick Shines

| Application | Kernel Used | Why |
|-------------|------------|-----|
| **Text classification** | Linear | Already high-dimensional |
| **Image recognition** | RBF, Polynomial | Nonlinear visual features |
| **Bioinformatics** | String kernels | Sequence similarity |
| **Graph analysis** | Graph kernels | Structural comparison |
| **Time series** | Dynamic time warping kernel | Temporal patterns |
| **Chemistry** | Molecular kernels | Molecular structure comparison |
| **NLP** | Tree kernels | Parse tree similarity |

### Custom Kernels for Domain-Specific Problems

```
The kernel trick is extensible:
├── String kernels: K counts common substrings
├── Graph kernels: K compares graph structures  
├── Set kernels: K measures set overlap
├── Distribution kernels: K compares distributions
└── Any PSD function works as a kernel!
```

---

## 12. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Feature mapping φ** | Transforms data to higher-dimensional space |
| **Kernel function K(x,y)** | K(x,y) = φ(x)·φ(y) — dot product in feature space |
| **Kernel trick** | Compute K(x,y) directly without computing φ |
| **Why it works** | SVM dual only uses dot products → replace with K |
| **Efficiency** | O(n) per kernel eval vs O(m) for explicit mapping |
| **Gram matrix** | Kᵢⱼ = K(xᵢ,xⱼ), N×N, symmetric, PSD |
| **Mercer's theorem** | K is valid kernel ⟺ Gram matrix is PSD for any data |
| **Kernel closure** | Sum, product, scaling of valid kernels are valid |
| **Feature space** | Can be infinite-dimensional (RBF kernel) |
| **Decision boundary** | Linear in feature space → nonlinear in original space |
| **Cover's theorem** | Higher dimensions → more likely linearly separable |
| **Prediction** | f(x) = Σ_{SVs} αᵢyᵢK(xᵢ,x) + b |

---

## 13. Revision Questions

### Q1: Core Concept
**Explain in your own words what the kernel trick is and why it's called a "trick."**

<details>
<summary>Answer</summary>

The kernel trick is a computational shortcut that allows SVMs to operate in high-dimensional (even infinite-dimensional) feature spaces without ever explicitly computing the coordinates in those spaces. It's called a "trick" because:

1. It exploits the fact that the SVM dual formulation only requires **dot products** between data points, not the individual coordinates
2. A kernel function K(x,y) computes what the dot product φ(x)·φ(y) **would be** in the high-dimensional space, using only the original low-dimensional coordinates
3. The computation stays O(n) regardless of the feature space dimension m, which can even be infinite

It's essentially "getting the benefits of high-dimensional computation for free" — a mathematical shortcut that makes the impossible possible.

</details>

### Q2: Verification
**Verify that K(x,y) = (x·y)² is a valid kernel for x,y ∈ ℝ². Find the explicit feature mapping φ.**

<details>
<summary>Answer</summary>

For x = (x₁,x₂) and y = (y₁,y₂):

K(x,y) = (x·y)² = (x₁y₁ + x₂y₂)²
       = x₁²y₁² + 2x₁x₂y₁y₂ + x₂²y₂²
       = (x₁²)(y₁²) + (√2·x₁x₂)(√2·y₁y₂) + (x₂²)(y₂²)

This is the dot product of:
φ(x) = (x₁², √2·x₁x₂, x₂²)
φ(y) = (y₁², √2·y₁y₂, y₂²)

So K(x,y) = φ(x)·φ(y) ✓

Validity: K(x,y) = (x·y)² = K_linear(x,y)². Since K_linear is a valid kernel and the square of a valid kernel is valid (closure under product), K is valid. ✓

</details>

### Q3: Computational Efficiency
**A dataset has 1000 samples in 50 dimensions. You want to use a polynomial kernel of degree 5. Compare the cost of explicit mapping vs kernel trick.**

<details>
<summary>Answer</summary>

**Explicit mapping dimension**: C(n+d, d) = C(55, 5) = 3,478,761 dimensions (≈3.5 million)

**Explicit approach**:
- Map 1000 points: 1000 × 3,478,761 = 3.48 billion operations
- Compute 1000² dot products in 3.5M-D: 1,000,000 × 3,478,761 ≈ 3.48 trillion operations
- Store feature matrix: 1000 × 3.5M × 8 bytes ≈ 26 GB

**Kernel trick**:
- Compute 1000² kernel values: 1,000,000 × (50 mults + 1 add + 1 power) ≈ 52 million operations
- Store kernel matrix: 1000 × 1000 × 8 bytes = 8 MB

**Speedup**: ~67,000x in computation, ~3,250x in memory. And this gap grows with higher degrees!

</details>

### Q4: Mercer's Theorem
**Why can't we use K(x,y) = x·y - 1 as a kernel? What would go wrong?**

<details>
<summary>Answer</summary>

K(x,y) = x·y - 1 is NOT a valid kernel because it can produce a non-PSD Gram matrix.

Counterexample: Take x₁ = 0. Then K(x₁, x₁) = 0·0 - 1 = -1 < 0.

The 1×1 Gram matrix [K(x₁,x₁)] = [-1] has eigenvalue -1 < 0 → NOT PSD.

What goes wrong if used anyway:
- The QP dual is no longer convex (H matrix has negative eigenvalues)
- No guaranteed global optimum → may converge to local minimum or diverge
- The "distances" in the implied feature space become imaginary (√(-1))
- The geometric interpretation breaks down completely

Note: K(x,y) = x·y + 1 (with + instead of -) IS valid (polynomial kernel with d=1).

</details>

### Q5: Practical Application
**You have a dataset where classes form concentric rings in 2D. Which kernel would you choose and why? How would the decision boundary look in the original space?**

<details>
<summary>Answer</summary>

**Best kernel**: **RBF (Gaussian) kernel** K(x,y) = exp(-γ‖x-y‖²)

**Why RBF**:
1. Concentric rings are radially symmetric — RBF naturally captures radial distance
2. RBF maps to infinite-dimensional space → can represent any smooth boundary
3. RBF decision boundaries naturally form closed curves (circles, ellipses)
4. A polynomial kernel of degree 2 would also work (since x₁²+x₂² separates rings), but RBF is more flexible

**Decision boundary in original space**: Approximately **circular boundaries** between the rings. The linear hyperplane in the infinite-dimensional RBF feature space maps back to concentric circles in 2D.

```python
svm = SVC(kernel='rbf', gamma='scale', C=1.0)
```

</details>

### Q6: Mathematical
**Show that if K₁ and K₂ are valid kernels, then K(x,y) = K₁(x,y) + K₂(x,y) is also a valid kernel.**

<details>
<summary>Answer</summary>

**Proof**: Since K₁ and K₂ are valid kernels, by Mercer's theorem, their Gram matrices K₁ and K₂ are PSD for any set of points.

For any vector z ∈ ℝⁿ:
- zᵀK₁z ≥ 0 (K₁ is PSD)
- zᵀK₂z ≥ 0 (K₂ is PSD)

The Gram matrix of K = K₁ + K₂ is K = K₁ + K₂.

Therefore: zᵀKz = zᵀ(K₁ + K₂)z = zᵀK₁z + zᵀK₂z ≥ 0 + 0 = 0

Since zᵀKz ≥ 0 for all z, K is PSD → K is a valid kernel. ✓

**Feature map interpretation**: If φ₁ and φ₂ are the feature maps for K₁ and K₂, then K corresponds to the concatenated feature map φ(x) = [φ₁(x), φ₂(x)], since:
φ(x)·φ(y) = φ₁(x)·φ₁(y) + φ₂(x)·φ₂(y) = K₁(x,y) + K₂(x,y) = K(x,y)

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Soft Margin](./03-soft-margin.md) | [Unit 9: SVM](../) | [Common Kernels →](./05-common-kernels.md) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 4 of 7 — The Kernel Trick*
