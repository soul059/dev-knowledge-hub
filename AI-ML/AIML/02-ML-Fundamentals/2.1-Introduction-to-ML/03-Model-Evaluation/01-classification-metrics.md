# 📊 Classification Metrics

## Chapter Overview
Classification metrics quantify how well a model distinguishes between discrete classes. **Accuracy alone can be dangerously misleading** on imbalanced datasets. This chapter covers every essential classification metric, when to use each, and how to implement them in Python.

---

## 1. The Confusion Matrix Foundation
All classification metrics derive from four fundamental counts:
```
                    Predicted
                 Pos        Neg
Actual  Pos  [  TP   |   FN  ]
        Neg  [  FP   |   TN  ]
```
| Symbol | Meaning | Description |
|--------|---------|-------------|
| **TP** | True Positive | Correctly predicted positive |
| **TN** | True Negative | Correctly predicted negative |
| **FP** | False Positive | Incorrectly predicted positive (Type I Error) |
| **FN** | False Negative | Incorrectly predicted negative (Type II Error) |

---

## 2. Accuracy
```
Accuracy = (TP + TN) / (TP + TN + FP + FN)
```
**Interpretation:** Fraction of all predictions that are correct.

### ⚠️ When Accuracy Fails — The Imbalanced Data Problem
Consider a dataset with 950 negatives and 50 positives. A model predicting **everything as negative** achieves **95% accuracy** yet catches zero positives. This is the *accuracy paradox*.

> **Rule of thumb:** Avoid accuracy as the primary metric when class ratio exceeds **80:20**.

---

## 3. Precision
```
Precision = TP / (TP + FP)
```
**Meaning:** "Of all instances the model **predicted positive**, how many were actually positive?"

**Optimize precision when** the cost of FP is high (spam filter blocking legit email, irrelevant recommendations).

---

## 4. Recall (Sensitivity / True Positive Rate)
```
Recall = TP / (TP + FN)
```
**Meaning:** "Of all **actual positive** instances, how many did the model find?"

**Optimize recall when** the cost of FN is high (cancer screening, fraud detection).

---

## 5. F1-Score
```
F1 = 2 · (Precision · Recall) / (Precision + Recall)
```
The **harmonic mean** of precision and recall — penalizes extreme imbalance more than the arithmetic mean:

| Precision | Recall | Arithmetic Mean | F1 (Harmonic Mean) |
|-----------|--------|-----------------|---------------------|
| 1.00 | 0.01 | 0.505 | 0.0198 |
| 0.80 | 0.80 | 0.800 | 0.8000 |
| 0.60 | 0.90 | 0.750 | 0.7200 |

**Fβ-Score (Generalized):** `Fβ = (1 + β²)·(P·R) / (β²·P + R)` — β < 1 weights precision more, β > 1 weights recall more.

---

## 6. Specificity & Balanced Accuracy
```
Specificity      = TN / (TN + FP)
Balanced Accuracy = (Sensitivity + Specificity) / 2
```
- **Specificity:** "Of all actual negatives, how many correctly identified?" Important in medical testing.
- **Balanced Accuracy:** Equal weight to each class regardless of size — ideal for **imbalanced datasets**.

---

## 7. ROC Curve and AUC
The **Receiver Operating Characteristic** curve plots TPR vs FPR at various thresholds.
```
  TPR (Recall)
  1.0 |        .-------*
      |      /
      |    /        (Good model)
  0.5 |  /
      | / . . . . . (Random = diagonal)
      |/
  0.0 +---------------
      0.0   0.5   1.0
         FPR (1 - Specificity)
```
| AUC Value | Interpretation |
|-----------|----------------|
| 1.0 | Perfect classifier |
| 0.9–1.0 | Excellent |
| 0.7–0.9 | Good |
| 0.5 | Random guessing (no skill) |
| < 0.5 | Worse than random |

**AUC** summarizes the ROC into a single number — the probability that the model ranks a random positive higher than a random negative.

---

## 8. Precision-Recall Curve and PR-AUC
When the positive class is rare, ROC-AUC can be overly optimistic. The **PR curve** plots Precision vs Recall:
```
  Precision
  1.0 |*
      | \
      |  \__
  0.5 |     \___
      |         \____
  0.0 +------------------
      0.0    0.5    1.0
            Recall
```
> **Use PR-AUC over ROC-AUC** when the positive class is very small (e.g., fraud detection at 0.1% fraud rate).

---

## 9. Multi-Class Averaging Strategies
| Strategy | Formula | When to Use |
|----------|---------|-------------|
| **Macro** | Average metric per class (equal weight) | All classes equally important |
| **Micro** | Aggregate TP/FP/FN globally, then compute | Large datasets, overall performance |
| **Weighted** | Average weighted by class support (count) | Imbalanced classes |

