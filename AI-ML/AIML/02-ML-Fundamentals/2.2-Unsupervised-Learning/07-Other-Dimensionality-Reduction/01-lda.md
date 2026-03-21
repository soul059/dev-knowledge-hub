# 📐 Linear Discriminant Analysis (LDA)

> **Chapter 7.1 — Supervised Dimensionality Reduction via Class Separability**

---

## 📌 Overview

**Linear Discriminant Analysis (LDA)** is a supervised dimensionality reduction and classification technique that finds a linear combination of features to **maximize class separability**. Unlike PCA, which finds directions of maximum variance regardless of class labels, LDA explicitly uses label information to project data onto axes that best separate the classes.

LDA was introduced by **Ronald A. Fisher (1936)** and is sometimes called **Fisher's Linear Discriminant** (for the two-class case).

### Key Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  PCA:  Find directions that maximize TOTAL VARIANCE                  │
│  LDA:  Find directions that maximize BETWEEN-CLASS VARIANCE          │
│         while minimizing WITHIN-CLASS VARIANCE                       │
│                                                                      │
│  Goal:  Maximize the ratio  σ²_between / σ²_within                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Motivation: Why Not Just PCA?

```
    Feature 2                          Feature 2
    ▲                                  ▲
    │  ○ ○ ○                           │  ○ ○ ○
    │    ○ ○   ● ●                     │    ○ ○   ● ●
    │  ○ ○       ● ●                   │  ○ ○       ● ●
    │          ● ●                     │          ● ●
    │        ●                         │        ●
    └──────────────► Feature 1         └──────────────► Feature 1

    PCA projection axis:               LDA projection axis:
    ──────────────────►                     ╱
    (max variance direction)              ╱  (max separation direction)
                                        ╱

    PCA result on 1D:                  LDA result on 1D:
    ○●○○●●○●●○  (MIXED!)              ○○○○○  ●●●●●  (SEPARATED!)
```

PCA may project onto a direction where classes overlap. LDA ensures classes are as far apart as possible in the projected space.

---

## 🔢 Mathematical Foundation

### Setup

- **N** data points in **d** dimensions
- **C** classes, with class *c* having **nₖ** samples
- **xᵢ** — individual data point
- **μₖ** — mean of class *k*
- **μ** — overall (grand) mean

### Step 1: Compute Class Means and Grand Mean

```
         1
μₖ  =  ───  Σ  xᵢ          (mean of class k)
        nₖ  xᵢ∈Cₖ

         1
μ   =  ───  Σ  xᵢ          (grand mean)
         N   i=1..N
```

### Step 2: Compute Scatter Matrices

#### Within-Class Scatter Matrix (Sw)

Measures the scatter (spread) of samples **within** each class:

```
         C
Sw  =   Σ   Sₖ
        k=1

where  Sₖ  =   Σ    (xᵢ - μₖ)(xᵢ - μₖ)ᵀ
              xᵢ∈Cₖ
```

```
┌─────────────────────────────────────────┐
│  Sw captures how "spread out" each      │
│  class is internally.                   │
│                                         │
│  Small Sw → Tight, compact clusters     │
│  Large Sw → Dispersed, spread clusters  │
└─────────────────────────────────────────┘
```

#### Between-Class Scatter Matrix (Sb)

Measures the scatter of class means **around** the grand mean:

```
         C
Sb  =   Σ   nₖ (μₖ - μ)(μₖ - μ)ᵀ
        k=1
```

```
┌─────────────────────────────────────────┐
│  Sb captures how "far apart" the class  │
│  means are from each other.             │
│                                         │
│  Large Sb → Classes well separated      │
│  Small Sb → Classes close together      │
└─────────────────────────────────────────┘
```

#### Total Scatter Matrix

```
St = Sw + Sb =  Σ  (xᵢ - μ)(xᵢ - μ)ᵀ
                i=1..N
```

### Step 3: Fisher's Criterion

We want to find projection vector **w** that **maximizes**:

```
              wᵀ Sb w
J(w)  =  ──────────────
              wᵀ Sw w
```

