# 📘 Chapter 6: Model Evaluation Metrics

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

Choosing the right metric is as important as choosing the right model. A model that looks great by one metric may be terrible by another. This chapter covers classification metrics (accuracy, precision, recall, F1, ROC-AUC), regression metrics (MSE, MAE, R²), confusion matrices, and custom scorers.

---

## 1. Classification Metrics

### 1.1 accuracy_score

```python
from sklearn.metrics import accuracy_score
import numpy as np

y_true = [0, 1, 1, 0, 1, 0, 1, 1]
y_pred = [0, 1, 0, 0, 1, 1, 1, 1]

acc = accuracy_score(y_true, y_pred)
print(f"Accuracy: {acc:.4f}")  # 0.7500

# ⚠️ Misleading for imbalanced data:
# If 95% of data is class 0, predicting all 0s gives 95% accuracy
```

### 1.2 Precision, Recall, F1-Score

```python
from sklearn.metrics import precision_score, recall_score, f1_score

precision = precision_score(y_true, y_pred)
recall    = recall_score(y_true, y_pred)
f1        = f1_score(y_true, y_pred)

print(f"Precision: {precision:.4f}")  # Of predicted positives, how many correct?
print(f"Recall:    {recall:.4f}")     # Of actual positives, how many found?
print(f"F1 Score:  {f1:.4f}")         # Harmonic mean of precision & recall
```

| Metric | Formula | Focus |
|---|---|---|
| **Precision** | TP / (TP + FP) | Minimize false positives (spam filter) |
| **Recall** | TP / (TP + FN) | Minimize false negatives (disease detection) |
| **F1 Score** | 2 × (P × R) / (P + R) | Balance precision and recall |

### 1.3 classification_report — All at Once

```python
from sklearn.metrics import classification_report
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

clf = RandomForestClassifier(random_state=42)
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)

print(classification_report(y_test, y_pred, target_names=load_iris().target_names))
```

Output:
```
              precision    recall  f1-score   support
      setosa       1.00      1.00      1.00        10
  versicolor       1.00      1.00      1.00        10
   virginica       1.00      1.00      1.00        10
    accuracy                           1.00        30
   macro avg       1.00      1.00      1.00        30
weighted avg       1.00      1.00      1.00        30
```

### 1.4 Multi-Class Averaging

```python
# For multi-class problems, specify the averaging method
f1_macro    = f1_score(y_test, y_pred, average='macro')    # unweighted mean per class
f1_weighted = f1_score(y_test, y_pred, average='weighted') # weighted by support
f1_micro    = f1_score(y_test, y_pred, average='micro')    # global TP/FP/FN

print(f"Macro F1:    {f1_macro:.4f}")
print(f"Weighted F1: {f1_weighted:.4f}")
print(f"Micro F1:    {f1_micro:.4f}")
```

| Average | Description | When to Use |
|---|---|---|
| `macro` | Mean of per-class metrics (equal weight) | All classes equally important |
| `weighted` | Weighted by class support | Imbalanced but care about majority |
| `micro` | Global counts (= accuracy for multi-class) | Large datasets |

---

## 2. Confusion Matrix

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

cm = confusion_matrix(y_test, y_pred)
print(cm)

# Visual display
disp = ConfusionMatrixDisplay(cm, display_labels=load_iris().target_names)
disp.plot(cmap='Blues')
plt.title('Confusion Matrix')
plt.tight_layout()
plt.savefig('confusion_matrix.png', dpi=150)
plt.show()
```

### Reading a Binary Confusion Matrix

```
                 Predicted
              Neg       Pos
Actual Neg [ TN        FP ]
Actual Pos [ FN        TP ]

TN = True Negatives   | FP = False Positives (Type I Error)
FN = False Negatives (Type II Error) | TP = True Positives
```

### Normalized Confusion Matrix

```python
cm_normalized = confusion_matrix(y_test, y_pred, normalize='true')
disp = ConfusionMatrixDisplay(cm_normalized, display_labels=load_iris().target_names)
disp.plot(cmap='Blues', values_format='.2f')
plt.title('Normalized Confusion Matrix')
plt.show()
```

---

## 3. ROC Curve and AUC

### 3.1 Binary Classification

```python
from sklearn.metrics import roc_curve, roc_auc_score
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
import matplotlib.pyplot as plt

X, y = make_classification(n_samples=500, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

clf = LogisticRegression(max_iter=200)
clf.fit(X_train, y_train)

# Get probability scores for the positive class
y_proba = clf.predict_proba(X_test)[:, 1]

# ROC curve
fpr, tpr, thresholds = roc_curve(y_test, y_proba)
auc = roc_auc_score(y_test, y_proba)

plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, label=f'Logistic Regression (AUC = {auc:.4f})')
plt.plot([0, 1], [0, 1], 'k--', label='Random Classifier')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

