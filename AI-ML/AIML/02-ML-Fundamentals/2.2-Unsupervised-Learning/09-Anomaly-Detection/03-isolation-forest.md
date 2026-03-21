# 🌲 Isolation Forest

> **Chapter 9.3 — Anomaly Detection via Isolation**

---

## 📌 Overview

**Isolation Forest** is an unsupervised anomaly detection algorithm introduced by **Fei Tony Liu, Kai Ming Ting, and Zhi-Hua Zhou (2008)**. Unlike most methods that model "normal" data and then flag deviations, Isolation Forest directly **isolates anomalies** by exploiting the fact that anomalies are *few* and *different* — making them easier to separate from the rest.

### Core Intuition

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  KEY INSIGHT:                                                        │
│  Anomalies are FEW and DIFFERENT →                                  │
│  They are EASIER TO ISOLATE with random splits.                      │
│                                                                      │
│  Normal Point:                    Anomaly:                           │
│  Needs MANY splits to isolate     Needs FEW splits to isolate       │
│                                                                      │
│  ●●●●●●●●                          ●●●●●●●                          │
│  ●●●●●●●●                          ●●●●●●●            ★            │
│  ●●●●●●●●  ← Buried in            ●●●●●●●                          │
│  ●●●●●●●●     the crowd!                                           │
│  ●●●●●●●●                          ★ is EASY to isolate!            │
│                                     (one split might do)             │
│                                                                      │
│  Short path = ANOMALY              Long path = NORMAL                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Algorithm: How Isolation Forest Works

### Building an Isolation Tree (iTree)

```
╔══════════════════════════════════════════════════════════════╗
║              ISOLATION TREE CONSTRUCTION                    ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Given: Subsample of data X' (size ψ, typically 256)         ║
║                                                              ║
║  Recursively:                                                ║
║  1. Randomly select a FEATURE q                              ║
║  2. Randomly select a SPLIT VALUE p between                  ║
║     [min(q), max(q)] of remaining data                       ║
║  3. Split data: left (q < p), right (q ≥ p)                 ║
║  4. Repeat until:                                            ║
║     a. Node has only 1 point (isolated!), or                 ║
║     b. All points have same value, or                        ║
║     c. Maximum tree height reached (⌈log₂(ψ)⌉)              ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### Visual Example

```
  Data: ● = normal points, ★ = anomaly

  Feature 2
  ▲
  │     ●●●●●
  │    ●●●●●●●
  │   ●●●●●●●●
  │    ●●●●●●●                    ★
  │     ●●●●●
  └───────────────────────────────────► Feature 1

  Random split 1 (Feature 1 at x=8):
  ┌─────────────────┬──────────────────┐
  │  ●●●●●          │                ★ │  ← ★ already isolated!
  │ ●●●●●●●         │                  │     Path length = 1
  │●●●●●●●●         │                  │
  │ ●●●●●●●         │                  │
  │  ●●●●●          │                  │
  └─────────────────┴──────────────────┘

  To isolate a normal point ● takes many more splits:
  Split 1 → Split 2 → Split 3 → ... → Split 7
  Path length = 7

  ★ Anomaly: path length ≈ 1-2  (SHORT = anomalous)
  ● Normal:  path length ≈ 6-8  (LONG = normal)
```

### Building the Isolation Forest

```
╔══════════════════════════════════════════════════════════════╗
║              ISOLATION FOREST ALGORITHM                     ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  TRAINING:                                                   ║
║  1. Set number of trees: t (default 100)                     ║
║  2. Set subsample size: ψ (default 256)                      ║
║  3. For each tree i = 1, ..., t:                             ║
║     a. Randomly sample ψ points from data                    ║
║     b. Build an iTree by recursive random splits             ║
║                                                              ║
║  SCORING:                                                    ║
║  1. For each point x:                                        ║
║     a. Pass x through ALL t trees                            ║
║     b. Record path length h(x) in each tree                  ║
║     c. Average: E[h(x)] = mean path length                   ║
║  2. Compute anomaly score:                                   ║
║                                                              ║
║     s(x, ψ) = 2^(-E[h(x)] / c(ψ))                          ║
║                                                              ║
║     where c(ψ) = 2H(ψ-1) - 2(ψ-1)/ψ                        ║
║     (average path length of unsuccessful BST search)         ║
║     H(k) = ln(k) + 0.5772... (Euler-Mascheroni constant)    ║
║                                                              ║
║  DECISION:                                                   ║
║     s(x) close to 1  → ANOMALY                              ║
║     s(x) close to 0  → NORMAL                               ║
║     s(x) ≈ 0.5       → UNCERTAIN                            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### Anomaly Score Interpretation

