# Chapter 4: Mutual Information

> **Unit 5 — Information Theory** | Mathematics for Machine Learning

---

## 1. Overview

**Mutual Information (MI)** quantifies how much knowing one random variable tells us about another. Unlike correlation, MI captures **all** statistical dependencies — including nonlinear ones. It is widely used in feature selection, clustering, and as a training objective in representation learning.

---

## 2. Conditional Entropy

Before defining MI, we need **conditional entropy** — the remaining uncertainty about X after observing Y:

```
H(X|Y) = - Σ  Σ  p(x, y) log p(x|y)
           y  x

       = H(X, Y) - H(Y)
```

**Intuition:** How much information about X is left after Y is revealed?

| Relationship      | Conditional Entropy     |
|-------------------|------------------------|
| X, Y independent  | H(X|Y) = H(X)          |
| Y determines X    | H(X|Y) = 0             |
| General case      | 0 ≤ H(X|Y) ≤ H(X)     |

---

## 3. Mutual Information — Definition

```
I(X; Y) = H(X) + H(Y) - H(X, Y)
```

Equivalent forms:

```
I(X; Y) = H(X) - H(X|Y)           ← reduction in uncertainty about X
I(X; Y) = H(Y) - H(Y|X)           ← reduction in uncertainty about Y
I(X; Y) = D_KL( p(x,y) ‖ p(x)p(y) )   ← divergence from independence

         = Σ  Σ  p(x,y) log [ p(x,y) / (p(x)p(y)) ]
           x  y
```

---

## 4. Venn Diagram of Information Measures (ASCII)

```
    ┌─────────────────────────────────────────┐
    │            H(X, Y)                      │
    │                                         │
    │   ┌──────────────┬──────────────┐       │
    │   │              │              │       │
    │   │    H(X|Y)    │   I(X;Y)    │H(Y|X)│       │
    │   │              │              │       │
    │   │  Unique to X │   Shared    │Unique │       │
    │   │              │             │ to Y  │       │
    │   └──────────────┴──────────────┘       │
    │                                         │
    │   ├── H(X) ──────┤                      │
    │                   ├──── H(Y) ───────────┤
    └─────────────────────────────────────────┘

    H(X)   = H(X|Y) + I(X;Y)
    H(Y)   = H(Y|X) + I(X;Y)
    H(X,Y) = H(X|Y) + H(Y|X) + I(X;Y)
           = H(X) + H(Y) - I(X;Y)
```

### All Relationships at a Glance

```
┌────────────────────────────────────────────────────────┐
│  I(X;Y) = H(X) + H(Y) - H(X,Y)                       │
│  I(X;Y) = H(X) - H(X|Y) = H(Y) - H(Y|X)             │
│  H(X,Y) = H(X) + H(Y|X) = H(Y) + H(X|Y)             │
│  H(X|Y) = H(X,Y) - H(Y)                              │
│  0 ≤ I(X;Y) ≤ min(H(X), H(Y))                        │
└────────────────────────────────────────────────────────┘
```

---

## 5. Properties of Mutual Information

| Property             | Statement                                              |
|----------------------|--------------------------------------------------------|
| **Non-negative**     | I(X;Y) ≥ 0                                            |
| **Symmetric**        | I(X;Y) = I(Y;X)                                       |
| **Zero iff independent** | I(X;Y) = 0 ⟺ X ⊥ Y                              |
| **Upper bound**      | I(X;Y) ≤ min(H(X), H(Y))                              |
| **Self-information** | I(X;X) = H(X) (entropy is self-MI)                    |
| **Data processing inequality** | X → Y → Z ⟹ I(X;Z) ≤ I(X;Y)            |

---

## 6. MI vs. Correlation

| Property              | Correlation (Pearson)    | Mutual Information       |
|----------------------|--------------------------|--------------------------|
| Captures linear deps  | ✓                        | ✓                        |
| Captures nonlinear    | ✗                        | ✓                        |
| Range                 | [-1, 1]                  | [0, ∞)                   |
| Zero means            | No linear relationship   | Complete independence    |
| Scale invariant       | ✗                        | Invariant to bijections  |

### Classic Example: X ~ Uniform(-1, 1), Y = X²

```
Pearson correlation:  ρ(X, Y) = 0   ← misses the relationship!
Mutual information:   I(X; Y) > 0   ← correctly detects dependence
```

---

## 7. Step-by-Step Examples

### Example 1: Joint Distribution Table

Consider random variables X ∈ {0, 1} and Y ∈ {0, 1}:

| p(x,y)   | Y=0  | Y=1  | p(x)  |
|-----------|------|------|-------|
| **X=0**   | 0.3  | 0.1  | 0.4   |
| **X=1**   | 0.2  | 0.4  | 0.6   |
| **p(y)**  | 0.5  | 0.5  |       |

