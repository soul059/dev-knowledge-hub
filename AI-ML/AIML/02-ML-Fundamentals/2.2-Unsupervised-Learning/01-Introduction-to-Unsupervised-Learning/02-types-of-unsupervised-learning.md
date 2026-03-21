# 1.2 Types of Unsupervised Learning

> **Chapter Overview:** Unsupervised learning is not a single technique — it's an umbrella covering several distinct families of methods. This chapter provides a comprehensive tour of each major type: clustering, dimensionality reduction, density estimation, association rule mining, anomaly detection, and generative models. Understanding these categories is essential for choosing the right tool for your problem.

---

## Table of Contents

1. [The Unsupervised Learning Taxonomy](#1-the-unsupervised-learning-taxonomy)
2. [Clustering](#2-clustering)
3. [Dimensionality Reduction](#3-dimensionality-reduction)
4. [Density Estimation](#4-density-estimation)
5. [Association Rule Mining](#5-association-rule-mining)
6. [Anomaly Detection](#6-anomaly-detection)
7. [Generative Models](#7-generative-models)
8. [Comparison Table](#8-comparison-table)
9. [How They Relate — The Big Picture](#9-how-they-relate--the-big-picture)
10. [Python Quick-Start for Each Type](#10-python-quick-start-for-each-type)
11. [Key Takeaways](#11-key-takeaways)
12. [Revision Questions](#12-revision-questions)

---

## 1. The Unsupervised Learning Taxonomy

```
                    UNSUPERVISED LEARNING
                           │
        ┌──────────┬───────┼───────┬──────────┬──────────┐
        │          │       │       │          │          │
   CLUSTERING   DIM.    DENSITY  ASSOC.   ANOMALY   GENERATIVE
               REDUC.   ESTIM.   RULES   DETECTION   MODELS
        │          │       │       │          │          │
   ┌────┴────┐   ┌┴──┐  ┌─┴──┐  ┌┴──┐    ┌──┴──┐   ┌──┴──┐
   │         │   │   │  │    │  │   │    │     │   │     │
K-Means  Hier. PCA t-SNE KDE GMM Apri. Isol. VAE  GAN
DBSCAN   Spec. UMAP     Parzen    FP-Gr Forest
GMM      Agg.  Auto-           LOF     Diffusion
               encoder         One-SVM
```

Each family answers a **different question** about unlabeled data:

| Family                 | Core Question                                        |
|------------------------|------------------------------------------------------|
| Clustering             | "Which data points belong together?"                 |
| Dimensionality Reduction | "Can we represent data in fewer dimensions?"       |
| Density Estimation     | "What is the probability distribution of the data?" |
| Association Rules      | "Which items frequently co-occur?"                   |
| Anomaly Detection      | "Which points are abnormal?"                         |
| Generative Models      | "Can we generate new realistic data points?"         |

---

## 2. Clustering

### Definition

Clustering partitions data into **groups (clusters)** such that points within a group are more similar to each other than to points in other groups.

### Mathematical Objective

```
    Given: X = {x₁, x₂, ..., xₙ} ∈ ℝᵈ
    Find:  Assignment function  C: xᵢ → {1, 2, ..., K}

    Optimize some criterion, e.g.:

    K-Means (minimize inertia):
                    K
        J = Σ     Σ     ‖xᵢ - μₖ‖²
           k=1  xᵢ∈Cₖ

    Where μₖ = (1/|Cₖ|) Σ xᵢ   (centroid of cluster k)
                        xᵢ∈Cₖ
```

### Types of Clustering

```
    ┌───────────────────────────────────────────────────────┐
    │              CLUSTERING METHODS                       │
    │                                                       │
    │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │
    │  │ PARTITIONAL │  │ HIERARCHICAL │  │ DENSITY-     │  │
    │  │             │  │              │  │ BASED        │  │
    │  │ • K-Means   │  │ • Agglom.    │  │ • DBSCAN     │  │
    │  │ • K-Medoids │  │ • Divisive   │  │ • OPTICS     │  │
    │  │ • K-Modes   │  │ • BIRCH      │  │ • HDBSCAN    │  │
    │  └─────────────┘  └──────────────┘  └─────────────┘  │
    │                                                       │
    │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │
    │  │ MODEL-BASED │  │ SPECTRAL     │  │ FUZZY        │  │
    │  │             │  │              │  │              │  │
    │  │ • GMM       │  │ • Spectral   │  │ • Fuzzy      │  │
    │  │ • Bayesian  │  │   Clustering │  │   C-Means    │  │
    │  │   Mixture   │  │ • Graph cuts │  │ • Possib.    │  │
    │  └─────────────┘  └──────────────┘  └─────────────┘  │
    └───────────────────────────────────────────────────────┘
```

### Example: Customer Segmentation

```
    Annual Income ($K)
    │
    80│          ■ ■ ■              High Income,
    70│        ■ ■ ■ ■             High Spending
    60│──────────────────────────
    50│  ● ● ●         ▲ ▲ ▲      
    40│  ● ● ●         ▲ ▲ ▲      ● Low Inc, Low Spend
    30│  ● ● ●         ▲ ▲ ▲      ▲ Low Inc, High Spend
    20│                            ■ High Inc, High Spend
    10│
      └──────────────────────────→ Spending Score
        10  20  30  40  50  60  70  80  90
```

### Key Properties

- **Hard vs Soft:** K-Means assigns each point to exactly one cluster (hard); GMM gives probabilities (soft).
- **K required vs not:** K-Means needs K; DBSCAN finds it automatically.
- **Shape assumptions:** K-Means assumes spherical clusters; DBSCAN handles arbitrary shapes.

---

## 3. Dimensionality Reduction

### Definition

Reduce the number of features (dimensions) while preserving as much **information** or **structure** as possible.

### Why Reduce Dimensions?

```
    Original: x ∈ ℝ¹⁰⁰⁰  →  Reduced: z ∈ ℝ²

    Benefits:
    ✓ Visualization (project to 2D/3D)
    ✓ Noise removal (low-variance dimensions are often noise)
    ✓ Computational efficiency (fewer features = faster training)
    ✓ Mitigate curse of dimensionality
    ✓ Feature extraction / representation learning
```

### Mathematical Framework (PCA)

```
    Principal Component Analysis (PCA):

    1. Center data:  X̃ = X - μ

    2. Covariance matrix:  C = (1/n) X̃ᵀX̃

    3. Eigendecomposition:  C = VΛVᵀ

    4. Project:  Z = X̃ · V_k    (keep top-k eigenvectors)

    Variance preserved = Σᵢ₌₁ᵏ λᵢ / Σᵢ₌₁ᵈ λᵢ
```

### Landscape of Methods

| Method       | Linear? | Preserves     | Best For               |
|-------------|---------|---------------|------------------------|
| **PCA**     | Yes     | Global variance | General-purpose        |
| **t-SNE**   | No      | Local structure | Visualization (2D/3D) |
| **UMAP**    | No      | Local + global  | Visualization + speed  |
| **Autoencoder** | No  | Learned features | Complex nonlinear     |
| **LDA**     | Yes     | Class separation | Supervised dim. red.  |
| **ICA**     | Yes     | Independence    | Signal separation      |

### ASCII: PCA Projection

```
    Original 2D data:              After PCA to 1D:
         y                              PC1
         │    · · ·                      │
         │  · · · · ·                    │·····│····│····│
         │· · · · · · ·         →        ├─────┤────┤────┤
         │  · · · · ·                    └─────────────────
         │    · · ·
         └──────────── x

    The first principal component captures the direction
    of maximum variance in the data.
```

---

## 4. Density Estimation

### Definition

Estimate the **probability density function (PDF)** p(x) from data samples, without assuming a specific parametric form.

### Mathematical Formulation

#### Parametric (assumes a distribution form)

```
    Gaussian:  p(x; μ, σ²) = (1/√(2πσ²)) exp(-(x-μ)²/(2σ²))

    Fit by Maximum Likelihood:
        μ̂ = (1/n) Σ xᵢ       σ̂² = (1/n) Σ (xᵢ - μ̂)²
```

#### Non-Parametric (Kernel Density Estimation)

```
    KDE:  p̂(x) = (1/nh) Σᵢ₌₁ⁿ K((x - xᵢ)/h)

    Where:
        K(u) = kernel function (e.g., Gaussian kernel = (1/√2π) e^(-u²/2))
        h    = bandwidth (smoothing parameter)
        n    = number of data points

    Small h → jagged estimate (overfitting)
    Large h → over-smooth (underfitting)
```

### ASCII: KDE Visualization

```
    p(x)
     │         ╱╲
     │        ╱  ╲       ╱╲
     │   ╱╲  ╱    ╲     ╱  ╲
     │  ╱  ╲╱      ╲   ╱    ╲
     │ ╱            ╲ ╱      ╲
     │╱              ╲        ╲
     └────────────────────────── x
     │ │  │ │ │ │   │  │ │  │ │  ← data points (rug plot)
```

### Applications

- **Novelty detection:** If p(x_new) < threshold → anomaly
- **Bayesian inference:** Use as prior or likelihood
- **Data visualization:** Understand data distribution shape

---

## 5. Association Rule Mining

### Definition

Discover interesting **relationships (associations)** between variables in large transactional datasets.

### Key Concepts

```
    Transaction Database:
    ┌─────┬──────────────────────────┐
    │ TID │ Items                    │
    ├─────┼──────────────────────────┤
    │ 1   │ {Bread, Butter, Milk}    │
    │ 2   │ {Bread, Diaper, Beer}    │
    │ 3   │ {Milk, Diaper, Beer}     │
    │ 4   │ {Bread, Milk, Diaper}    │
    │ 5   │ {Bread, Butter, Diaper}  │
    └─────┴──────────────────────────┘

    Rule:  {Bread, Butter} → {Milk}
```

### Metrics

```
    Support(A → B)    = P(A ∪ B) = Count(A ∪ B) / N

    Confidence(A → B) = P(B | A) = Support(A ∪ B) / Support(A)

    Lift(A → B)       = Confidence(A → B) / Support(B)
                       = P(A ∪ B) / (P(A) · P(B))

    Interpretation:
    • Lift > 1  → A and B co-occur more than expected (positive association)
    • Lift = 1  → Independent
    • Lift < 1  → Negative association
```

### Common Algorithms

| Algorithm     | Key Idea                           | Complexity        |
|--------------|-------------------------------------|-------------------|
| **Apriori**  | Level-wise candidate generation     | Exponential worst  |
| **FP-Growth**| Frequent Pattern tree (no candidates)| Much faster       |
| **Eclat**    | Vertical data format + set intersection | Efficient      |

---

## 6. Anomaly Detection

### Definition

Identify **rare observations** that deviate significantly from the majority of data. Also called **outlier detection** or **novelty detection**.

### Approaches

```
    ┌──────────────────────────────────────────────────┐
    │             ANOMALY DETECTION METHODS             │
    │                                                   │
    │  Statistical:                                     │
    │    • Z-score (|x - μ| / σ > threshold)           │
    │    • Grubbs' test, Dixon's Q test                │
    │                                                   │
    │  Distance-based:                                  │
    │    • k-NN distance                               │
    │    • Local Outlier Factor (LOF)                   │
    │                                                   │
    │  Density-based:                                   │
    │    • KDE-based (low density → anomaly)           │
    │    • DBSCAN (noise points = anomalies)           │
    │                                                   │
    │  Model-based:                                     │
    │    • One-Class SVM                               │
    │    • Isolation Forest                            │
    │    • Autoencoder (high reconstruction error)     │
    │                                                   │
    │  Ensemble:                                        │
    │    • Feature bagging                             │
    │    • Isolation Forest (inherently ensemble)      │
    └──────────────────────────────────────────────────┘
```

### ASCII: Isolation Forest Intuition

```
    Normal points need MANY splits       Anomalies need FEW splits
    to isolate:                          to isolate:

         ┌───────────┐                        ┌───────────┐
         │ ○○○○○○○○  │                        │ ○○○○○○○○  │
         │ ○○○○○○○○  │                        │ ○○○○○○○○  │
         │ ○○○○●○○○  │ ← many splits         │ ○○○○○○○○  │
         │ ○○○○○○○○  │    needed              │ ○○○○○○○○  │
         │ ○○○○○○○○  │                        │          ★│ ← one split!
         └───────────┘                        └───────────┘

    Average path length for anomalies is SHORT → anomaly score is HIGH
```

---

## 7. Generative Models

### Definition

Learn the **joint distribution p(x)** or a mechanism to **generate new data points** that resemble the training data.

### Why Generative?

```
    Discriminative: p(y | x)  → "Given this image, is it a cat?"
    Generative:     p(x)      → "What does a typical cat image look like?"
                    p(x, y)   → "Generate a cat image"
```

### Major Approaches

#### Variational Autoencoders (VAE)

```
    Input x → Encoder → μ, σ (latent distribution) → Sample z → Decoder → x̂

    Loss = Reconstruction Loss + KL Divergence
         = ‖x - x̂‖² + KL(q(z|x) ‖ p(z))

    ┌───────┐     ┌───────────┐     ┌───────┐     ┌───────────┐     ┌───────┐
    │ Input │ →   │  Encoder  │ →   │  z~   │ →   │  Decoder  │ →   │ Output│
    │  x    │     │ q(z | x)  │     │N(μ,σ²)│     │ p(x | z)  │     │  x̂   │
    └───────┘     └───────────┘     └───────┘     └───────────┘     └───────┘
```

#### Generative Adversarial Networks (GAN)

```
    Random noise z → Generator G(z) → Fake image x_fake
                                            │
                                     ┌──────┴──────┐
                                     │ Discriminator│
                                     │   D(x)      │
                                     └──────┬──────┘
                                            │
    Real image x_real ──────────────────────┘

    min max  E[log D(x)] + E[log(1 - D(G(z)))]
     G   D
```

#### Diffusion Models

```
    Forward process:  x₀ → x₁ → x₂ → ... → xₜ  (add noise step by step)
    Reverse process:  xₜ → xₜ₋₁ → ... → x₀     (denoise step by step)

    Learn the reverse process to generate new samples from pure noise.
    Used in: DALL-E, Stable Diffusion, Midjourney
```

---

## 8. Comparison Table

```
┌───────────────────┬───────────────┬──────────────────┬────────────────────┬──────────────────┐
│ Type              │ Input         │ Output           │ Key Algorithms     │ Example Use      │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Clustering        │ Feature matrix│ Cluster labels   │ K-Means, DBSCAN,   │ Customer         │
│                   │               │                  │ Hierarchical, GMM  │ segmentation     │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Dim. Reduction    │ High-dim data │ Low-dim embed.   │ PCA, t-SNE, UMAP,  │ Visualization,   │
│                   │               │                  │ Autoencoders       │ compression      │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Density Estim.    │ Data samples  │ PDF p̂(x)        │ KDE, GMM,          │ Probability      │
│                   │               │                  │ Histogram          │ modeling         │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Association Rules │ Transactions  │ Rules {A}→{B}    │ Apriori, FP-Growth │ Market basket    │
│                   │               │                  │                    │ analysis         │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Anomaly Detection │ Feature matrix│ Anomaly scores   │ Isolation Forest,  │ Fraud detection  │
│                   │               │ or labels        │ LOF, One-Class SVM │                  │
├───────────────────┼───────────────┼──────────────────┼────────────────────┼──────────────────┤
│ Generative Models │ Data samples  │ New synthetic    │ VAE, GAN,          │ Image generation,│
│                   │               │ samples          │ Diffusion Models   │ data augment.    │
└───────────────────┴───────────────┴──────────────────┴────────────────────┴──────────────────┘
```

---

## 9. How They Relate — The Big Picture

```
                    Unsupervised Learning
                           │
                    Learn p(x) or structure
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    STRUCTURE         DISTRIBUTION       GENERATION
    DISCOVERY         ESTIMATION         (Synthesis)
         │                 │                 │
    ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
    │         │       │         │       │         │
 Cluster   Reduce   Density  Anomaly   VAE      GAN
    │         │       │    (tails of   │         │
    │         │       │     density)   │         │
    │    Can feed     │         │      └────┬────┘
    │    into each    │         │           │
    │    other!       │         │      Generate new
    │         │       │         │      data that
    └────┬────┘       └────┬────┘      follows p(x)
         │                 │
    PCA → K-Means    KDE → Anomaly
    (reduce dims     (low density
     then cluster)    = outlier)
```

### Cross-Pollination Examples

1. **PCA + K-Means:** Reduce dimensions first, then cluster (handles high-d data better).
2. **GMM as both clustering AND density estimation:** Assigns cluster probabilities AND models p(x).
3. **Autoencoder for both dim. reduction AND anomaly detection:** High reconstruction error = anomaly.
4. **DBSCAN for both clustering AND anomaly detection:** "Noise" points = anomalies.

---

## 10. Python Quick-Start for Each Type

### Clustering (K-Means)

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

X, _ = make_blobs(n_samples=300, centers=4, random_state=42)
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
labels = kmeans.fit_predict(X)
print(f"Cluster sizes: {np.bincount(labels)}")
```

### Dimensionality Reduction (PCA)

```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits

X, y = load_digits(return_X_y=True)  # 64 features
pca = PCA(n_components=2)
X_2d = pca.fit_transform(X)
print(f"Variance explained: {pca.explained_variance_ratio_.sum():.2%}")
```

### Density Estimation (KDE)

```python
from sklearn.neighbors import KernelDensity
import numpy as np

X = np.random.randn(1000, 1)
kde = KernelDensity(kernel='gaussian', bandwidth=0.5)
kde.fit(X)
# Score = log-density
log_density = kde.score_samples(np.linspace(-4, 4, 100).reshape(-1, 1))
```

### Association Rules (mlxtend)

```python
# pip install mlxtend
from mlxtend.frequent_patterns import apriori, association_rules
import pandas as pd

# One-hot encoded transaction data
df = pd.DataFrame({
    'Bread': [1, 1, 0, 1, 1],
    'Butter': [1, 0, 0, 0, 1],
    'Milk': [1, 0, 1, 1, 0],
    'Diaper': [0, 1, 1, 1, 1],
    'Beer': [0, 1, 1, 0, 0]
})
frequent = apriori(df, min_support=0.4, use_colnames=True)
rules = association_rules(frequent, metric="lift", min_threshold=1.0)
print(rules[['antecedents', 'consequents', 'support', 'confidence', 'lift']])
```

### Anomaly Detection (Isolation Forest)

```python
from sklearn.ensemble import IsolationForest
import numpy as np

np.random.seed(42)
X_normal = np.random.randn(300, 2)
X_anomaly = np.random.uniform(low=-6, high=6, size=(15, 2))
X = np.vstack([X_normal, X_anomaly])

iso = IsolationForest(contamination=0.05, random_state=42)
predictions = iso.fit_predict(X)  # 1 = normal, -1 = anomaly
print(f"Anomalies detected: {(predictions == -1).sum()}")
```

### Generative Model (Simple VAE concept with sklearn's PCA inverse)

```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
import numpy as np

X, _ = load_digits(return_X_y=True)
pca = PCA(n_components=10)
X_compressed = pca.fit_transform(X)

# "Generate" by sampling in latent space and decoding
z_sample = np.random.randn(5, 10) * X_compressed.std(axis=0)
X_generated = pca.inverse_transform(z_sample)
print(f"Generated {X_generated.shape[0]} samples of shape {X_generated.shape[1]}")
```

---

## 11. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Unsupervised learning encompasses **6+ major families**, each answering a different question |
| 2 | **Clustering** groups similar data; **dimensionality reduction** compresses data |
| 3 | **Density estimation** models p(x); **anomaly detection** finds low-density points |
| 4 | **Association rules** find co-occurrence patterns; **generative models** create new data |
| 5 | Many methods serve **dual purposes** (GMM = clustering + density; DBSCAN = clustering + anomaly) |
| 6 | Methods are often **combined** in pipelines (PCA → K-Means, Autoencoder → Anomaly) |

---

## 12. Revision Questions

1. **Taxonomy:** List the six main types of unsupervised learning discussed in this chapter. For each, give one sentence describing its goal and one example algorithm.

2. **Clustering vs Classification:** Explain why clustering is unsupervised while classification is supervised, even though both produce categorical outputs.

3. **Density Estimation:** What is the difference between parametric and non-parametric density estimation? Give an example of each.

4. **Association Rules:** Define support, confidence, and lift for the rule {Bread} → {Milk}. What does a lift value of 2.5 mean?

5. **Generative Models:** Compare VAEs and GANs in terms of (a) training objective, (b) architecture, and (c) quality of generated samples.

6. **Cross-Pollination:** Describe two real-world scenarios where you would combine methods from different unsupervised learning families in a single pipeline.

---

<div align="center">

| [← Supervised vs Unsupervised](./01-supervised-vs-unsupervised.md) | [Up: Introduction](./README.md) | [Next: Applications & Use Cases →](./03-applications-and-use-cases.md) |
|:------------------------------------------------------------------:|:-------------------------------:|:---------------------------------------------------------------------:|

</div>
