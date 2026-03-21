# 📘 Chapter 5: Pipeline Creation

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

Scikit-learn **Pipelines** chain preprocessing steps and models into a single object. They prevent data leakage, simplify code, and make your ML workflow reproducible. This chapter covers `Pipeline`, `make_pipeline`, `FeatureUnion`, custom transformers, and serialization.

---

## 1. Why Pipelines?

| Without Pipeline | With Pipeline |
|---|---|
| Manual fit/transform for each step | One `fit()` and `predict()` call |
| Easy to accidentally leak data | Automatically fits on train only |
| Hard to reproduce | Single serializable object |
| Messy cross-validation | Works seamlessly with `GridSearchCV` |

---

## 2. Pipeline Class

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Create pipeline with named steps
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression(max_iter=200))
])

# fit() calls scaler.fit_transform() then classifier.fit()
pipe.fit(X_train, y_train)

# predict() calls scaler.transform() then classifier.predict()
y_pred = pipe.predict(X_test)

# score() does transform + score
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")
```

### How Pipeline Works Internally

```
fit(X_train, y_train):
    Step 1: scaler.fit_transform(X_train)  → X_scaled
    Step 2: classifier.fit(X_scaled, y_train)

predict(X_test):
    Step 1: scaler.transform(X_test)       → X_scaled
    Step 2: classifier.predict(X_scaled)
```

> 💡 All steps except the last must be **transformers** (have `transform()`). The last step can be any estimator.

---

## 3. make_pipeline — Shortcut

Automatically generates step names from class names.

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import MinMaxScaler
from sklearn.svm import SVC

# Equivalent to Pipeline([('minmaxscaler', MinMaxScaler()), ('svc', SVC())])
pipe = make_pipeline(MinMaxScaler(), SVC(kernel='rbf'))

pipe.fit(X_train, y_train)
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")

# Step names are auto-generated (lowercase class names)
print(pipe.steps)
# [('minmaxscaler', MinMaxScaler()), ('svc', SVC())]
```

---

## 4. Multi-Step Pipeline

```python
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=2)),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
])

pipe.fit(X_train, y_train)
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")

# Access intermediate results
X_scaled = pipe['scaler'].transform(X_test)
X_pca = pipe['pca'].transform(X_scaled)
```

---

## 5. FeatureUnion — Parallel Feature Processing

Combines features from multiple transformers side by side (horizontally).

```python
from sklearn.pipeline import FeatureUnion
from sklearn.decomposition import PCA
from sklearn.feature_selection import SelectKBest, f_classif

# Combine PCA features with best original features
combined_features = FeatureUnion([
    ('pca', PCA(n_components=2)),
    ('select_best', SelectKBest(f_classif, k=2))
])

pipe = Pipeline([
    ('features', combined_features),
    ('classifier', LogisticRegression(max_iter=200))
])

pipe.fit(X_train, y_train)
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")

# Output has 2 (PCA) + 2 (SelectKBest) = 4 features
```

---

## 6. ColumnTransformer in Pipelines

Handle mixed data types (numerical + categorical) within a pipeline.

```python
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import GradientBoostingClassifier

# Sample data with mixed types
df = pd.DataFrame({
    'age': [25, 30, 35, 40, 45, 28, 33, 50],
    'salary': [50000, 60000, 70000, 80000, 90000, 55000, 65000, 95000],
    'city': ['NYC', 'LA', 'NYC', 'SF', 'LA', 'NYC', 'SF', 'LA'],
    'education': ['BS', 'MS', 'PhD', 'BS', 'MS', 'BS', 'PhD', 'MS']
})
y = [0, 1, 1, 0, 1, 0, 1, 1]

num_features = ['age', 'salary']
cat_features = ['city', 'education']

# ColumnTransformer applies different transformers to different columns
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), num_features),
        ('cat', OneHotEncoder(handle_unknown='ignore', sparse_output=False), cat_features)
    ]
)

# Full pipeline: preprocessing + model
pipe = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', GradientBoostingClassifier(random_state=42))
])

pipe.fit(df, y)
print(f"Predictions: {pipe.predict(df)}")
```

---

## 7. Custom Transformers

Create your own transformer by inheriting from `TransformerMixin` and `BaseEstimator`.

```python
from sklearn.base import BaseEstimator, TransformerMixin
import numpy as np

class LogTransformer(BaseEstimator, TransformerMixin):
    """Applies log(1 + x) transformation to all features."""

    def __init__(self, add_original=False):
        self.add_original = add_original

    def fit(self, X, y=None):
        # Nothing to learn — just return self
        return self

    def transform(self, X):
        X_log = np.log1p(np.abs(X))
        if self.add_original:
            return np.hstack([X, X_log])
        return X_log


# Use in a pipeline
pipe = Pipeline([
    ('log_transform', LogTransformer(add_original=True)),
    ('scaler', StandardScaler()),
    ('model', LogisticRegression(max_iter=200))
])

pipe.fit(X_train, y_train)
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")
```

### Custom Transformer with Learned State

