# Handling Imbalanced Data in Text Classification

## Overview

In real-world text classification, classes are rarely balanced. Spam might be 1% of emails; toxic comments might be 5% of all comments. Imbalanced datasets cause models to be biased toward the majority class, achieving high accuracy while failing on the minority class that often matters most.

---

## The Problem

```
Dataset: Customer complaints classification
  General Inquiry:   8,000 samples (80%)
  Billing Issue:     1,500 samples (15%)
  Safety Concern:      500 samples (5%)    ← Most critical!

A model that always predicts "General Inquiry" gets 80% accuracy
but NEVER catches safety concerns.
```

---

## Solutions

### 1. Resampling Techniques

```python
# Oversampling: duplicate minority class
from imblearn.over_sampling import RandomOverSampler, SMOTE

# Random oversampling
ros = RandomOverSampler(random_state=42)
X_resampled, y_resampled = ros.fit_resample(X_train, y_train)

# SMOTE: create synthetic minority samples
# (works with numerical features, not raw text — apply after TF-IDF)
smote = SMOTE(random_state=42)
X_smote, y_smote = smote.fit_resample(X_train_tfidf, y_train)
```

```python
# Undersampling: reduce majority class
from imblearn.under_sampling import RandomUnderSampler

rus = RandomUnderSampler(random_state=42)
X_under, y_under = rus.fit_resample(X_train, y_train)
# Warning: loses majority class data
```

### 2. Class Weights

```python
from sklearn.utils.class_weight import compute_class_weight
import numpy as np

# Automatically compute weights inversely proportional to frequency
weights = compute_class_weight('balanced', classes=np.unique(y_train), y=y_train)
class_weight_dict = dict(zip(np.unique(y_train), weights))
# {'General': 0.42, 'Billing': 2.22, 'Safety': 6.67}

# Use in sklearn
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(class_weight='balanced')

# Use in PyTorch
import torch
weight_tensor = torch.FloatTensor(weights)
criterion = torch.nn.CrossEntropyLoss(weight=weight_tensor)
```

### 3. Focal Loss (for Neural Networks)

```python
import torch
import torch.nn as nn

class FocalLoss(nn.Module):
    def __init__(self, alpha=1, gamma=2):
        super().__init__()
        self.alpha = alpha
        self.gamma = gamma
    
    def forward(self, inputs, targets):
        ce_loss = nn.functional.cross_entropy(inputs, targets, reduction='none')
        p_t = torch.exp(-ce_loss)
        focal_loss = self.alpha * (1 - p_t) ** self.gamma * ce_loss
        return focal_loss.mean()

# gamma=0: standard CE loss
# gamma=2: down-weights easy (majority) examples significantly
```

### 4. Threshold Adjustment

```python
# Lower threshold for minority class to increase recall
probabilities = model.predict_proba(X_test)

# Instead of argmax, use custom thresholds
# Safety concern: lower threshold to catch more
thresholds = {'General': 0.5, 'Billing': 0.4, 'Safety': 0.2}
```

---

## Strategy Comparison

| Strategy | Pros | Cons | Best For |
|----------|------|------|----------|
| Oversampling | No data loss | Overfitting on duplicates | Small datasets |
| SMOTE | Creates novel samples | Only works on features, not text | After vectorization |
| Undersampling | Reduces training time | Loses majority data | Very large datasets |
| Class weights | Simple, no data change | May not fully solve | First approach to try |
| Focal loss | Elegant, adaptive | Requires tuning γ | Deep learning models |
| Threshold tuning | Post-hoc, flexible | Need validation set | After training |

---

## Evaluation for Imbalanced Data

```
❌ DON'T use: Accuracy (misleading with imbalanced classes)

✅ DO use:
  • Precision, Recall, F1 per class
  • Macro F1 (equal weight to all classes)
  • PR-AUC (Precision-Recall AUC)
  • Confusion matrix (see where errors happen)
```

---

## Revision Questions

1. **Why is accuracy a misleading metric for imbalanced datasets?**
2. **How does `class_weight='balanced'` work in sklearn?**
3. **What is focal loss and how does it address class imbalance?**
4. **When would you choose oversampling vs class weights?**
5. **Why might you use a lower classification threshold for the minority class?**

---

[Previous: 05-multi-label.md](05-multi-label.md) | [Next: 07-evaluation-metrics.md](07-evaluation-metrics.md)
