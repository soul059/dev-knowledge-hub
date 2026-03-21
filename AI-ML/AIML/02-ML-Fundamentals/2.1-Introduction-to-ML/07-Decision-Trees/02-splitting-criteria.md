# 📘 7.2 — Splitting Criteria

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 2 of 7 — Gini Impurity, Entropy, Information Gain, Gain Ratio — the mathematics of choosing the best split.*

---

## 📑 Table of Contents

1. [Why Do We Need Splitting Criteria?](#1-why-do-we-need-splitting-criteria)
2. [Gini Impurity](#2-gini-impurity)
3. [Entropy](#3-entropy)
4. [Information Gain](#4-information-gain)
5. [Gain Ratio](#5-gain-ratio)
6. [Gini vs Entropy — Detailed Comparison](#6-gini-vs-entropy--detailed-comparison)
7. [Step-by-Step Splitting Example](#7-step-by-step-splitting-example)
8. [Choosing Features & Thresholds](#8-choosing-features--thresholds)
9. [Continuous vs Categorical Features](#9-continuous-vs-categorical-features)
10. [Python — Gini & Entropy from Scratch](#10-python--gini--entropy-from-scratch)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Why Do We Need Splitting Criteria?

At every internal node of a decision tree, the algorithm must choose:

1. **Which feature** to split on.
2. **What threshold or partition** to use.

The goal is to make the child nodes as **pure** as possible — ideally each child should contain samples from only one class.

```
  IMPURE parent              PURE children (ideal)
  ┌──────────┐               ┌──────┐   ┌──────┐
  │ ●●●○○○○  │    split →    │ ●●●  │   │ ○○○○ │
  │           │               │      │   │      │
  └──────────┘               └──────┘   └──────┘
   4○ + 3●                    3●          4○
   Impurity: high             Impurity: 0  Impurity: 0
```

A **splitting criterion** is a mathematical function that measures impurity (or information content) of a node. The algorithm evaluates every possible split and picks the one that **reduces impurity the most**.

---

## 2. Gini Impurity

### 2.1 Definition

For a node with *K* classes and class proportions p₁, p₂, …, p_K:

```
                    K
Gini(node) = 1 −  Σ  pₖ²
                   k=1
```

Equivalently:

```
                    K
Gini(node) =  Σ  pₖ (1 − pₖ)
                   k=1
```

### 2.2 Intuition

- **Gini = 0** → the node is perfectly **pure** (all samples belong to one class).
- **Gini is maximised** when classes are **equally distributed**.
  - For binary classification: max Gini = 1 − (0.5² + 0.5²) = **0.5**
  - For K classes: max Gini = 1 − K × (1/K)² = 1 − 1/K

### 2.3 Gini for Binary Classification

With two classes, let p = proportion of class 1. Then:

```
Gini = 1 − p² − (1−p)² = 2p(1 − p)
```

Graph:

```
Gini
0.5 │          ·  ·  ·
    │        ·          ·
    │      ·              ·
0.4 │    ·                  ·
    │   ·                    ·
0.3 │  ·                      ·
    │ ·                        ·
0.2 │·                          ·
    │                            ·
0.1 │                             ·
    │·                             ·
  0 ┼──────────────────────────────┼── p
    0         0.25   0.5   0.75    1
```

### 2.4 Worked Calculation

A node has 10 samples: 7 belong to class A, 3 belong to class B.

```
p_A = 7/10 = 0.70
p_B = 3/10 = 0.30

Gini = 1 − (0.70² + 0.30²)
     = 1 − (0.49 + 0.09)
     = 1 − 0.58
     = 0.42
```

### 2.5 Weighted Gini After a Split

If a split creates two child nodes (left with nₗ samples, right with nᵣ samples):

```
                     nₗ                    nᵣ
Gini_split = ────────────── × Gini(left) + ────────────── × Gini(right)
              nₗ + nᵣ                      nₗ + nᵣ
```

The algorithm picks the split that gives the **lowest weighted Gini**.

---

## 3. Entropy

### 3.1 Definition

From information theory, **entropy** measures the average amount of "surprise" or "information" in a random variable.

```
                     K
H(node) = −  Σ  pₖ log₂(pₖ)
                    k=1
```

> Convention: 0 × log₂(0) = 0

### 3.2 Intuition

- **H = 0** → perfectly pure (no surprise — you know the class).
- **H is maximised** when all classes are equally likely.
  - Binary: max H = −0.5 log₂(0.5) − 0.5 log₂(0.5) = **1.0 bit**
  - K classes: max H = log₂(K)

### 3.3 Entropy for Binary Classification

Let p = proportion of class 1:

```
H = −p log₂(p) − (1−p) log₂(1−p)
```

Graph:

```
H (bits)
1.0 │          ·  ·  ·
    │        ·          ·
    │      ·              ·
0.8 │    ·                  ·
    │   ·                    ·
0.6 │  ·                      ·
    │ ·                        ·
0.4 │·                          ·
    │                            ·
0.2 │                             ·
    │·                             ·
  0 ┼──────────────────────────────┼── p
    0         0.25   0.5   0.75    1
```

### 3.4 Worked Calculation

Same node: 7 class A, 3 class B.

```
p_A = 0.70,  p_B = 0.30

H = −(0.70 × log₂(0.70)) − (0.30 × log₂(0.30))
  = −(0.70 × (−0.5146)) − (0.30 × (−1.7370))
  = 0.3602 + 0.5211
  = 0.8813 bits
```

---

## 4. Information Gain

### 4.1 Definition

**Information Gain (IG)** is the reduction in entropy (or Gini) achieved by splitting a node.

```
IG(parent, split) = H(parent) − H_weighted(children)
```

Where:

```
                          |Sₗ|                |Sᵣ|
H_weighted(children) = ───────── × H(Sₗ) + ───────── × H(Sᵣ)
                        |Sₗ|+|Sᵣ|           |Sₗ|+|Sᵣ|
```

### 4.2 Worked Example

**Parent node:** 14 samples — 9 Yes, 5 No

```
H(parent) = −(9/14)log₂(9/14) − (5/14)log₂(5/14)
           = −(0.6429)(−0.6376) − (0.3571)(−1.4854)
           = 0.4100 + 0.5305
           = 0.9405 bits
```

**Proposed split on "Wind":**
- Wind = Weak: 8 samples (6 Yes, 2 No)
- Wind = Strong: 6 samples (3 Yes, 3 No)

```
H(Weak)   = −(6/8)log₂(6/8) − (2/8)log₂(2/8)
           = −(0.75)(−0.4150) − (0.25)(−2.0)
           = 0.3113 + 0.5000
           = 0.8113 bits

H(Strong) = −(3/6)log₂(3/6) − (3/6)log₂(3/6)
           = −(0.5)(−1.0) − (0.5)(−1.0)
           = 1.0000 bit

H_weighted = (8/14)(0.8113) + (6/14)(1.0000)
           = 0.4636 + 0.4286
           = 0.8922 bits

IG(Wind)   = 0.9405 − 0.8922 = 0.0483 bits
```

### 4.3 Algorithm Selects the Maximum IG

The tree-building algorithm computes IG for **every possible split** on every feature, then selects the split with the **highest Information Gain**.

---

## 5. Gain Ratio

### 5.1 The Problem with Information Gain

Information Gain is **biased toward features with many distinct values**. Consider a feature like "CustomerID" — it has a unique value per sample, so splitting on it creates perfectly pure leaves (each leaf has one sample). The IG would be maximal, but this split is useless for generalisation.

### 5.2 Definition

**Gain Ratio** normalises Information Gain by the **intrinsic value (split information)** of the feature:

```
                      IG(parent, feature)
GainRatio(feature) = ─────────────────────
                      SplitInfo(feature)
```

Where Split Information measures how evenly the split divides the data:

```
                        K    |Sₖ|        |Sₖ|
SplitInfo(feature) = − Σ  ───────  log₂ ───────
                       k=1  |S|          |S|
```

### 5.3 Worked Example

For the "Wind" split above:

```
SplitInfo(Wind) = −(8/14)log₂(8/14) − (6/14)log₂(6/14)
                = −(0.5714)(−0.8074) − (0.4286)(−1.2224)
                = 0.4614 + 0.5238
                = 0.9852

GainRatio(Wind) = 0.0483 / 0.9852 = 0.0490
```

For a feature with many values (high SplitInfo), the ratio is naturally reduced, correcting the bias.

### 5.4 Usage

- **C4.5** uses Gain Ratio as its splitting criterion.
- **ID3** uses raw Information Gain (and suffers from the bias).
- **CART** uses Gini Impurity (avoids the problem by using binary splits).

---

## 6. Gini vs Entropy — Detailed Comparison

### 6.1 Mathematical Comparison (Binary Case)

| p | Gini = 2p(1−p) | Entropy = −p log₂p − (1−p)log₂(1−p) | Scaled Entropy × 0.5 |
|---|-------|---------|----------------|
| 0.0 | 0.000 | 0.000 | 0.000 |
| 0.1 | 0.180 | 0.469 | 0.234 |
| 0.2 | 0.320 | 0.722 | 0.361 |
| 0.3 | 0.420 | 0.881 | 0.441 |
| 0.4 | 0.480 | 0.971 | 0.486 |
| 0.5 | 0.500 | 1.000 | 0.500 |

When entropy is scaled by 0.5, both curves are nearly identical. **In practice, Gini and Entropy produce almost the same splits** ~95% of the time.

### 6.2 Side-by-Side Comparison

```
            Gini vs Entropy (Binary Classification)

 Value
 1.0  │                      ·  ·  ·                    Entropy
      │                   ·          ·
 0.8  │                ·                ·
      │             ·                      ·
 0.6  │          ·    ◆  ◆  ◆                ·
      │        ·  ◆              ◆             ·
 0.5  │      · ◆          Gini     ◆            ·
 0.4  │    ◆·                         ◆           ·
      │  ◆                               ◆
 0.2  │◆                                    ◆        ·
      │                                       ◆
 0.0  ◆─────────────────────────────────────────◆──────·
      0       0.2      0.4      0.6      0.8       1.0
                         p (class proportion)
```

### 6.3 Feature-by-Feature Comparison

| Aspect | Gini Impurity | Entropy / Info Gain |
|--------|--------------|---------------------|
| **Formula** | 1 − Σ pₖ² | −Σ pₖ log₂ pₖ |
| **Range (binary)** | [0, 0.5] | [0, 1.0] |
| **Range (K classes)** | [0, 1 − 1/K] | [0, log₂ K] |
| **Computation** | Faster (no logarithm) | Slower (requires log) |
| **Used by** | CART (sklearn default) | ID3, C4.5 |
| **Sensitivity** | Slightly favours pure nodes | Slightly more sensitive to class balance |
| **Practical difference** | Very small | Very small |
| **sklearn parameter** | `criterion='gini'` | `criterion='entropy'` |

### 6.4 When Does It Matter?

- **Rarely.** Research (Raileanu & Stoffel, 2004) showed that Gini and Entropy lead to **identical trees in ~98% of cases** on standard benchmarks.
- Gini is marginally faster to compute (no log), so sklearn uses it as default.
- If you must choose: try both with cross-validation and pick the one with better validation performance.

---

## 7. Step-by-Step Splitting Example

### 7.1 Dataset

| Sample | Colour | Size  | Shape  | Class |
|--------|--------|-------|--------|-------|
| 1      | Red    | Large | Circle | A     |
| 2      | Red    | Small | Circle | A     |
| 3      | Blue   | Large | Square | B     |
| 4      | Blue   | Small | Circle | B     |
| 5      | Green  | Large | Square | B     |
| 6      | Red    | Large | Square | A     |
| 7      | Blue   | Small | Square | B     |
| 8      | Green  | Large | Circle | A     |

Total: 4 A, 4 B → parent Gini = 1 − (0.5² + 0.5²) = **0.5**

### 7.2 Evaluate Every Feature

#### Feature: Colour (Red / Blue / Green)

Since CART is binary, we try all possible partitions into two groups:

**Split 1: {Red} vs {Blue, Green}**

| Subset | Samples | A | B | Gini |
|--------|---------|---|---|------|
| Red | 1,2,6 → 3 samples | 3 | 0 | 1 − (1² + 0²) = 0.000 |
| Blue+Green | 3,4,5,7,8 → 5 samples | 1 | 4 | 1 − (0.2² + 0.8²) = 0.320 |

```
Gini_split = (3/8)(0.000) + (5/8)(0.320) = 0.000 + 0.200 = 0.200
```

**Split 2: {Blue} vs {Red, Green}**

| Subset | Samples | A | B | Gini |
|--------|---------|---|---|------|
| Blue | 3,4,7 → 3 samples | 0 | 3 | 0.000 |
| Red+Green | 1,2,5,6,8 → 5 samples | 4 | 1 | 1 − (0.8² + 0.2²) = 0.320 |

```
Gini_split = (3/8)(0.000) + (5/8)(0.320) = 0.200
```

**Split 3: {Green} vs {Red, Blue}**

| Subset | Samples | A | B | Gini |
|--------|---------|---|---|------|
| Green | 5,8 → 2 samples | 1 | 1 | 0.500 |
| Red+Blue | 1,2,3,4,6,7 → 6 samples | 3 | 3 | 0.500 |

```
Gini_split = (2/8)(0.500) + (6/8)(0.500) = 0.125 + 0.375 = 0.500
```

**Split 4: {Red, Blue} vs {Green}** — same as Split 3 by symmetry = 0.500

**Split 5: {Red, Green} vs {Blue}** — same as Split 2 by symmetry = 0.200

**Split 6: {Blue, Green} vs {Red}** — same as Split 1 by symmetry = 0.200

Best Colour split: **{Red} vs {Blue, Green}** or **{Blue} vs {Red, Green}** with Gini = **0.200**

#### Feature: Size (Large / Small)

| Subset | Samples | A | B | Gini |
|--------|---------|---|---|------|
| Large | 1,3,5,6,8 → 5 | 3 | 2 | 1 − (0.6² + 0.4²) = 0.480 |
| Small | 2,4,7 → 3 | 1 | 2 | 1 − (0.333² + 0.667²) = 0.444 |

```
Gini_split = (5/8)(0.480) + (3/8)(0.444) = 0.300 + 0.167 = 0.467
```

#### Feature: Shape (Circle / Square)

| Subset | Samples | A | B | Gini |
|--------|---------|---|---|------|
| Circle | 1,2,4,8 → 4 | 3 | 1 | 1 − (0.75² + 0.25²) = 0.375 |
| Square | 3,5,6,7 → 4 | 1 | 3 | 1 − (0.25² + 0.75²) = 0.375 |

```
Gini_split = (4/8)(0.375) + (4/8)(0.375) = 0.375
```

### 7.3 Select the Best Split

| Feature | Best Split | Weighted Gini | ΔGini |
|---------|-----------|--------------|-------|
| **Colour** | {Red} vs {Blue, Green} | **0.200** | 0.300 |
| Size | Large vs Small | 0.467 | 0.033 |
| Shape | Circle vs Square | 0.375 | 0.125 |

**Winner: Colour** with the split `{Red} vs {Blue, Green}` — lowest Gini (highest Gini decrease).

### 7.4 Resulting Tree After First Split

```
            [Colour ∈ {Red}?]
             /              \
           Yes               No
           /                   \
     ┌─────────┐         ┌─────────────┐
     │ Samples │         │ Samples     │
     │ 1, 2, 6 │         │ 3,4,5,7,8   │
     │ 3A, 0B  │         │ 1A, 4B      │
     │ Gini=0  │         │ Gini=0.32   │
     │→ LEAF A │         │→ split more │
     └─────────┘         └─────────────┘
```

The left child is pure (Gini = 0) → becomes a **leaf** predicting class A.  
The right child is impure → the algorithm **recurses** and finds the next best split among its 5 samples.

---

## 8. Choosing Features & Thresholds

### 8.1 The Algorithm

```
for each feature f in dataset:
    for each possible split point t of feature f:
        compute Gini_split (or IG) for splitting on f at t
        if Gini_split < best_so_far:
            best_so_far = Gini_split
            best_feature = f
            best_threshold = t
```

### 8.2 Number of Candidate Splits

| Feature Type | # Possible Splits |
|-------------|-------------------|
| Binary (e.g., Yes/No) | 1 |
| Categorical with k values (CART) | 2^(k−1) − 1 subsets |
| Categorical with k values (ID3) | 1 (k-way split) |
| Continuous with n distinct values | n − 1 thresholds |

For continuous features, the standard approach is:
1. Sort the feature values.
2. Consider thresholds at the **midpoint** between consecutive distinct values.

---

## 9. Continuous vs Categorical Features

### 9.1 Continuous Features

For a continuous feature with sorted values x₁ ≤ x₂ ≤ … ≤ xₙ:

- Candidate thresholds: t₁ = (x₁+x₂)/2, t₂ = (x₂+x₃)/2, …, tₙ₋₁ = (xₙ₋₁+xₙ)/2
- Each threshold creates a binary split: `feature ≤ t` vs `feature > t`

```
   Feature values:  2.1  3.5  3.5  4.8  7.2  9.0
   Candidate thresholds:  2.8   4.15   6.0   8.1
                          ↑      ↑      ↑     ↑
                     (between consecutive distinct values)
```

### 9.2 Categorical Features

**CART (binary splits):** For a categorical feature with values {A, B, C, D}, test all subsets:
- {A} vs {B,C,D}
- {B} vs {A,C,D}
- {A,B} vs {C,D}
- {A,C} vs {B,D}
- ... etc.
- Total: 2^(k−1) − 1 = 7 possible splits

**ID3 / C4.5 (multi-way):** Create one branch per category:
- Branch A, Branch B, Branch C, Branch D

### 9.3 Optimisation for Binary Classification

For binary classification with a categorical feature, there's a shortcut:
1. Compute the proportion of class 1 for each category.
2. Sort categories by this proportion.
3. Only consider splits that keep this sorted order contiguous.

This reduces the number of candidate splits from 2^(k−1) − 1 to k − 1.

---

## 10. Python — Gini & Entropy from Scratch

```python
import numpy as np
from collections import Counter

# ──────────────────────────────────────────────────────
#  Impurity Functions
# ──────────────────────────────────────────────────────

def gini_impurity(labels):
    """Compute Gini impurity of a list of labels."""
    n = len(labels)
    if n == 0:
        return 0.0
    counts = Counter(labels)
    return 1.0 - sum((c / n) ** 2 for c in counts.values())


def entropy(labels):
    """Compute entropy (in bits) of a list of labels."""
    n = len(labels)
    if n == 0:
        return 0.0
    counts = Counter(labels)
    probs = [c / n for c in counts.values()]
    return -sum(p * np.log2(p) for p in probs if p > 0)


def weighted_impurity(left_labels, right_labels, criterion='gini'):
    """Compute the weighted impurity after a binary split."""
    func = gini_impurity if criterion == 'gini' else entropy
    n_left = len(left_labels)
    n_right = len(right_labels)
    n_total = n_left + n_right
    if n_total == 0:
        return 0.0
    return (n_left / n_total) * func(left_labels) + \
           (n_right / n_total) * func(right_labels)


def information_gain(parent_labels, left_labels, right_labels, criterion='entropy'):
    """Compute information gain of a split."""
    func = gini_impurity if criterion == 'gini' else entropy
    parent_impurity = func(parent_labels)
    child_impurity = weighted_impurity(left_labels, right_labels, criterion)
    return parent_impurity - child_impurity


# ──────────────────────────────────────────────────────
#  Demo
# ──────────────────────────────────────────────────────

if __name__ == "__main__":
    # Parent node: 9 Yes, 5 No
    parent = ['Yes'] * 9 + ['No'] * 5

    print("=== Parent Node ===")
    print(f"  Gini impurity : {gini_impurity(parent):.4f}")
    print(f"  Entropy       : {entropy(parent):.4f} bits")

    # Split by Wind
    left  = ['Yes'] * 6 + ['No'] * 2   # Wind = Weak
    right = ['Yes'] * 3 + ['No'] * 3   # Wind = Strong

    print("\n=== Split by Wind ===")
    print(f"  Left  (Weak)   — Gini: {gini_impurity(left):.4f}, "
          f"Entropy: {entropy(left):.4f}")
    print(f"  Right (Strong) — Gini: {gini_impurity(right):.4f}, "
          f"Entropy: {entropy(right):.4f}")
    print(f"  Weighted Gini     : {weighted_impurity(left, right, 'gini'):.4f}")
    print(f"  Weighted Entropy  : {weighted_impurity(left, right, 'entropy'):.4f}")
    print(f"  Info Gain (Gini)  : {information_gain(parent, left, right, 'gini'):.4f}")
    print(f"  Info Gain (Entropy): {information_gain(parent, left, right, 'entropy'):.4f}")

    # ── Find best threshold for a continuous feature ──
    print("\n=== Finding Best Threshold for a Continuous Feature ===")
    
    feature_vals = np.array([2.0, 3.5, 1.0, 4.5, 3.0, 5.0, 2.5, 4.0])
    labels = np.array(['A', 'B', 'A', 'B', 'A', 'B', 'A', 'B'])
    
    sorted_idx = np.argsort(feature_vals)
    sorted_vals = feature_vals[sorted_idx]
    sorted_labels = labels[sorted_idx]
    
    best_threshold = None
    best_gain = -1
    
    for i in range(1, len(sorted_vals)):
        if sorted_vals[i] == sorted_vals[i - 1]:
            continue
        threshold = (sorted_vals[i - 1] + sorted_vals[i]) / 2
        left_mask = feature_vals <= threshold
        left_l = labels[left_mask]
        right_l = labels[~left_mask]
        gain = information_gain(labels, left_l, right_l, 'gini')
        print(f"  Threshold {threshold:.2f} → IG (Gini) = {gain:.4f}")
        if gain > best_gain:
            best_gain = gain
            best_threshold = threshold
    
    print(f"\n  Best threshold: {best_threshold:.2f} with IG = {best_gain:.4f}")
```

**Expected output:**

```
=== Parent Node ===
  Gini impurity : 0.4592
  Entropy       : 0.9403 bits

=== Split by Wind ===
  Left  (Weak)   — Gini: 0.3750, Entropy: 0.8113
  Right (Strong) — Gini: 0.5000, Entropy: 1.0000
  Weighted Gini     : 0.4286
  Weighted Entropy  : 0.8922
  Info Gain (Gini)  : 0.0306
  Info Gain (Entropy): 0.0481
```

---

## 11. Applications

| Domain | Splitting Criteria in Action |
|--------|------------------------------|
| **Medical diagnosis** | Splitting on symptoms — which symptom best separates sick from healthy? Entropy highlights informative symptoms. |
| **Credit scoring** | Gini impurity identifies which financial features (income, debt ratio) most cleanly separate defaulters from non-defaulters. |
| **Email spam filtering** | Information Gain ranks which keywords are most predictive of spam vs ham. |
| **Customer churn** | Gain Ratio prevents overfitting to high-cardinality features like ZIP code. |
| **Fraud detection** | Continuous features (transaction amount) use threshold-based splits to flag anomalies. |

---

## 12. Summary Table

| Criterion | Formula | Range (Binary) | Used By | Pros | Cons |
|-----------|---------|----------------|---------|------|------|
| **Gini Impurity** | 1 − Σ pₖ² | [0, 0.5] | CART, sklearn | Fast (no log), simple | Slightly less theoretically grounded |
| **Entropy** | −Σ pₖ log₂ pₖ | [0, 1.0] | ID3, C4.5 | Information-theoretic basis | Needs logarithm (slower) |
| **Information Gain** | H(parent) − H_weighted(children) | [0, H(parent)] | ID3 | Simple, intuitive | Biased toward high-cardinality features |
| **Gain Ratio** | IG / SplitInfo | [0, 1] | C4.5 | Corrects cardinality bias | Can be unstable when SplitInfo ≈ 0 |

---

## 13. Revision Questions

1. **Compute** the Gini impurity and Entropy for a node with 12 samples: 8 class A, 4 class B. Show all working.

2. A split produces two children: Left (5A, 1B) and Right (3A, 3B). **Calculate** the weighted Gini impurity and the Information Gain compared to the parent from Q1.

3. **Explain** why Information Gain is biased toward features with many categories. Give a concrete example.

4. **Derive** the Gain Ratio for the split in Q2. What is the SplitInfo?

5. You have a continuous feature with values [1, 3, 5, 7, 9]. How many candidate thresholds are there? **List** them.

6. In sklearn, the default splitting criterion for `DecisionTreeClassifier` is _______ . If you wanted to use entropy, what parameter would you set?

---

## 14. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.1 Tree Structure & Terminology](01-tree-structure-terminology.md) | **7.2 Splitting Criteria** | [7.3 Building a Decision Tree →](03-building-decision-tree.md) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | **Splitting Criteria** (you are here) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"The best split is the one that tells you the most about what you don't yet know."*