print(f"AUC: {auc:.4f}")
```

### 3.2 Multi-Class ROC-AUC

```python
from sklearn.metrics import roc_auc_score

y_proba_multi = clf.predict_proba(X_test)

# One-vs-Rest AUC
auc_ovr = roc_auc_score(y_test, y_proba_multi, multi_class='ovr', average='macro')
print(f"Multi-class AUC (OvR): {auc_ovr:.4f}")
```

---

## 4. Precision-Recall Curve

Better than ROC for **imbalanced datasets**.

```python
from sklearn.metrics import precision_recall_curve, average_precision_score
import matplotlib.pyplot as plt

precision_vals, recall_vals, thresholds = precision_recall_curve(y_test, y_proba)
ap = average_precision_score(y_test, y_proba)

plt.figure(figsize=(8, 6))
plt.plot(recall_vals, precision_vals, label=f'AP = {ap:.4f}')
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.title('Precision-Recall Curve')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

### Choosing the Right Threshold

```python
# Find threshold that balances precision and recall
f1_scores = 2 * (precision_vals[:-1] * recall_vals[:-1]) / (precision_vals[:-1] + recall_vals[:-1] + 1e-10)
best_idx = np.argmax(f1_scores)
best_threshold = thresholds[best_idx]

print(f"Best threshold: {best_threshold:.4f}")
print(f"Precision at best: {precision_vals[best_idx]:.4f}")
print(f"Recall at best: {recall_vals[best_idx]:.4f}")

# Apply custom threshold
y_custom = (y_proba >= best_threshold).astype(int)
```

---

## 5. Regression Metrics

```python
from sklearn.metrics import (mean_squared_error, mean_absolute_error,
                             r2_score, explained_variance_score,
                             root_mean_squared_error)
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression

X, y = make_regression(n_samples=200, n_features=5, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

reg = LinearRegression()
reg.fit(X_train, y_train)
y_pred = reg.predict(X_test)

mse  = mean_squared_error(y_test, y_pred)
rmse = root_mean_squared_error(y_test, y_pred)
mae  = mean_absolute_error(y_test, y_pred)
r2   = r2_score(y_test, y_pred)
ev   = explained_variance_score(y_test, y_pred)

print(f"MSE:  {mse:.4f}")
print(f"RMSE: {rmse:.4f}")
print(f"MAE:  {mae:.4f}")
print(f"R²:   {r2:.4f}")
print(f"Explained Variance: {ev:.4f}")
```

### Regression Metrics Reference

| Metric | Formula | Range | Interpretation |
|---|---|---|---|
| **MSE** | mean((y - ŷ)²) | [0, ∞) | Penalizes large errors heavily |
| **RMSE** | √MSE | [0, ∞) | Same unit as target variable |
| **MAE** | mean(\|y - ŷ\|) | [0, ∞) | Robust to outliers |
| **R²** | 1 - SS_res/SS_tot | (-∞, 1] | 1 = perfect, 0 = mean baseline |
| **Explained Var** | 1 - Var(y-ŷ)/Var(y) | (-∞, 1] | Like R² but ignores bias |

### When to Use Which

```python
# MAE vs MSE: MAE is more robust to outliers
y_true_out = np.array([1, 2, 3, 100])  # outlier at 100
y_pred_out = np.array([1.1, 2.2, 2.8, 4])

print(f"MAE:  {mean_absolute_error(y_true_out, y_pred_out):.2f}")   # ~24.1
print(f"MSE:  {mean_squared_error(y_true_out, y_pred_out):.2f}")    # ~2304
# MSE is dominated by the outlier, MAE is more balanced
```

---

## 6. Using Metrics with Cross-Validation

```python
from sklearn.model_selection import cross_val_score

clf = RandomForestClassifier(random_state=42)

# Classification metrics
acc_scores = cross_val_score(clf, X, y, cv=5, scoring='accuracy')
f1_scores  = cross_val_score(clf, X, y, cv=5, scoring='f1_macro')

print(f"Accuracy: {acc_scores.mean():.4f} ± {acc_scores.std():.4f}")
print(f"F1 Macro: {f1_scores.mean():.4f} ± {f1_scores.std():.4f}")

# Regression metrics (note: negated!)
reg = LinearRegression()
mse_scores = cross_val_score(reg, X, y, cv=5, scoring='neg_mean_squared_error')
r2_scores  = cross_val_score(reg, X, y, cv=5, scoring='r2')

print(f"MSE: {-mse_scores.mean():.4f}")  # negate to get positive MSE
print(f"R²:  {r2_scores.mean():.4f}")
```

