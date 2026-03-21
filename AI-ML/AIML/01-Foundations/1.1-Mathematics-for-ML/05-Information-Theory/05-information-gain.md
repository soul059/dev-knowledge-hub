# Chapter 5: Information Gain

> **Unit 5 — Information Theory** | Mathematics for Machine Learning

---

## 1. Overview

**Information Gain (IG)** measures the reduction in entropy achieved by partitioning a dataset on a given feature. It is the core splitting criterion in the **ID3** and **C4.5** decision tree algorithms. IG connects Shannon entropy directly to supervised learning — making it one of the most practical concepts in information theory for ML.

---

## 2. Information Gain — Definition

Given a dataset S with target variable Y and a feature A that splits S into subsets {S₁, S₂, …, Sₖ}:

```
                          k    |Sᵥ|
IG(S, A) = H(S) -  Σ   ────── H(Sᵥ)
                         v=1   |S|
```

Where:
- **H(S)** = entropy of the target before splitting
- **H(Sᵥ)** = entropy of subset Sᵥ after splitting on A
- **|Sᵥ|/|S|** = fraction of samples in subset Sᵥ

**IG = Parent Entropy − Weighted Average of Child Entropies**

Equivalently, IG(S, A) = I(Y; A) — the mutual information between the feature and the target.

---

## 3. Decision Tree Splitting with IG

```
                    Dataset S
                   H(S) = 0.94
                   14 samples
                  /            \
         A = "Yes"            A = "No"
         9 samples            5 samples
         H(S₁)= 0.92         H(S₂)= 0.72

  IG(S, A) = 0.94 - (9/14)·0.92 - (5/14)·0.72
           = 0.94 - 0.592 - 0.257
           = 0.091 bits
```

### Algorithm: Greedy Feature Selection

```
ID3 / C4.5 Decision Tree Algorithm:
─────────────────────────────────────
1. Compute H(S) for current node
2. For each feature A:
     a. Split S into subsets by A's values
     b. Compute IG(S, A)
3. Select A* = argmax_A IG(S, A)
4. Create child nodes for each value of A*
5. Recurse on each child until:
     - All samples same class (H = 0), OR
     - No features left, OR
     - Max depth reached
```

---

## 4. Complete Worked Example: Play Tennis Dataset

### The Dataset

| Day | Outlook  | Temp | Humidity | Wind   | Play? |
|-----|----------|------|----------|--------|-------|
| 1   | Sunny    | Hot  | High     | Weak   | No    |
| 2   | Sunny    | Hot  | High     | Strong | No    |
| 3   | Overcast | Hot  | High     | Weak   | Yes   |
| 4   | Rain     | Mild | High     | Weak   | Yes   |
| 5   | Rain     | Cool | Normal   | Weak   | Yes   |
| 6   | Rain     | Cool | Normal   | Strong | No    |
| 7   | Overcast | Cool | Normal   | Strong | Yes   |
| 8   | Sunny    | Mild | High     | Weak   | No    |
| 9   | Sunny    | Cool | Normal   | Weak   | Yes   |
| 10  | Rain     | Mild | Normal   | Weak   | Yes   |
| 11  | Sunny    | Mild | Normal   | Strong | Yes   |
| 12  | Overcast | Mild | High     | Strong | Yes   |
| 13  | Overcast | Hot  | Normal   | Weak   | Yes   |
| 14  | Rain     | Mild | High     | Strong | No    |

**Total:** 9 Yes, 5 No out of 14 samples.

### Step 1: Compute Parent Entropy H(S)

```
H(S) = -(9/14) log₂(9/14) - (5/14) log₂(5/14)
     = -(0.643)(−0.637) - (0.357)(−1.486)
     = 0.410 + 0.531
     = 0.940 bits
```

### Step 2: IG for "Outlook"

| Outlook  | Total | Yes | No | H(Sᵥ)                                      |
|----------|-------|-----|----|---------------------------------------------|
| Sunny    | 5     | 2   | 3  | -(2/5)log₂(2/5)-(3/5)log₂(3/5) = 0.971     |
| Overcast | 4     | 4   | 0  | -(4/4)log₂(4/4) = 0.0                       |
| Rain     | 5     | 3   | 2  | -(3/5)log₂(3/5)-(2/5)log₂(2/5) = 0.971     |