```python
class OutlierClipper(BaseEstimator, TransformerMixin):
    """Clips outliers beyond quantile-based bounds."""

    def __init__(self, lower_quantile=0.01, upper_quantile=0.99):
        self.lower_quantile = lower_quantile
        self.upper_quantile = upper_quantile

    def fit(self, X, y=None):
        self.lower_bound_ = np.quantile(X, self.lower_quantile, axis=0)
        self.upper_bound_ = np.quantile(X, self.upper_quantile, axis=0)
        return self

    def transform(self, X):
        return np.clip(X, self.lower_bound_, self.upper_bound_)
```

---

## 8. Pipeline with GridSearchCV

Access pipeline step parameters using `stepname__parameter` syntax.

```python
from sklearn.model_selection import GridSearchCV

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA()),
    ('classifier', RandomForestClassifier(random_state=42))
])

# Use double underscore to reference nested parameters
param_grid = {
    'pca__n_components': [2, 3, 4],
    'classifier__n_estimators': [50, 100, 200],
    'classifier__max_depth': [3, 5, None]
}

grid_search = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', n_jobs=-1)
grid_search.fit(X_train, y_train)

print(f"Best params: {grid_search.best_params_}")
print(f"Best CV score: {grid_search.best_score_:.4f}")
print(f"Test score: {grid_search.score(X_test, y_test):.4f}")
```

### Switching Estimators in Grid Search

```python
from sklearn.svm import SVC

# Try different classifiers in the same pipeline
param_grid = [
    {
        'classifier': [LogisticRegression(max_iter=200)],
        'classifier__C': [0.1, 1, 10]
    },
    {
        'classifier': [SVC()],
        'classifier__C': [0.1, 1, 10],
        'classifier__kernel': ['rbf', 'linear']
    }
]

grid_search = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy')
grid_search.fit(X_train, y_train)
```

---

## 9. Accessing Pipeline Steps

```python
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=2)),
    ('classifier', LogisticRegression(max_iter=200))
])
pipe.fit(X_train, y_train)

# Access by name
print(pipe.named_steps['scaler'].mean_)
print(pipe.named_steps['pca'].explained_variance_ratio_)

# Access by index
print(pipe[0])  # StandardScaler
print(pipe[1])  # PCA

# Slice the pipeline (returns a new Pipeline)
preprocessing_pipe = pipe[:2]  # scaler + PCA only
X_processed = preprocessing_pipe.transform(X_test)

# Get all parameters
print(pipe.get_params())
```

---

## 10. Serialization with joblib

Save and load trained pipelines for deployment.

```python
import joblib

# Train a complete pipeline
pipe = make_pipeline(StandardScaler(), LogisticRegression(max_iter=200))
pipe.fit(X_train, y_train)

# Save to disk
joblib.dump(pipe, 'trained_pipeline.joblib')

# Load from disk (in production)
loaded_pipe = joblib.load('trained_pipeline.joblib')

# Use directly — no need to preprocess manually
predictions = loaded_pipe.predict(X_test)
print(f"Loaded model accuracy: {loaded_pipe.score(X_test, y_test):.4f}")
```

### Using pickle (alternative)

```python
import pickle

# Save
with open('pipeline.pkl', 'wb') as f:
    pickle.dump(pipe, f)

# Load
with open('pipeline.pkl', 'rb') as f:
    loaded_pipe = pickle.load(f)
```

> 💡 **joblib** is preferred over pickle for scikit-learn objects because it handles large NumPy arrays more efficiently.

---

## 11. Best Practices

| Practice | Why |
|---|---|
| Always use pipelines for preprocessing + model | Prevents data leakage |
| Use `make_pipeline` for quick prototyping | Less boilerplate |
| Name steps meaningfully in `Pipeline` | Easier grid search parameter names |
| Serialize entire pipeline, not just the model | Ensures consistent preprocessing in production |
| Use `ColumnTransformer` for mixed data types | Clean handling of numeric + categorical |
| Test custom transformers independently | Easier debugging |

---

## 📋 Summary Table

| Tool | Purpose | Key Point |
|---|---|---|
| `Pipeline` | Chain steps sequentially | Named steps with `(name, estimator)` tuples |
| `make_pipeline` | Shortcut for Pipeline | Auto-generates step names |
| `FeatureUnion` | Combine features in parallel | Concatenates horizontally |
| `ColumnTransformer` | Different transforms per column | Essential for mixed-type data |
| `TransformerMixin` | Base class for custom transformers | Provides `fit_transform` for free |
| `BaseEstimator` | Base class for all estimators | Provides `get_params`/`set_params` |
| `stepname__param` | Access nested params | Used in `GridSearchCV` |
| `joblib.dump/load` | Save/load pipelines | Preferred for scikit-learn objects |

---

## ❓ Revision Questions

1. **How does a pipeline prevent data leakage during cross-validation?**
2. **What is the difference between `Pipeline` and `make_pipeline`?**
3. **How do you specify hyperparameters for a pipeline step in `GridSearchCV`?**
4. **What is `FeatureUnion` and how does it differ from `ColumnTransformer`?**
5. **Why should you inherit from both `BaseEstimator` and `TransformerMixin` when creating a custom transformer?**
6. **Why is `joblib` preferred over `pickle` for saving scikit-learn pipelines?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← 04 - Cross-Validation](04-cross-validation.md) | [Unit 5: Scikit-learn Basics](../README.md) | [06 - Model Evaluation Metrics →](06-model-evaluation-metrics.md) |
