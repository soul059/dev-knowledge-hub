# Text Classification: Problem Formulation

## Overview

Text classification assigns predefined labels to text documents. It's the most common NLP task — powering spam detection, sentiment analysis, topic categorization, intent detection, and content moderation. This chapter frames text classification as a machine learning problem.

---

## Problem Definition

```
Input:   Document x (text string)
Output:  Label y ∈ {c₁, c₂, ..., cₖ}

Goal:    Learn function f(x) → y that generalizes to unseen documents

Examples:
  "This movie was fantastic!" → Positive
  "Server error 500"          → Bug Report
  "How do I reset password?"  → Account Support
```

---

## Classification Types

```
Binary Classification (2 classes):
  Spam / Not Spam
  Positive / Negative
  Toxic / Not Toxic

Multi-class Classification (k > 2, one label per doc):
  Sports | Politics | Technology | Entertainment
  (each doc gets exactly one label)

Multi-label Classification (multiple labels per doc):
  [Technology, AI, Business]  ← a doc can have multiple tags
  [Sports, Health]
```

---

## The Text Classification Pipeline

```
┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Raw Text │──▶│ Preprocessing │──▶│ Feature  │──▶│  Model   │──▶│ Predicted│
│          │   │ tokenize,     │   │ Extract  │   │ train/   │   │  Label   │
│          │   │ normalize     │   │ BoW/TFIDF│   │ predict  │   │          │
│          │   │               │   │ /embed   │   │          │   │          │
└──────────┘   └──────────────┘   └──────────┘   └──────────┘   └──────────┘
```

---

## Dataset Structure

```python
import pandas as pd

# Typical text classification dataset
data = pd.DataFrame({
    'text': [
        "Great product, highly recommend!",
        "Terrible experience, waste of money",
        "Average quality, nothing special",
        "Absolutely loved it, will buy again",
        "Broke after one day, very disappointed"
    ],
    'label': ['positive', 'negative', 'neutral', 'positive', 'negative']
})

# Train/test split
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    data['text'], data['label'], test_size=0.2, random_state=42,
    stratify=data['label']  # maintain class distribution
)
```

---

## Quick End-to-End Example

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report

# Build pipeline
pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=5000, ngram_range=(1, 2))),
    ('clf', LogisticRegression(max_iter=1000))
])

# Train
pipeline.fit(X_train, y_train)

# Predict and evaluate
predictions = pipeline.predict(X_test)
print(classification_report(y_test, predictions))
```

---

## Key Considerations

| Consideration | Questions to Ask |
|--------------|------------------|
| Number of classes | Binary or multi-class? Multi-label? |
| Class balance | Equal samples per class or imbalanced? |
| Text length | Short (tweets) or long (documents)? |
| Domain | General or specialized (medical, legal)? |
| Data size | Hundreds? Thousands? Millions? |
| Latency | Real-time prediction needed? |

---

## Revision Questions

1. **What is the difference between multi-class and multi-label classification?**
2. **Why is stratified train/test splitting important for text classification?**
3. **What are the main steps in a text classification pipeline?**
4. **How does the choice of feature representation affect classifier performance?**
5. **Name five real-world applications of text classification.**

---

[Previous: Unit 4 - 07-oov-handling.md](../04-Word-Embeddings/07-oov-handling.md) | [Next: 02-feature-extraction.md](02-feature-extraction.md)