```
IG(S, Outlook) = 0.940 - (5/14)(0.971) - (4/14)(0.0) - (5/14)(0.971)
               = 0.940 - 0.347 - 0.0 - 0.347
               = 0.247 bits
```

### Step 3: IG for "Temperature"

| Temp | Total | Yes | No | H(Sᵥ) |
|------|-------|-----|----|--------|
| Hot  | 4     | 2   | 2  | 1.000  |
| Mild | 6     | 4   | 2  | 0.918  |
| Cool | 4     | 3   | 1  | 0.811  |

```
IG(S, Temp) = 0.940 - (4/14)(1.0) - (6/14)(0.918) - (4/14)(0.811)
            = 0.940 - 0.286 - 0.394 - 0.232
            = 0.029 bits
```

### Step 4: IG for "Humidity"

| Humidity | Total | Yes | No | H(Sᵥ) |
|----------|-------|-----|----|--------|
| High     | 7     | 3   | 4  | 0.985  |
| Normal   | 7     | 6   | 1  | 0.592  |

```
IG(S, Humidity) = 0.940 - (7/14)(0.985) - (7/14)(0.592)
               = 0.940 - 0.493 - 0.296
               = 0.152 bits
```

### Step 5: IG for "Wind"

| Wind   | Total | Yes | No | H(Sᵥ) |
|--------|-------|-----|----|--------|
| Weak   | 8     | 6   | 2  | 0.811  |
| Strong | 6     | 3   | 3  | 1.000  |

```
IG(S, Wind) = 0.940 - (8/14)(0.811) - (6/14)(1.0)
            = 0.940 - 0.463 - 0.429
            = 0.048 bits
```

### Step 6: Select Best Feature

| Feature     | Information Gain |
|-------------|-----------------|
| **Outlook** | **0.247** ✓ Best|
| Humidity    | 0.152           |
| Wind        | 0.048           |
| Temperature | 0.029           |

**"Outlook" wins** → split the root node on Outlook.

### Resulting Tree (First Level)

```
                      [Outlook]
                    /     |     \
              Sunny   Overcast   Rain
              /          |          \
         [5 samples]  [4 samples]  [5 samples]
         2Y, 3N       4Y, 0N       3Y, 2N
         H=0.971      H=0 (leaf!)  H=0.971
         split more   → Yes        split more
```

The "Overcast" branch is pure (all Yes) → becomes a **leaf node**. Continue recursion on "Sunny" and "Rain" branches.

---

## 5. Comparison: Information Gain vs. Gini Impurity

### Gini Impurity Definition

```
                K
Gini(S) = 1 - Σ  pₖ²
               k=1
```

For binary classification: Gini(S) = 2p(1-p)

### Side-by-Side Comparison

```
  Impurity
  1.0 |
      |       * * * *              ← Entropy (scaled)
  0.8 |     *   G G   *
      |    * G         G *
  0.6 |   *G             G*
      |  *G               G*      ← Gini
  0.4 | *G                 G*
      |*G                   G*
  0.2 |G                     G
      |                       *
  0.0 +---+---+---+---+---+---→ p
      0  0.2  0.4  0.5  0.8  1.0
```

| Criterion          | Entropy / IG                  | Gini Impurity               |
|--------------------|------------------------------|-----------------------------|
| Formula            | -Σ pₖ log₂ pₖ               | 1 - Σ pₖ²                  |
| Range (binary)     | [0, 1] bit                   | [0, 0.5]                   |
| Computation        | Requires log (slower)        | Only multiplications (faster)|
| Bias               | Prefers multi-way splits     | Prefers binary splits       |
| Used in            | ID3, C4.5                    | CART, scikit-learn default  |
| Result quality     | Very similar in practice     | Very similar in practice    |

**In practice**, both produce nearly identical trees. CART / scikit-learn defaults to Gini for computational efficiency.

---

## 6. Gain Ratio — Correcting for Bias

### The Problem with IG

IG is **biased toward features with many distinct values**. A feature like "Customer ID" (unique per sample) would have maximum IG because each split is pure — but it's useless for generalization.

### Split Information

```
                       k    |Sᵥ|       |Sᵥ|
SplitInfo(S, A) = - Σ  ────── log₂ ──────
                      v=1   |S|        |S|
```

