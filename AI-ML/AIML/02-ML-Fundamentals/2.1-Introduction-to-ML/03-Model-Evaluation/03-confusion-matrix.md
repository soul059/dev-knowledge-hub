# 🔢 Confusion Matrix

## Chapter Overview
The confusion matrix is the **single most informative structure** in classification evaluation. Every metric — accuracy, precision, recall, F1, specificity — is derived from it. This chapter covers construction, interpretation, multi-class extensions, error types, and Python implementation.

---

## 1. Core Definitions — TP, TN, FP, FN
Consider a binary classifier predicting **spam** (Positive) or **not spam** (Negative):

| Term | Full Name | Meaning | Spam Example |
|------|-----------|---------|--------------|
| **TP** | True Positive | Model says +, actually + | Spam correctly caught |
| **TN** | True Negative | Model says −, actually − | Legit email correctly passed |
| **FP** | False Positive | Model says +, actually − | Legit email wrongly blocked |
| **FN** | False Negative | Model says −, actually + | Spam slipped through |

> **Memory aid:** True/False = was model **correct**? Positive/Negative = **what model predicted**.

---

## 2. Constructing the 2×2 Confusion Matrix
```
                       PREDICTED
                   Positive    Negative
               ┌────────────┬────────────┐
    Positive   │     TP     │     FN     │  ← Actual Positives
  ACTUAL       ├────────────┼────────────┤
    Negative   │     FP     │     TN     │  ← Actual Negatives
               └────────────┴────────────┘
```

### Concrete Example — COVID Test (200 people: 50 infected, 150 healthy)
```
                   COVID+      COVID-
  ACTUAL  +    │  TP = 45   │  FN = 5    │   50
          -    │  FP = 10   │  TN = 140  │  150
                   55 pred+    145 pred-    200 total
```

---

## 3. Deriving ALL Metrics from the Confusion Matrix
From the COVID example (TP=45, FP=10, FN=5, TN=140):

```
Accuracy    = (TP+TN) / Total        = (45+140)/200    = 0.925
Precision   = TP / (TP+FP)           = 45/55           = 0.818
Recall      = TP / (TP+FN)           = 45/50           = 0.900
Specificity = TN / (TN+FP)           = 140/150         = 0.933
F1          = 2·(P·R)/(P+R)          = 2·0.736/1.718   = 0.857
FPR         = FP / (FP+TN)           = 10/150          = 0.067
```

```
  ┌─────────────────────────────────────────────┐
  │            CONFUSION MATRIX                 │
  │           TP=45  FN=5                       │
  │           FP=10  TN=140                     │
  ├─────────────────────────────────────────────┤
  │  Accuracy=0.925  Precision=0.818            │
  │  Recall=0.900    Specificity=0.933          │
  │  F1=0.857        FPR=0.067                  │
  └─────────────────────────────────────────────┘
```

---

## 4. Type I vs Type II Errors
| Error Type | Alias | Definition |
|------------|-------|------------|
| **Type I** | False Positive (FP) | False alarm — predicted +, actually − |
| **Type II** | False Negative (FN) | Missed detection — predicted −, actually + |

### Real-World Consequences
| Domain | Type I (FP) Cost | Type II (FN) Cost |
|--------|-----------------|-------------------|
| Medical diagnosis | Unnecessary treatment, anxiety | Missed disease → death |
| Criminal justice | Innocent person jailed | Guilty person goes free |
| Spam filter | Important email lost | Spam in inbox |
| Fraud detection | Legitimate transaction blocked | Fraud goes undetected |
| Fire alarm | False evacuation | No alarm during fire |

> **Key insight:** The relative cost of Type I vs Type II errors determines whether to optimize **precision** (minimize FP) or **recall** (minimize FN).

---

## 5. Multi-Class Confusion Matrix (N×N)
For K classes, the matrix becomes K×K. Cell (i,j) = actual class i predicted as class j.

### Example: 3-Class (Cat / Dog / Bird)
```
                     PREDICTED
                Cat    Dog    Bird
          ┌────────┬────────┬────────┐
  Cat     │   8    │   1    │   1    │  10
  ACTUAL  ├────────┼────────┼────────┤
  Dog     │   2    │   7    │   1    │  10
          ├────────┼────────┼────────┤
  Bird    │   0    │   1    │   9    │  10
          └────────┴────────┴────────┘
```
- **Diagonal** = correct predictions (8+7+9=24) | **Off-diagonal** = misclassifications
- Row sums = actual class counts | Column sums = predicted class counts

**Per-class metrics (for "Cat"):**
```
TP=8 (Cat→Cat)  FP=2 (Dog→Cat + Bird→Cat)  FN=2 (Cat→Dog + Cat→Bird)  TN=18
Precision(Cat) = 8/10 = 0.80    Recall(Cat) = 8/10 = 0.80
```

---

## 6. Normalized Confusion Matrix
Raw counts mislead when class sizes differ. Two normalization approaches:

**Row-Normalized** (by actual class) — shows **recall** per class (each row sums to 1.0):
```
  Cat  │  0.80  │  0.10  │  0.10  │
  Dog  │  0.20  │  0.70  │  0.10  │
  Bird │  0.00  │  0.10  │  0.90  │
```
**Column-Normalized** (by predicted class) — shows **precision** per class (each column sums to 1.0).

> **Row-normalized** is more common — it directly shows per-class recall rates.

---

