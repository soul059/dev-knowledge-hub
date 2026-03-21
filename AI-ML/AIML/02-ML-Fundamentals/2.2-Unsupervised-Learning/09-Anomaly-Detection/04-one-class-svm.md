# 🛡️ One-Class SVM

> **Chapter 9.4 — Boundary-Based Anomaly Detection with Support Vector Machines**

---

## 📌 Overview

**One-Class SVM (OCSVM)** is a semi-supervised anomaly detection method that learns a **decision boundary** enclosing the "normal" data in feature space. Points falling **outside** this boundary are classified as anomalies. It extends the traditional SVM framework to the unsupervised setting, using kernel functions to handle nonlinear boundaries.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Standard SVM (2 classes):       One-Class SVM:                     │
│                                                                      │
│    ● ●  │  ○ ○                    ┌─────────────┐                    │
│    ● ●  │  ○ ○                    │  ● ● ● ●    │                    │
│    ● ●  │  ○ ○                    │  ● ● ● ● ●  │  ★ ← Anomaly     │
│         │                         │  ● ● ● ●    │                    │
│    Separates 2 classes            │  ● ● ●      │                    │
│                                   └─────────────┘                    │
│                                   Boundary around                    │
│                                   ONE class (normal)                 │
│                                                                      │
│  INSIDE boundary → Normal                                           │
│  OUTSIDE boundary → Anomaly                                          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Mathematical Foundation

### The Optimization Problem

One-Class SVM (Schölkopf et al., 2001) finds a hyperplane in **kernel feature space** that maximally separates the data from the **origin**:

```
  Primal Problem:

  min    ½‖w‖² + (1/νN) Σᵢ ξᵢ  -  ρ
  w,ξ,ρ

  subject to:
    wᵀΦ(xᵢ) ≥ ρ - ξᵢ    for all i
    ξᵢ ≥ 0               for all i

  where:
    w     = normal vector to the hyperplane
    ρ     = offset (distance from origin)
    ξᵢ    = slack variables (allow some points inside)
    ν     = upper bound on fraction of outliers
    Φ(x)  = feature map (kernel function)
    N     = number of training points
```

### The ν Parameter

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ν (nu) has a DUAL interpretation:                           │
│                                                              │
│  1. Upper bound on the FRACTION OF OUTLIERS                  │
│     in the training set                                      │
│                                                              │
│  2. Lower bound on the FRACTION OF SUPPORT VECTORS           │
│                                                              │
│  Example:                                                    │
│  ν = 0.05 → At most 5% of training points can be outliers   │
│             At least 5% of points will be support vectors    │
│                                                              │
│  EFFECT:                                                     │
│  Small ν (0.01) → Tight boundary, few outliers tolerated     │
│  Large ν (0.20) → Loose boundary, many outliers allowed      │
│                                                              │
│  ν    Boundary                                               │
│  0.01 ┌──────┐  Very tight (strict)                          │
│       │●●●●●●│                                               │
│       └──────┘                                               │
│                                                              │
│  0.20 ┌──────────────┐  Loose (lenient)                      │
│       │  ●●●●●●      │                                       │
│       │    ●●●●      │                                       │
│       └──────────────┘                                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Decision Function

```
  f(x) = sign(wᵀΦ(x) - ρ)

  f(x) > 0  →  NORMAL  (inside boundary)
  f(x) < 0  →  ANOMALY (outside boundary)
  f(x) = 0  →  ON the decision boundary
```

### Kernel Trick

```
  The kernel function avoids explicitly computing Φ(x):

  K(xᵢ, xⱼ) = Φ(xᵢ)ᵀ Φ(xⱼ)

  Common kernels:

  ┌──────────────┬────────────────────────────┬──────────────────────┐
  │ Kernel       │ Formula                    │ Decision Boundary    │
  ├──────────────┼────────────────────────────┼──────────────────────┤
  │ Linear       │ K(x,y) = xᵀy              │ Hyperplane           │
  │ RBF (Gauss.) │ K(x,y) = exp(-γ‖x-y‖²)   │ Flexible, nonlinear  │
  │ Polynomial   │ K(x,y) = (γxᵀy + r)^d    │ Polynomial surface   │
  │ Sigmoid      │ K(x,y) = tanh(γxᵀy + r)  │ Sigmoidal            │
  └──────────────┴────────────────────────────┴──────────────────────┘

  RBF is the DEFAULT and usually best choice for One-Class SVM
```