### Gain Ratio (C4.5)

```
                     IG(S, A)
GainRatio(S, A) = ──────────────
                   SplitInfo(S, A)
```

### Example: Gain Ratio for Outlook

```
SplitInfo(S, Outlook) = -(5/14)log₂(5/14) - (4/14)log₂(4/14) - (5/14)log₂(5/14)
                      = 0.530 + 0.516 + 0.530
                      = 1.577 bits

GainRatio(S, Outlook) = 0.247 / 1.577 = 0.157
```

### Gain Ratio for All Features

| Feature     | IG    | SplitInfo | Gain Ratio |
|-------------|-------|-----------|------------|
| Outlook     | 0.247 | 1.577     | 0.157      |
| Humidity    | 0.152 | 1.000     | **0.152**  |
| Wind        | 0.048 | 0.985     | 0.049      |
| Temperature | 0.029 | 1.557     | 0.019      |

With gain ratio, Outlook still wins, but the gap narrows — correcting for its 3-way split advantage.

---

## 7. Connection to ID3 / C4.5 / CART Algorithms

| Algorithm | Year | Splitting Criterion     | Feature Types      | Pruning |
|-----------|------|------------------------|--------------------|---------|
| **ID3**   | 1986 | Information Gain        | Categorical only   | None    |
| **C4.5**  | 1993 | Gain Ratio              | Cat. + Continuous  | Error-based|
| **CART**  | 1984 | Gini Impurity           | Cat. + Continuous  | Cost-complexity |

### ID3 Limitations (Fixed by C4.5)
1. IG bias toward high-cardinality features → **Gain Ratio**
2. Cannot handle continuous features → **Binary thresholds**
3. No pruning → **Post-pruning with error estimation**
4. Cannot handle missing values → **Fractional instances**

---

## 8. Python Implementation

```python
import numpy as np
from collections import Counter

def entropy(labels):
    """Compute entropy of a label sequence."""
    n = len(labels)
    if n == 0:
        return 0.0
    counts = Counter(labels)
    probs = np.array([c / n for c in counts.values()])
    return -np.sum(probs * np.log2(probs))

def information_gain(parent_labels, splits):
    """
    Compute IG given parent labels and a list of child label arrays.
    splits: list of arrays, each containing labels for a subset.
    """
    h_parent = entropy(parent_labels)
    n = len(parent_labels)
    h_children = sum(len(s) / n * entropy(s) for s in splits)
    return h_parent - h_children

def gain_ratio(parent_labels, splits):
    """Compute gain ratio (C4.5 criterion)."""
    ig = information_gain(parent_labels, splits)
    n = len(parent_labels)
    split_info = -sum(
        (len(s)/n) * np.log2(len(s)/n) for s in splits if len(s) > 0
    )
    return ig / split_info if split_info > 0 else 0.0

# --- Play Tennis Example ---
labels = ['No','No','Yes','Yes','Yes','No','Yes','No',
          'Yes','Yes','Yes','Yes','Yes','No']

# Splits by Outlook
sunny    = ['No','No','No','Yes','Yes']
overcast = ['Yes','Yes','Yes','Yes']
rain     = ['Yes','Yes','No','Yes','No']

ig_outlook = information_gain(labels, [sunny, overcast, rain])
gr_outlook = gain_ratio(labels, [sunny, overcast, rain])

# Splits by Humidity
high   = ['No','No','Yes','Yes','No','No','Yes']
normal = ['Yes','Yes','Yes','Yes','Yes','Yes','No']

ig_humidity = information_gain(labels, [high, normal])

# Splits by Wind
weak   = ['No','Yes','Yes','Yes','No','Yes','Yes','Yes']
strong = ['No','No','Yes','Yes','Yes','No']

ig_wind = information_gain(labels, [weak, strong])

print("Information Gain Results:")
print(f"  Outlook:     IG = {ig_outlook:.4f},  GR = {gr_outlook:.4f}")
print(f"  Humidity:    IG = {ig_humidity:.4f}")
print(f"  Wind:        IG = {ig_wind:.4f}")
print(f"\nBest split: Outlook (highest IG = {ig_outlook:.4f})")
```

**Output:**
```
Information Gain Results:
  Outlook:     IG = 0.2467,  GR = 0.1564
  Humidity:    IG = 0.1518
  Wind:        IG = 0.0481

Best split: Outlook (highest IG = 0.2467)
```