```
  Anomaly Score s(x):

  0.0          0.5          1.0
  │─────────────┼─────────────│
  │   NORMAL    │   ANOMALY   │
  │             │             │
  │  Long path  │  Short path │
  │  (deep in   │  (easily    │
  │   tree)     │  isolated)  │

  s → 1:  E[h(x)] → 0  (very short path → definite anomaly)
  s → 0:  E[h(x)] → ∞  (very long path → definitely normal)
  s ≈ 0.5: E[h(x)] ≈ c(ψ) (average path → no clear distinction)
```

---

## 📝 Worked Example

### Small Dataset

```
Points: A(1,1), B(2,2), C(1.5,1.5), D(2,1), E(1,2), F(8,8)

  Feature 2
  ▲
  8 │                              F★
  │
  2 │ E●  B●
  │  C●
  1 │ A●  D●
  └──┼──┼──┼──┼──┼──┼──┼──┼──► Feature 1
     1  2  3  4  5  6  7  8
```

### Tree 1: Random Splits

```
  Split 1: Feature 1 at x=5.3
  ┌───────────────────┬─────────────┐
  │ A,B,C,D,E (left)  │    F (right)│ ← F isolated! Path=1
  └───────────────────┴─────────────┘

  Continue splitting left branch:
  Split 2: Feature 2 at y=1.7
  ┌──────────────┬──────────┐
  │ A,D (below)  │ B,C,E    │
  └──────────────┴──────────┘

  Split 3: Feature 1 at x=1.3
  ┌──────┬───────┐
  │ A    │ D     │ ← A isolated at path=3
  └──────┴───────┘    D isolated at path=3
```

### Path Lengths

```
  ┌───────┬──────────────┬──────────────────────┐
  │ Point │ Path Length   │ Interpretation        │
  ├───────┼──────────────┼──────────────────────┤
  │ F     │ 1            │ Easily isolated → ★   │
  │ A     │ 3            │ Harder to isolate → ● │
  │ D     │ 3            │ Harder to isolate → ● │
  │ B     │ 4            │ Deep in tree → ●      │
  │ C     │ 4            │ Deep in tree → ●      │
  │ E     │ 4            │ Deep in tree → ●      │
  └───────┴──────────────┴──────────────────────┘

  F (anomaly) has the SHORTEST path → highest anomaly score!
```

---

## 🐍 Python Implementation

### Scikit-learn

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

# Generate data with anomalies
np.random.seed(42)
X_normal = 0.3 * np.random.randn(300, 2)
X_outliers = np.random.uniform(low=-3, high=3, size=(20, 2))
X = np.vstack([X_normal, X_outliers])

# Train Isolation Forest
clf = IsolationForest(
    n_estimators=100,       # Number of trees (default 100)
    max_samples='auto',     # Subsample size (default 256 or N if N<256)
    contamination=0.05,     # Expected proportion of anomalies
    random_state=42,
    max_features=1.0,       # Features to draw (default 1.0 = all)
    bootstrap=False         # Sample without replacement
)

# Fit and predict
clf.fit(X)
y_pred = clf.predict(X)           # 1=normal, -1=anomaly
scores = clf.decision_function(X) # Lower = more anomalous
anomaly_scores = -scores          # Flip so higher = more anomalous

print(f"Total points: {len(X)}")
print(f"Anomalies detected: {(y_pred == -1).sum()}")
print(f"Normal points: {(y_pred == 1).sum()}")

# Visualization
plt.figure(figsize=(10, 8))

# Decision boundary
xx, yy = np.meshgrid(np.linspace(-4, 4, 100), np.linspace(-4, 4, 100))
Z = clf.decision_function(np.c_[xx.ravel(), yy.ravel()])
Z = Z.reshape(xx.shape)