This is the **generalized Rayleigh quotient**. The solution is found by solving the **generalized eigenvalue problem**:

```
Sb w  =  λ Sw w

or equivalently (if Sw is invertible):

Sw⁻¹ Sb w  =  λ w
```

The eigenvectors corresponding to the **largest eigenvalues** give the LDA projection directions.

### Step 4: Dimensionality Limit

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  CRITICAL: LDA can produce AT MOST (C - 1)          │
│  discriminant components, where C = number of       │
│  classes.                                           │
│                                                     │
│  Why? Sb has rank at most (C - 1) because it is     │
│  computed from C class means, which lie in a        │
│  (C-1)-dimensional subspace.                        │
│                                                     │
│  Examples:                                          │
│  • 2 classes → at most 1 LDA component              │
│  • 5 classes → at most 4 LDA components             │
│  • 10 classes → at most 9 LDA components            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 📊 LDA Algorithm: Step-by-Step

```
╔══════════════════════════════════════════════════════════╗
║                    LDA ALGORITHM                        ║
╠══════════════════════════════════════════════════════════╣
║                                                         ║
║  INPUT: Data X (N×d), Labels y (C classes)              ║
║                                                         ║
║  1. Compute class means μ₁, μ₂, ..., μc and grand μ    ║
║                                                         ║
║  2. Compute Within-Class Scatter Matrix Sw              ║
║     Sw = Σₖ Σ_{x∈Cₖ} (x - μₖ)(x - μₖ)ᵀ               ║
║                                                         ║
║  3. Compute Between-Class Scatter Matrix Sb             ║
║     Sb = Σₖ nₖ(μₖ - μ)(μₖ - μ)ᵀ                       ║
║                                                         ║
║  4. Compute eigenvalues/eigenvectors of Sw⁻¹Sb         ║
║                                                         ║
║  5. Sort eigenvectors by eigenvalue (descending)        ║
║                                                         ║
║  6. Select top k ≤ (C-1) eigenvectors → W (d×k)        ║
║                                                         ║
║  7. Project: X_new = X · W                              ║
║                                                         ║
║  OUTPUT: Projected data X_new (N×k)                     ║
║                                                         ║
╚══════════════════════════════════════════════════════════╝
```

---

## 📐 Two-Class LDA (Fisher's Linear Discriminant)

For the binary case, the solution simplifies elegantly:

```
w  =  Sw⁻¹ (μ₁ - μ₂)
```

The projected score for a new point x is:

```
y = wᵀ x
```

Classification: compare projected value to threshold (typically the midpoint of projected class means).

```
         Class 1          Class 2
    ┌──────────────┐ ┌──────────────┐
    │ ○  ○  ○  ○   │ │   ●  ●  ●  ● │
    └──────┬───────┘ └───────┬──────┘
           │                 │
    ───────▼────────┬────────▼─────────►  LDA axis
           μ₁'     threshold   μ₂'
```

---

## 📝 Worked Example

### Dataset: 2 Features, 2 Classes

```
Class 0 (○):  (4,2), (2,4), (2,3), (3,6), (4,4)
Class 1 (●):  (9,10), (6,8), (9,5), (8,7), (10,8)
```

### Step 1: Class Means

```
μ₀ = (4+2+2+3+4)/5 , (2+4+3+6+4)/5 = (3.0, 3.8)
μ₁ = (9+6+9+8+10)/5, (10+8+5+7+8)/5 = (8.4, 7.6)
μ  = (3.0+8.4)/2, (3.8+7.6)/2 = (5.7, 5.7)  [weighted by class size]
```

### Step 2: Within-Class Scatter

```
S₀ = Σ (xᵢ - μ₀)(xᵢ - μ₀)ᵀ  for class 0
   = (4-3)²+(2-3)²+...  → compute full 2×2 matrix

S₀ = │ 4.0   0.4 │
     │ 0.4   8.8 │

S₁ = │ 10.8  -0.4│
     │ -0.4  13.2│

Sw = S₀ + S₁ = │14.8   0.0│
               │ 0.0  22.0│
```

### Step 3: Between-Class Scatter

