# Evaluation Metrics for Text Classification

## Overview

Choosing the right evaluation metric is as important as choosing the right model. Accuracy can be misleading; F1 balances precision and recall; AUC measures ranking quality. This chapter covers all key metrics with formulas, intuition, and when to use each.

---

## Confusion Matrix Foundation

```
                    Predicted
                 Pos        Neg
Actual  Pos  [  TP    |    FN  ]
        Neg  [  FP    |    TN  ]

TP = True Positive:  Correctly predicted positive
FP = False Positive: Incorrectly predicted positive (Type I error)
FN = False Negative: Missed positive (Type II error)
TN = True Negative:  Correctly predicted negative
```

---

## Core Metrics

### Accuracy
```
Accuracy = (TP + TN) / (TP + TN + FP + FN)

Good when: Classes are balanced
Bad when:  Imbalanced data (95% accuracy by predicting majority class)
```

### Precision
```
Precision = TP / (TP + FP)

"Of all items predicted positive, how many were actually positive?"
High precision = few false alarms

Important for: Spam detection (don't mark real email as spam)
```

### Recall (Sensitivity)
```
Recall = TP / (TP + FN)

"Of all actual positives, how many did we catch?"
High recall = few missed positives

Important for: Disease detection (don't miss sick patients)
```

### F1 Score
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)

Harmonic mean of precision and recall.
F1 = 1.0 → perfect
F1 = 0.0 → worst

Use when: You need to balance precision and recall
```

---

## Multi-class Averaging

```python
from sklearn.metrics import f1_score

# Macro: unweighted mean across classes (treats all classes equally)
f1_macro = f1_score(y_true, y_pred, average='macro')

# Micro: global TP, FP, FN (dominated by large classes)
f1_micro = f1_score(y_true, y_pred, average='micro')

# Weighted: weighted by class frequency
f1_weighted = f1_score(y_true, y_pred, average='weighted')
```

| Averaging | Formula | Best When |
|-----------|---------|-----------|
| Macro | Mean(F1 per class) | All classes equally important |
| Micro | Global TP/FP/FN | Large dataset, classes similar size |
| Weighted | Frequency-weighted mean | Want to account for class size |

---

## Complete Evaluation in Python

```python
from sklearn.metrics import (classification_report, confusion_matrix,
                              roc_auc_score, average_precision_score)
import numpy as np

y_true = [0, 1, 1, 0, 1, 0, 1, 1, 0, 1]
y_pred = [0, 1, 0, 0, 1, 1, 1, 1, 0, 0]

# Full classification report
print(classification_report(y_true, y_pred, target_names=['Neg', 'Pos']))
```
```
              precision    recall  f1-score   support
         Neg       0.75      0.75      0.75         4
         Pos       0.80      0.67      0.73         6
    accuracy                           0.70        10
   macro avg       0.78      0.71      0.74        10
weighted avg       0.78      0.70      0.73        10
```

```python
# Confusion matrix
cm = confusion_matrix(y_true, y_pred)
print(cm)
# [[3, 1],    ← TN=3, FP=1
#  [2, 4]]    ← FN=2, TP=4
```

---

## ROC-AUC and PR-AUC

```
ROC Curve: TPR vs FPR at different thresholds
  TPR (Recall) = TP / (TP + FN)
  FPR          = FP / (FP + TN)
  AUC = 1.0 → perfect, 0.5 → random

PR Curve: Precision vs Recall at different thresholds
  Better than ROC for imbalanced datasets
  
Use ROC-AUC for: balanced binary classification
Use PR-AUC for:  imbalanced datasets (focus on positive class)
```

```python
# Requires probability scores, not hard predictions
y_probs = model.predict_proba(X_test)[:, 1]

roc_auc = roc_auc_score(y_true, y_probs)
pr_auc = average_precision_score(y_true, y_probs)
print(f"ROC-AUC: {roc_auc:.3f}, PR-AUC: {pr_auc:.3f}")
```

---

## Metric Selection Guide

| Scenario | Best Metric | Why |
|----------|-------------|-----|
| Balanced binary | Accuracy or F1 | Both informative |
| Imbalanced binary | F1, PR-AUC | Accuracy is misleading |
| Multi-class balanced | Macro F1 | Equal weight to all classes |
| Multi-class imbalanced | Macro F1 or Weighted F1 | Depends on goal |
| Ranking quality | ROC-AUC | Threshold-independent |
| Cost of FP >> FN | Precision | Minimize false alarms |
| Cost of FN >> FP | Recall | Catch all positives |

---

## Revision Questions

1. **Why is accuracy misleading for a dataset with 95% negative samples?**
2. **When would you prioritize precision over recall?**
3. **What is the difference between macro and micro F1?**
4. **Why is PR-AUC preferred over ROC-AUC for imbalanced datasets?**
5. **Calculate F1 given precision=0.8 and recall=0.6.**
6. **How do you interpret a confusion matrix for a 3-class problem?**

---

[Previous: 06-imbalanced-data.md](06-imbalanced-data.md) | [Next: Unit 6 - 01-pos-tagging.md](../06-Sequence-Labeling/01-pos-tagging.md)
