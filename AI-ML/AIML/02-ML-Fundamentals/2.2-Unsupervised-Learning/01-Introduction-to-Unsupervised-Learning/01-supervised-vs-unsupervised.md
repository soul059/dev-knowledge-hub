# 1.1 Supervised vs Unsupervised Learning

> **Chapter Overview:** This chapter draws a clear line between learning paradigms—supervised, unsupervised, semi-supervised, and self-supervised—by examining how the presence or absence of labels fundamentally changes the learning objective, the algorithms we choose, and the way we evaluate results.

---

## Table of Contents

1. [The Role of Labels in Machine Learning](#1-the-role-of-labels-in-machine-learning)
2. [Supervised Learning Recap](#2-supervised-learning-recap)
3. [Unsupervised Learning — The Core Idea](#3-unsupervised-learning--the-core-idea)
4. [Side-by-Side Paradigm Comparison](#4-side-by-side-paradigm-comparison)
5. [When No Labels Are Available](#5-when-no-labels-are-available)
6. [Semi-Supervised Learning](#6-semi-supervised-learning)
7. [Self-Supervised Learning](#7-self-supervised-learning)
8. [Mathematical Framing](#8-mathematical-framing)
9. [Python Example — Visualizing Both Paradigms](#9-python-example--visualizing-both-paradigms)
10. [Applications Summary Table](#10-applications-summary-table)
11. [Key Takeaways](#11-key-takeaways)
12. [Revision Questions](#12-revision-questions)

---

## 1. The Role of Labels in Machine Learning

A **label** (or **target** / **response variable**) is the "answer" associated with each training example. Its presence or absence determines the entire learning pipeline:

```
┌──────────────────────────────────────────────────────────────────┐
│                      TRAINING DATA                               │
│                                                                  │
│   Features (X)           Label (y)         Paradigm              │
│   ─────────────          ──────────        ──────────────        │
│   [5.1, 3.5, 1.4, 0.2]  "Setosa"     →   Supervised            │
│   [5.1, 3.5, 1.4, 0.2]  ???          →   Unsupervised          │
│   [5.1, 3.5, 1.4, 0.2]  "Setosa"  ┐                            │
│   [6.7, 3.0, 5.2, 2.3]  ???       ├─→   Semi-Supervised        │
│   [4.9, 3.0, 1.4, 0.2]  ???       ┘                            │
└──────────────────────────────────────────────────────────────────┘
```

**Key insight:** In real-world datasets, labeling is expensive—medical images need expert radiologists, text documents need domain annotators, and sensor data may have no natural label at all. This economic reality is one of the strongest motivations for unsupervised learning.

---

## 2. Supervised Learning Recap

In supervised learning, we are given a dataset **D = {(x₁, y₁), (x₂, y₂), …, (xₙ, yₙ)}** and the goal is to learn a mapping **f: X → Y** that minimizes some loss function.

### Objective

```
                      n
    min   L(θ) =  Σ  ℓ(f(xᵢ; θ), yᵢ)
     θ           i=1

Where:
    xᵢ ∈ ℝᵈ       — feature vector
    yᵢ              — known label (continuous for regression, discrete for classification)
    f(x; θ)         — model with parameters θ
    ℓ(ŷ, y)         — loss function (MSE, cross-entropy, etc.)
```

### Subtypes

| Subtype            | Label Type   | Example                        | Common Algorithms             |
|--------------------|-------------|--------------------------------|-------------------------------|
| **Classification** | Discrete     | Spam / Not Spam                | Logistic Reg., SVM, Trees, NN |
| **Regression**     | Continuous   | House price prediction         | Linear Reg., Ridge, SVR       |

### Evaluation

Because labels are known, we can directly measure **accuracy, precision, recall, F1, MSE, R²**, etc.

---

## 3. Unsupervised Learning — The Core Idea

In unsupervised learning, we only have **D = {x₁, x₂, …, xₙ}** — no labels. The goal shifts from prediction to **discovering hidden structure**.

### What "Structure" Means

```
┌─────────────────────────────────────────────────────────────┐
│              HIDDEN STRUCTURE IN DATA                        │
│                                                             │
│   ┌──────────┐   ┌──────────────┐   ┌──────────────────┐   │
│   │ CLUSTERS │   │ LOW-DIM      │   │ DENSITY          │   │
│   │ (groups) │   │ MANIFOLDS    │   │ ESTIMATION       │   │
│   │          │   │ (compression)│   │ (prob. landscape)│   │
│   │  ○ ○ ○   │   │   ↙         │   │   ∧              │   │
│   │  ○ ○     │   │  ↙  PCA     │   │  / \  p(x)      │   │
│   │    ● ● ● │   │ ↙           │   │ /   \            │   │
│   │    ● ●   │   │              │   │/     \____       │   │
│   └──────────┘   └──────────────┘   └──────────────────┘   │
│                                                             │
│   ┌──────────────┐   ┌──────────────────────────────────┐   │
│   │ ASSOCIATION  │   │ ANOMALY DETECTION                │   │
│   │ RULES        │   │ (what doesn't fit?)              │   │
│   │ {A} → {B}    │   │   ○ ○ ○ ○ ○ ○                   │   │
│   │ bread→butter │   │   ○ ○ ○ ○        ★ ← outlier    │   │
│   └──────────────┘   └──────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Objective (General)

There is no single loss function. Instead, we optimize proxies:

```
Clustering:        min  Σ  Σ  ‖xᵢ - μₖ‖²          (within-cluster sum of squares)
                    μ  k  xᵢ∈Cₖ

Dimensionality     max  Var(Xw)  s.t. ‖w‖ = 1      (PCA: maximize variance)
Reduction:          w

Density            max  Σ  log p(xᵢ; θ)             (maximum likelihood)
Estimation:         θ  i
```

---

## 4. Side-by-Side Paradigm Comparison

```
┌────────────────────────┬──────────────────────────┬──────────────────────────┐
│      Aspect            │   SUPERVISED             │   UNSUPERVISED           │
├────────────────────────┼──────────────────────────┼──────────────────────────┤
│ Training Data          │ (X, y) — labeled         │ X only — no labels       │
│ Goal                   │ Predict y from x         │ Discover structure in X  │
│ Feedback Signal        │ Explicit (error on y)    │ Implicit (internal       │
│                        │                          │   criteria)              │
│ Evaluation             │ Accuracy, MSE, F1, AUC   │ Silhouette, inertia,     │
│                        │                          │   reconstruction error   │
│ Example Tasks          │ Classification,          │ Clustering, PCA,         │
│                        │ Regression               │ Anomaly Detection        │
│ Algorithms             │ SVM, Trees, Linear Reg., │ K-Means, DBSCAN, PCA,   │
│                        │ Neural Networks          │ t-SNE, Autoencoders      │
│ Overfitting Check      │ Train/Val/Test split     │ No standard split;       │
│                        │                          │   use stability analysis │
│ Data Labeling Cost     │ HIGH (expert required)   │ LOW (no labels needed)   │
│ Typical Output         │ Predicted label/value    │ Cluster IDs, embeddings, │
│                        │                          │   transformed data       │
│ Human Involvement      │ Before training (label)  │ After training (interp.) │
└────────────────────────┴──────────────────────────┴──────────────────────────┘
```

### ASCII Decision Flow

```
        START: Do you have labeled data?
                    │
          ┌─── YES ─┤── NO ───┐
          │         │          │
          ▼         │          ▼
    Supervised      │    Unsupervised
    Learning        │    Learning
          │         │          │
    ┌─────┴─────┐   │    ┌─────┴──────────┐
    │           │   │    │                │
Classification  Regression  Clustering   Dim. Reduction
                    │
              Do you have SOME labels?
                    │
              ┌─── YES ───┐
              │            │
              ▼            ▼
        Semi-Supervised   Self-Supervised
        Learning          Learning
```

---

## 5. When No Labels Are Available

### Common Scenarios

1. **Exploratory Data Analysis (EDA):** You've just received a new dataset and want to understand its structure before deciding on a supervised task.

2. **Prohibitive Labeling Cost:** Medical imaging (each scan needs a radiologist), satellite imagery (millions of pixels), genomics (wet-lab validation per gene).

3. **Naturally Unlabeled Data:** Web logs, IoT sensor streams, social media posts — they arrive without a "correct answer."

4. **Preprocessing for Supervised Learning:** Reduce dimensions (PCA), create features (clustering IDs as new features), or detect and remove outliers before training a classifier.

5. **Privacy Constraints:** Labels may exist but cannot be shared (e.g., patient diagnoses under HIPAA).

### The Annotation Cost Problem

```
    Cost per label (approximate):

    Task                          Cost/Label    Time/Label
    ─────────────────────────     ──────────    ──────────
    Sentiment (text)              $0.05–0.10    10 sec
    Image classification          $0.10–0.50    15 sec
    Object detection (bbox)       $0.50–2.00    30 sec
    Medical image diagnosis        $5–50        5 min
    Semantic segmentation          $5–15        10 min
    Autonomous driving (3D)        $10–100      30 min

    → If you have 1M unlabeled images and labeling costs $1/image,
      that's $1,000,000 just for labels!
```

---

## 6. Semi-Supervised Learning

Semi-supervised learning sits between supervised and unsupervised. It uses a **small amount of labeled data** together with a **large amount of unlabeled data**.

### Core Assumption

The underlying data distribution **p(x)** carries information about the decision boundary. Two key assumptions:

1. **Smoothness Assumption:** Points close in feature space should share labels.
2. **Cluster Assumption:** Points in the same cluster likely share a label.
3. **Manifold Assumption:** High-dimensional data lies on a lower-dimensional manifold.

### How It Works (Conceptual)

```
    LABELED:       ● ● ●  (class A)        ■ ■  (class B)
    UNLABELED:     ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○

    Step 1: Train initial model on labeled data only
    Step 2: Use model to predict labels for unlabeled data
    Step 3: Add high-confidence predictions to training set
    Step 4: Retrain → Repeat (self-training / pseudo-labeling)

    After convergence:
    ● ● ● ● ● ● ● ●  (class A)        ■ ■ ■ ■ ■ ■ ■  (class B)
```

### Common Techniques

- **Self-training / Pseudo-labeling**
- **Label propagation** (graph-based)
- **Generative models** (EM with mixture models)
- **Consistency regularization** (modern deep learning: MixMatch, FixMatch)

---

## 7. Self-Supervised Learning

Self-supervised learning creates **supervisory signals from the data itself** — no human annotation needed.

### The Pretext Task Idea

```
    ┌──────────────────────────────────────────────────┐
    │  Original Data  →  Create Artificial Labels      │
    │                                                  │
    │  Image:    Rotate 90° → Predict rotation angle   │
    │  Text:     Mask word  → Predict masked word      │
    │  Video:    Shuffle frames → Predict order        │
    │  Audio:    Mask segment → Predict segment        │
    └──────────────────────────────────────────────────┘

    The model learns useful REPRESENTATIONS while solving
    these pretext tasks, even without real labels.
```

### Notable Examples

| Domain   | Method       | Pretext Task                     | Downstream Use           |
|----------|-------------|----------------------------------|--------------------------|
| NLP      | BERT         | Masked Language Modeling         | Classification, QA, NER  |
| NLP      | GPT          | Next-token prediction            | Generation, Few-shot     |
| Vision   | SimCLR       | Contrastive learning on augments | Image classification     |
| Vision   | MAE          | Masked image patches             | Object detection         |

### Relationship to Unsupervised Learning

Self-supervised learning is technically unsupervised (no human labels), but it's distinguished by:
- Learning **representations** rather than structure
- Using **pretext tasks** that mimic supervised objectives
- Typically applied in **deep learning** contexts

---

## 8. Mathematical Framing

### Supervised Learning — Conditional Distribution

We model **p(y | x; θ)** — the conditional probability of the label given features:

```
    Classification:    p(y = k | x; θ) = softmax(f(x; θ))ₖ

    Regression:        p(y | x; θ) = N(y; f(x; θ), σ²)

    Objective:         max  Σ  log p(yᵢ | xᵢ; θ)        (MLE)
                        θ  i=1
```

### Unsupervised Learning — Marginal Distribution

We model **p(x; θ)** — the data distribution itself:

```
    Density Estimation:   max  Σ  log p(xᵢ; θ)
                           θ  i=1

    Latent Variable Model: p(x; θ) = ∫ p(x | z; θ) p(z) dz

    Where z is a latent (hidden) variable — e.g., cluster identity,
    latent factors, compressed representation.
```

### The Key Difference

```
    Supervised:    Learn  f: X → Y        (input → output mapping)
    Unsupervised:  Learn  p(X)            (data distribution)
                   or     g: X → Z        (input → latent space)
```

---

## 9. Python Example — Visualizing Both Paradigms

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs
from sklearn.linear_model import LogisticRegression
from sklearn.cluster import KMeans

# Generate synthetic data with 3 clusters
X, y_true = make_blobs(n_samples=300, centers=3, 
                         cluster_std=1.0, random_state=42)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# ---- Panel 1: Raw data (no labels visible) ----
axes[0].scatter(X[:, 0], X[:, 1], c='gray', alpha=0.6, edgecolors='k', s=40)
axes[0].set_title("Raw Unlabeled Data", fontsize=14)
axes[0].set_xlabel("Feature 1")
axes[0].set_ylabel("Feature 2")

# ---- Panel 2: Supervised Learning (if labels are available) ----
clf = LogisticRegression(max_iter=1000)
clf.fit(X, y_true)
y_pred_sup = clf.predict(X)
axes[1].scatter(X[:, 0], X[:, 1], c=y_true, cmap='viridis', 
                alpha=0.6, edgecolors='k', s=40)
axes[1].set_title(f"Supervised (Logistic Reg.) — Acc: {clf.score(X, y_true):.2f}", 
                  fontsize=14)
axes[1].set_xlabel("Feature 1")

# ---- Panel 3: Unsupervised Learning (no labels used) ----
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
y_pred_unsup = kmeans.fit_predict(X)
axes[2].scatter(X[:, 0], X[:, 1], c=y_pred_unsup, cmap='viridis', 
                alpha=0.6, edgecolors='k', s=40)
centers = kmeans.cluster_centers_
axes[2].scatter(centers[:, 0], centers[:, 1], c='red', marker='X', 
                s=200, edgecolors='k', linewidths=2, label='Centroids')
axes[2].set_title("Unsupervised (K-Means)", fontsize=14)
axes[2].set_xlabel("Feature 1")
axes[2].legend()

plt.tight_layout()
plt.savefig("supervised_vs_unsupervised.png", dpi=150)
plt.show()
```

### Key Observations from the Code

- **Supervised (Panel 2):** Uses `y_true` labels during training → can measure accuracy directly.
- **Unsupervised (Panel 3):** Uses `X` only → discovers clusters without labels → cluster IDs may not match true labels (label permutation problem).

---

## 10. Applications Summary Table

| Paradigm          | When to Use                                    | Example Application              |
|-------------------|------------------------------------------------|----------------------------------|
| Supervised        | Labeled data is abundant and task is well-defined | Email spam detection            |
| Unsupervised      | No labels; goal is exploration or structure     | Customer segmentation            |
| Semi-Supervised   | Few labels + lots of unlabeled data            | Medical image classification     |
| Self-Supervised   | Large unlabeled corpus; want pretrained features | BERT pretraining on text        |

---

## 11. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Supervised learning requires labeled data **(X, y)** and optimizes a prediction loss |
| 2 | Unsupervised learning works with **X** only and discovers hidden structure |
| 3 | The choice of paradigm is driven by **data availability**, not algorithm preference |
| 4 | Semi-supervised bridges the gap when only **a few labels** are available |
| 5 | Self-supervised creates **artificial labels** from the data itself (e.g., BERT, GPT) |
| 6 | Unsupervised learning evaluation is **harder** — no ground truth to compare against |
| 7 | In practice, paradigms are often **combined** (e.g., PCA for preprocessing → SVM for classification) |

---

## 12. Revision Questions

1. **Conceptual:** Explain the fundamental difference between the objective function in supervised vs unsupervised learning. Why does the absence of labels change what we optimize?

2. **Practical:** You have 10 million customer transaction records with no labels. Name three unsupervised learning tasks you could perform on this data and what business value each would provide.

3. **Mathematical:** Write out the maximum likelihood objective for (a) supervised classification and (b) unsupervised density estimation. How do they differ?

4. **Semi-Supervised:** A hospital has 500 labeled MRI scans and 50,000 unlabeled ones. Describe how semi-supervised learning could leverage both. What assumptions must hold?

5. **Self-Supervised:** Explain how BERT uses self-supervision. What is the pretext task, and why does solving it produce useful representations for downstream tasks?

6. **Critical Thinking:** "Unsupervised learning is always harder than supervised learning." Do you agree or disagree? Justify your answer with specific examples.

---

<div align="center">

| ← Previous | [Up: Introduction to Unsupervised Learning](./README.md) | [Next: Types of Unsupervised Learning →](./02-types-of-unsupervised-learning.md) |
|:-----------:|:--------------------------------------------------------:|:--------------------------------------------------------------------------------:|

</div>