```
Sb = 5·(μ₀-μ)(μ₀-μ)ᵀ + 5·(μ₁-μ)(μ₁-μ)ᵀ

(μ₀ - μ) = (-2.7, -1.9)
(μ₁ - μ) = (2.7, 1.9)

Sb = 5·│7.29  5.13│ + 5·│7.29  5.13│ = │72.9  51.3│
       │5.13  3.61│     │5.13  3.61│   │51.3  36.1│
```

### Step 4: Solve Sw⁻¹Sb

```
Sw⁻¹ = │1/14.8    0   │ = │0.0676    0   │
       │  0    1/22.0 │   │  0    0.0455 │

Sw⁻¹ Sb = │0.0676·72.9  0.0676·51.3│ = │4.928  3.468│
          │0.0455·51.3  0.0455·36.1│   │2.334  1.643│
```

### Step 5: Eigenvectors

```
The dominant eigenvector ≈ w = [0.816, 0.578] (normalized)

This is the LDA projection direction.
```

### Step 6: Project

```
Projected class 0 points:
  (4,2)·w = 4·0.816 + 2·0.578 = 4.42
  ...

Projected class 1 points:
  (9,10)·w = 9·0.816 + 10·0.578 = 13.12
  ...

Classes are well separated on this 1D projection!
```

---

## 🐍 Python Implementation

### From Scratch

```python
import numpy as np

def lda_from_scratch(X, y, n_components=None):
    """
    Linear Discriminant Analysis from scratch.
    
    Parameters:
        X: ndarray of shape (n_samples, n_features)
        y: ndarray of shape (n_samples,)
        n_components: number of LDA components (max C-1)
    
    Returns:
        W: projection matrix (n_features, n_components)
        X_projected: projected data (n_samples, n_components)
    """
    classes = np.unique(y)
    n_features = X.shape[1]
    C = len(classes)
    
    if n_components is None:
        n_components = min(C - 1, n_features)
    
    # Grand mean
    mean_overall = np.mean(X, axis=0)
    
    # Initialize scatter matrices
    Sw = np.zeros((n_features, n_features))
    Sb = np.zeros((n_features, n_features))
    
    for cls in classes:
        X_cls = X[y == cls]
        mean_cls = np.mean(X_cls, axis=0)
        n_cls = X_cls.shape[0]
        
        # Within-class scatter
        diff = X_cls - mean_cls
        Sw += diff.T @ diff
        
        # Between-class scatter
        mean_diff = (mean_cls - mean_overall).reshape(-1, 1)
        Sb += n_cls * (mean_diff @ mean_diff.T)
    
    # Solve generalized eigenvalue problem
    A = np.linalg.inv(Sw) @ Sb
    eigenvalues, eigenvectors = np.linalg.eig(A)
    
    # Sort by eigenvalue (descending)
    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx].real
    eigenvectors = eigenvectors[:, idx].real
    
    # Select top components
    W = eigenvectors[:, :n_components]
    
    # Project data
    X_projected = X @ W
    
    return W, X_projected, eigenvalues[:n_components]

# Example usage
np.random.seed(42)
# Generate sample data
X_class0 = np.random.randn(50, 2) + np.array([2, 3])
X_class1 = np.random.randn(50, 2) + np.array([6, 7])
X = np.vstack([X_class0, X_class1])
y = np.array([0]*50 + [1]*50)

W, X_proj, eigenvalues = lda_from_scratch(X, y, n_components=1)
print(f"Projection vector: {W.flatten()}")
print(f"Eigenvalue: {eigenvalues[0]:.4f}")
print(f"Class 0 mean (projected): {X_proj[y==0].mean():.4f}")
print(f"Class 1 mean (projected): {X_proj[y==1].mean():.4f}")
```

### Scikit-learn Implementation

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.datasets import load_iris
import matplotlib.pyplot as plt
import numpy as np

# Load Iris dataset (4 features, 3 classes)
iris = load_iris()
X, y = iris.data, iris.target

# LDA: 3 classes → max 2 components
lda = LinearDiscriminantAnalysis(n_components=2)
X_lda = lda.fit_transform(X, y)