### RBF Kernel: Effect of γ

```
  Small γ (e.g., 0.01):         Large γ (e.g., 10):
  ┌──────────────────┐         ┌──────────────────┐
  │  ╭──────────────╮│         │    ╭──╮          │
  │  │  ●● ●●●      ││         │    │●●│ ╭──╮    │
  │  │  ●●●●●●  ●   ││         │    │●●│ │● │    │
  │  │  ●●●●●●      ││         │    ╰──╯ ╰──╯    │
  │  ╰──────────────╯│         │                  │
  │  Smooth, broad    │         │  Tight, complex  │
  │  boundary         │         │  boundary         │
  └──────────────────┘         └──────────────────┘

  γ = 1/(n_features · X.var())  ← 'scale' default in sklearn
```

---

## 🐍 Python Implementation

### Scikit-learn

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import OneClassSVM

# Generate data
np.random.seed(42)
X_train = 0.3 * np.random.randn(200, 2)  # Normal data (for training)
X_test_normal = 0.3 * np.random.randn(50, 2)
X_test_anomaly = np.random.uniform(low=-3, high=3, size=(20, 2))
X_test = np.vstack([X_test_normal, X_test_anomaly])
y_test = np.array([1]*50 + [-1]*20)

# Train One-Class SVM
ocsvm = OneClassSVM(
    kernel='rbf',       # Kernel type (default: 'rbf')
    gamma='scale',      # Kernel coefficient (1/(n_features·var))
    nu=0.05,            # Upper bound on outlier fraction
)
ocsvm.fit(X_train)

# Predict
y_pred = ocsvm.predict(X_test)      # 1=normal, -1=anomaly
scores = ocsvm.decision_function(X_test)  # Distance from boundary

print(f"Anomalies detected: {(y_pred == -1).sum()}")
print(f"True anomalies: {(y_test == -1).sum()}")

# Evaluate
from sklearn.metrics import classification_report
print(classification_report(y_test, y_pred, target_names=['Anomaly', 'Normal']))

# Visualization
plt.figure(figsize=(10, 8))

# Decision boundary
xx, yy = np.meshgrid(np.linspace(-4, 4, 200), np.linspace(-4, 4, 200))
Z = ocsvm.decision_function(np.c_[xx.ravel(), yy.ravel()])
Z = Z.reshape(xx.shape)

plt.contourf(xx, yy, Z, levels=np.linspace(Z.min(), 0, 7), 
             cmap='PuBu', alpha=0.3)
plt.contour(xx, yy, Z, levels=[0], linewidths=2, colors='darkred')
plt.contourf(xx, yy, Z, levels=[0, Z.max()], colors='palegreen', alpha=0.3)

# Training data
plt.scatter(X_train[:, 0], X_train[:, 1], c='white', s=20, 
            edgecolors='gray', label='Training (normal)')

# Test data
normal_mask = y_pred == 1
plt.scatter(X_test[normal_mask, 0], X_test[normal_mask, 1], 
            c='green', s=30, label='Predicted Normal')
plt.scatter(X_test[~normal_mask, 0], X_test[~normal_mask, 1], 
            c='red', s=60, marker='*', label='Predicted Anomaly')

plt.title('One-Class SVM Decision Boundary')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### Effect of ν Parameter

```python
fig, axes = plt.subplots(1, 4, figsize=(20, 5))
nu_values = [0.01, 0.05, 0.10, 0.30]

for ax, nu in zip(axes, nu_values):
    ocsvm = OneClassSVM(kernel='rbf', gamma='scale', nu=nu)
    ocsvm.fit(X_train)
    
    Z = ocsvm.decision_function(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    
    ax.contourf(xx, yy, Z, levels=[0, Z.max()], colors='palegreen', alpha=0.3)
    ax.contour(xx, yy, Z, levels=[0], linewidths=2, colors='red')
    ax.scatter(X_train[:, 0], X_train[:, 1], c='blue', s=5)
    
    n_outliers = (ocsvm.predict(X_train) == -1).sum()
    ax.set_title(f'ν={nu}\n({n_outliers} outliers in train)')
    ax.set_xlim(-2, 2)
    ax.set_ylim(-2, 2)

plt.suptitle('One-Class SVM: Effect of ν Parameter', fontsize=14)
plt.tight_layout()
plt.show()
```