plt.contourf(xx, yy, Z, levels=np.linspace(Z.min(), 0, 7), cmap='Blues_r', alpha=0.3)
plt.contour(xx, yy, Z, levels=[0], linewidths=2, colors='red', linestyles='--')

# Plot points
normal_mask = y_pred == 1
plt.scatter(X[normal_mask, 0], X[normal_mask, 1], c='blue', s=20, label='Normal')
plt.scatter(X[~normal_mask, 0], X[~normal_mask, 1], c='red', s=50, 
            marker='*', label='Anomaly')

plt.title('Isolation Forest Anomaly Detection')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('isolation_forest.png', dpi=150)
plt.show()
```

### Anomaly Score Distribution

```python
# Visualize anomaly score distribution
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Score distribution
axes[0].hist(scores[y_pred == 1], bins=30, alpha=0.7, label='Normal', color='blue')
axes[0].hist(scores[y_pred == -1], bins=15, alpha=0.7, label='Anomaly', color='red')
axes[0].axvline(x=0, color='black', linestyle='--', label='Threshold')
axes[0].set_xlabel('Decision Score')
axes[0].set_ylabel('Count')
axes[0].set_title('Decision Score Distribution')
axes[0].legend()

# Scatter with score coloring
scatter = axes[1].scatter(X[:, 0], X[:, 1], c=anomaly_scores, 
                          cmap='RdYlBu_r', s=20)
plt.colorbar(scatter, ax=axes[1], label='Anomaly Score')
axes[1].set_title('Points Colored by Anomaly Score')

plt.tight_layout()
plt.show()
```

### From Scratch (Simplified)

```python
import numpy as np

class IsolationTreeNode:
    def __init__(self, depth=0):
        self.depth = depth
        self.left = None
        self.right = None
        self.split_feature = None
        self.split_value = None
        self.size = 0  # For external nodes
    
    @property
    def is_leaf(self):
        return self.left is None and self.right is None

def build_itree(X, depth=0, max_depth=10):
    """Build a single isolation tree."""
    node = IsolationTreeNode(depth)
    n_samples, n_features = X.shape
    
    if n_samples <= 1 or depth >= max_depth:
        node.size = n_samples
        return node
    
    # Random feature and split value
    feature = np.random.randint(n_features)
    min_val, max_val = X[:, feature].min(), X[:, feature].max()
    
    if min_val == max_val:
        node.size = n_samples
        return node
    
    split_val = np.random.uniform(min_val, max_val)
    
    node.split_feature = feature
    node.split_value = split_val
    
    left_mask = X[:, feature] < split_val
    node.left = build_itree(X[left_mask], depth + 1, max_depth)
    node.right = build_itree(X[~left_mask], depth + 1, max_depth)
    
    return node

def path_length(x, node):
    """Compute path length for a single point in an isolation tree."""
    if node.is_leaf:
        # Add adjustment for unsplit data
        return node.depth + _c(node.size)
    
    if x[node.split_feature] < node.split_value:
        return path_length(x, node.left)
    else:
        return path_length(x, node.right)

def _c(n):
    """Average path length of unsuccessful search in BST."""
    if n <= 1:
        return 0
    return 2 * (np.log(n - 1) + 0.5772156649) - 2 * (n - 1) / n

def isolation_forest_score(X, n_trees=100, subsample_size=256):
    """Compute anomaly scores using Isolation Forest."""
    n_samples = X.shape[0]
    psi = min(subsample_size, n_samples)
    max_depth = int(np.ceil(np.log2(psi)))
    
    # Build forest
    trees = []
    for _ in range(n_trees):
        idx = np.random.choice(n_samples, size=psi, replace=False)
        tree = build_itree(X[idx], max_depth=max_depth)
        trees.append(tree)
    
    # Score each point
    scores = np.zeros(n_samples)
    for i, x in enumerate(X):
        avg_path = np.mean([path_length(x, tree) for tree in trees])
        scores[i] = 2 ** (-avg_path / _c(psi))
    
    return scores

