# 📘 7.3 — Building a Decision Tree

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 3 of 7 — CART, ID3, C4.5 algorithms, recursive partitioning, stopping criteria, and a complete worked example building a tree from scratch.*

---

## 📑 Table of Contents

1. [Overview — How a Decision Tree Is Built](#1-overview--how-a-decision-tree-is-built)
2. [The Greedy Approach](#2-the-greedy-approach)
3. [CART Algorithm (Classification And Regression Trees)](#3-cart-algorithm-classification-and-regression-trees)
4. [ID3 Algorithm](#4-id3-algorithm)
5. [C4.5 Algorithm](#5-c45-algorithm)
6. [Algorithm Comparison Table](#6-algorithm-comparison-table)
7. [Stopping Criteria](#7-stopping-criteria)
8. [Complete Worked Example — Building a Tree Step by Step](#8-complete-worked-example--building-a-tree-step-by-step)
9. [ASCII Visualisation of the Growing Tree](#9-ascii-visualisation-of-the-growing-tree)
10. [Python — Building with sklearn & From Scratch](#10-python--building-with-sklearn--from-scratch)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Overview — How a Decision Tree Is Built

Building a decision tree is a **top-down, recursive process** called **recursive partitioning** (also known as **top-down induction of decision trees — TDIDT**).

### High-Level Algorithm

```
function BUILD_TREE(data, features):
    1. If stopping criterion met:
           return LEAF(majority_class or mean_target)
    
    2. Find the BEST SPLIT:
           - For each feature in features:
               - For each possible split point:
                   - Compute impurity reduction (Gini, Entropy, MSE)
           - Select the split with maximum impurity reduction
    
    3. Split data into subsets based on best split
    
    4. Recursively:
           left_child  = BUILD_TREE(left_subset,  features)
           right_child = BUILD_TREE(right_subset, features)
    
    5. Return NODE(best_split, left_child, right_child)
```

### Visual Process

```
  Iteration 1:  Full dataset → find best split → create root
  
       ┌──────────────────┐
       │ ALL DATA (N=100) │
       │ Best split: X₁≤5 │
       └────────┬─────────┘
                │
       ┌────────┴────────┐
       │                 │
  ┌────┴─────┐    ┌─────┴────┐
  │ Left     │    │ Right    │
  │ (N=60)   │    │ (N=40)   │
  └──────────┘    └──────────┘

  Iteration 2:  Recurse on left subset → find best split

       ┌──────────────────┐
       │ ALL DATA (N=100) │
       │ X₁ ≤ 5           │
       └────────┬─────────┘
                │
       ┌────────┴────────┐
       │                 │
  ┌────┴─────┐    ┌─────┴────┐
  │ X₂ ≤ 3  │    │ Right    │
  │ (N=60)  │    │ (N=40)   │
  └──┬───┬──┘    └──────────┘
     │   │
   ┌─┴┐ ┌┴──┐
   │30│ │30 │
   └──┘ └───┘

  ... and so on until stopping criteria are met.
```

---

## 2. The Greedy Approach

Decision tree algorithms use a **greedy** strategy:

- At each node, they pick the **locally optimal split** (the one that maximises impurity reduction at that node).
- They do **not** consider the global impact of a split on future splits.
- This is a heuristic — finding the globally optimal tree is NP-complete.

### Why Greedy?

| Approach | Time Complexity | Quality |
|----------|----------------|---------|
| **Exhaustive search** (globally optimal) | O(2^n) — infeasible | Perfect |
| **Greedy** (locally optimal at each step) | O(n × m × log n) per node | Very good in practice |

Where *n* = number of samples, *m* = number of features.

### Consequence of Greediness

```
A greedy tree might miss this globally optimal structure:

        [Feature B ≤ 7]         ← Greedy chose Feature A first,
         /            \            missing this better global tree
      [Pure]       [Feature C ≤ 2]
                    /          \
                 [Pure]      [Pure]
```

This limitation motivates ensemble methods (Random Forests, Gradient Boosting) which combine many trees to overcome individual tree limitations.

---

## 3. CART Algorithm (Classification And Regression Trees)

### 3.1 Overview

**CART** (Breiman et al., 1984) is the most widely used decision tree algorithm. It is the algorithm behind `sklearn.tree.DecisionTreeClassifier` and `DecisionTreeRegressor`.

### 3.2 Key Properties

| Property | CART |
|----------|------|
| **Splitting** | Always **binary** splits |
| **Classification criterion** | Gini impurity (default) or Entropy |
| **Regression criterion** | MSE (mean squared error) or MAE |
| **Feature handling** | Continuous: threshold split. Categorical: subset split |
| **Pruning** | Cost-complexity pruning (post-pruning) |
| **Output** | A single binary tree |

### 3.3 CART — Classification

```
function CART_CLASSIFY(S, features):
    if all samples in S have the same class:
        return Leaf(that class)
    if no features left or other stopping criterion:
        return Leaf(majority class in S)
    
    best_feature, best_threshold = None, None
    best_gini = +∞
    
    for each feature f:
        for each candidate threshold t:
            S_left  = {x ∈ S : x[f] ≤ t}
            S_right = {x ∈ S : x[f] > t}
            
            weighted_gini = |S_left|/|S| × Gini(S_left)
                          + |S_right|/|S| × Gini(S_right)
            
            if weighted_gini < best_gini:
                best_gini = weighted_gini
                best_feature = f
                best_threshold = t
    
    Split S into S_left and S_right using best_feature ≤ best_threshold
    
    left_child  = CART_CLASSIFY(S_left, features)
    right_child = CART_CLASSIFY(S_right, features)
    
    return Node(best_feature, best_threshold, left_child, right_child)
```

### 3.4 CART — Regression

For regression, CART uses **variance reduction** (MSE) instead of Gini:

```
                  1    n
MSE(node) =  ────── Σ  (yᵢ − ȳ)²
                n   i=1

where ȳ = mean of y values in the node
```

The best split minimises the weighted MSE of the children:

```
MSE_split = (nₗ/n) × MSE(left) + (nᵣ/n) × MSE(right)
```

The prediction at a leaf is the **mean** of the target values in that leaf.

---

## 4. ID3 Algorithm

### 4.1 Overview

**ID3** (Iterative Dichotomiser 3, Quinlan 1986) was one of the first decision tree algorithms.

### 4.2 Key Properties

| Property | ID3 |
|----------|-----|
| **Splitting** | **Multi-way** (one branch per category) |
| **Criterion** | Information Gain (Entropy-based) |
| **Feature handling** | Categorical only (no continuous features) |
| **Pruning** | None (no built-in pruning) |
| **Feature reuse** | A feature used at a node is **removed** from consideration in subtrees |

### 4.3 Algorithm

```
function ID3(S, features):
    if all samples in S have the same class:
        return Leaf(that class)
    if features is empty:
        return Leaf(majority class in S)
    
    # Select feature with highest Information Gain
    best_feature = argmax_{f ∈ features} IG(S, f)
    
    Create a new node for best_feature
    remaining_features = features − {best_feature}
    
    for each value v of best_feature:
        S_v = {x ∈ S : x[best_feature] = v}
        if S_v is empty:
            add Leaf(majority class in S) as child
        else:
            add ID3(S_v, remaining_features) as child
    
    return node
```

### 4.4 Limitations of ID3

1. **No continuous features** — must be discretised beforehand.
2. **Biased toward high-cardinality features** — Information Gain favours features with many values.
3. **No pruning** — prone to overfitting.
4. **No missing value handling.**

---

## 5. C4.5 Algorithm

### 5.1 Overview

**C4.5** (Quinlan, 1993) is the successor to ID3, addressing its major limitations.

### 5.2 Key Improvements Over ID3

| Limitation in ID3 | Solution in C4.5 |
|-------------------|-------------------|
| No continuous features | Handles continuous features via threshold splits |
| Bias toward high-cardinality | Uses **Gain Ratio** instead of IG |
| No pruning | **Error-based pruning** (post-pruning) |
| No missing values | Handles missing values by distributing samples probabilistically |
| Multi-way categorical only | Still multi-way for categorical, binary for continuous |

### 5.3 Algorithm Sketch

```
function C4.5(S, features):
    if all samples have the same class:
        return Leaf(that class)
    if features is empty:
        return Leaf(majority class)
    
    for each feature f:
        if f is continuous:
            find best binary threshold t
            compute GainRatio for split f ≤ t
        else:
            compute GainRatio for multi-way split on f
    
    best_feature = feature with highest GainRatio
    
    Create node for best_feature
    Split S and recurse on each child
    
    # Post-pruning step
    Prune subtrees if pruning improves estimated error on unseen data
    
    return node
```

### 5.4 Missing Value Handling in C4.5

When a sample has a missing value for the split feature:
1. Distribute it to **all branches** with fractional weights.
2. The weight for branch *v* is proportional to the number of samples with that value.

```
Example: 10 samples at a node, feature "Outlook":
  - 5 Sunny, 3 Rainy, 2 missing
  
  The 2 missing samples are distributed:
  - 5/8 of each → Sunny branch
  - 3/8 of each → Rainy branch
```

---

## 6. Algorithm Comparison Table

| Feature | CART | ID3 | C4.5 |
|---------|------|-----|------|
| **Year** | 1984 | 1986 | 1993 |
| **Author** | Breiman et al. | Quinlan | Quinlan |
| **Split type** | Binary | Multi-way | Multi-way (cat.), Binary (cont.) |
| **Criterion** | Gini / MSE | Information Gain | Gain Ratio |
| **Continuous features** | ✅ Threshold split | ❌ Must discretise | ✅ Threshold split |
| **Categorical features** | Subset split (binary) | One branch per value | One branch per value |
| **Missing values** | Surrogate splits | ❌ Not handled | ✅ Probabilistic distribution |
| **Pruning** | Cost-complexity (post) | ❌ None | Error-based (post) |
| **Regression** | ✅ | ❌ | ❌ |
| **Implementation** | sklearn, R `rpart` | Custom, Weka | Weka (J48) |
| **Feature reuse** | Features can be reused | Feature removed after use | Feature can be reused |

---

## 7. Stopping Criteria

The recursive splitting must stop at some point. **Stopping criteria** determine when a node becomes a leaf.

### 7.1 Natural Stopping (Always Applied)

| Condition | Action |
|-----------|--------|
| All samples in the node belong to the **same class** | Create leaf with that class |
| **No features left** to split on | Create leaf with majority class |
| **No samples** in the node | Create leaf with parent's majority class |

### 7.2 Hyperparameter-Based Stopping (Pre-Pruning)

| Parameter (sklearn) | Description | Typical Values |
|---------------------|-------------|----------------|
| `max_depth` | Maximum depth of the tree | 3–20, or None |
| `min_samples_split` | Min samples required to split a node | 2–50 |
| `min_samples_leaf` | Min samples required in each leaf | 1–20 |
| `max_leaf_nodes` | Max number of leaf nodes | 10–100 |
| `min_impurity_decrease` | Min impurity decrease to justify a split | 0.0–0.05 |
| `max_features` | Max features to consider per split | 'sqrt', 'log2', int, float |

### 7.3 Effect of Each Parameter

```
max_depth controls:      How "deep" questions go
                         ┌──────────────────────────┐
  depth=1:  [root]       │  Very simple model       │
             / \         │  High bias, low variance  │
           [L] [L]       └──────────────────────────┘
  
  depth=10: [root]       ┌──────────────────────────┐
             / \         │  Very complex model       │
           ...  ...      │  Low bias, high variance  │
          (1024 leaves)  └──────────────────────────┘

min_samples_split controls:    Minimum crowd to ask a question
  
  min_samples_split=2:   Even 2 samples → split further (overfit risk)
  min_samples_split=50:  Need at least 50 samples to split (conservative)

min_samples_leaf controls:     Minimum crowd in each answer
  
  min_samples_leaf=1:    A leaf can have just 1 sample (overfit risk)
  min_samples_leaf=20:   Each leaf must represent ≥ 20 samples (smoother)
```

### 7.4 Choosing Stopping Criteria

Use **cross-validation** to tune these hyperparameters:

```python
from sklearn.model_selection import GridSearchCV
from sklearn.tree import DecisionTreeClassifier

param_grid = {
    'max_depth': [3, 5, 7, 10, 15, None],
    'min_samples_split': [2, 5, 10, 20],
    'min_samples_leaf': [1, 5, 10],
}

grid_search = GridSearchCV(
    DecisionTreeClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
)
grid_search.fit(X_train, y_train)
print(f"Best params: {grid_search.best_params_}")
print(f"Best CV accuracy: {grid_search.best_score_:.4f}")
```

---

## 8. Complete Worked Example — Building a Tree Step by Step

### 8.1 Dataset

We'll build a tree to predict whether a customer will **buy** a product (Yes/No):

| ID | Age   | Income | Student | Credit | Buys? |
|----|-------|--------|---------|--------|-------|
| 1  | Youth | High   | No      | Fair   | No    |
| 2  | Youth | High   | No      | Excellent | No |
| 3  | Middle| High   | No      | Fair   | Yes   |
| 4  | Senior| Medium | No      | Fair   | Yes   |
| 5  | Senior| Low    | Yes     | Fair   | Yes   |
| 6  | Senior| Low    | Yes     | Excellent | No |
| 7  | Middle| Low    | Yes     | Excellent | Yes |
| 8  | Youth | Medium | No      | Fair   | No    |
| 9  | Youth | Low    | Yes     | Fair   | Yes   |
| 10 | Senior| Medium | Yes     | Fair   | Yes   |
| 11 | Youth | Medium | Yes     | Excellent | Yes |
| 12 | Middle| Medium | No      | Excellent | Yes |
| 13 | Middle| High   | Yes     | Fair   | Yes   |
| 14 | Senior| Medium | No      | Excellent | No |

**Total:** 14 samples — 9 Yes, 5 No

### 8.2 Step 1 — Compute Parent Entropy

```
p(Yes) = 9/14 = 0.6429
p(No)  = 5/14 = 0.3571

H(parent) = −0.6429 × log₂(0.6429) − 0.3571 × log₂(0.3571)
          = −0.6429 × (−0.6376) − 0.3571 × (−1.4854)
          = 0.4099 + 0.5305
          = 0.9403 bits
```

### 8.3 Step 2 — Compute Information Gain for Each Feature

#### Feature: Age (Youth / Middle / Senior)

| Value | Samples | Yes | No | Entropy |
|-------|---------|-----|-----|---------|
| Youth | 5 | 2 | 3 | −(2/5)log₂(2/5) − (3/5)log₂(3/5) = 0.9710 |
| Middle | 4 | 4 | 0 | 0.0000 |
| Senior | 5 | 3 | 2 | −(3/5)log₂(3/5) − (2/5)log₂(2/5) = 0.9710 |

```
H_weighted(Age) = (5/14)(0.9710) + (4/14)(0.0000) + (5/14)(0.9710)
                = 0.3468 + 0 + 0.3468
                = 0.6936

IG(Age) = 0.9403 − 0.6936 = 0.2467 bits
```

#### Feature: Income (High / Medium / Low)

| Value | Samples | Yes | No | Entropy |
|-------|---------|-----|-----|---------|
| High | 4 | 2 | 2 | 1.0000 |
| Medium | 6 | 4 | 2 | 0.9183 |
| Low | 4 | 3 | 1 | 0.8113 |

```
H_weighted(Income) = (4/14)(1.0) + (6/14)(0.9183) + (4/14)(0.8113)
                   = 0.2857 + 0.3936 + 0.2318
                   = 0.9111

IG(Income) = 0.9403 − 0.9111 = 0.0292 bits
```

#### Feature: Student (Yes / No)

| Value | Samples | Yes | No | Entropy |
|-------|---------|-----|-----|---------|
| Yes | 7 | 6 | 1 | −(6/7)log₂(6/7)−(1/7)log₂(1/7) = 0.5917 |
| No | 7 | 3 | 4 | −(3/7)log₂(3/7)−(4/7)log₂(4/7) = 0.9852 |

```
H_weighted(Student) = (7/14)(0.5917) + (7/14)(0.9852)
                    = 0.2959 + 0.4926
                    = 0.7885

IG(Student) = 0.9403 − 0.7885 = 0.1518 bits
```

#### Feature: Credit (Fair / Excellent)

| Value | Samples | Yes | No | Entropy |
|-------|---------|-----|-----|---------|
| Fair | 8 | 6 | 2 | −(6/8)log₂(6/8)−(2/8)log₂(2/8) = 0.8113 |
| Excellent | 6 | 3 | 3 | 1.0000 |

```
H_weighted(Credit) = (8/14)(0.8113) + (6/14)(1.0000)
                   = 0.4636 + 0.4286
                   = 0.8922

IG(Credit) = 0.9403 − 0.8922 = 0.0481 bits
```

### 8.4 Step 3 — Select the Root

| Feature | Information Gain |
|---------|-----------------|
| **Age** | **0.2467** ← highest! |
| Income | 0.0292 |
| Student | 0.1518 |
| Credit | 0.0481 |

**Root node = Age** ✅

### 8.5 Step 4 — Split and Recurse

After splitting on Age:

```
                    [Age?]
                 /    |     \
           Youth   Middle    Senior
            /        |         \
     {5 samples}  {4 samples}  {5 samples}
      2Y, 3N       4Y, 0N      3Y, 2N
```

**Middle branch:** All 4 samples are "Yes" → **Leaf: Yes** (pure node, entropy = 0) ✅

**Youth branch:** (IDs 1, 2, 8, 9, 11) — 2 Yes, 3 No → Need to split further.

Remaining features: {Income, Student, Credit}

| Feature | IG (on Youth subset) |
|---------|---------------------|
| Income | 0.0196 |
| **Student** | **0.9710** |
| Credit | 0.0196 |

> IG(Student on Youth) = 0.9710 − 0 = 0.9710 (because each child is pure!)

Split Youth on **Student**:
- Student = No → IDs 1, 2, 8 → all "No" → **Leaf: No** ✅
- Student = Yes → IDs 9, 11 → all "Yes" → **Leaf: Yes** ✅

**Senior branch:** (IDs 4, 5, 6, 10, 14) — 3 Yes, 2 No → Need to split further.

Remaining features: {Income, Student, Credit}

| Feature | IG (on Senior subset) |
|---------|----------------------|
| Income | 0.9710 |
| Student | 0.0196 |
| **Credit** | **0.9710** |

Both Income and Credit have the same IG. We pick **Credit**:
- Credit = Fair → IDs 4, 5, 10 → all "Yes" → **Leaf: Yes** ✅
- Credit = Excellent → IDs 6, 14 → all "No" → **Leaf: No** ✅

### 8.6 Final Tree

```
                         [Age?]
                      /    |     \
               Youth    Middle    Senior
                /         |          \
          [Student?]    YES        [Credit?]
           /      \                 /      \
         No       Yes           Fair    Excellent
         /          \            /           \
        NO          YES        YES           NO
```

**Tree depth:** 2  
**Number of leaves:** 5  
**Training accuracy:** 14/14 = 100%

---

## 9. ASCII Visualisation of the Growing Tree

### Stage 1: Root Selection

```
   [All 14 samples]
    9 Yes, 5 No
    H = 0.9403
    Best split: Age
         │
         ▼
       [Age?]
      /  |   \
    Y    M    S
```

### Stage 2: Middle Branch Resolved

```
       [Age?]
      /  |   \
    ?    |    ?
   5s   4s   5s
  2Y,3N 4Y,0N 3Y,2N
          │
         YES  ← LEAF (pure)
```

### Stage 3: Youth Branch Resolved

```
          [Age?]
         /  |   \
   [Student?] YES  ?
    /      \      5s
  No       Yes   3Y,2N
  3s       2s
  0Y,3N   2Y,0N
   │        │
  NO      YES    ← LEAVES (pure)
```

### Stage 4: Senior Branch Resolved — COMPLETE TREE

```
               [Age?]
            /    |     \
      [Student?] YES  [Credit?]
       /    \          /      \
     NO    YES      YES       NO
```

---

## 10. Python — Building with sklearn & From Scratch

### 10.1 Using sklearn

```python
import numpy as np
import pandas as pd
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.preprocessing import LabelEncoder

# --- Create the dataset ---
data = {
    'Age':     ['Youth','Youth','Middle','Senior','Senior',
                'Senior','Middle','Youth','Youth','Senior',
                'Youth','Middle','Middle','Senior'],
    'Income':  ['High','High','High','Medium','Low',
                'Low','Low','Medium','Low','Medium',
                'Medium','Medium','High','Medium'],
    'Student': ['No','No','No','No','Yes',
                'Yes','Yes','No','Yes','Yes',
                'Yes','No','Yes','No'],
    'Credit':  ['Fair','Excellent','Fair','Fair','Fair',
                'Excellent','Excellent','Fair','Fair','Fair',
                'Excellent','Excellent','Fair','Excellent'],
    'Buys':    ['No','No','Yes','Yes','Yes',
                'No','Yes','No','Yes','Yes',
                'Yes','Yes','Yes','No']
}

df = pd.DataFrame(data)

# Encode categorical features
le = {}
for col in df.columns:
    le[col] = LabelEncoder()
    df[col] = le[col].fit_transform(df[col])

X = df.drop('Buys', axis=1).values
y = df['Buys'].values
feature_names = ['Age', 'Income', 'Student', 'Credit']

# --- Build tree with entropy (to match ID3 behaviour) ---
clf = DecisionTreeClassifier(criterion='entropy', random_state=42)
clf.fit(X, y)

# --- Display ---
print("Decision Tree Rules:")
print(export_text(clf, feature_names=feature_names))
print(f"Depth: {clf.get_depth()}")
print(f"Leaves: {clf.get_n_leaves()}")
print(f"Training accuracy: {clf.score(X, y):.4f}")
```

### 10.2 From Scratch — Minimal Implementation

```python
import numpy as np
from collections import Counter


class DecisionNode:
    """A node in the decision tree."""
    def __init__(self, feature=None, threshold=None, left=None, right=None,
                 value=None):
        self.feature = feature        # index of feature to split on
        self.threshold = threshold    # threshold for the split
        self.left = left              # left child
        self.right = right            # right child
        self.value = value            # class label (if leaf)

    def is_leaf(self):
        return self.value is not None


class SimpleDecisionTree:
    """A minimal CART-style decision tree classifier."""

    def __init__(self, max_depth=10, min_samples_split=2):
        self.max_depth = max_depth
        self.min_samples_split = min_samples_split
        self.root = None

    def fit(self, X, y):
        self.n_classes = len(set(y))
        self.root = self._build(X, y, depth=0)

    def predict(self, X):
        return np.array([self._traverse(x, self.root) for x in X])

    # ── Private methods ──

    def _gini(self, y):
        counts = Counter(y)
        n = len(y)
        return 1.0 - sum((c / n) ** 2 for c in counts.values())

    def _best_split(self, X, y):
        best_gain = -1
        best_feat, best_thresh = None, None
        parent_gini = self._gini(y)
        n = len(y)

        for feat in range(X.shape[1]):
            thresholds = np.unique(X[:, feat])
            for t in thresholds:
                left_mask = X[:, feat] <= t
                right_mask = ~left_mask
                if left_mask.sum() == 0 or right_mask.sum() == 0:
                    continue
                
                weighted = (left_mask.sum() / n * self._gini(y[left_mask]) +
                            right_mask.sum() / n * self._gini(y[right_mask]))
                gain = parent_gini - weighted

                if gain > best_gain:
                    best_gain = gain
                    best_feat = feat
                    best_thresh = t

        return best_feat, best_thresh, best_gain

    def _build(self, X, y, depth):
        # Stopping criteria
        if (depth >= self.max_depth or
                len(y) < self.min_samples_split or
                len(set(y)) == 1):
            return DecisionNode(value=Counter(y).most_common(1)[0][0])

        feat, thresh, gain = self._best_split(X, y)

        if gain <= 0:
            return DecisionNode(value=Counter(y).most_common(1)[0][0])

        left_mask = X[:, feat] <= thresh
        left = self._build(X[left_mask], y[left_mask], depth + 1)
        right = self._build(X[~left_mask], y[~left_mask], depth + 1)

        return DecisionNode(feature=feat, threshold=thresh,
                            left=left, right=right)

    def _traverse(self, x, node):
        if node.is_leaf():
            return node.value
        if x[node.feature] <= node.threshold:
            return self._traverse(x, node.left)
        return self._traverse(x, node.right)

    def print_tree(self, node=None, indent=""):
        if node is None:
            node = self.root
        if node.is_leaf():
            print(f"{indent}Predict: {node.value}")
        else:
            print(f"{indent}Feature[{node.feature}] <= {node.threshold}")
            print(f"{indent}├── True:")
            self.print_tree(node.left, indent + "│   ")
            print(f"{indent}└── False:")
            self.print_tree(node.right, indent + "    ")


# ── Demo ──
if __name__ == "__main__":
    from sklearn.datasets import load_iris
    from sklearn.model_selection import train_test_split

    iris = load_iris()
    X_train, X_test, y_train, y_test = train_test_split(
        iris.data, iris.target, test_size=0.3, random_state=42
    )

    tree = SimpleDecisionTree(max_depth=4, min_samples_split=5)
    tree.fit(X_train, y_train)

    print("=== Tree Structure ===")
    tree.print_tree()

    train_acc = np.mean(tree.predict(X_train) == y_train)
    test_acc = np.mean(tree.predict(X_test) == y_test)
    print(f"\nTrain accuracy: {train_acc:.4f}")
    print(f"Test accuracy:  {test_acc:.4f}")
```

---

## 11. Applications

| Domain | Application |
|--------|-------------|
| **Data exploration** | Build a quick tree to understand feature-target relationships |
| **Rule extraction** | Convert a tree into business rules (if-then-else) for deployment |
| **Medical triage** | Flowchart-based patient routing (symptom → test → diagnosis) |
| **Ensemble building** | Decision trees are the base learner in Random Forests and Gradient Boosting |
| **Feature selection** | Important features appear near the root; unimportant features are never used |

---

## 12. Summary Table

| Concept | Key Points |
|---------|-----------|
| **Recursive partitioning** | Top-down, split data at each node, recurse on children |
| **Greedy approach** | Picks locally best split; globally optimal tree is NP-hard |
| **CART** | Binary splits, Gini/MSE, cost-complexity pruning, sklearn default |
| **ID3** | Multi-way, Information Gain, categorical only, no pruning |
| **C4.5** | Multi-way, Gain Ratio, handles continuous & missing, error-based pruning |
| **Stopping criteria** | max_depth, min_samples_split, min_samples_leaf, min_impurity_decrease |
| **Training complexity** | O(n × m × log n × d) where n=samples, m=features, d=depth |
| **Prediction complexity** | O(d) per sample |

---

## 13. Revision Questions

1. **Describe** the recursive partitioning algorithm in your own words. Why is it called "top-down"?

2. **Compare** CART, ID3, and C4.5 on these dimensions: split type, criterion, handling of continuous features, and pruning.

3. Using the dataset in Section 8, verify that the Information Gain for the feature "Student" at the root is 0.1518 bits. Show all working.

4. **Explain** why decision tree algorithms are greedy. What is the consequence of this greediness?

5. You set `max_depth=1` in sklearn. How many leaf nodes will the tree have? What is this model called?

6. A node has 100 samples (80 class A, 20 class B). You set `min_samples_leaf=25`. Can this node be split? Why or why not?

---

## 14. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.2 Splitting Criteria](02-splitting-criteria.md) | **7.3 Building a Decision Tree** | [7.4 Pruning Techniques →](04-pruning-techniques.md) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | **Building a Decision Tree** (you are here) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"The art of building a tree is knowing when to stop asking questions."*
