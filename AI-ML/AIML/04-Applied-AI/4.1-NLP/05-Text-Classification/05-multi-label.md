# Multi-label Text Classification

## Overview

In multi-label classification, each document can belong to **multiple classes simultaneously**. A news article might be tagged [Politics, Economy, International]. Unlike multi-class (exactly one label), multi-label requires different loss functions, architectures, and evaluation metrics.

---

## Multi-class vs Multi-label

```
Multi-class (mutually exclusive):
  "The cat sat on the mat" → Animal  (one label only)

Multi-label (can overlap):
  "AI-powered drug discovery startup raises $50M"
  → [Technology, Healthcare, Business]  (multiple labels)

  Output layer difference:
    Multi-class:  Softmax → probabilities sum to 1
    Multi-label:  Sigmoid → independent probability per label
```

---

## Architecture

```python
import torch
import torch.nn as nn

class MultiLabelClassifier(nn.Module):
    def __init__(self, input_dim, num_labels):
        super().__init__()
        self.fc = nn.Sequential(
            nn.Linear(input_dim, 256),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(256, num_labels),
            nn.Sigmoid()  # Independent probability per label
        )
    
    def forward(self, x):
        return self.fc(x)

# Loss function: Binary Cross Entropy (independent per label)
criterion = nn.BCELoss()

# Example
num_labels = 5  # Technology, Sports, Politics, Health, Business
model = MultiLabelClassifier(768, num_labels)

# Labels are multi-hot encoded
# "AI health startup" → [1, 0, 0, 1, 1]  (Tech, Health, Business)
labels = torch.tensor([[1, 0, 0, 1, 1]], dtype=torch.float)
```

---

## Sklearn Multi-label

```python
from sklearn.multioutput import MultiOutputClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import MultiLabelBinarizer

# Convert labels to binary matrix
mlb = MultiLabelBinarizer()
y_binary = mlb.fit_transform(y_train_labels)
# [["tech", "health"]] → [[1, 0, 1, 0, 0]]

# One classifier per label (Binary Relevance)
pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=10000)),
    ('clf', MultiOutputClassifier(LogisticRegression(max_iter=1000)))
])

pipeline.fit(X_train, y_binary)
predictions = pipeline.predict(X_test)
```

---

## Thresholding

```python
# Sigmoid outputs probabilities per label
probs = model(features)
# tensor([0.92, 0.15, 0.03, 0.87, 0.71])

# Default threshold = 0.5
preds_default = (probs > 0.5).int()
# tensor([1, 0, 0, 1, 1]) → Tech, Health, Business

# Optimized threshold per label (tune on validation set)
thresholds = [0.4, 0.3, 0.5, 0.45, 0.6]
preds_custom = torch.tensor([
    int(p > t) for p, t in zip(probs, thresholds)
])
```

---

## Evaluation Metrics

| Metric | Formula | Handles Multi-label? |
|--------|---------|:---:|
| Exact Match Ratio | All labels must match exactly | Strict |
| Hamming Loss | Fraction of wrong labels | ✅ Per-label |
| Micro F1 | Global TP, FP, FN across all labels | ✅ |
| Macro F1 | Average F1 per label | ✅ |
| Sample F1 | F1 per sample, then average | ✅ |

```python
from sklearn.metrics import hamming_loss, f1_score

print(f"Hamming Loss: {hamming_loss(y_true, y_pred):.3f}")
print(f"Micro F1:     {f1_score(y_true, y_pred, average='micro'):.3f}")
print(f"Macro F1:     {f1_score(y_true, y_pred, average='macro'):.3f}")
```

---

## Revision Questions

1. **What is the key difference between multi-class and multi-label classification?**
2. **Why do we use Sigmoid instead of Softmax for multi-label?**
3. **What loss function is appropriate for multi-label classification and why?**
4. **How does threshold selection affect multi-label predictions?**
5. **Explain the difference between Micro F1 and Macro F1 in multi-label context.**

---

[Previous: 04-deep-learning-approaches.md](04-deep-learning-approaches.md) | [Next: 06-imbalanced-data.md](06-imbalanced-data.md)
