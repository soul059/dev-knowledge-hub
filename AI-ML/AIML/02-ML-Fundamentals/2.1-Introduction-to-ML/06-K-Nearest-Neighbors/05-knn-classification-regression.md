# 🔮 Unit 6, Chapter 5: KNN for Classification & Regression

> **K-Nearest Neighbors — Part 5 of 6**
>
> KNN is one of the few algorithms that naturally handles both classification and regression with the same core logic — find neighbors, aggregate their outputs. This chapter covers the complete algorithm in depth: pseudocode, from-scratch Python implementation, scikit-learn usage, and practical examples on real datasets.

---

## Table of Contents

1. [KNN for Classification — Majority Voting](#1-knn-for-classification--majority-voting)
2. [KNN for Regression — Averaging Neighbors](#2-knn-for-regression--averaging-neighbors)
3. [The KNN Algorithm — Pseudocode](#3-the-knn-algorithm--pseudocode)
4. [Complete Python Implementation from Scratch](#4-complete-python-implementation-from-scratch)
5. [Using scikit-learn: KNeighborsClassifier](#5-using-scikit-learn-kneighborsclassifier)
6. [Using scikit-learn: KNeighborsRegressor](#6-using-scikit-learn-kneighborsregressor)
7. [Practical Example: Classification on Iris Dataset](#7-practical-example-classification-on-iris-dataset)
8. [Practical Example: Regression on Boston-style Housing](#8-practical-example-regression-on-housing-data)
9. [Decision Boundary Visualization](#9-decision-boundary-visualization)
10. [KNN Probability Estimates](#10-knn-probability-estimates)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. KNN for Classification — Majority Voting

### How It Works

```
  Given: Training set {(x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)}
         Query point: x_q
         Number of neighbors: K

  Step 1: Compute distance from x_q to every training point
  Step 2: Select the K nearest points (smallest distances)
  Step 3: Count the class labels among those K neighbors
  Step 4: Predict the class with the MOST votes (majority)

  ┌────────────────────────────────────────────────────┐
  │  ŷ = mode({y_i : x_i ∈ K-nearest-neighbors(x_q)}) │
  └────────────────────────────────────────────────────┘
```

### Worked Example

```
  Dataset (5 training points):
  ┌────────┬─────────┬───────┐
  │ Point  │ (x₁,x₂) │ Class │
  ├────────┼─────────┼───────┤
  │  A     │ (1, 2)  │  Red  │
  │  B     │ (2, 3)  │  Red  │
  │  C     │ (3, 1)  │ Blue  │
  │  D     │ (5, 4)  │ Blue  │
  │  E     │ (4, 2)  │ Blue  │
  └────────┴─────────┴───────┘

  Query: Q = (3, 3),  K = 3

  Step 1: Compute distances
    d(Q,A) = √[(3-1)² + (3-2)²] = √[4+1]  = √5  ≈ 2.236
    d(Q,B) = √[(3-2)² + (3-3)²] = √[1+0]  = √1  = 1.000
    d(Q,C) = √[(3-3)² + (3-1)²] = √[0+4]  = √4  = 2.000
    d(Q,D) = √[(3-5)² + (3-4)²] = √[4+1]  = √5  ≈ 2.236
    d(Q,E) = √[(3-4)² + (3-2)²] = √[1+1]  = √2  ≈ 1.414

  Step 2: Sort by distance
    B (1.000), E (1.414), C (2.000), A (2.236), D (2.236)

  Step 3: Select K=3 nearest
    B → Red,  E → Blue,  C → Blue

  Step 4: Vote
    Red: 1 vote,  Blue: 2 votes

  Prediction: Blue ✓

  ┌──────────────────────────────────────┐
  │        y                             │
  │  4 ─       ● D (Blue)               │
  │  3 ─  ● B (Red)  ╳ Q               │
  │  2 ─  ● A (Red)      ● E (Blue)    │
  │  1 ─        ● C (Blue)              │
  │  0 ─                                │
  │     └──┬──┬──┬──┬──┬──→ x           │
  │        1  2  3  4  5                │
  │                                      │
  │  3 nearest to Q: B(Red), E(Blue),   │
  │  C(Blue) → Predict Blue             │
  └──────────────────────────────────────┘
```

---

## 2. KNN for Regression — Averaging Neighbors

### How It Works

```
  Same as classification, but instead of voting:
  Take the AVERAGE (or weighted average) of neighbor values.

  ┌──────────────────────────────────────────────┐
  │            1                                  │
  │  ŷ = ───── Σ  y_i     (uniform average)     │
  │          K  i∈KNN                             │
  │                                               │
  │         Σ w_i · y_i                           │
  │  ŷ = ───────────────   (weighted average)    │
  │          Σ w_i                                │
  └──────────────────────────────────────────────┘
```

### Worked Example

```
  Dataset: Predict house price based on area (sq ft)

  ┌────────┬──────────┬──────────┐
  │ Point  │ Area     │ Price    │
  ├────────┼──────────┼──────────┤
  │  A     │  800     │ $150,000 │
  │  B     │ 1000     │ $200,000 │
  │  C     │ 1200     │ $250,000 │
  │  D     │ 1500     │ $300,000 │
  │  E     │ 2000     │ $400,000 │
  └────────┴──────────┴──────────┘

  Query: Area = 1100 sq ft,  K = 3

  Step 1: Distances (|area_q - area_i|)
    d(Q,A) = |1100 - 800|  = 300
    d(Q,B) = |1100 - 1000| = 100
    d(Q,C) = |1100 - 1200| = 100
    d(Q,D) = |1100 - 1500| = 400
    d(Q,E) = |1100 - 2000| = 900

  Step 2: K=3 nearest: B (100), C (100), A (300)

  Step 3a: Uniform average
    ŷ = (200K + 250K + 150K) / 3 = $200,000

  Step 3b: Weighted average (w = 1/d)
    w_B = 1/100 = 0.010
    w_C = 1/100 = 0.010
    w_A = 1/300 = 0.00333

    ŷ = (0.010×200K + 0.010×250K + 0.00333×150K)
        / (0.010 + 0.010 + 0.00333)
      = (2000 + 2500 + 500) / 0.02333
      = 5000 / 0.02333
      = $214,286

  Weighted average gives more weight to equally-distant B and C,
  less to the farther A → arguably a better estimate.
```

### Regression vs Classification Comparison

```
  ┌──────────────────┬────────────────────┬────────────────────┐
  │  Aspect          │  Classification    │  Regression        │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Output          │  Discrete class    │  Continuous value  │
  │  Aggregation     │  Majority vote     │  Average (mean)    │
  │  Weighted        │  Weighted vote     │  Weighted average  │
  │  Confidence      │  Vote proportions  │  Std of neighbors  │
  │  sklearn class   │  KNeighborsClassifier │ KNeighborsRegressor │
  └──────────────────┴────────────────────┴────────────────────┘
```

---

## 3. The KNN Algorithm — Pseudocode

### Classification

```
  ALGORITHM: KNN_Classify(training_set, query_point, K)

  INPUT:
    training_set = {(x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)}
    query_point  = x_q
    K            = number of neighbors

  OUTPUT:
    predicted_class

  PROCEDURE:
    1. FOR each training point (xᵢ, yᵢ):
         Compute distance dᵢ = dist(x_q, xᵢ)

    2. Sort all distances in ascending order

    3. Select the K points with smallest distances:
         neighbors = {(x_{i₁}, y_{i₁}), ..., (x_{iₖ}, y_{iₖ})}

    4. Count class frequencies among neighbors:
         FOR each class c:
           count_c = number of neighbors with class c

    5. RETURN class with highest count:
         predicted_class = argmax_c(count_c)

    (If tie: break by distance or randomly)
```

### Regression

```
  ALGORITHM: KNN_Regress(training_set, query_point, K)

  INPUT:
    training_set = {(x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)}
    query_point  = x_q
    K            = number of neighbors

  OUTPUT:
    predicted_value

  PROCEDURE:
    1-3. Same as classification (find K nearest neighbors)

    4. Compute prediction:
         predicted_value = (1/K) × Σ yᵢ  for i in neighbors

    OR (weighted version):
         predicted_value = Σ(wᵢ·yᵢ) / Σ(wᵢ)
         where wᵢ = 1/dᵢ
```

### Time Complexity

```
  Training:  O(1)       — just store the data (lazy learner!)
  Prediction: O(n·d)    — compute n distances in d dimensions
              + O(n log n)  — sort distances (or O(n log K) with heap)

  Total prediction: O(n·d + n log K) per query point

  ┌────────────────────────────────────────────┐
  │  KNN has NO training time but EXPENSIVE    │
  │  prediction time. It's a "lazy learner."   │
  └────────────────────────────────────────────┘
```

---

## 4. Complete Python Implementation from Scratch

```python
import numpy as np
from collections import Counter

class KNNFromScratch:
    """
    K-Nearest Neighbors for Classification and Regression.
    Implemented from scratch using only NumPy.
    """

    def __init__(self, k=5, weights='uniform', task='classification'):
        """
        Parameters:
            k: Number of neighbors
            weights: 'uniform' or 'distance'
            task: 'classification' or 'regression'
        """
        self.k = k
        self.weights = weights
        self.task = task

    def fit(self, X, y):
        """Store training data (lazy learning — no computation here)."""
        self.X_train = np.array(X, dtype=float)
        self.y_train = np.array(y)
        return self

    def _euclidean_distances(self, x):
        """Compute Euclidean distance from x to all training points."""
        return np.sqrt(np.sum((self.X_train - x) ** 2, axis=1))

    def _get_neighbors(self, x):
        """Find K nearest neighbors and their distances."""
        distances = self._euclidean_distances(x)
        k_indices = np.argsort(distances)[:self.k]
        k_distances = distances[k_indices]
        k_labels = self.y_train[k_indices]
        return k_distances, k_labels

    def _compute_weights(self, distances):
        """Compute weights based on distances."""
        if self.weights == 'uniform':
            return np.ones_like(distances)
        elif self.weights == 'distance':
            # Handle zero distance: give infinite weight (only that point matters)
            if np.any(distances == 0):
                w = np.zeros_like(distances)
                w[distances == 0] = 1.0
                return w
            return 1.0 / distances
        else:
            raise ValueError(f"Unknown weights: {self.weights}")

    def _predict_one_classification(self, x):
        """Predict class for one point."""
        distances, labels = self._get_neighbors(x)
        weights = self._compute_weights(distances)

        # Weighted vote for each class
        class_weights = {}
        for label, weight in zip(labels, weights):
            class_weights[label] = class_weights.get(label, 0) + weight

        return max(class_weights, key=class_weights.get)

    def _predict_one_regression(self, x):
        """Predict value for one point."""
        distances, values = self._get_neighbors(x)
        weights = self._compute_weights(distances)

        # Weighted average
        return np.sum(weights * values) / np.sum(weights)

    def predict(self, X):
        """Predict for multiple query points."""
        X = np.array(X, dtype=float)
        if self.task == 'classification':
            return np.array([self._predict_one_classification(x) for x in X])
        else:
            return np.array([self._predict_one_regression(x) for x in X])

    def score(self, X, y):
        """Compute accuracy (classification) or R² (regression)."""
        predictions = self.predict(X)
        y = np.array(y)

        if self.task == 'classification':
            return np.mean(predictions == y)
        else:
            # R² score
            ss_res = np.sum((y - predictions) ** 2)
            ss_tot = np.sum((y - np.mean(y)) ** 2)
            return 1 - (ss_res / ss_tot)

    def predict_proba(self, X):
        """
        Predict class probabilities (classification only).
        Returns array of shape (n_samples, n_classes).
        """
        if self.task != 'classification':
            raise ValueError("predict_proba only for classification")

        X = np.array(X, dtype=float)
        classes = np.unique(self.y_train)
        probas = []

        for x in X:
            distances, labels = self._get_neighbors(x)
            weights = self._compute_weights(distances)
            total_weight = np.sum(weights)

            proba = []
            for c in classes:
                mask = labels == c
                proba.append(np.sum(weights[mask]) / total_weight)
            probas.append(proba)

        return np.array(probas)


# ═══════════════════════════════════════════
#         CLASSIFICATION DEMO
# ═══════════════════════════════════════════
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

print("=" * 55)
print("   KNN CLASSIFICATION — FROM SCRATCH")
print("=" * 55)

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

for k in [1, 3, 5, 7, 9]:
    for w in ['uniform', 'distance']:
        model = KNNFromScratch(k=k, weights=w, task='classification')
        model.fit(X_train_s, y_train)
        acc = model.score(X_test_s, y_test)
        print(f"  K={k:2d}, weights={w:10s} → Accuracy = {acc:.4f}")

# Probability predictions
model = KNNFromScratch(k=5, weights='distance', task='classification')
model.fit(X_train_s, y_train)
probas = model.predict_proba(X_test_s[:5])
print(f"\nProbability predictions (first 5 test samples):")
print(f"  {'Sample':>8}  {'P(class0)':>10}  {'P(class1)':>10}  {'P(class2)':>10}  {'Predicted':>10}  {'Actual':>8}")
for i in range(5):
    pred = np.argmax(probas[i])
    print(f"  {i+1:8d}  {probas[i][0]:10.4f}  {probas[i][1]:10.4f}  {probas[i][2]:10.4f}  {pred:10d}  {y_test[i]:8d}")

# ═══════════════════════════════════════════
#           REGRESSION DEMO
# ═══════════════════════════════════════════
print(f"\n{'=' * 55}")
print("   KNN REGRESSION — FROM SCRATCH")
print("=" * 55)

from sklearn.datasets import make_regression

X_reg, y_reg = make_regression(n_samples=200, n_features=3,
                                noise=10, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(
    X_reg, y_reg, test_size=0.3, random_state=42
)

scaler_reg = StandardScaler()
X_tr_s = scaler_reg.fit_transform(X_tr)
X_te_s = scaler_reg.transform(X_te)

for k in [1, 3, 5, 7, 9]:
    for w in ['uniform', 'distance']:
        model = KNNFromScratch(k=k, weights=w, task='regression')
        model.fit(X_tr_s, y_tr)
        r2 = model.score(X_te_s, y_te)
        print(f"  K={k:2d}, weights={w:10s} → R² = {r2:.4f}")
```

**Expected Output (abbreviated):**
```
=======================================================
   KNN CLASSIFICATION — FROM SCRATCH
=======================================================
  K= 1, weights=uniform    → Accuracy = 0.9556
  K= 1, weights=distance   → Accuracy = 0.9556
  K= 3, weights=uniform    → Accuracy = 0.9778
  K= 3, weights=distance   → Accuracy = 0.9778
  K= 5, weights=uniform    → Accuracy = 0.9778
  K= 5, weights=distance   → Accuracy = 0.9778
  ...

=======================================================
   KNN REGRESSION — FROM SCRATCH
=======================================================
  K= 1, weights=uniform    → R² = 0.8234
  K= 3, weights=uniform    → R² = 0.8912
  K= 5, weights=uniform    → R² = 0.8856
  K= 5, weights=distance   → R² = 0.8973
  ...
```

---

## 5. Using scikit-learn: KNeighborsClassifier

### Basic Usage

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, confusion_matrix

# Load data
X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

# ALWAYS scale features for KNN
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Create and train model
knn = KNeighborsClassifier(
    n_neighbors=5,          # K value
    weights='distance',     # 'uniform' or 'distance'
    metric='euclidean',     # Distance metric
    algorithm='auto',       # 'auto', 'ball_tree', 'kd_tree', 'brute'
    p=2,                    # Power for Minkowski metric
    n_jobs=-1               # Use all CPU cores
)

knn.fit(X_train, y_train)

# Predict
y_pred = knn.predict(X_test)
y_proba = knn.predict_proba(X_test)

# Evaluate
print(f"Accuracy: {knn.score(X_test, y_test):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=load_iris().target_names))

print(f"Confusion Matrix:")
print(confusion_matrix(y_test, y_pred))
```

### All KNeighborsClassifier Parameters

```python
KNeighborsClassifier(
    n_neighbors=5,          # Number of neighbors (K)
    weights='uniform',      # 'uniform', 'distance', or callable
    algorithm='auto',       # Search algorithm:
                            #   'brute'    : O(n) brute force
                            #   'kd_tree'  : KD-Tree (fast for low d)
                            #   'ball_tree': Ball Tree (works for any metric)
                            #   'auto'     : picks best automatically
    leaf_size=30,           # Leaf size for tree algorithms
    p=2,                    # Minkowski power (1=Manhattan, 2=Euclidean)
    metric='minkowski',     # Distance metric
    metric_params=None,     # Additional metric parameters
    n_jobs=None             # Parallel processing (-1 = all cores)
)
```

---

## 6. Using scikit-learn: KNeighborsRegressor

### Basic Usage

```python
from sklearn.neighbors import KNeighborsRegressor
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
import numpy as np

# Load data
housing = fetch_california_housing()
X, y = housing.data, housing.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Scale features
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Train KNN regressor
knn_reg = KNeighborsRegressor(
    n_neighbors=7,
    weights='distance',
    metric='euclidean',
    n_jobs=-1
)

knn_reg.fit(X_train, y_train)
y_pred = knn_reg.predict(X_test)

# Evaluate
print(f"R² Score:  {r2_score(y_test, y_pred):.4f}")
print(f"MAE:       {mean_absolute_error(y_test, y_pred):.4f}")
print(f"RMSE:      {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")

# Show sample predictions
print(f"\n{'Actual':>10}  {'Predicted':>10}  {'Error':>10}")
print("-" * 35)
for actual, pred in zip(y_test[:10], y_pred[:10]):
    print(f"{actual:10.3f}  {pred:10.3f}  {actual-pred:10.3f}")
```

---

## 7. Practical Example: Classification on Iris Dataset

### Complete End-to-End Pipeline

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import (
    train_test_split, cross_val_score, GridSearchCV
)
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report, confusion_matrix

# ───────────── 1. Load Data ─────────────
iris = load_iris()
X, y = iris.data, iris.target
print(f"Dataset: {iris.data.shape[0]} samples, {iris.data.shape[1]} features")
print(f"Classes: {iris.target_names}")
print(f"Features: {iris.feature_names}")

# ───────────── 2. Create Pipeline ─────────────
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier())
])

# ───────────── 3. Hyperparameter Tuning ─────────────
param_grid = {
    'knn__n_neighbors': list(range(1, 21, 2)),
    'knn__weights': ['uniform', 'distance'],
    'knn__metric': ['euclidean', 'manhattan'],
}

grid = GridSearchCV(
    pipe, param_grid, cv=10,
    scoring='accuracy', return_train_score=True, n_jobs=-1
)

grid.fit(X, y)

print(f"\nBest Parameters: {grid.best_params_}")
print(f"Best CV Accuracy: {grid.best_score_:.4f}")

# ───────────── 4. Final Evaluation ─────────────
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

best_model = grid.best_estimator_
best_model.fit(X_train, y_train)
y_pred = best_model.predict(X_test)

print(f"\nTest Accuracy: {best_model.score(X_test, y_test):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=iris.target_names))

print(f"Confusion Matrix:")
cm = confusion_matrix(y_test, y_pred)
print(cm)

# ASCII confusion matrix
print(f"\n  {'':>12} {'setosa':>8} {'versicolor':>12} {'virginica':>10}")
for i, name in enumerate(iris.target_names):
    row = '  '.join(f'{v:5d}' for v in cm[i])
    print(f"  {name:>12} {row}")
```

---

## 8. Practical Example: Regression on Housing Data

```python
import numpy as np
from sklearn.datasets import fetch_california_housing
from sklearn.neighbors import KNeighborsRegressor
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import mean_squared_error, r2_score

# ───────────── 1. Load Data ─────────────
housing = fetch_california_housing()
X, y = housing.data, housing.target

print(f"Dataset: {X.shape[0]} samples, {X.shape[1]} features")
print(f"Features: {housing.feature_names}")
print(f"Target: Median house value (×$100,000)")
print(f"Target range: {y.min():.2f} — {y.max():.2f}")

# ───────────── 2. Split Data ─────────────
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# ───────────── 3. Pipeline with GridSearch ─────────────
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsRegressor())
])

param_grid = {
    'knn__n_neighbors': [3, 5, 7, 9, 11, 15, 21],
    'knn__weights': ['uniform', 'distance'],
}

grid = GridSearchCV(pipe, param_grid, cv=5,
                    scoring='r2', n_jobs=-1)
grid.fit(X_train, y_train)

print(f"\nBest Parameters: {grid.best_params_}")
print(f"Best CV R²: {grid.best_score_:.4f}")

# ───────────── 4. Evaluate Best Model ─────────────
y_pred = grid.predict(X_test)

r2 = r2_score(y_test, y_pred)
rmse = np.sqrt(mean_squared_error(y_test, y_pred))

print(f"\nTest R²:   {r2:.4f}")
print(f"Test RMSE: {rmse:.4f}")
print(f"(RMSE in dollars: ${rmse * 100000:,.0f})")

# ───────────── 5. Residual Analysis ─────────────
residuals = y_test - y_pred
print(f"\nResidual Statistics:")
print(f"  Mean:   {residuals.mean():.4f}")
print(f"  Std:    {residuals.std():.4f}")
print(f"  Min:    {residuals.min():.4f}")
print(f"  Max:    {residuals.max():.4f}")
print(f"  Median: {np.median(residuals):.4f}")
```

---

## 9. Decision Boundary Visualization

### Understanding Decision Boundaries

```
  KNN creates a decision boundary that is the locus of points
  equidistant (in terms of vote outcome) from two classes.

  K = 1: Voronoi Tessellation
  ┌─────────────────────────────────┐
  │  ●  │   │ ○    │   ○  │        │
  │     │ ● │      │      │  ○     │   Each region is the
  │  ───┘   │   ○  │  ────┘        │   set of all points
  │  ●      └──────┘     ○    ○    │   closest to one
  │     ●  │          │            │   particular training
  │        │  ●    ───┘   ○        │   point.
  │  ●  ●  │         ○            │
  └─────────────────────────────────┘

  K = 5: Smoother boundary
  ┌─────────────────────────────────┐
  │  ●        ○ ○                   │
  │    ● ●  ╲        ○             │   Boundary is smoother
  │      ●   ╲   ○                 │   because it's based on
  │    ●      ╲    ○ ○             │   majority of 5 neighbors,
  │   ●        ╲      ○           │   not just the nearest one.
  └─────────────────────────────────┘

  K = 50: Very smooth (nearly linear)
  ┌─────────────────────────────────┐
  │  ●       │ ○ ○                  │
  │    ● ●   │       ○             │   Boundary is almost a
  │      ●   │   ○                 │   straight line — large K
  │    ●     │   ○ ○               │   averages over many points.
  │   ●      │     ○               │
  └─────────────────────────────────┘
```

### Python Code for Decision Boundary (2D)

```python
import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from sklearn.neighbors import KNeighborsClassifier
from sklearn.datasets import make_moons
from sklearn.preprocessing import StandardScaler

# Generate 2D dataset (for visualization)
X, y = make_moons(n_samples=200, noise=0.2, random_state=42)
scaler = StandardScaler()
X = scaler.fit_transform(X)

# Create mesh grid for decision boundary
h = 0.02  # Step size
x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
y_min, y_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                      np.arange(y_min, y_max, h))

# Plot decision boundaries for different K values
fig, axes = plt.subplots(1, 3, figsize=(18, 5))
k_values = [1, 5, 25]

for ax, k in zip(axes, k_values):
    knn = KNeighborsClassifier(n_neighbors=k)
    knn.fit(X, y)

    # Predict on mesh grid
    Z = knn.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)

    # Plot
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
    ax.scatter(X[y == 0, 0], X[y == 0, 1], c='blue', edgecolors='k', s=30)
    ax.scatter(X[y == 1, 0], X[y == 1, 1], c='red', edgecolors='k', s=30)
    ax.set_title(f'K = {k}', fontsize=14)
    ax.set_xlabel('Feature 1')
    ax.set_ylabel('Feature 2')

plt.suptitle('KNN Decision Boundaries', fontsize=16)
plt.tight_layout()
plt.savefig('knn_decision_boundaries.png', dpi=150)
print("Decision boundary plot saved as 'knn_decision_boundaries.png'")
```

---

## 10. KNN Probability Estimates

### How KNN Estimates Probabilities

```
  For classification, KNN can output class probabilities:

                    Number of neighbors with class c
  P(class = c) = ────────────────────────────────────
                           K (total neighbors)

  Or with distance weighting:

                   Σ w_i  (for neighbors with class c)
  P(class = c) = ──────────────────────────────────────
                        Σ w_i  (all K neighbors)
```

### Example

```
  K = 7 neighbors for query Q:

  Classes: [A, A, A, B, B, A, C]
  
  Uniform probabilities:
    P(A) = 4/7 = 0.571
    P(B) = 2/7 = 0.286
    P(C) = 1/7 = 0.143

  With distances [0.5, 0.8, 1.0, 1.2, 1.5, 2.0, 2.5]:
  Weights (1/d): [2.0, 1.25, 1.0, 0.83, 0.67, 0.5, 0.4]
  Total weight = 6.65

  Weighted probabilities:
    P(A) = (2.0 + 1.25 + 1.0 + 0.5) / 6.65 = 4.75/6.65 = 0.714
    P(B) = (0.83 + 0.67) / 6.65 = 1.5/6.65 = 0.226
    P(C) = 0.4 / 6.65 = 0.060

  Distance-weighted → MORE confident in A (the closer class).
```

### Using predict_proba in scikit-learn

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.datasets import load_iris
import numpy as np

X, y = load_iris(return_X_y=True)
knn = KNeighborsClassifier(n_neighbors=5, weights='distance')
knn.fit(X, y)

# Predict probabilities for a new sample
sample = X[0:1]  # First sample
proba = knn.predict_proba(sample)
print(f"Probabilities: {proba[0]}")
print(f"Predicted class: {knn.predict(sample)[0]}")
print(f"Confidence: {proba[0].max():.2%}")
```

---

## 11. Summary Table

| Aspect | Classification | Regression |
|--------|:---:|:---:|
| **Output** | Discrete class label | Continuous value |
| **Aggregation** | Majority vote / weighted vote | Mean / weighted mean |
| **Evaluation** | Accuracy, F1, AUC | R², MAE, RMSE |
| **Probabilities** | Vote proportions | N/A (use prediction intervals) |
| **sklearn class** | `KNeighborsClassifier` | `KNeighborsRegressor` |
| **Key parameters** | `n_neighbors`, `weights`, `metric` | Same |
| **Must scale features** | ✅ Yes | ✅ Yes |
| **Training time** | O(1) — just store data | O(1) |
| **Prediction time** | O(n·d) per query | O(n·d) per query |
| **Best for** | Non-linear boundaries, multi-class | Local patterns, non-parametric |

---

## 12. Revision Questions

**Q1.** Walk through the KNN classification algorithm step-by-step for the following scenario: Training data has points A(1,1)=Red, B(2,3)=Blue, C(4,2)=Red, D(3,5)=Blue, E(5,4)=Red. Predict the class for Q(3,3) using K=3 with uniform voting. Show all distance calculations.

**Q2.** For KNN regression with K=4, your neighbors have values [10, 20, 30, 100] at distances [1, 2, 3, 10]. Calculate the prediction using (a) uniform averaging and (b) distance-weighted averaging. Which is more robust to the outlier value of 100?

**Q3.** Explain why KNN is called a "lazy learner." What are the implications for training time vs prediction time? When is this an advantage and when is it a disadvantage?

**Q4.** Write a Python function `knn_classify(X_train, y_train, x_query, k)` from scratch (using only NumPy) that returns the predicted class for a single query point using Euclidean distance and uniform voting.

**Q5.** You train a KNN classifier with K=5 on the Iris dataset and get `predict_proba` output of [0.6, 0.2, 0.2] for a test sample. What does this mean? How would the probabilities change if you switched from `weights='uniform'` to `weights='distance'` and the 3 nearest neighbors are all class 0?

**Q6.** Compare the decision boundary of KNN at K=1 (Voronoi tessellation) vs K=n (majority class everywhere). Describe what happens at intermediate K values and how this relates to the bias-variance tradeoff.

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [6.4 Curse of Dimensionality](./04-curse-of-dimensionality.md) | **6.5 KNN Classification & Regression** | [6.6 Advantages & Limitations](./06-advantages-and-limitations.md) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 5 of 6*
