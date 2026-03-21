# 📘 Chapter 2: Supervised, Unsupervised & Reinforcement Learning

> **Unit 2.1 – Introduction to Machine Learning** | File 2 of 7

---

## 📋 Overview

This chapter provides an **in-depth comparison** of the three core ML paradigms:
Supervised, Unsupervised, and Reinforcement Learning. You will learn how each
paradigm works, what kind of data it requires, the key sub-categories within each,
and see practical Python implementations.

---

## 1. Supervised Learning — Learning from Labels

### 1.1 How It Works

The model receives a training set of **input-output pairs** `{(x₁, y₁), (x₂, y₂), …, (xₙ, yₙ)}`
and learns a function `f` such that `f(xᵢ) ≈ yᵢ`.

```
   Training Data                    Model
  ┌────────────────┐              ┌──────────┐
  │ x₁ → y₁       │              │          │
  │ x₂ → y₂       │──  train  ──▶│  f(x;θ)  │──▶  ŷ = prediction
  │ ...            │              │          │
  │ xₙ → yₙ       │              └──────────┘
  └────────────────┘
```

**Loss function** measures how far predictions are from true labels:

```
  Mean Squared Error (Regression):    MSE = (1/n) Σ (yᵢ - ŷᵢ)²
  Cross-Entropy (Classification):    L = -Σ yᵢ log(ŷᵢ)
```

### 1.2 Classification vs Regression

| Aspect | Classification | Regression |
|--------|---------------|------------|
| **Output** | Discrete class labels | Continuous numerical value |
| **Example** | Is this email spam? (Yes/No) | What is the house price? ($350K) |
| **Loss metric** | Accuracy, F1, AUC-ROC | MSE, MAE, R² |
| **Algorithms** | Logistic Regression, SVM, Random Forest | Linear Regression, SVR, XGBoost |
| **Decision boundary** | Separating hyperplane or regions | Best-fit curve or surface |

### 1.3 Python Example — Supervised Classification

```python
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

X, y = load_wine(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

clf = RandomForestClassifier(n_estimators=100, random_state=42)
clf.fit(X_train, y_train)
print(classification_report(y_test, clf.predict(X_test), target_names=load_wine().target_names))
```

---

## 2. Unsupervised Learning — Finding Hidden Structure

### 2.1 How It Works

No labels are provided. The model discovers **patterns, groupings, or compressed
representations** from raw data `{x₁, x₂, …, xₙ}`.

```
   Unlabeled Data                   Model
  ┌────────────────┐              ┌──────────────┐
  │ x₁             │              │              │
  │ x₂             │──  train  ──▶│  Discover    │──▶  clusters,
  │ ...            │              │  Structure   │     components,
  │ xₙ             │              │              │     anomalies
  └────────────────┘              └──────────────┘
```

### 2.2 Clustering vs Dimensionality Reduction

| Aspect | Clustering | Dimensionality Reduction |
|--------|-----------|-------------------------|
| **Goal** | Group similar data points together | Reduce number of features while preserving information |
| **Output** | Cluster assignments | Lower-dimensional representation |
| **Algorithms** | K-Means, DBSCAN, Hierarchical | PCA, t-SNE, UMAP, Autoencoders |
| **Use case** | Customer segmentation | Data visualization, noise removal |
| **Evaluation** | Silhouette score, inertia | Explained variance ratio |

**K-Means Objective:** `Minimize J = Σₖ Σ_{xᵢ ∈ Cₖ} ‖xᵢ - μₖ‖²` where μₖ = centroid of cluster Cₖ

**PCA Objective:** `Maximize Var(Xw) = wᵀΣw` subject to `‖w‖ = 1`, where Σ = covariance matrix

### 2.3 Python Example — K-Means Clustering

```python
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans
import numpy as np

# Generate unlabeled data with 3 natural clusters
X, _ = make_blobs(n_samples=300, centers=3, cluster_std=0.8, random_state=42)

# Apply K-Means (we don't use labels — unsupervised)
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
kmeans.fit(X)

print(f"Cluster centers:\n{kmeans.cluster_centers_}")
print(f"Inertia (within-cluster sum of squares): {kmeans.inertia_:.2f}")
print(f"Labels assigned: {np.unique(kmeans.labels_)}")
```

---

## 3. Reinforcement Learning — Learning by Interaction

### 3.1 How It Works

An **agent** takes **actions** in an **environment**, observes the resulting **state**
and **reward**, and learns a **policy** π(s) → a that maximizes cumulative reward.

