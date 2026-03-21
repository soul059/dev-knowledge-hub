# 📘 Chapter 1: The Estimator API

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

Scikit-learn is built on a **consistent API design** that makes every algorithm feel familiar once you learn one. The central abstraction is the **Estimator** — any object that learns from data. This chapter covers the philosophy, interfaces, and conventions that make scikit-learn powerful and predictable.

---

## 1. Scikit-learn Design Philosophy

Scikit-learn follows five key principles:

| Principle | Description |
|---|---|
| **Consistency** | All estimators share the same interface (`fit`, `predict`, `transform`) |
| **Inspection** | Parameters are public attributes; learned values use trailing `_` |
| **Non-proliferation** | Data is NumPy arrays or SciPy sparse matrices — no custom types |
| **Composition** | Pipelines and meta-estimators combine building blocks |
| **Sensible Defaults** | Every parameter has a reasonable default value |

---

## 2. The Estimator Interface

### 2.1 Core Methods

```python
from sklearn.linear_model import LinearRegression

# Every estimator is instantiated with hyperparameters
model = LinearRegression(fit_intercept=True)

# fit() — learn from data (always returns self)
model.fit(X_train, y_train)

# predict() — make predictions on new data
predictions = model.predict(X_test)

# score() — evaluate model performance
r2 = model.score(X_test, y_test)
```

### 2.2 Transformers (fit + transform)

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()

# fit() learns parameters (mean, std) from training data
scaler.fit(X_train)

# transform() applies the learned transformation
X_train_scaled = scaler.transform(X_train)

# fit_transform() combines both steps (more efficient)
X_train_scaled = scaler.fit_transform(X_train)

# IMPORTANT: only transform on test data (never fit again)
X_test_scaled = scaler.transform(X_test)
```

---

## 3. Types of Estimators

### 3.1 Classifiers — predict discrete labels

```python
from sklearn.ensemble import RandomForestClassifier

clf = RandomForestClassifier(n_estimators=100, random_state=42)
clf.fit(X_train, y_train)
labels = clf.predict(X_test)
probabilities = clf.predict_proba(X_test)  # class probabilities
```

### 3.2 Regressors — predict continuous values

```python
from sklearn.linear_model import Ridge

reg = Ridge(alpha=1.0)
reg.fit(X_train, y_train)
values = reg.predict(X_test)
```

### 3.3 Transformers — transform data

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)
```

### 3.4 Clusterers — discover groups

```python
from sklearn.cluster import KMeans

kmeans = KMeans(n_clusters=3, random_state=42)
kmeans.fit(X)
cluster_labels = kmeans.predict(X_new)
# or directly: cluster_labels = kmeans.fit_predict(X)
```

---

## 4. The fit/predict Pattern — Complete Workflow

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

# Step 1: Load data
iris = load_iris()
X, y = iris.data, iris.target

# Step 2: Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Step 3: Create estimator with hyperparameters
knn = KNeighborsClassifier(n_neighbors=5, metric='euclidean')

# Step 4: Fit on training data
knn.fit(X_train, y_train)

# Step 5: Predict on test data
y_pred = knn.predict(X_test)

# Step 6: Evaluate
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.4f}")
```

---

## 5. get_params / set_params / clone

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.base import clone

dt = DecisionTreeClassifier(max_depth=5, min_samples_split=10)

# Inspect all parameters (including defaults)
print(dt.get_params())
# {'ccp_alpha': 0.0, 'max_depth': 5, 'min_samples_split': 10, ...}

# Modify parameters after creation
dt.set_params(max_depth=10, criterion='entropy')

# clone() creates an unfitted copy with the same parameters
dt_clone = clone(dt)
# dt_clone has same hyperparameters but NO learned attributes
```

### When to use `clone()`

```python
from sklearn.base import clone

# Useful in cross-validation loops or comparing models
base_model = DecisionTreeClassifier(max_depth=5)

results = []
for train_idx, val_idx in kfold.split(X, y):
    model = clone(base_model)  # fresh unfitted copy each fold
    model.fit(X[train_idx], y[train_idx])
    score = model.score(X[val_idx], y[val_idx])
    results.append(score)
```

---

## 6. Learned Attributes (Trailing Underscore `_` Convention)

Attributes ending with `_` are **only available after `fit()`** and represent what the model learned from data.

```python
from sklearn.linear_model import LinearRegression
import numpy as np

X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2, 4, 6, 8, 10])

model = LinearRegression()
model.fit(X, y)

# Learned attributes (available only after fit)
print(model.coef_)          # array([2.]) — slope
print(model.intercept_)     # 0.0 — y-intercept
print(model.n_features_in_) # 1 — number of features seen during fit

# Hyperparameters (available anytime — no underscore)
print(model.fit_intercept)  # True
```

### Common Learned Attributes by Estimator Type

| Estimator | Key Attributes | Description |
|---|---|---|
| `LinearRegression` | `coef_`, `intercept_` | Weights and bias |
| `DecisionTreeClassifier` | `feature_importances_`, `tree_` | Feature rankings, tree structure |
| `KMeans` | `cluster_centers_`, `labels_`, `inertia_` | Centroids, assignments, SSE |
| `PCA` | `components_`, `explained_variance_ratio_` | Principal axes, variance explained |
| `StandardScaler` | `mean_`, `scale_`, `var_` | Per-feature statistics |

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

# Feature importances (Gini-based)
importances = rf.feature_importances_
for name, imp in zip(iris.feature_names, importances):
    print(f"{name}: {imp:.4f}")

# Classes the model learned
print(rf.classes_)       # array([0, 1, 2])
print(rf.n_classes_)     # 3
```

---

## 7. Complete End-to-End Workflow

```python
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

# Generate synthetic data
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    n_classes=2, random_state=42
)

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Preprocess (fit on train, transform both)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Train
clf = LogisticRegression(C=1.0, max_iter=200, random_state=42)
clf.fit(X_train_scaled, y_train)

# Evaluate
y_pred = clf.predict(X_test_scaled)
print(classification_report(y_test, y_pred))

# Inspect learned parameters
print(f"Coefficients shape: {clf.coef_.shape}")
print(f"Intercept: {clf.intercept_[0]:.4f}")
print(f"Iterations used: {clf.n_iter_[0]}")
```

---

## 📋 Summary Table

| Concept | Key Point |
|---|---|
| `fit(X, y)` | Learn from data; always returns `self` |
| `predict(X)` | Generate predictions (classifiers, regressors) |
| `transform(X)` | Transform data (scalers, encoders, PCA) |
| `fit_transform(X)` | `fit()` + `transform()` in one step |
| `score(X, y)` | Default evaluation metric |
| `get_params()` | Returns dict of hyperparameters |
| `set_params()` | Modifies hyperparameters |
| `clone(est)` | Unfitted copy with same hyperparameters |
| `attribute_` | Trailing `_` = learned from data |

---

## ❓ Revision Questions

1. **What is the difference between `fit_transform()` and calling `fit()` then `transform()` separately?**
2. **Why do learned attributes have a trailing underscore? Give three examples.**
3. **When should you use `clone()` instead of creating a new estimator instance?**
4. **What method would you call to get class probabilities from a classifier?**
5. **Explain why you should call `fit()` only on training data for transformers but `transform()` on both train and test data.**
6. **What are the four main types of estimators in scikit-learn? Give one example class for each.**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| — | [Unit 5: Scikit-learn Basics](../README.md) | [02 - Data Preprocessing →](02-data-preprocessing.md) |