print(f"Original shape: {X.shape}")         # (150, 4)
print(f"LDA shape:      {X_lda.shape}")     # (150, 2)
print(f"Explained variance ratio: {lda.explained_variance_ratio_}")

# Visualization
plt.figure(figsize=(10, 6))
colors = ['#e74c3c', '#2ecc71', '#3498db']
target_names = iris.target_names

for color, i, name in zip(colors, [0, 1, 2], target_names):
    plt.scatter(X_lda[y == i, 0], X_lda[y == i, 1],
                color=color, alpha=0.7, label=name, s=60)

plt.xlabel('LD 1')
plt.ylabel('LD 2')
plt.title('LDA of Iris Dataset')
plt.legend(loc='best')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('lda_iris.png', dpi=150)
plt.show()
```

### LDA for Classification

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# LDA as classifier
lda_clf = LinearDiscriminantAnalysis()
lda_clf.fit(X_train, y_train)

y_pred = lda_clf.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred, target_names=iris.target_names))

# LDA as dimensionality reduction + other classifier
from sklearn.neighbors import KNeighborsClassifier

lda_reducer = LinearDiscriminantAnalysis(n_components=2)
X_train_lda = lda_reducer.fit_transform(X_train, y_train)
X_test_lda = lda_reducer.transform(X_test)

knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train_lda, y_train)
y_pred_knn = knn.predict(X_test_lda)
print(f"KNN Accuracy (after LDA): {accuracy_score(y_test, y_pred_knn):.4f}")
```

---

## ⚖️ LDA vs PCA: Comprehensive Comparison

```
┌─────────────────────┬─────────────────────┬─────────────────────┐
│     Aspect          │       PCA           │       LDA           │
├─────────────────────┼─────────────────────┼─────────────────────┤
│ Type                │ Unsupervised        │ Supervised          │
│ Objective           │ Max total variance  │ Max class           │
│                     │                     │ separability        │
│ Labels required?    │ No                  │ Yes                 │
│ Max components      │ min(N, d)           │ C - 1               │
│ Optimal for         │ General reduction   │ Classification      │
│ Assumes             │ None (linear)       │ Gaussian classes,   │
│                     │                     │ equal covariance    │
│ Scatter matrix      │ Total scatter       │ Sw⁻¹ Sb             │
│ Invariant to        │ Orthogonal rotation │ Class relabeling    │
│ Outlier sensitive   │ Yes                 │ Yes                 │
│ Overfitting risk    │ Low                 │ Higher (uses labels)│
│ Use case            │ Exploration,        │ Pre-classification  │
│                     │ preprocessing       │ dimensionality      │
│                     │                     │ reduction           │
└─────────────────────┴─────────────────────┴─────────────────────┘
```

### Visual Comparison

```
  Original Data (2 classes)         PCA Projection         LDA Projection
  ─────────────────────────        ────────────────        ────────────────
        ●●●                              ●●●
       ●●●●   ○○                        ●●●●○○              
      ●●●●  ○○○○           PC1: ───●●●●○○○○○──►    LD1: ──●●●●│○○○○──►
       ●●● ○○○○                   (some overlap)         (clean separation)
        ●   ○○○
            ○
```

---

## ⚠️ Assumptions and Limitations

### Assumptions

1. **Gaussian distributed classes** — Each class follows a multivariate normal distribution
2. **Equal covariance matrices** — All classes share the same covariance (homoscedasticity)
3. **Linearity** — Decision boundaries are linear (hyperplanes)
4. **Non-singular Sw** — Within-class scatter matrix must be invertible

### Handling Singular Sw (Small Sample Problem)

When **N < d** (more features than samples), Sw is singular. Solutions:

```python
# Solution 1: Regularized LDA (shrinkage)
lda = LinearDiscriminantAnalysis(solver='lsqr', shrinkage='auto')

# Solution 2: PCA first, then LDA
from sklearn.decomposition import PCA
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('pca', PCA(n_components=50)),      # Reduce to 50 dims first
    ('lda', LinearDiscriminantAnalysis(n_components=2))
])
X_reduced = pipe.fit_transform(X, y)

# Solution 3: Use pseudoinverse
# Sw_inv = np.linalg.pinv(Sw)
```