```
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │   ┌─────────┐   action aₜ   ┌──────────────┐   │
  │   │         │──────────────▶│              │   │
  │   │  AGENT  │               │ ENVIRONMENT  │   │
  │   │  π(s)   │◀──────────────│              │   │
  │   │         │  reward rₜ    │              │   │
  │   └─────────┘  + state sₜ₊₁ └──────────────┘   │
  │                                                 │
  │   Goal: Maximize  G = Σ γᵗ rₜ                  │
  │          (discounted cumulative reward)          │
  │          γ ∈ [0, 1] = discount factor           │
  └─────────────────────────────────────────────────┘
```

### 3.2 Key Concepts

| Concept | Description |
|---------|-------------|
| **State (s)** | Current situation of the environment |
| **Action (a)** | Decision made by the agent |
| **Reward (r)** | Scalar feedback signal (+1 for good, -1 for bad) |
| **Policy (π)** | Strategy: mapping from states to actions |
| **Value Function V(s)** | Expected cumulative reward from state s |
| **Q-Function Q(s,a)** | Expected cumulative reward for taking action a in state s |
| **Discount Factor (γ)** | How much to value future vs immediate rewards |

**Bellman Equation (foundation of RL):**

```
  Q(s, a) = r + γ · max_a' Q(s', a')

  The optimal Q-value for (state, action) equals the immediate reward
  plus the discounted best future value.
```

### 3.3 Python Example — Simple Q-Learning (Grid World)

```python
import numpy as np

n_states, n_actions = 5, 2  # 5 states, actions: 0=left, 1=right
Q = np.zeros((n_states, n_actions))
gamma, alpha = 0.9, 0.1

for _ in range(500):
    state = 0
    while state != 4:  # goal is state 4
        action = np.argmax(Q[state]) if np.random.rand() > 0.1 else np.random.randint(2)
        next_state = max(0, min(4, state + (1 if action == 1 else -1)))
        reward = 1.0 if next_state == 4 else -0.01
        Q[state, action] += alpha * (reward + gamma * np.max(Q[next_state]) - Q[state, action])
        state = next_state

print("Learned Q-table:\n", Q)  # Right action (1) gets higher values
```

---

## 4. Labeled vs Unlabeled Data

| Aspect | Labeled Data | Unlabeled Data |
|--------|-------------|---------------|
| **Format** | (Image → "Cat"), (Email → "Spam") | Image, Email (no target) |
| **Used by** | Supervised learning | Unsupervised learning |
| **Availability** | Scarce, expensive | Abundant, cheap |
| **Cost** | 10K medical images ≈ $50K+ | Often free |

---

## 5. Master Comparison Table

| Aspect | Supervised | Unsupervised | Reinforcement |
|--------|-----------|-------------|---------------|
| **Data** | Labeled (x, y) | Unlabeled (x) | Reward signals |
| **Goal** | Predict output | Find patterns | Maximize reward |
| **Feedback** | Correct answer provided | No feedback | Delayed reward signal |
| **Sub-types** | Classification, Regression | Clustering, Dim. Reduction | Model-free, Model-based |
| **Key Algorithms** | LR, SVM, RF, Neural Nets | K-Means, PCA, DBSCAN | Q-Learning, PPO, A3C |
| **Evaluation** | Accuracy, MSE, F1 | Silhouette, Variance | Cumulative reward |
| **Example** | Spam filter | Customer segments | Game-playing bot |
| **Data cost** | High (labeling needed) | Low (no labels) | Medium (simulation) |
| **Complexity** | Moderate | Moderate | High |

---

## ✅ Revision Questions

1. **Explain with an example** the difference between classification and regression
   in supervised learning. What kind of output does each produce?

2. **You have 1 million customer transaction records with no labels.** Which ML
   paradigm would you use and why? Name two algorithms suitable for this task.

3. **Write the Bellman equation** for Q-learning and explain each term. Why is the
   discount factor γ important?

4. **Compare K-Means clustering and PCA.** Both are unsupervised — how do their
   goals and outputs differ?

5. **A hospital has 500 labeled X-rays and 50,000 unlabeled ones.** Which learning
   paradigm(s) could be used? Justify your answer.

6. **In the Q-learning code example**, what would happen if we set `gamma = 0`?
   How would the agent's behavior change?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| [← 01 – Definition and Types](01-definition-and-types.md) | [03 – Training, Validation & Testing →](03-training-validation-testing.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