**Step 1: Compute H(X)**
```
H(X) = -0.4 log₂(0.4) - 0.6 log₂(0.6)
     = -0.4(-1.322) - 0.6(-0.737)
     = 0.529 + 0.442 = 0.971 bits
```

**Step 2: Compute H(Y)**
```
H(Y) = -0.5 log₂(0.5) - 0.5 log₂(0.5)
     = 1.0 bit
```

**Step 3: Compute H(X,Y)**
```
H(X,Y) = -0.3 log₂(0.3) - 0.1 log₂(0.1) - 0.2 log₂(0.2) - 0.4 log₂(0.4)
       = 0.521 + 0.332 + 0.464 + 0.529
       = 1.846 bits
```

**Step 4: Compute I(X;Y)**
```
I(X;Y) = H(X) + H(Y) - H(X,Y)
       = 0.971 + 1.0 - 1.846
       = 0.125 bits
```

X and Y share 0.125 bits of information — they are **not independent**.

### Example 2: Independent Variables

If X ⊥ Y, then p(x,y) = p(x)p(y) for all x, y:

| p(x,y) | Y=0  | Y=1  |
|---------|------|------|
| X=0     | 0.20 | 0.20 |
| X=1     | 0.30 | 0.30 |

```
H(X,Y) = H(X) + H(Y)   →   I(X;Y) = 0 bits
```

### Example 3: Perfectly Dependent Variables

If Y = X (deterministic):

| p(x,y) | Y=0  | Y=1  |
|---------|------|------|
| X=0     | 0.5  | 0.0  |
| X=1     | 0.0  | 0.5  |

```
H(X) = H(Y) = 1.0 bit
H(X,Y) = -0.5 log₂(0.5) - 0.5 log₂(0.5) = 1.0 bit
I(X;Y) = 1.0 + 1.0 - 1.0 = 1.0 bit = H(X) = H(Y)
```

Maximum MI — knowing Y tells us everything about X.

---

## 8. Python Implementation

```python
import numpy as np
from sklearn.metrics import mutual_info_score
from sklearn.feature_selection import mutual_info_classif

def mutual_information(joint_prob, base=2):
    """Compute MI from a joint probability table (2D array)."""
    joint = np.array(joint_prob)
    px = joint.sum(axis=1)      # marginal of X
    py = joint.sum(axis=0)      # marginal of Y
    
    mi = 0.0
    for i in range(joint.shape[0]):
        for j in range(joint.shape[1]):
            if joint[i, j] > 0:
                mi += joint[i, j] * np.log(joint[i, j] / (px[i] * py[j]))
    return mi / np.log(base)

# --- Example 1: From joint table ---
joint = np.array([[0.3, 0.1],
                  [0.2, 0.4]])
print(f"I(X;Y) = {mutual_information(joint):.4f} bits")

# --- Example 2: Using sklearn for discrete data ---
X_labels = [0, 0, 1, 1, 0, 1, 1, 0, 1, 1]
Y_labels = [0, 0, 1, 1, 1, 1, 0, 0, 1, 1]
mi_sklearn = mutual_info_score(X_labels, Y_labels)
print(f"MI (sklearn, nats): {mi_sklearn:.4f}")

# --- Example 3: Feature selection with MI ---
from sklearn.datasets import load_iris

iris = load_iris()
mi_scores = mutual_info_classif(iris.data, iris.target, random_state=42)
for name, score in zip(iris.feature_names, mi_scores):
    print(f"  {name:20s}: MI = {score:.4f} nats")
```

**Output:**
```
I(X;Y) = 0.1245 bits
MI (sklearn, nats): 0.2781
  sepal length (cm)   : MI = 0.5124 nats
  sepal width (cm)    : MI = 0.2720 nats
  petal length (cm)   : MI = 0.9917 nats
  petal width (cm)    : MI = 1.0052 nats
```

Petal features have the highest MI with the target → most informative for classification.

---

## 9. MI Estimation Methods

For continuous variables, MI cannot be computed exactly from finite samples. Key estimation methods:

| Method                      | Description                                          |
|-----------------------------|------------------------------------------------------|
| **Binning / Histograms**    | Discretize continuous vars; simple but bin-sensitive  |
| **KDE-based**               | Kernel density estimation of p(x), p(y), p(x,y)     |
| **KSG Estimator**           | k-nearest neighbor method (Kraskov et al., 2004)     |
| **MINE**                    | Neural network MI estimator via Donsker-Varadhan bound|
| **InfoNCE**                 | Contrastive lower bound; used in CPC and SimCLR      |

### KSG Estimator (Most Popular)