---

## ⚖️ One-Class SVM vs Isolation Forest

```
┌──────────────────────┬─────────────────────┬─────────────────────┐
│     Aspect           │   One-Class SVM     │  Isolation Forest   │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Approach             │ Boundary-based      │ Isolation-based     │
│ Training             │ Train on normal     │ Train on all data   │
│                      │ data only           │ (mixed)             │
│ Theory               │ SVM/kernel methods  │ Random forests      │
│ Handles nonlinear    │ Yes (kernel trick)  │ Yes (random splits) │
│ Scalability          │ O(N²)-O(N³)         │ O(t·ψ·log ψ)       │
│ Large datasets       │ Slow (>10K)         │ Fast (millions)     │
│ High-D data          │ Struggles           │ Handles well        │
│ Key parameter        │ ν, γ, kernel        │ contamination       │
│ Tuning difficulty    │ Hard (γ sensitive)  │ Easy (few params)   │
│ Novelty detection    │ ✅ Natural fit       │ ✅ Possible         │
│ Memory               │ O(N·d) for SVs      │ O(t·ψ)             │
│ Interpretability     │ Moderate (boundary) │ Low (ensemble)      │
│ sklearn              │ OneClassSVM         │ IsolationForest     │
└──────────────────────┴─────────────────────┴─────────────────────┘

RULE OF THUMB:
• Small-medium data (<10K): Either works; OCSVM gives cleaner boundary
• Large data (>10K): Use Isolation Forest (much faster)
• Clean training data available: OCSVM (novelty detection)
• Mixed/unlabeled data: Isolation Forest
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Algorithm** | One-Class SVM (Schölkopf et al., 2001) |
| **Approach** | Learn boundary around normal data |
| **Key Idea** | Maximize margin between data and origin in kernel space |
| **ν Parameter** | Upper bound on outlier fraction / lower bound on SVs |
| **Kernel** | RBF (default), linear, polynomial |
| **Decision** | f(x) > 0 → normal, f(x) < 0 → anomaly |
| **Training** | On normal data only (semi-supervised) |
| **Complexity** | O(N²) to O(N³) |
| **Best For** | Clean training data, novelty detection, small-medium N |
| **sklearn** | `OneClassSVM(kernel='rbf', nu=0.05)` |

---

## ❓ Revision Questions

1. **Explain how One-Class SVM adapts the standard SVM framework for anomaly detection. What does "separating from the origin" mean geometrically?**
   Discuss the mapping to feature space and the hyperplane construction.

2. **What is the ν parameter and its dual interpretation? How does changing ν affect the decision boundary?**
   Explain the trade-off between tight boundaries (few false positives) and loose boundaries (fewer missed anomalies).

3. **Why is the RBF kernel the default choice for One-Class SVM? What happens with a linear kernel?**
   Discuss the ability to create nonlinear boundaries and the effect of γ.

4. **Compare One-Class SVM with Isolation Forest for a dataset of 1 million network packets. Which would you choose and why?**
   Consider computational complexity, scalability, and practical constraints.

5. **One-Class SVM is trained on normal data only. What happens if the training data is contaminated with anomalies?**
   Discuss the impact on the learned boundary and potential mitigation strategies.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Isolation Forest](./03-isolation-forest.md) | [📂 Unsupervised Learning](../README.md) | [Local Outlier Factor →](./05-local-outlier-factor.md) |

---

> **Key Takeaway:** One-Class SVM learns a tight decision boundary around normal data in kernel-transformed space. It excels at novelty detection when clean training data is available and datasets are moderate in size. Use the RBF kernel for nonlinear boundaries and tune ν based on your expected outlier fraction.