# Test
scores_custom = isolation_forest_score(X, n_trees=100, subsample_size=256)
print(f"Top 5 anomaly scores: {np.sort(scores_custom)[-5:]}")
```

---

## 🎛️ Key Parameters

```
┌─────────────────────┬─────────┬─────────────────────────────────────┐
│ Parameter           │ Default │ Effect                              │
├─────────────────────┼─────────┼─────────────────────────────────────┤
│ n_estimators        │ 100     │ Number of trees in the forest       │
│                     │         │ More → more stable scores           │
│                     │         │ Diminishing returns after ~100      │
├─────────────────────┼─────────┼─────────────────────────────────────┤
│ max_samples          │ 'auto'  │ Subsample size ψ (default 256)     │
│                     │         │ Larger → better but slower          │
│                     │         │ 256 is usually sufficient           │
├─────────────────────┼─────────┼─────────────────────────────────────┤
│ contamination       │ 'auto'  │ Expected fraction of anomalies     │
│                     │         │ Sets the decision threshold         │
│                     │         │ 'auto' uses offset from paper       │
├─────────────────────┼─────────┼─────────────────────────────────────┤
│ max_features         │ 1.0     │ Fraction of features per tree      │
│                     │         │ <1.0 → more randomness              │
└─────────────────────┴─────────┴─────────────────────────────────────┘
```

---

## ✅ Advantages and ❌ Limitations

```
┌──────────────────────────────────────────────────────────────┐
│ ADVANTAGES:                                                  │
│ ✅ Linear time complexity O(t·ψ·log(ψ))                     │
│ ✅ No distance/density computation needed                    │
│ ✅ Handles high-dimensional data well                        │
│ ✅ Works without assuming data distribution                  │
│ ✅ Small memory footprint (subsampling)                      │
│ ✅ Few hyperparameters to tune                               │
│ ✅ Naturally handles mixed feature types                     │
│                                                              │
│ LIMITATIONS:                                                 │
│ ❌ Assumes anomalies are "few and different"                 │
│ ❌ Axis-aligned splits can miss some anomalies               │
│ ❌ Struggles with local anomalies in dense regions           │
│ ❌ Contamination parameter needs to be set                   │
│ ❌ Not ideal for time-series anomalies                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Algorithm** | Isolation Forest (Liu et al., 2008) |
| **Approach** | Isolate anomalies (not model normal) |
| **Key Idea** | Anomalies have shorter path lengths in random trees |
| **Score** | s(x) = 2^(-E[h(x)]/c(ψ)); close to 1 = anomaly |
| **Complexity** | O(t · ψ · log ψ) — linear in N |
| **Key Params** | `n_estimators`, `contamination`, `max_samples` |
| **Strengths** | Fast, scalable, no distribution assumption |
| **sklearn** | `IsolationForest(contamination=0.05)` |

---

## ❓ Revision Questions

1. **Explain the core intuition behind Isolation Forest. Why do anomalies have shorter path lengths?**
   Relate to the properties of anomalies (few, different) and random splitting.

2. **What is the anomaly score formula s(x, ψ) = 2^(-E[h(x)]/c(ψ))? What does c(ψ) represent and why is it needed?**
   Discuss normalization and the connection to binary search tree analysis.

3. **Why does Isolation Forest use subsampling (ψ=256)? What problems does this solve compared to using all data?**
   Consider swamping, masking, and computational efficiency.

4. **Compare Isolation Forest with statistical methods (Z-score, Mahalanobis). When would you choose each?**
   Consider assumptions, dimensionality, and types of anomalies.

5. **The `contamination` parameter is set to 0.05. What happens if the true contamination is 0.01? Or 0.20?**
   Discuss the effect on precision, recall, and the decision threshold.

6. **Build an isolation tree by hand for the points: (1,1), (2,2), (3,3), (10,10). Show 2-3 random splits and compute path lengths.**
   Demonstrate that (10,10) is isolated faster than the clustered points.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Statistical Methods](./02-statistical-methods.md) | [📂 Unsupervised Learning](../README.md) | [One-Class SVM →](./04-one-class-svm.md) |

---

> **Key Takeaway:** Isolation Forest flips the anomaly detection paradigm — instead of modeling normal behavior, it directly isolates anomalies using random partitioning. Anomalies, being few and different, require fewer splits to isolate (shorter paths). This makes Isolation Forest fast, scalable, and assumption-free — the go-to method for general-purpose anomaly detection.