`Macro-F1 = (F1_class1 + F1_class2 + ... + F1_classK) / K`

---

## 10. When to Use Which Metric — Decision Guide
| Scenario | Recommended Metric |
|----------|-------------------|
| Balanced classes | Accuracy, F1 |
| Imbalanced classes | F1, PR-AUC, Balanced Accuracy |
| Cost of FP is high | Precision, Specificity |
| Cost of FN is high | Recall (Sensitivity) |
| Ranking/threshold tuning | ROC-AUC |
| Rare positive class | PR-AUC |
| Multi-class problem | Macro/Weighted F1 |

---

## 11. Step-by-Step Example
**Given:** TP=80, FP=20, FN=10, TN=890
```
Accuracy     = (80+890)/(80+890+20+10)        = 970/1000 = 0.970
Precision    = 80/(80+20)                      = 80/100   = 0.800
Recall       = 80/(80+10)                      = 80/90    = 0.889
F1           = 2·(0.800·0.889)/(0.800+0.889)              = 0.842
Specificity  = 890/(890+20)                    = 890/910  = 0.978
Balanced Acc = (0.889+0.978)/2                             = 0.934
```

---

## 12. Python Implementation
```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score,
    f1_score, roc_auc_score, classification_report,
    roc_curve, auc
)
import numpy as np

y_true = [1, 1, 1, 0, 0, 0, 1, 0, 1, 0]
y_pred = [1, 0, 1, 0, 0, 1, 1, 0, 1, 0]
y_prob = [0.9, 0.4, 0.8, 0.1, 0.2, 0.7, 0.85, 0.05, 0.75, 0.3]

print(f"Accuracy:  {accuracy_score(y_true, y_pred):.4f}")
print(f"Precision: {precision_score(y_true, y_pred):.4f}")
print(f"Recall:    {recall_score(y_true, y_pred):.4f}")
print(f"F1-Score:  {f1_score(y_true, y_pred):.4f}")
print(f"ROC-AUC:   {roc_auc_score(y_true, y_prob):.4f}")
print(classification_report(y_true, y_pred))

# Multi-class averaging
y_true_mc = [0, 1, 2, 0, 1, 2, 0, 2]
y_pred_mc = [0, 2, 2, 0, 0, 2, 0, 1]
print(f"Macro F1:    {f1_score(y_true_mc, y_pred_mc, average='macro'):.4f}")
print(f"Weighted F1: {f1_score(y_true_mc, y_pred_mc, average='weighted'):.4f}")

# ROC curve data
fpr, tpr, thresholds = roc_curve(y_true, y_prob)
roc_auc = auc(fpr, tpr)
```

---

## 13. Real-World ML Applications
| Domain | Primary Metric | Reason |
|--------|---------------|--------|
| Medical diagnosis | Recall | Missing disease is life-threatening |
| Email spam filter | Precision | Losing legitimate email is costly |
| Credit card fraud | PR-AUC | Fraud is extremely rare (~0.1%) |
| Autonomous driving | Recall | Missing a pedestrian is catastrophic |
| Search engine ranking | Precision@K | Users only see top K results |

---

## 📋 Summary Table
| Metric | Formula | Range | Best For |
|--------|---------|-------|----------|
| Accuracy | (TP+TN)/Total | [0,1] | Balanced data |
| Precision | TP/(TP+FP) | [0,1] | Low FP tolerance |
| Recall | TP/(TP+FN) | [0,1] | Low FN tolerance |
| F1 | 2PR/(P+R) | [0,1] | Balance P & R |
| Specificity | TN/(TN+FP) | [0,1] | True negative rate |
| Balanced Acc | (Sens+Spec)/2 | [0,1] | Imbalanced data |
| ROC-AUC | Area under ROC | [0,1] | Threshold tuning |
| PR-AUC | Area under PR | [0,1] | Rare positive class |

---

## ❓ Revision Questions
1. **A model achieves 98% accuracy on a dataset where 97% of samples are negative. Is this model good? What metric would you use instead?**
2. **A cancer screening system has Precision=0.95 and Recall=0.60. Should you deploy it? Which metric matters more here and why?**
3. **Calculate F1-Score given: TP=50, FP=10, FN=30, TN=910. Show all intermediate steps.**
4. **When would you prefer PR-AUC over ROC-AUC? Give a concrete example with class distribution.**
5. **Explain the difference between macro-averaging and weighted-averaging for F1 in a 3-class problem where class sizes are 100, 500, and 50.**
6. **A fraud detection model has ROC-AUC=0.95 but PR-AUC=0.30. What does this tell you about the model's performance? Which metric should you trust?**

---

## 🧭 Navigation
| | Link |
|------|------|
| ⬅️ Previous | — |
| ➡️ Next | [Regression Metrics](02-regression-metrics.md) |
| 📂 Up | [Model Evaluation](README.md) |