### Limitations

```
┌────────────────────────────────────────────────────────────┐
│ LIMITATIONS OF LDA                                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│ 1. Max (C-1) components — severe limit with few classes    │
│ 2. Assumes Gaussian distributions — fails for non-Gaussian │
│ 3. Equal covariance assumption — rarely true in practice   │
│ 4. Linear boundaries only — can't capture nonlinear        │
│    decision boundaries                                     │
│ 5. Sensitive to outliers — means and covariances affected  │
│ 6. Requires labeled data — not always available            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 🏭 Real-World Applications

| Domain | Application | Details |
|--------|-------------|---------|
| **Face Recognition** | Fisherfaces | LDA on face images for recognition |
| **Medical Diagnosis** | Disease classification | Reduce biomarker dimensions |
| **Finance** | Credit scoring | Separate good/bad borrowers |
| **NLP** | Text classification | Reduce TF-IDF dimensions by topic |
| **Bioinformatics** | Gene expression | Classify cancer subtypes |
| **Signal Processing** | EEG/ECG classification | Brain-computer interfaces |

---

## 🔄 Variants of LDA

| Variant | Description |
|---------|-------------|
| **Quadratic DA (QDA)** | Each class has its own covariance matrix |
| **Regularized LDA** | Adds regularization to handle singular Sw |
| **Kernel LDA** | Nonlinear extension using kernel trick |
| **Flexible DA (FDA)** | Uses nonparametric regression for boundaries |
| **Heteroscedastic LDA** | Handles unequal covariance matrices |

```python
# QDA — allows different covariance per class
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis

qda = QuadraticDiscriminantAnalysis()
qda.fit(X_train, y_train)
print(f"QDA Accuracy: {qda.score(X_test, y_test):.4f}")
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Full Name** | Linear Discriminant Analysis |
| **Type** | Supervised dimensionality reduction / classification |
| **Objective** | Maximize between-class / minimize within-class variance |
| **Criterion** | Fisher's criterion: J(w) = wᵀSbw / wᵀSww |
| **Solution** | Eigenvectors of Sw⁻¹Sb |
| **Max Components** | C − 1 (C = number of classes) |
| **Assumptions** | Gaussian classes, equal covariance, linear boundaries |
| **Complexity** | O(d²N + d³) |
| **sklearn** | `LinearDiscriminantAnalysis(n_components=k)` |
| **Key Parameters** | `solver`, `shrinkage`, `n_components` |

---

## ❓ Revision Questions

1. **What is Fisher's criterion and how does it differ from PCA's objective?**
   Explain the mathematical formulation and the intuition behind maximizing between-class variance while minimizing within-class variance.

2. **Why is LDA limited to at most (C−1) components?**
   Discuss the rank of the between-class scatter matrix and its relationship to the number of classes.

3. **A dataset has 1000 samples, 500 features, and 3 classes. What is the maximum number of LDA components? What problem might arise, and how would you solve it?**
   Consider the singularity of Sw and discuss regularization or PCA preprocessing.

4. **Compare LDA and PCA for a classification task on a dataset where classes have very different variances but similar means. Which would perform better and why?**
   Think about what each method optimizes and how overlapping classes affect projections.

5. **What assumptions does LDA make, and what happens when they are violated?**
   Discuss Gaussian assumption, equal covariance, and alternatives like QDA when assumptions fail.

6. **Implement LDA from scratch for a 3-class problem. Walk through the scatter matrix computations and eigendecomposition.**
   Show the step-by-step process including class means, Sw, Sb, and the final projection.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← PCA](../06-Principal-Component-Analysis/) | [📂 Unsupervised Learning](../README.md) | [t-SNE →](./02-tsne.md) |

---

> **Key Takeaway:** LDA is the go-to method when you have labeled data and want to reduce dimensions while preserving class separability. Remember: it's supervised (unlike PCA), limited to C−1 components, and assumes Gaussian classes with equal covariance.