```python
from sklearn.feature_selection import mutual_info_regression

# Estimates MI between continuous features and continuous target
X_continuous = np.random.randn(1000, 3)
y_continuous = X_continuous[:, 0]**2 + 0.5 * np.random.randn(1000)

mi_est = mutual_info_regression(X_continuous, y_continuous, random_state=42)
print("MI estimates:", np.round(mi_est, 3))
# Feature 0 will have high MI (nonlinear dependency), others ≈ 0
```

---

## 10. Feature Selection Using MI

MI-based feature selection ranks features by their shared information with the target:

```
Algorithm: MI Feature Selection
─────────────────────────────────
1. For each feature Xᵢ:
     Compute I(Xᵢ; Y) using KSG or binning
2. Rank features by I(Xᵢ; Y) descending
3. Select top-k features

Advanced: MRMR (Max-Relevance Min-Redundancy)
     Score(Xᵢ) = I(Xᵢ; Y) - (1/|S|) Σ I(Xᵢ; Xⱼ)
                                       Xⱼ∈S
     Maximizes relevance, minimizes redundancy among selected features
```

### Advantages Over Correlation-Based Selection

- Captures nonlinear feature-target relationships
- Works for both categorical and continuous features
- No assumption about data distribution

---

## 11. Real-World ML Applications

| Application                   | How MI Is Used                                        |
|-------------------------------|-------------------------------------------------------|
| **Feature Selection**         | Rank features by I(feature; target)                   |
| **Decision Trees**            | Information gain = MI between feature split and label  |
| **Contrastive Learning**      | SimCLR, MoCo maximize MI between augmented views      |
| **InfoGAN**                   | Maximize MI between latent codes and generated output  |
| **Representation Learning**   | Deep InfoMax maximizes MI between local/global repr.  |
| **Clustering**                | Normalized MI (NMI) measures clustering quality       |
| **Independent Component Analysis** | Minimize MI between components                  |
| **Channel Capacity**          | C = max_p(x) I(X; Y) — Shannon's channel coding theorem |

---

## 12. Normalized Mutual Information (NMI)

For comparing clusterings or measuring agreement:

```
              2 · I(X; Y)
NMI(X, Y) = ─────────────
              H(X) + H(Y)
```

- Range: [0, 1]
- NMI = 0: independent
- NMI = 1: perfect agreement

```python
from sklearn.metrics import normalized_mutual_info_score

labels_true = [0, 0, 1, 1, 2, 2]
labels_pred = [0, 0, 1, 2, 2, 2]
nmi = normalized_mutual_info_score(labels_true, labels_pred)
print(f"NMI = {nmi:.4f}")  # 0 to 1
```

---

## 13. Summary Table

| Concept               | Formula / Key Fact                                     |
|-----------------------|-------------------------------------------------------|
| Conditional Entropy   | H(X|Y) = H(X,Y) - H(Y)                              |
| Mutual Information    | I(X;Y) = H(X) + H(Y) - H(X,Y)                       |
| Alt. form (KL)        | I(X;Y) = D_KL(p(x,y) ‖ p(x)p(y))                    |
| Alt. form (cond.)     | I(X;Y) = H(X) - H(X|Y)                               |
| Symmetric             | I(X;Y) = I(Y;X)                                       |
| Bounds                | 0 ≤ I(X;Y) ≤ min(H(X), H(Y))                         |
| Independence          | I(X;Y) = 0 ⟺ X ⊥ Y                                  |
| Self-MI               | I(X;X) = H(X)                                         |
| NMI                   | 2·I(X;Y) / (H(X) + H(Y)), range [0,1]                |

---

## 14. Quick Revision Questions

1. **Define mutual information in terms of entropies.**
   > I(X;Y) = H(X) + H(Y) - H(X,Y), or equivalently H(X) - H(X|Y).

2. **Why is MI preferred over Pearson correlation for feature selection?**
   > MI captures all dependencies (including nonlinear), while Pearson correlation only detects linear relationships.

3. **If I(X;Y) = H(X), what can you conclude about X and Y?**
   > Y completely determines X — knowing Y leaves zero uncertainty about X (H(X|Y) = 0).

4. **What does the data processing inequality state?**
   > For a Markov chain X → Y → Z: I(X;Z) ≤ I(X;Y). Processing data can only lose information, never gain it.

5. **How is MI related to KL divergence?**
   > I(X;Y) = D_KL(p(x,y) ‖ p(x)p(y)) — it measures how far the joint distribution is from the product of marginals.

6. **What is NMI and when is it used?**
   > Normalized Mutual Information = 2·I(X;Y)/(H(X)+H(Y)), scaled to [0,1]. It's used to evaluate clustering quality against ground truth.

---

[← Previous Chapter: 03-KL-Divergence](./03-kl-divergence.md) | [Next Chapter: 05-Information-Gain →](./05-information-gain.md)