### Using scikit-learn

```python
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.preprocessing import LabelEncoder
import numpy as np

# Encode the Play Tennis dataset
outlook  = ['Sunny','Sunny','Overcast','Rain','Rain','Rain',
            'Overcast','Sunny','Sunny','Rain','Sunny','Overcast',
            'Overcast','Rain']
humidity = ['High','High','High','High','Normal','Normal','Normal',
            'High','Normal','Normal','Normal','High','Normal','High']
wind     = ['Weak','Strong','Weak','Weak','Weak','Strong','Strong',
            'Weak','Weak','Weak','Strong','Strong','Weak','Strong']
play     = ['No','No','Yes','Yes','Yes','No','Yes','No',
            'Yes','Yes','Yes','Yes','Yes','No']

le = LabelEncoder()
X = np.column_stack([le.fit_transform(outlook),
                     le.fit_transform(humidity),
                     le.fit_transform(wind)])
y = le.fit_transform(play)

# criterion='entropy' uses Information Gain
tree = DecisionTreeClassifier(criterion='entropy', random_state=42)
tree.fit(X, y)
print(export_text(tree, feature_names=['Outlook','Humidity','Wind']))
```

---

## 9. Real-World ML Applications

| Application                 | How IG / Entropy Splitting Is Used                   |
|-----------------------------|------------------------------------------------------|
| **Decision Trees (ID3/C4.5)** | Core splitting criterion                           |
| **Random Forests**          | Each tree uses IG or Gini for splits                 |
| **Gradient Boosted Trees**  | XGBoost/LightGBM can use entropy-based splits        |
| **Feature Importance**      | Total IG contributed by each feature across the tree  |
| **Feature Selection**       | Pre-filter features by their IG with the target       |
| **Text Classification**     | Select top words/n-grams by IG with class labels     |
| **Medical Diagnosis**       | Build interpretable decision rules from patient data |

---

## 10. Summary Table

| Concept            | Formula / Key Fact                                       |
|--------------------|----------------------------------------------------------|
| Information Gain   | IG(S,A) = H(S) - Σ (|Sᵥ|/|S|) H(Sᵥ)                   |
| Equivalence        | IG(S,A) = I(Y; A) (mutual information)                   |
| Entropy            | H(S) = -Σ pₖ log₂ pₖ                                   |
| Gini Impurity      | Gini(S) = 1 - Σ pₖ²                                     |
| Split Information   | SI(S,A) = -Σ (|Sᵥ|/|S|) log₂(|Sᵥ|/|S|)                |
| Gain Ratio         | GR(S,A) = IG(S,A) / SI(S,A)                             |
| IG bias            | Favors features with many values → use gain ratio        |
| ID3                | Uses IG, categorical features only                       |
| C4.5               | Uses gain ratio, handles continuous + missing values     |
| CART               | Uses Gini impurity, binary splits only                   |

---

## 11. Quick Revision Questions

1. **What is information gain, and what does a high IG value indicate?**
   > IG is the reduction in entropy after splitting on a feature. High IG means the feature creates purer subsets — it's very informative.

2. **Why does ID3 use entropy rather than Gini impurity?**
   > ID3 was designed with entropy from information theory. In practice, both produce very similar trees; CART uses Gini for computational simplicity.

3. **What is the bias of information gain, and how does gain ratio fix it?**
   > IG favors features with many distinct values (high cardinality). Gain ratio normalizes by SplitInfo, penalizing features that create many small partitions.

4. **In the Play Tennis example, why does Outlook have the highest IG?**
   > Because "Overcast" creates a perfectly pure subset (all Yes, H=0), contributing a large entropy reduction.

5. **How is information gain related to mutual information?**
   > They are equivalent: IG(S, A) = I(Y; A). Both measure how much knowing A reduces uncertainty about Y.

6. **When would you choose `criterion='entropy'` over `criterion='gini'` in scikit-learn?**
   > When you need multi-way splits or want information-theoretic interpretability. In practice, results are similar; Gini is slightly faster to compute.

---

[← Previous Chapter: 04-Mutual-Information](./04-mutual-information.md) | [Next Chapter: Unit 6 →](../../1.1-Mathematics-for-ML/)
