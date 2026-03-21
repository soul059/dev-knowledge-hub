# 📘 7.4 — Pruning Techniques

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 4 of 7 — Overfitting, pre-pruning, post-pruning, cost-complexity pruning, finding optimal alpha, and practical Python examples.*

---

## 📑 Table of Contents

1. [Why Prune? — The Overfitting Problem](#1-why-prune--the-overfitting-problem)
2. [Pre-Pruning (Early Stopping)](#2-pre-pruning-early-stopping)
3. [Post-Pruning Overview](#3-post-pruning-overview)
4. [Reduced Error Pruning (REP)](#4-reduced-error-pruning-rep)
5. [Cost-Complexity Pruning (CCP) — Minimal Cost-Complexity](#5-cost-complexity-pruning-ccp--minimal-cost-complexity)
6. [The Alpha Parameter (ccp_alpha)](#6-the-alpha-parameter-ccp_alpha)
7. [Finding Optimal Alpha with Cross-Validation](#7-finding-optimal-alpha-with-cross-validation)
8. [Pruned vs Unpruned — Visual Comparison](#8-pruned-vs-unpruned--visual-comparison)
9. [Python — Complete Pruning Pipeline](#9-python--complete-pruning-pipeline)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Why Prune? — The Overfitting Problem

### 1.1 The Problem

An unpruned decision tree will keep splitting until every leaf is pure (or until it runs out of features). This creates a tree that:

- **Memorises** the training data (including noise).
- **Fails to generalise** — poor performance on unseen data.
- Is **unnecessarily complex** — hundreds or thousands of nodes.

```
  Error
    ↑
    │  ·                              
    │   ·    ·  ·  ·  ·  ·  ·  ·     Test Error
    │    ·  ·                    ·    (gets worse after optimal point)
    │     ··
    │      ·                         ← Sweet spot
    │       ·
    │  · · · · · · · · · · · · · ·   Training Error
    │                                (keeps decreasing)
    └──────────────────────────────── Tree Complexity →
              ↑
         Optimal complexity
         (pruned tree)
```

### 1.2 Overfitting Illustrated

```
  Unpruned tree (depth=20):          Pruned tree (depth=4):
  ┌───────────────────────┐         ┌───────────────────────┐
  │ Train Acc: 100.0%     │         │ Train Acc: 95.2%      │
  │ Test Acc:   72.3%     │         │ Test Acc:  91.8%      │
  │ Nodes: 847            │         │ Nodes: 23             │
  │ Leaves: 424           │         │ Leaves: 12            │
  │ Overfitting!          │         │ Generalises well!     │
  └───────────────────────┘         └───────────────────────┘
```

### 1.3 Two Strategies to Combat Overfitting

| Strategy | When Applied | How |
|----------|-------------|-----|
| **Pre-pruning** (early stopping) | During tree construction | Stop splitting early based on criteria |
| **Post-pruning** | After tree is fully grown | Remove branches that don't improve generalisation |

---

## 2. Pre-Pruning (Early Stopping)

### 2.1 Concept

Pre-pruning stops the tree from growing too large by **imposing constraints during construction**. The tree never reaches full depth.

### 2.2 Techniques

| Technique | sklearn Parameter | Description |
|-----------|-------------------|-------------|
| **Max depth** | `max_depth` | Stop splitting when the tree reaches a fixed depth |
| **Min samples to split** | `min_samples_split` | Don't split a node if it has fewer than *n* samples |
| **Min samples per leaf** | `min_samples_leaf` | Don't create a split if any child would have fewer than *n* samples |
| **Max leaf nodes** | `max_leaf_nodes` | Grow the tree in best-first fashion, stopping at *n* leaves |
| **Min impurity decrease** | `min_impurity_decrease` | Don't split if the impurity decrease is below a threshold |
| **Max features** | `max_features` | Consider only a subset of features per split (adds randomness) |

### 2.3 How Each Technique Works

#### Max Depth

```
max_depth = 2:

      [Root]              ← depth 0
      /    \
   [N1]    [N2]           ← depth 1
   / \      / \
 [L1][L2] [L3][L4]        ← depth 2 → STOP (all become leaves)
```

#### Min Samples Split

```
min_samples_split = 20:

      [100 samples]       ← 100 ≥ 20 → split ✅
       /         \
   [60 samples] [40 samples]  ← 60 ≥ 20, 40 ≥ 20 → split ✅
    /    \        /    \
 [35] [25]    [15] [25]      ← 15 < 20 → STOP ❌ (becomes leaf)
```

#### Min Impurity Decrease

```
min_impurity_decrease = 0.01:

Node Gini = 0.48
Best split → weighted child Gini = 0.475
Decrease = 0.48 − 0.475 = 0.005 < 0.01 → STOP ❌
```

### 2.4 Pros and Cons of Pre-Pruning

| Pros | Cons |
|------|------|
| Computationally efficient — avoid building unnecessary branches | May stop too early (the **horizon effect**) |
| Easy to implement and tune | A bad split early on might enable a great split later |
| Directly controls model complexity | Requires careful hyperparameter tuning |

### 2.5 The Horizon Effect

```
Pre-pruning might miss this:

      [Weak split — low gain]        ← Pre-pruning stops here
       /                  \
   [Excellent split!]  [Excellent split!]   ← Never reached
    /        \            /        \
  [Pure]   [Pure]      [Pure]   [Pure]
```

The first split has low information gain, so pre-pruning stops. But the children would have had excellent splits. **Post-pruning** avoids this problem because it builds the full tree first.

---

## 3. Post-Pruning Overview

### 3.1 Concept

Post-pruning builds the **full tree first**, then removes branches that don't improve (or hurt) generalisation performance.

### 3.2 General Process

```
1. Build a COMPLETE, unpruned tree T_max
2. Generate a sequence of pruned subtrees: T_0 ⊃ T_1 ⊃ T_2 ⊃ ... ⊃ T_root
3. Evaluate each subtree using validation data or cross-validation
4. Select the subtree with best generalisation performance
```

### 3.3 Two Main Post-Pruning Methods

| Method | Used By | How |
|--------|---------|-----|
| **Reduced Error Pruning (REP)** | General / C4.5 variant | Use a validation set to decide which branches to remove |
| **Cost-Complexity Pruning (CCP)** | CART / sklearn | Add a complexity penalty; find optimal penalty via CV |

---

## 4. Reduced Error Pruning (REP)

### 4.1 Algorithm

```
function REDUCED_ERROR_PRUNING(tree, validation_data):
    repeat:
        for each internal node n (bottom-up):
            temporarily replace n's subtree with a leaf
            evaluate accuracy on validation_data
            if accuracy does not decrease:
                permanently prune (replace subtree with leaf)
    until no more pruning improves validation accuracy
```

### 4.2 Worked Example

```
Original tree:                    After REP:

      [A ≤ 5]                        [A ≤ 5]
      /     \                        /     \
   [B ≤ 3]  [C ≤ 7]              Leaf:1  Leaf:0
   /    \     /    \
 L:1   L:0  L:0   L:0

Validation accuracy:
  Original: 82%
  Prune [C ≤ 7] → replace with Leaf:0 → accuracy: 84% → PRUNE ✅
  Prune [B ≤ 3] → replace with Leaf:1 → accuracy: 85% → PRUNE ✅
  
  Final pruned tree has just the root with two leaves.
```

### 4.3 Pros and Cons

| Pros | Cons |
|------|------|
| Simple and intuitive | Requires a separate validation set (reduces training data) |
| Guaranteed to not decrease validation performance | Order of pruning can matter |
| Fast | May under-prune if validation set is small |

---

## 5. Cost-Complexity Pruning (CCP) — Minimal Cost-Complexity

### 5.1 Concept

**Cost-Complexity Pruning (CCP)**, also called **weakest link pruning** or **minimal cost-complexity pruning**, is the pruning method used by CART and implemented in sklearn.

The key idea: **penalise tree complexity** just like regularisation in linear models.

### 5.2 The Cost-Complexity Measure

For a tree *T*, define:

```
Rα(T) = R(T) + α × |T̃|
```

Where:
- **R(T)** = total weighted impurity (misclassification rate) of the tree
- **|T̃|** = number of leaf nodes in the tree (a measure of complexity)
- **α** (alpha) = the complexity parameter (≥ 0)

```
  Rα(T) = Training error + α × Number of leaves
           ─────────────   ──────────────────────
           Fit to data     Penalty for complexity
```

### 5.3 How Alpha Controls Pruning

```
α = 0:       No penalty for complexity → full tree (unpruned)
α = small:   Slight penalty → prune only the weakest branches
α = large:   Heavy penalty → aggressive pruning → very small tree
α = ∞:       Maximum penalty → tree is just the root (single leaf)
```

### 5.4 The Pruning Sequence

CCP generates a nested sequence of subtrees by progressively pruning the **weakest link** — the internal node whose removal causes the smallest increase in training error per leaf removed.

```
T₀ ⊃ T₁ ⊃ T₂ ⊃ ... ⊃ Tₖ = {root}

where:
  T₀ = full tree (unpruned)
  Tₖ = root only (just one leaf)
  
Each Tᵢ is optimal for some range of α values.
```

### 5.5 Effective Alpha for a Node

The **effective alpha** of an internal node *t* is:

```
                  R(t) − R(Tₜ)
α_eff(t) = ─────────────────────
              |T̃ₜ| − 1
```

Where:
- R(t) = impurity if we replace the subtree rooted at *t* with a single leaf
- R(Tₜ) = total impurity of the subtree rooted at *t*
- |T̃ₜ| = number of leaves in the subtree rooted at *t*

**Interpretation:** α_eff(t) is the "cost" of keeping the subtree at *t*. The node with the **smallest** α_eff is the **weakest link** — pruning it gives the most complexity reduction per unit of accuracy loss.

### 5.6 Algorithm

```
function COST_COMPLEXITY_PRUNING(T_max):
    T = T_max
    sequence = [T]
    alphas = [0]
    
    while T has more than 1 leaf:
        for each internal node t in T:
            compute α_eff(t)
        
        t_weakest = node with smallest α_eff
        
        Replace subtree at t_weakest with a leaf
        (use majority class or mean value)
        
        sequence.append(T)
        alphas.append(α_eff(t_weakest))
    
    return sequence, alphas
```

### 5.7 Worked Example

```
Full tree T₀:           α_eff for each internal node:

      [A]                  α_eff(A) = (0.30 − 0.10) / (4 − 1) = 0.0667
     /   \                 α_eff(B) = (0.15 − 0.10) / (2 − 1) = 0.0500  ← weakest!
   [B]   [C]               α_eff(C) = (0.12 − 0.00) / (2 − 1) = 0.1200
   / \    / \
  L1 L2  L3  L4

Step 1: Prune B (weakest link), α₁ = 0.05

      [A]
     /   \
   Leaf  [C]    ← T₁ (3 leaves → 3 nodes)
          / \
        L3  L4

Step 2: Compute new α_eff values
  α_eff(A) = recalculated
  α_eff(C) = 0.1200
  → Prune the one with smaller α_eff

Step 3: Continue until only the root remains → T_final = {root}

Pruning sequence: T₀, T₁, T₂, T₃ (root only)
Alpha sequence:   0,  0.05, ..., large
```

---

## 6. The Alpha Parameter (ccp_alpha)

### 6.1 In sklearn

The `ccp_alpha` parameter in `DecisionTreeClassifier` and `DecisionTreeRegressor` controls cost-complexity pruning:

```python
from sklearn.tree import DecisionTreeClassifier

# No pruning (default)
clf_full = DecisionTreeClassifier(ccp_alpha=0.0)

# Moderate pruning
clf_pruned = DecisionTreeClassifier(ccp_alpha=0.02)

# Aggressive pruning
clf_aggressive = DecisionTreeClassifier(ccp_alpha=0.1)
```

### 6.2 Getting the Pruning Path

sklearn provides `cost_complexity_pruning_path()` to get all effective alphas and corresponding impurities:

```python
clf = DecisionTreeClassifier(random_state=42)
clf.fit(X_train, y_train)

# Get pruning path
path = clf.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas        # array of alpha values
impurities = path.impurities        # total impurity at each alpha

print(f"Number of pruning steps: {len(ccp_alphas)}")
print(f"Alpha range: [{ccp_alphas[0]:.4f}, {ccp_alphas[-1]:.4f}]")
```

### 6.3 Visualising the Pruning Path

```
  Total      │ ·
  Impurity   │   ·
             │     ·
             │       · · ·
             │              · · · · ·
             │                        · · · · · · · · ·
             └──────────────────────────────────────────── α →
               0    0.01   0.02   0.03   0.04   0.05

  As α increases → more pruning → higher impurity → simpler tree
```

---

## 7. Finding Optimal Alpha with Cross-Validation

### 7.1 The Process

```
1. Build full tree on training data
2. Get the pruning path → list of candidate α values
3. For each α:
     - Train a tree with ccp_alpha = α
     - Evaluate using k-fold cross-validation
4. Select α that maximises mean CV score
5. (Optional) Apply 1-SE rule: pick largest α within 1 SE of the best score
```

### 7.2 The 1-SE Rule

The **1 standard error rule** selects the simplest model (largest α) whose performance is within one standard error of the best model. This adds a preference for simplicity.

```
CV Score
  ↑
  │     ·  ·  ·  ·  ·
  │   ·                ·
  │  ·                   ·  ·           ← 1 SE band
  │ ·  ──────────────────────── ·  ·  
  │·                                ·  ·
  │                                      · 
  └──────────────────────────────────────── α →
        ↑                   ↑
    α_best           α_1SE (simpler, preferred)
```

### 7.3 Python Implementation

```python
import numpy as np
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import cross_val_score

def find_optimal_alpha(X_train, y_train, cv=5):
    """Find the optimal ccp_alpha using cross-validation."""
    
    # Step 1: Get pruning path
    clf = DecisionTreeClassifier(random_state=42)
    clf.fit(X_train, y_train)
    path = clf.cost_complexity_pruning_path(X_train, y_train)
    ccp_alphas = path.ccp_alphas
    
    # Step 2: Cross-validate each alpha
    mean_scores = []
    std_scores = []
    
    for alpha in ccp_alphas:
        clf_temp = DecisionTreeClassifier(ccp_alpha=alpha, random_state=42)
        scores = cross_val_score(clf_temp, X_train, y_train, cv=cv,
                                 scoring='accuracy')
        mean_scores.append(scores.mean())
        std_scores.append(scores.std())
    
    mean_scores = np.array(mean_scores)
    std_scores = np.array(std_scores)
    
    # Step 3: Best alpha
    best_idx = np.argmax(mean_scores)
    best_alpha = ccp_alphas[best_idx]
    
    # Step 4: 1-SE rule
    threshold = mean_scores[best_idx] - std_scores[best_idx]
    se_candidates = np.where(mean_scores >= threshold)[0]
    alpha_1se = ccp_alphas[se_candidates[-1]]  # largest alpha above threshold
    
    return best_alpha, alpha_1se, ccp_alphas, mean_scores, std_scores
```

---

## 8. Pruned vs Unpruned — Visual Comparison

### 8.1 Tree Complexity

```
  Unpruned Tree (α = 0):                Pruned Tree (α = 0.02):

          [X₁ ≤ 5.5]                          [X₁ ≤ 5.5]
         /           \                       /           \
    [X₂ ≤ 3.1]   [X₃ ≤ 1.8]           [X₂ ≤ 3.1]    Leaf: B
    /       \       /       \           /       \
  [X₄≤2]  [X₅≤7] [X₆≤4]  [X₇≤9]    Leaf: A  Leaf: B
  / \    / \    / \    / \
 L  L   L  L  L  L   L  L

  16 leaves, depth = 4                  3 leaves, depth = 2
  Train: 100%, Test: 78%               Train: 93%, Test: 89%
```

### 8.2 Decision Boundary Comparison

```
  Unpruned (overfits):               Pruned (smooth):

  X₂ ↑                              X₂ ↑
     │ ·B·B│····│                       │ · B B B B B B
     │ ·B··│·A··│                       │ · B B B B B B
     │─────┼────┤                       │ B B B B B B B
     │ A A │B│B │                       │─────────────────
     │ A A │·│A·│                       │ A A A A A A A
     │ A A A A A│                       │ A A A A A A A
     └──────────── X₁ →                 └──────────────── X₁ →

  Jagged boundaries = noise           Smooth boundaries = signal
```

### 8.3 Performance Comparison (Typical)

| Metric | Unpruned | Pre-Pruned | Post-Pruned (CCP) |
|--------|----------|------------|---------------------|
| Training accuracy | ~100% | ~93% | ~95% |
| Test accuracy | ~75% | ~88% | ~90% |
| Number of leaves | ~200 | ~15 | ~12 |
| Tree depth | ~20 | 5 | 6 |
| Interpretability | Poor | Good | Good |

---

## 9. Python — Complete Pruning Pipeline

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.model_selection import train_test_split, cross_val_score

# ── Load data ──
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.3, random_state=42
)

# ──────────────────────────────────────────────────────
#  1. Unpruned Tree (baseline)
# ──────────────────────────────────────────────────────
clf_full = DecisionTreeClassifier(random_state=42)
clf_full.fit(X_train, y_train)
print("=== UNPRUNED TREE ===")
print(f"  Depth:    {clf_full.get_depth()}")
print(f"  Leaves:   {clf_full.get_n_leaves()}")
print(f"  Train acc: {clf_full.score(X_train, y_train):.4f}")
print(f"  Test acc:  {clf_full.score(X_test, y_test):.4f}")

# ──────────────────────────────────────────────────────
#  2. Pre-Pruned Tree
# ──────────────────────────────────────────────────────
clf_pre = DecisionTreeClassifier(
    max_depth=4, min_samples_leaf=10, random_state=42
)
clf_pre.fit(X_train, y_train)
print("\n=== PRE-PRUNED TREE ===")
print(f"  Depth:    {clf_pre.get_depth()}")
print(f"  Leaves:   {clf_pre.get_n_leaves()}")
print(f"  Train acc: {clf_pre.score(X_train, y_train):.4f}")
print(f"  Test acc:  {clf_pre.score(X_test, y_test):.4f}")

# ──────────────────────────────────────────────────────
#  3. Cost-Complexity Pruning (Post-Pruning)
# ──────────────────────────────────────────────────────

# Get the pruning path
path = clf_full.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas[:-1]  # exclude the trivial root-only tree

# Cross-validate each alpha
cv_means = []
cv_stds = []
train_scores = []
test_scores_list = []
depths = []
n_leaves = []

for alpha in ccp_alphas:
    clf_temp = DecisionTreeClassifier(ccp_alpha=alpha, random_state=42)
    scores = cross_val_score(clf_temp, X_train, y_train, cv=5,
                             scoring='accuracy')
    cv_means.append(scores.mean())
    cv_stds.append(scores.std())
    
    clf_temp.fit(X_train, y_train)
    train_scores.append(clf_temp.score(X_train, y_train))
    test_scores_list.append(clf_temp.score(X_test, y_test))
    depths.append(clf_temp.get_depth())
    n_leaves.append(clf_temp.get_n_leaves())

cv_means = np.array(cv_means)
cv_stds = np.array(cv_stds)

# Find best alpha
best_idx = np.argmax(cv_means)
best_alpha = ccp_alphas[best_idx]

# 1-SE rule
threshold = cv_means[best_idx] - cv_stds[best_idx]
se_candidates = np.where(cv_means >= threshold)[0]
alpha_1se = ccp_alphas[se_candidates[-1]]

print(f"\n=== COST-COMPLEXITY PRUNING ===")
print(f"  Best alpha:        {best_alpha:.6f}")
print(f"  1-SE alpha:        {alpha_1se:.6f}")

# Final pruned tree
clf_pruned = DecisionTreeClassifier(ccp_alpha=alpha_1se, random_state=42)
clf_pruned.fit(X_train, y_train)
print(f"  Depth:             {clf_pruned.get_depth()}")
print(f"  Leaves:            {clf_pruned.get_n_leaves()}")
print(f"  Train acc:         {clf_pruned.score(X_train, y_train):.4f}")
print(f"  Test acc:          {clf_pruned.score(X_test, y_test):.4f}")

# ──────────────────────────────────────────────────────
#  4. Visualise
# ──────────────────────────────────────────────────────
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Plot 1: CV accuracy vs alpha
ax = axes[0, 0]
ax.errorbar(ccp_alphas, cv_means, yerr=cv_stds, fmt='o-', markersize=3,
            label='Mean CV accuracy')
ax.axvline(best_alpha, color='green', linestyle='--', label=f'Best α={best_alpha:.4f}')
ax.axvline(alpha_1se, color='red', linestyle='--', label=f'1-SE α={alpha_1se:.4f}')
ax.set_xlabel('ccp_alpha')
ax.set_ylabel('Accuracy')
ax.set_title('Cross-Validation Accuracy vs Alpha')
ax.legend(fontsize=8)

# Plot 2: Train vs Test accuracy
ax = axes[0, 1]
ax.plot(ccp_alphas, train_scores, 'o-', markersize=3, label='Train')
ax.plot(ccp_alphas, test_scores_list, 'o-', markersize=3, label='Test')
ax.axvline(alpha_1se, color='red', linestyle='--', label=f'1-SE α')
ax.set_xlabel('ccp_alpha')
ax.set_ylabel('Accuracy')
ax.set_title('Train vs Test Accuracy')
ax.legend()

# Plot 3: Tree depth vs alpha
ax = axes[1, 0]
ax.plot(ccp_alphas, depths, 'o-', markersize=3)
ax.axvline(alpha_1se, color='red', linestyle='--')
ax.set_xlabel('ccp_alpha')
ax.set_ylabel('Tree Depth')
ax.set_title('Tree Depth vs Alpha')

# Plot 4: Number of leaves vs alpha
ax = axes[1, 1]
ax.plot(ccp_alphas, n_leaves, 'o-', markersize=3)
ax.axvline(alpha_1se, color='red', linestyle='--')
ax.set_xlabel('ccp_alpha')
ax.set_ylabel('Number of Leaves')
ax.set_title('Number of Leaves vs Alpha')

plt.tight_layout()
plt.savefig('pruning_analysis.png', dpi=150)
plt.show()

# ──────────────────────────────────────────────────────
#  5. Print the pruned tree rules
# ──────────────────────────────────────────────────────
print("\n=== PRUNED TREE RULES ===")
print(export_text(clf_pruned, feature_names=list(data.feature_names)))

# ──────────────────────────────────────────────────────
#  6. Comparison Summary
# ──────────────────────────────────────────────────────
print("\n=== COMPARISON SUMMARY ===")
print(f"{'Model':<20} {'Depth':<8} {'Leaves':<8} {'Train':<10} {'Test':<10}")
print("-" * 56)
print(f"{'Unpruned':<20} {clf_full.get_depth():<8} {clf_full.get_n_leaves():<8} "
      f"{clf_full.score(X_train, y_train):<10.4f} {clf_full.score(X_test, y_test):<10.4f}")
print(f"{'Pre-pruned':<20} {clf_pre.get_depth():<8} {clf_pre.get_n_leaves():<8} "
      f"{clf_pre.score(X_train, y_train):<10.4f} {clf_pre.score(X_test, y_test):<10.4f}")
print(f"{'CCP (1-SE)':<20} {clf_pruned.get_depth():<8} {clf_pruned.get_n_leaves():<8} "
      f"{clf_pruned.score(X_train, y_train):<10.4f} {clf_pruned.score(X_test, y_test):<10.4f}")
```

---

## 10. Applications

| Domain | How Pruning Helps |
|--------|-------------------|
| **Medical diagnosis** | Pruned trees are safer — fewer spurious rules based on noise |
| **Credit scoring** | Regulatory requirement: models must be interpretable → simpler trees preferred |
| **Manufacturing** | Pruned trees provide robust quality control rules that don't change with every batch |
| **Ensemble methods** | Random Forests use many unpruned trees (diversity). Gradient Boosting uses shallow pre-pruned trees. |
| **Edge deployment** | Pruned trees fit in memory-constrained devices (IoT, embedded systems) |

---

## 11. Summary Table

| Concept | Key Points |
|---------|-----------|
| **Overfitting** | Unpruned trees memorise noise; high train accuracy, low test accuracy |
| **Pre-pruning** | Stop early via max_depth, min_samples_split, min_samples_leaf, etc. |
| **Horizon effect** | Pre-pruning may miss good splits hidden behind weak initial splits |
| **Post-pruning** | Build full tree first, then remove branches |
| **REP** | Prune branches that don't help on a validation set |
| **CCP** | Rα(T) = R(T) + α × \|T̃\| — penalise complexity |
| **Effective alpha** | α_eff(t) = (R(t) − R(Tₜ)) / (\|T̃ₜ\| − 1) — "cost" of keeping a subtree |
| **Weakest link** | Node with smallest α_eff is pruned first |
| **ccp_alpha** | sklearn parameter; 0 = no pruning, larger = more pruning |
| **Optimal alpha** | Found via cross-validation; 1-SE rule prefers simpler models |

---

## 12. Revision Questions

1. **Explain** why an unpruned decision tree tends to overfit. What happens to the bias and variance as tree depth increases?

2. **List** four pre-pruning hyperparameters in sklearn and describe what each controls.

3. **Define** the cost-complexity measure Rα(T). What does each term represent?

4. A subtree rooted at node *t* has 6 leaves and a total impurity of 0.08. If replacing it with a single leaf would give an impurity of 0.20, what is the **effective alpha** α_eff(t)?

5. **Describe** the 1-SE rule and why it is useful when selecting the optimal alpha.

6. **Compare** pre-pruning and post-pruning. When would you prefer one over the other?

---

## 13. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.3 Building a Decision Tree](03-building-decision-tree.md) | **7.4 Pruning Techniques** | [7.5 Advantages & Limitations →](05-advantages-and-limitations.md) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | **Pruning Techniques** (you are here) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"Pruning is the art of knowing which branches bear fruit and which are just dead wood."*