---

## 7. make_scorer — Custom Metrics

```python
from sklearn.metrics import make_scorer, fbeta_score

# F-beta score with beta=2 (weights recall higher)
f2_score = make_scorer(fbeta_score, beta=2)
scores = cross_val_score(clf, X, y, cv=5, scoring=f2_score)

# Custom metric function
def specificity(y_true, y_pred):
    """True Negative Rate = TN / (TN + FP)"""
    tn = ((y_true == 0) & (y_pred == 0)).sum()
    fp = ((y_true == 0) & (y_pred == 1)).sum()
    return tn / (tn + fp) if (tn + fp) > 0 else 0.0

specificity_scorer = make_scorer(specificity)
scores = cross_val_score(clf, X, y, cv=5, scoring=specificity_scorer)
print(f"Specificity: {scores.mean():.4f}")
```

### make_scorer for Regression

```python
def mean_absolute_percentage_error(y_true, y_pred):
    """MAPE — percentage-based error."""
    return np.mean(np.abs((y_true - y_pred) / (y_true + 1e-10))) * 100

mape_scorer = make_scorer(mean_absolute_percentage_error, greater_is_better=False)
scores = cross_val_score(reg, X, y, cv=5, scoring=mape_scorer)
print(f"MAPE: {-scores.mean():.2f}%")
```

---

## 8. Comparing Models with Multiple Metrics

```python
from sklearn.model_selection import cross_validate
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC

models = {
    'Logistic Regression': LogisticRegression(max_iter=200),
    'Random Forest': RandomForestClassifier(random_state=42),
    'SVM': SVC(probability=True)
}

scoring = ['accuracy', 'f1_macro', 'roc_auc_ovr']

for name, model in models.items():
    results = cross_validate(model, X, y, cv=5, scoring=scoring)
    print(f"\n{name}:")
    for metric in scoring:
        key = f'test_{metric}'
        print(f"  {metric}: {results[key].mean():.4f} ± {results[key].std():.4f}")
```

### cross_validate vs cross_val_score

```python
# cross_validate returns a dict with multiple metrics + timing info
results = cross_validate(
    clf, X, y, cv=5,
    scoring=['accuracy', 'f1_macro'],
    return_train_score=True  # also compute training scores
)

print(f"Test accuracy:  {results['test_accuracy'].mean():.4f}")
print(f"Train accuracy: {results['train_accuracy'].mean():.4f}")
# Large gap between train and test → overfitting
```

---

## 9. Metric Selection Guide

| Scenario | Recommended Metric |
|---|---|
| Balanced binary classification | Accuracy, F1 |
| Imbalanced binary classification | F1, Precision-Recall AUC, Recall |
| Multi-class classification | F1 (macro/weighted), accuracy |
| Ranking/scoring | ROC-AUC |
| Regression (general) | RMSE, R² |
| Regression with outliers | MAE |
| Regression (percentage errors) | MAPE |
| Cost-sensitive problems | Custom scorer with business costs |

---

## 📋 Summary Table

| Metric/Tool | Task | Key Function |
|---|---|---|
| `accuracy_score` | Classification | Overall correctness |
| `precision_score` | Classification | Positive prediction quality |
| `recall_score` | Classification | Positive detection rate |
| `f1_score` | Classification | Precision-recall balance |
| `classification_report` | Classification | Complete per-class report |
| `confusion_matrix` | Classification | Prediction breakdown |
| `roc_auc_score` | Classification | Ranking quality |
| `mean_squared_error` | Regression | Average squared error |
| `mean_absolute_error` | Regression | Average absolute error |
| `r2_score` | Regression | Variance explained |
| `make_scorer` | Any | Custom metric for CV |
| `cross_validate` | Any | Multi-metric CV evaluation |

---

## ❓ Revision Questions

1. **Why is accuracy a poor metric for imbalanced datasets? What should you use instead?**
2. **Explain the difference between precision and recall. In what scenario would you prioritize recall over precision?**
3. **What does the ROC-AUC score represent and why is it useful for comparing models?**
4. **When would you prefer MAE over RMSE for evaluating a regression model?**
5. **How do you use `make_scorer` to create a custom evaluation metric for cross-validation?**
6. **What does a large gap between `train_accuracy` and `test_accuracy` in `cross_validate` results indicate?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← 05 - Pipeline Creation](05-pipeline-creation.md) | [Unit 5: Scikit-learn Basics](../README.md) | — |