## 7. Visualization Techniques
| Technique | Best For | Tool |
|-----------|----------|------|
| Annotated heatmap | 2-class and small multi-class | `ConfusionMatrixDisplay` |
| Color-coded heatmap | Large multi-class (10+ classes) | `seaborn.heatmap` |
| Normalized heatmap | Imbalanced classes | Normalize then plot |

**Goal:** Dark diagonal (high TP/TN), light off-diagonal (low FP/FN).

---

## 8. Practical Interpretation Examples

### Example A: Cancer Screening (1000 patients, 20 have cancer)
```
  Actual +  │  TP=18  │  FN=2   │     Accuracy  = 948/1000 = 0.948
  Actual -  │  FP=50  │  TN=930 │     Precision = 18/68    = 0.265
                                      Recall    = 18/20    = 0.900
```
High accuracy is misleading. Low precision (many false alarms), but high recall. For screening, **high recall is acceptable** — better to investigate 50 false alarms than miss 2 cancer patients.

### Example B: Spam Detection (10,000 emails, 500 spam)
```
  Spam     │  480  │   20   │        Precision = 480/495 = 0.970
  Not Spam │   15  │ 9,485  │        Recall    = 480/500 = 0.960
```
Both metrics high. The 15 FPs (legit emails blocked) are the main concern — **precision matters** to avoid losing important emails.

---

## 9. Python Implementation
```python
from sklearn.metrics import (
    confusion_matrix, ConfusionMatrixDisplay,
    classification_report, accuracy_score,
    precision_score, recall_score, f1_score
)
import numpy as np
import matplotlib.pyplot as plt

# --- Binary Classification ---
y_true = [1,1,1,1,1, 0,0,0,0,0, 1,0,1,0,1]
y_pred = [1,1,0,1,1, 0,0,1,0,0, 1,1,1,0,0]

cm = confusion_matrix(y_true, y_pred)
tn, fp, fn, tp = cm.ravel()
print(f"TP={tp}, TN={tn}, FP={fp}, FN={fn}")
print(f"Accuracy:  {accuracy_score(y_true, y_pred):.4f}")
print(f"Precision: {precision_score(y_true, y_pred):.4f}")
print(f"Recall:    {recall_score(y_true, y_pred):.4f}")
print(f"F1:        {f1_score(y_true, y_pred):.4f}")
print(classification_report(y_true, y_pred))

# Visualization
disp = ConfusionMatrixDisplay(confusion_matrix=cm,
                               display_labels=["Negative", "Positive"])
disp.plot(cmap="Blues", values_format="d")
plt.title("Binary Confusion Matrix")
plt.tight_layout()
plt.savefig("confusion_matrix.png", dpi=150)

# --- Multi-Class ---
y_true_mc = [0,0,0, 1,1,1, 2,2,2]
y_pred_mc = [0,1,0, 1,1,2, 0,2,2]
cm_mc = confusion_matrix(y_true_mc, y_pred_mc)
cm_norm = confusion_matrix(y_true_mc, y_pred_mc, normalize='true')
print("Multi-class CM:\n", cm_mc)
print("Normalized:\n", np.round(cm_norm, 2))

disp_mc = ConfusionMatrixDisplay(confusion_matrix=cm_mc,
                                  display_labels=["Cat","Dog","Bird"])
disp_mc.plot(cmap="Oranges", values_format="d")
plt.title("Multi-Class Confusion Matrix")
plt.tight_layout()
plt.savefig("confusion_matrix_multiclass.png", dpi=150)
```

---

## 10. Real-World ML Applications
| Application | Matrix Focus | Key Concern |
|-------------|-------------|-------------|
| Cancer screening | Minimize FN | Missing cancer is life-threatening |
| Spam filter | Minimize FP | Losing important emails |
| Fraud detection | Minimize FN | Undetected fraud causes losses |
| Autonomous vehicles | Minimize FN | Missing obstacles is catastrophic |
| Quality control | Minimize both | Defects shipped & good products rejected |

---

## 📋 Summary Table
| Concept | Key Point |
|---------|-----------|
| Confusion Matrix | 2×2 table of TP, TN, FP, FN |
| Diagonal | Correct predictions (TP + TN) |
| Off-diagonal | Errors (FP + FN) |
| Type I Error | FP — false alarm |
| Type II Error | FN — missed detection |
| Row-normalized | Shows recall per class |
| Column-normalized | Shows precision per class |
| Multi-class | K×K matrix for K classes |
| Perfect matrix | All values on diagonal, zeros elsewhere |

---

## ❓ Revision Questions
1. **Given the confusion matrix [[85, 15], [10, 90]], compute accuracy, precision, recall, F1, and specificity. Show all steps.**
2. **In a fire alarm system, which is more dangerous — Type I or Type II error? Explain with reference to FP and FN.**
3. **A 3-class confusion matrix shows class A is often misclassified as class B. What does this suggest, and how would you investigate?**
4. **Why is row-normalization preferred over column-normalization when comparing class-wise performance on imbalanced datasets?**
5. **A model has TP=100, FP=900, FN=0, TN=9000. Calculate precision and recall. Is this a good model? What is the likely data characteristic?**
6. **Explain how you would extract per-class TP, FP, FN, and TN from a 4×4 multi-class confusion matrix.**

---

## 🧭 Navigation
| | Link |
|------|------|
| ⬅️ Previous | [Regression Metrics](02-regression-metrics.md) |
| ➡️ Next | [Cross-Validation Techniques](04-cross-validation-techniques.md) |
| 📂 Up | [Model Evaluation](README.md) |
