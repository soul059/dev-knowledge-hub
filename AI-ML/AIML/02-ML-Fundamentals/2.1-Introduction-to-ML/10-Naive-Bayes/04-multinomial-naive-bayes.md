# 📘 Chapter 4: Multinomial Naive Bayes

> **Unit 10: Naive Bayes** | **Section 4 of 7**
> Naive Bayes for discrete count features using the multinomial distribution. Master document-term matrices, word frequency features, parameter estimation for text data, and build complete text classification pipelines with MultinomialNB and TF-IDF in scikit-learn.

---

## 📑 Table of Contents

1. [Introduction: Count-Based Features](#1-introduction-count-based-features)
2. [The Multinomial Distribution](#2-the-multinomial-distribution)
3. [Document-Term Matrix](#3-document-term-matrix)
4. [Parameter Estimation](#4-parameter-estimation)
5. [Classification with Multinomial NB](#5-classification-with-multinomial-nb)
6. [Worked Example: Text Classification](#6-worked-example-text-classification)
7. [MultinomialNB in scikit-learn](#7-multinomialnb-in-scikit-learn)
8. [TF-IDF Integration](#8-tf-idf-integration)
9. [Complete Text Classification Pipeline](#9-complete-text-classification-pipeline)
10. [Python Implementation](#10-python-implementation)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Introduction: Count-Based Features

### 1.1 When Gaussian Doesn't Fit

[Gaussian NB](03-gaussian-naive-bayes.md) works for continuous, bell-shaped features. But many real-world features are **discrete counts**:

```
    ┌─────────────────────────────────────────────────────────────┐
    │  COUNT-BASED FEATURES:                                     │
    │                                                             │
    │  • Word frequencies in documents   (NLP / text mining)     │
    │  • Pixel intensity counts          (image histograms)      │
    │  • Categorical feature counts      (survey responses)      │
    │  • Event occurrence counts         (web analytics)         │
    │  • Gene expression levels          (bioinformatics)        │
    │                                                             │
    │  These features are:                                        │
    │    ✓ Non-negative integers (0, 1, 2, 3, ...)              │
    │    ✓ Often sparse (many zeros)                              │
    │    ✓ NOT bell-shaped (right-skewed, zero-inflated)         │
    └─────────────────────────────────────────────────────────────┘
```

### 1.2 The Text Classification Use Case

Multinomial NB is **most famous** for text classification:

```
    Document → Bag of Words → Word Counts → Multinomial NB → Class

    Example email:
    "Buy cheap products now! Free shipping on all orders!"

    Word count vector:
    buy=1, cheap=1, products=1, now=1, free=1, shipping=1, all=1, orders=1

    This count vector is the input to Multinomial NB.
```

---

## 2. The Multinomial Distribution

### 2.1 Definition

The multinomial distribution models the probability of observing a specific set of counts across multiple categories:

```
    ╔═══════════════════════════════════════════════════════════════╗
    ║                                                               ║
    ║  Given a document of length L with vocabulary V = {w₁,...,wₘ} ║
    ║                                                               ║
    ║                    L!                                         ║
    ║  P(x|c) = ──────────────── × Π  P(wⱼ|c)^xⱼ                 ║
    ║            x₁! × x₂! ×...   j=1                             ║
    ║                                                               ║
    ║  Where:                                                       ║
    ║    xⱼ = count of word wⱼ in the document                    ║
    ║    P(wⱼ|c) = probability of word wⱼ in class c             ║
    ║    L = Σxⱼ = total number of words in the document          ║
    ║                                                               ║
    ║  For classification, the multinomial coefficient is constant  ║
    ║  across classes, so we only need:                             ║
    ║                                                               ║
    ║  P(x|c) ∝ Π P(wⱼ|c)^xⱼ                                    ║
    ║                                                               ║
    ╚═══════════════════════════════════════════════════════════════╝
```

### 2.2 Intuition: The Bag-of-Words Model

```
    The "bag of words" treats a document as an unordered collection:
    
    Original: "the cat sat on the mat"
    
    Bag of words:
    ┌─────────────────────────────────┐
    │  the: 2  │  cat: 1  │  sat: 1  │
    │  on: 1   │  mat: 1  │          │
    └─────────────────────────────────┘
    
    Word ORDER is lost → "the cat sat" = "sat cat the"
    Word COUNTS are preserved → "the" appears twice
    
    This is a SIMPLIFICATION, but works surprisingly well
    for classification tasks!
```

### 2.3 Multinomial vs Categorical

```
    ┌────────────────────────────────────────────────────┐
    │  CATEGORICAL (single trial):                      │
    │    Roll one die → {1, 2, 3, 4, 5, 6}            │
    │    One outcome from K categories                  │
    │                                                    │
    │  MULTINOMIAL (multiple trials):                   │
    │    Roll a die N times → count each outcome        │
    │    "4 ones, 2 twos, 1 three, 0 fours, ..."      │
    │    Multiple outcomes from K categories             │
    │                                                    │
    │  A document is like rolling a "word die" N times: │
    │    N = document length                             │
    │    K = vocabulary size                             │
    │    Each "roll" picks a word from the vocabulary    │
    └────────────────────────────────────────────────────┘
```

---

## 3. Document-Term Matrix

### 3.1 Structure

The **document-term matrix (DTM)** is the standard input for text classification:

```
    Documents (rows) × Words (columns) = Count Matrix
    
              │ buy │ free │ click │ meeting │ report │ project │
    ──────────┼─────┼──────┼───────┼─────────┼────────┼─────────┤
    Doc 1     │  2  │  3   │   1   │    0    │   0    │    0    │  → Spam
    Doc 2     │  0  │  1   │   0   │    0    │   0    │    0    │  → Spam
    Doc 3     │  0  │  0   │   0   │    2    │   1    │    3    │  → Ham
    Doc 4     │  1  │  0   │   0   │    1    │   2    │    1    │  → Ham
    Doc 5     │  3  │  2   │   2   │    0    │   0    │    0    │  → Spam
    
    Properties:
    • Non-negative integer values
    • Very SPARSE (most entries are 0)
    • Rows sum to document length
    • Columns represent features (words)
```

### 3.2 Creating a DTM

```
    Step 1: Collect all documents
            D₁ = "buy free click buy free free"
            D₂ = "free"
            D₃ = "meeting project project report project"
            ...

    Step 2: Build vocabulary (unique words)
            V = {buy, free, click, meeting, report, project}

    Step 3: Count word occurrences per document
            D₁ → [2, 3, 1, 0, 0, 0]
            D₂ → [0, 1, 0, 0, 0, 0]
            D₃ → [0, 0, 0, 1, 1, 3]

    Step 4: Stack into matrix → DTM
```

---

## 4. Parameter Estimation

### 4.1 Estimating Word Probabilities

For each class c, we estimate the probability of each word:

```
    ╔══════════════════════════════════════════════════════════════╗
    ║                                                              ║
    ║  Without smoothing:                                         ║
    ║                                                              ║
    ║              count(wⱼ, c)      Total occurrences of wⱼ     ║
    ║  P(wⱼ|c) = ────────────── =  in documents of class c       ║
    ║              Σₖ count(wₖ, c)   Total words in class c       ║
    ║                                                              ║
    ║  With Laplace smoothing (α = 1):                            ║
    ║                                                              ║
    ║              count(wⱼ, c) + 1                               ║
    ║  P(wⱼ|c) = ─────────────────────                            ║
    ║              Σₖ count(wₖ, c) + |V|                          ║
    ║                                                              ║
    ║  Where |V| = vocabulary size                                ║
    ║                                                              ║
    ╚══════════════════════════════════════════════════════════════╝
```

### 4.2 Estimation Example

```
    Training corpus:
    
    Spam docs: "buy free free buy"  +  "free click buy"
    Ham docs:  "meeting report project"  +  "project meeting"
    
    ─────────────────────────────────────────────────────
    
    Vocabulary: {buy, free, click, meeting, report, project}
    |V| = 6
    
    Word counts per class:
    
    Word      │ Spam count │ Ham count │
    ──────────┼────────────┼───────────┤
    buy       │     3      │     0     │
    free      │     3      │     0     │
    click     │     1      │     0     │
    meeting   │     0      │     2     │
    report    │     0      │     1     │
    project   │     0      │     2     │
    ──────────┼────────────┼───────────┤
    TOTAL     │     7      │     5     │
    
    Without smoothing:
    P(buy|Spam) = 3/7 = 0.4286    P(buy|Ham) = 0/5 = 0.0000 ← ZERO!
    P(free|Spam) = 3/7 = 0.4286   P(free|Ham) = 0/5 = 0.0000 ← ZERO!
    
    ⚠️ ZERO probabilities kill the entire product!
    Any word not seen in a class → P(doc|class) = 0
    
    With Laplace smoothing (α=1):
    P(buy|Spam) = (3+1)/(7+6) = 4/13 = 0.3077
    P(buy|Ham) = (0+1)/(5+6) = 1/11 = 0.0909
    P(free|Spam) = (3+1)/(7+6) = 4/13 = 0.3077
    P(free|Ham) = (0+1)/(5+6) = 1/11 = 0.0909
    P(click|Spam) = (1+1)/(7+6) = 2/13 = 0.1538
    P(click|Ham) = (0+1)/(5+6) = 1/11 = 0.0909
    P(meeting|Spam) = (0+1)/(7+6) = 1/13 = 0.0769
    P(meeting|Ham) = (2+1)/(5+6) = 3/11 = 0.2727
    P(report|Spam) = (0+1)/(7+6) = 1/13 = 0.0769
    P(report|Ham) = (1+1)/(5+6) = 2/11 = 0.1818
    P(project|Spam) = (0+1)/(7+6) = 1/13 = 0.0769
    P(project|Ham) = (2+1)/(5+6) = 3/11 = 0.2727
    
    Priors: P(Spam) = 2/4 = 0.5,  P(Ham) = 2/4 = 0.5
```

---

## 5. Classification with Multinomial NB

### 5.1 The Classification Rule

```
    For a new document x with word counts (x₁, x₂, ..., xₘ):
    
    ŷ = argmax  [ log P(c) + Σⱼ xⱼ · log P(wⱼ|c) ]
          c
    
    In log space:
    log_score(c) = log P(c) + x₁ · log P(w₁|c) + x₂ · log P(w₂|c) + ...
    
    Note: xⱼ multiplies the log probability because the word appears xⱼ times
```

### 5.2 The Flow

```
    ┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
    │  New Document │ ──→ │  Count Vectorize  │ ──→ │  Word Count  │
    │  "buy free    │     │  (bag of words)   │     │  Vector      │
    │   buy now"    │     │                    │     │ [2,1,0,0,0] │
    └──────────────┘     └──────────────────┘     └──────┬───────┘
                                                          │
                                                          ▼
                                                  ┌──────────────┐
                                                  │ For each     │
                                                  │ class c:     │
                                                  │              │
                                                  │ score(c) =   │
                                                  │  log P(c) +  │
                                                  │  Σ xⱼ·log    │
                                                  │   P(wⱼ|c)   │
                                                  └──────┬───────┘
                                                          │
                                                          ▼
                                                  ┌──────────────┐
                                                  │  argmax →    │
                                                  │  Prediction  │
                                                  └──────────────┘
```

---

## 6. Worked Example: Text Classification

### Problem Setup

Classify a new document: **"free money free"**

Using the parameters computed in Section 4.2 (with Laplace smoothing):

### Step 1: Convert to Count Vector

```
    "free money free" → word counts:
    
    buy=0, free=2, click=0, meeting=0, report=0, project=0
    
    Note: "money" is NOT in our vocabulary → ignored (OOV word)
    
    x = [0, 2, 0, 0, 0, 0]
```

### Step 2: Compute Log Scores

```
    log_score(Spam):
    = log P(Spam) + 0·log P(buy|Spam) + 2·log P(free|Spam) 
      + 0·log P(click|Spam) + 0·log P(meeting|Spam)
      + 0·log P(report|Spam) + 0·log P(project|Spam)
    
    = log(0.5) + 0 + 2·log(0.3077) + 0 + 0 + 0 + 0
    = -0.6931 + 2·(-1.1787)
    = -0.6931 + (-2.3574)
    = -3.0506

    log_score(Ham):
    = log P(Ham) + 0·log P(buy|Ham) + 2·log P(free|Ham)
      + 0·... + 0·... + 0·... + 0·...
    
    = log(0.5) + 2·log(0.0909)
    = -0.6931 + 2·(-2.3979)
    = -0.6931 + (-4.7958)
    = -5.4889
```

### Step 3: Compare and Classify

```
    log_score(Spam) = -3.0506
    log_score(Ham)  = -5.4889
    
    -3.0506 > -5.4889  →  Spam wins!
    
    ─────────────────────────────────────────────────
    
    Converting to probabilities (optional):
    
    P(Spam|x) = exp(-3.0506) / [exp(-3.0506) + exp(-5.4889)]
              = 0.04734 / (0.04734 + 0.00413)
              = 0.04734 / 0.05147
              = 0.9198 ≈ 92.0%
    
    P(Ham|x) = 0.00413 / 0.05147 = 0.0802 ≈ 8.0%
    
    ╔═══════════════════════════════════╗
    ║  "free money free" → SPAM (92%)  ║
    ╚═══════════════════════════════════╝
```

---

## 7. MultinomialNB in scikit-learn

### 7.1 Basic Usage

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# Sample data
documents = [
    "buy cheap products now free shipping",
    "free free free click here buy",
    "limited offer buy now free",
    "meeting agenda for tomorrow",
    "project report due friday",
    "quarterly meeting with team",
    "please review the project proposal",
    "schedule meeting next week",
    "urgent buy now limited time free",
    "team project update report",
]
labels = [1, 1, 1, 0, 0, 0, 0, 0, 1, 0]  # 1=Spam, 0=Ham

# Vectorize
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(documents)

# Train/test split
X_train, X_test, y_train, y_test = train_test_split(
    X, labels, test_size=0.3, random_state=42
)

# Train Multinomial NB
mnb = MultinomialNB(alpha=1.0)  # alpha=1 → Laplace smoothing
mnb.fit(X_train, y_train)

# Predict
y_pred = mnb.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"Predictions: {y_pred}")
```

### 7.2 Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `alpha` | 1.0 | Additive smoothing (1.0 = Laplace, 0 = no smoothing) |
| `fit_prior` | True | Whether to learn priors from data |
| `class_prior` | None | Manually specify prior probabilities |

### 7.3 Inspecting the Model

```python
# Feature log probabilities: log P(wⱼ|c) for each class
print("Feature names:", vectorizer.get_feature_names_out()[:10])

# Log probabilities per class
import numpy as np
feature_names = vectorizer.get_feature_names_out()

for i, class_label in enumerate(['Ham', 'Spam']):
    print(f"\nTop 5 words for {class_label}:")
    log_probs = mnb.feature_log_prob_[i]
    top_indices = np.argsort(log_probs)[-5:][::-1]
    for idx in top_indices:
        print(f"  {feature_names[idx]:>15}: log P = {log_probs[idx]:.4f}"
              f"  (P = {np.exp(log_probs[idx]):.4f})")

# Class priors
print(f"\nClass priors: {np.exp(mnb.class_log_prior_)}")
```

---

## 8. TF-IDF Integration

### 8.1 Why TF-IDF?

Raw word counts can be misleading — common words like "the", "is", "a" dominate:

```
    ┌──────────────────────────────────────────────────────────────┐
    │  TF-IDF = Term Frequency × Inverse Document Frequency      │
    │                                                              │
    │  TF(t,d) = count(t in d) / total_words(d)                  │
    │           "How frequent is this word in THIS document?"     │
    │                                                              │
    │  IDF(t) = log(N / df(t))                                    │
    │           "How rare is this word across ALL documents?"     │
    │                                                              │
    │  TF-IDF(t,d) = TF(t,d) × IDF(t)                           │
    │                                                              │
    │  High TF-IDF: word is frequent in document but rare overall │
    │  Low TF-IDF:  word is common everywhere (uninformative)     │
    └──────────────────────────────────────────────────────────────┘
```

### 8.2 TF-IDF + MultinomialNB

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline

# TF-IDF vectorization + Multinomial NB pipeline
tfidf_nb_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(
        max_features=10000,
        ngram_range=(1, 2),    # unigrams and bigrams
        stop_words='english',
        min_df=2,              # ignore very rare words
        max_df=0.95,           # ignore very common words
    )),
    ('classifier', MultinomialNB(alpha=0.1)),
])

# Fit and predict
tfidf_nb_pipeline.fit(train_texts, train_labels)
predictions = tfidf_nb_pipeline.predict(test_texts)
```

### 8.3 Count vs TF-IDF Comparison

```
    ┌────────────────────────┬────────────────────────────────────┐
    │  CountVectorizer       │  TfidfVectorizer                  │
    ├────────────────────────┼────────────────────────────────────┤
    │ Raw word counts        │ Weighted word importance           │
    │ Integer values         │ Float values [0, 1]               │
    │ Common words dominate  │ Common words down-weighted        │
    │ Simple, fast           │ Slightly slower                    │
    │ Good baseline          │ Often better accuracy             │
    │ True multinomial input │ Technically not multinomial        │
    │                        │ (but MNB still works well on it)  │
    └────────────────────────┴────────────────────────────────────┘
    
    Note: MultinomialNB requires NON-NEGATIVE inputs.
    Both Count and TF-IDF satisfy this requirement.
```

---

## 9. Complete Text Classification Pipeline

### 9.1 Full Pipeline Architecture

```
    ┌─────────┐    ┌──────────┐    ┌───────────┐    ┌──────────┐    ┌──────────┐
    │ Raw Text │───→│ Preprocess│───→│ Vectorize │───→│   Train  │───→│ Evaluate │
    │          │    │          │    │           │    │   MNB    │    │          │
    └─────────┘    └──────────┘    └───────────┘    └──────────┘    └──────────┘
         │              │               │                │               │
         │         • Lowercase      • CountVec       • alpha=1       • Accuracy
         │         • Remove punct   • TfidfVec       • fit_prior     • Precision
         │         • Tokenize       • max_features   • class_prior   • Recall
         │         • Remove stops   • ngram_range                    • F1-score
         │         • Stemming       • min_df/max_df                  • Confusion
         │                                                             matrix
```

### 9.2 Production-Ready Pipeline

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.metrics import classification_report, confusion_matrix
import numpy as np

def build_text_classifier(train_texts, train_labels, test_texts, test_labels):
    """Build, tune, and evaluate a Multinomial NB text classifier."""
    
    # Define pipeline
    pipeline = Pipeline([
        ('tfidf', TfidfVectorizer()),
        ('clf', MultinomialNB()),
    ])
    
    # Hyperparameter grid
    param_grid = {
        'tfidf__max_features': [5000, 10000, 20000],
        'tfidf__ngram_range': [(1, 1), (1, 2)],
        'tfidf__stop_words': [None, 'english'],
        'clf__alpha': [0.01, 0.1, 0.5, 1.0],
    }
    
    # Grid search with cross-validation
    grid_search = GridSearchCV(
        pipeline, param_grid, cv=5, scoring='f1_weighted', n_jobs=-1
    )
    grid_search.fit(train_texts, train_labels)
    
    # Best model
    print(f"Best parameters: {grid_search.best_params_}")
    print(f"Best CV score: {grid_search.best_score_:.4f}")
    
    # Evaluate on test set
    y_pred = grid_search.predict(test_texts)
    print(f"\nTest set results:")
    print(classification_report(test_labels, y_pred))
    
    return grid_search.best_estimator_
```

---

## 10. Python Implementation

### 10.1 Multinomial NB from Scratch

```python
import numpy as np
from collections import defaultdict

class MultinomialNBFromScratch:
    """Multinomial Naive Bayes classifier from scratch."""
    
    def __init__(self, alpha=1.0):
        self.alpha = alpha  # smoothing parameter
    
    def fit(self, X, y):
        """
        X: document-term matrix (n_docs × n_features), integer counts
        y: class labels
        """
        n_samples, n_features = X.shape
        self.classes = np.unique(y)
        n_classes = len(self.classes)
        
        # Priors: P(c) = Nc / N
        self.class_log_prior = np.zeros(n_classes)
        # Feature log probabilities: log P(wⱼ|c)
        self.feature_log_prob = np.zeros((n_classes, n_features))
        
        for idx, c in enumerate(self.classes):
            # Select documents belonging to class c
            X_c = X[y == c]
            
            # Prior
            self.class_log_prior[idx] = np.log(len(X_c) / n_samples)
            
            # Count total occurrences of each word in class c
            word_counts = X_c.sum(axis=0)
            if hasattr(word_counts, 'A1'):  # handle sparse matrices
                word_counts = word_counts.A1
            
            # Total words in class c
            total_words = word_counts.sum()
            
            # Smoothed word probabilities
            smoothed_counts = word_counts + self.alpha
            smoothed_total = total_words + self.alpha * n_features
            
            self.feature_log_prob[idx] = np.log(smoothed_counts / smoothed_total)
        
        return self
    
    def predict_log_proba(self, X):
        """Compute log P(c|x) for each class."""
        # log P(c|x) ∝ log P(c) + Σ xⱼ · log P(wⱼ|c)
        # X @ feature_log_prob.T computes the dot product for all docs & classes
        if hasattr(X, 'toarray'):
            X_dense = X.toarray()
        else:
            X_dense = np.array(X)
        
        log_proba = X_dense @ self.feature_log_prob.T + self.class_log_prior
        return log_proba
    
    def predict(self, X):
        log_proba = self.predict_log_proba(X)
        return self.classes[np.argmax(log_proba, axis=1)]
    
    def predict_proba(self, X):
        log_proba = self.predict_log_proba(X)
        # Log-sum-exp for numerical stability
        max_log = np.max(log_proba, axis=1, keepdims=True)
        log_proba_shifted = log_proba - max_log
        proba = np.exp(log_proba_shifted)
        proba /= proba.sum(axis=1, keepdims=True)
        return proba


# ─── Test: Compare with sklearn ─────────────────────────────
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

documents = [
    "buy cheap products now free shipping",
    "free free free click here buy now",
    "limited offer buy now free deal",
    "meeting agenda for tomorrow morning",
    "project report due friday deadline",
    "quarterly meeting with team review",
    "please review the project proposal draft",
    "schedule meeting next week planning",
    "urgent buy now limited time free offer",
    "team project update report status",
]
labels = np.array([1, 1, 1, 0, 0, 0, 0, 0, 1, 0])

# Vectorize
vec = CountVectorizer()
X = vec.fit_transform(documents)

# Our implementation
mnb_scratch = MultinomialNBFromScratch(alpha=1.0)
mnb_scratch.fit(X, labels)
pred_scratch = mnb_scratch.predict(X)

# sklearn
mnb_sklearn = MultinomialNB(alpha=1.0)
mnb_sklearn.fit(X, labels)
pred_sklearn = mnb_sklearn.predict(X)

print("Predictions match:", np.all(pred_scratch == pred_sklearn))
print(f"Scratch predictions: {pred_scratch}")
print(f"Sklearn predictions: {pred_sklearn}")
```

### 10.2 Working with Real Data — 20 Newsgroups

```python
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report
import numpy as np

# Load subset of newsgroups
categories = ['alt.atheism', 'soc.religion.christian', 'comp.graphics', 'sci.med']
train = fetch_20newsgroups(subset='train', categories=categories, shuffle=True)
test = fetch_20newsgroups(subset='test', categories=categories, shuffle=True)

# Build pipeline
pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=10000, stop_words='english')),
    ('clf', MultinomialNB(alpha=0.1)),
])

# Train and evaluate
pipeline.fit(train.data, train.target)
y_pred = pipeline.predict(test.data)

print(f"Accuracy: {np.mean(y_pred == test.target):.4f}")
print(f"\nClassification Report:")
print(classification_report(test.target, y_pred, target_names=test.target_names))

# Show most informative features per class
feature_names = pipeline.named_steps['tfidf'].get_feature_names_out()
log_probs = pipeline.named_steps['clf'].feature_log_prob_

for i, category in enumerate(train.target_names):
    top10 = np.argsort(log_probs[i])[-10:][::-1]
    print(f"\n{category}: {', '.join(feature_names[top10])}")
```

### 10.3 Spam Filter Example

```python
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import Pipeline
import numpy as np

# Simulated spam dataset
emails = [
    "Win a free iPhone! Click now to claim your prize",
    "Free money! Act now, limited time offer",
    "Buy cheap medications online free shipping",
    "Congratulations you won the lottery click here",
    "Hi, the meeting is scheduled for 3pm tomorrow",
    "Please review the attached project proposal",
    "Can we discuss the quarterly report this week?",
    "Team lunch planned for Friday at noon",
    "Your account has been compromised click to verify",
    "Dear friend please help transfer funds urgently",
    "Don't forget to submit your timesheet by Friday",
    "The new project requirements are ready for review",
    "URGENT: You've been selected for a cash prize!",
    "Weekly team standup notes attached",
    "Free trial expired! Renew now for special price",
    "Client meeting moved to next Tuesday at 2pm",
]
labels = [1,1,1,1, 0,0,0,0, 1,1,0,0,1,0,1,0]

# Compare Count vs TF-IDF
for VecClass, name in [(CountVectorizer, 'Count'), (TfidfVectorizer, 'TF-IDF')]:
    pipeline = Pipeline([
        ('vec', VecClass(stop_words='english')),
        ('clf', MultinomialNB(alpha=1.0)),
    ])
    scores = cross_val_score(pipeline, emails, labels, cv=3, scoring='accuracy')
    print(f"{name:>8} Vectorizer: accuracy = {scores.mean():.4f} ± {scores.std():.4f}")

# Classify a new email
pipeline = Pipeline([
    ('vec', CountVectorizer(stop_words='english')),
    ('clf', MultinomialNB(alpha=1.0)),
])
pipeline.fit(emails, labels)

new_emails = [
    "Free money click here to claim your prize now!",
    "Please review the project report before our meeting",
]
predictions = pipeline.predict(new_emails)
probas = pipeline.predict_proba(new_emails)

for email, pred, proba in zip(new_emails, predictions, probas):
    label = "SPAM" if pred == 1 else "HAM"
    conf = max(proba) * 100
    print(f"\n  '{email[:50]}...'")
    print(f"  → {label} ({conf:.1f}% confidence)")
```

---

## 11. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **Multinomial NB** | P(x\|c) ∝ Π P(wⱼ\|c)^xⱼ | For discrete count features |
| **Word probability** | P(wⱼ\|c) = count(wⱼ,c) / Σcount(wₖ,c) | Fraction of total words |
| **Laplace smoothing** | P(wⱼ\|c) = (count+α) / (total+α\|V\|) | Prevents zero probabilities |
| **Log classification** | log P(c) + Σ xⱼ·log P(wⱼ\|c) | Numerically stable |
| **DTM** | Docs × Words count matrix | Standard text representation |
| **Bag of words** | Ignore word order, keep counts | Simplification that works |
| **TF-IDF** | TF × IDF weighting | Down-weights common words |
| **CountVectorizer** | Raw word counts | True multinomial input |
| **TfidfVectorizer** | Weighted counts | Often better accuracy |
| **alpha parameter** | Smoothing (1.0=Laplace) | Critical for text data |
| **Best use case** | Text classification | Fast, effective baseline |

---

## 12. Revision Questions

### Q1: Parameter Estimation
**Q:** Given two classes with the following word counts and vocabulary {a, b, c, d}, compute all smoothed word probabilities using α=1.

| Word | Class 0 count | Class 1 count |
|------|--------------|--------------|
| a | 5 | 1 |
| b | 2 | 8 |
| c | 3 | 1 |
| d | 0 | 0 |

**A:**
- Class 0 total: 5+2+3+0 = 10. With smoothing: (count+1)/(10+4) = (count+1)/14
  - P(a|0)=(5+1)/14=6/14=**0.4286**, P(b|0)=3/14=**0.2143**, P(c|0)=4/14=**0.2857**, P(d|0)=1/14=**0.0714**
- Class 1 total: 1+8+1+0 = 10. With smoothing: (count+1)/(10+4) = (count+1)/14
  - P(a|1)=2/14=**0.1429**, P(b|1)=9/14=**0.6429**, P(c|1)=2/14=**0.1429**, P(d|1)=1/14=**0.0714**
- Verification: each class sums to 14/14 = 1.0 ✓

---

### Q2: Classification Calculation
**Q:** Using the probabilities from Q1 and equal priors P(C₀)=P(C₁)=0.5, classify a document with word counts: a=2, b=0, c=1, d=0. Show all steps in log space.

**A:**
- log_score(C₀) = log(0.5) + 2·log(0.4286) + 0·log(0.2143) + 1·log(0.2857) + 0·log(0.0714)
  = -0.693 + 2·(-0.847) + (-1.252) = -0.693 - 1.694 - 1.252 = **-3.639**
- log_score(C₁) = log(0.5) + 2·log(0.1429) + 0·log(0.6429) + 1·log(0.1429) + 0·log(0.0714)
  = -0.693 + 2·(-1.946) + (-1.946) = -0.693 - 3.892 - 1.946 = **-6.531**
- -3.639 > -6.531 → **Class 0** wins
- P(C₀|x) = exp(-3.639)/[exp(-3.639)+exp(-6.531)] = 0.0264/(0.0264+0.00146) = **0.948 (94.8%)**

---

### Q3: Count vs TF-IDF
**Q:** Explain why TF-IDF often improves Multinomial NB performance over raw counts. Give a specific example where raw counts would mislead.

**A:** TF-IDF down-weights words that appear in many documents (high document frequency) and up-weights words that are distinctive to specific documents. Example: the word "the" appears in nearly every document, so its raw count might be high (e.g., 10 in a spam email, 12 in a ham email). Raw counts give "the" significant influence on classification despite being uninformative. TF-IDF assigns "the" a near-zero IDF, effectively neutralizing it. Meanwhile, discriminative words like "lottery" or "meeting" get high TF-IDF scores because they appear in few documents, concentrating the classifier's attention on informative features.

---

### Q4: Vocabulary Size Impact
**Q:** How does increasing vocabulary size |V| affect Multinomial NB's (a) training time, (b) accuracy, and (c) the effect of smoothing?

**A:**
- **(a) Training time:** Increases linearly — more word probabilities to estimate, but still fast (single pass through data)
- **(b) Accuracy:** Initially improves (more informative features), then may degrade (noise from rare/uninformative words). Use max_features or min_df to cap vocabulary
- **(c) Smoothing effect:** With larger |V|, the denominator (total + α·|V|) grows, spreading more probability mass to unseen words. Smoothing "dilutes" observed counts more. Very large vocabularies may need smaller α to avoid over-smoothing

---

### Q5: Zero Probability Problem
**Q:** Without Laplace smoothing, why does a single unseen word in one class cause Multinomial NB to completely ignore all other evidence? Show mathematically.

**A:** If word wⱼ never appears in class c, then P(wⱼ|c) = 0. The class score is: P(c) × Π P(wᵢ|c) = P(c) × ... × **0** × ... = **0**, regardless of how strong all other word probabilities are. In log space: log(0) = -∞, so log_score = -∞. A single zero probability **zeroes out the entire product**, making all other evidence irrelevant. This is catastrophic because the absence of a word in training data for one class doesn't mean it can't occur — it just means we haven't seen it yet. Smoothing adds a small count to prevent any probability from being exactly zero.

---

### Q6: MultinomialNB vs GaussianNB for Text
**Q:** Why is MultinomialNB preferred over GaussianNB for text classification? What would go wrong if you used GaussianNB on a document-term matrix?

**A:** Document-term matrices have word counts that are: (1) non-negative integers, (2) extremely sparse (mostly zeros), (3) right-skewed (few common words, many rare ones). GaussianNB assumes each feature follows a bell-shaped normal distribution — terrible for this data. Problems: negative probabilities for count=0 (the mode), the Gaussian PDF assigns significant density to negative values (impossible for counts), and the bell curve doesn't capture the spike at zero. MultinomialNB's multinomial distribution naturally models: non-negative counts, the "rolling a word die" generation process, and sparse data. Empirically, MultinomialNB significantly outperforms GaussianNB on text tasks.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 3: Gaussian Naive Bayes](03-gaussian-naive-bayes.md) | **Chapter 4: Multinomial Naive Bayes** | [Chapter 5: Bernoulli Naive Bayes →](05-bernoulli-naive-bayes.md) |

### Unit 10 Chapters
1. [Bayes' Theorem Review](01-bayes-theorem-review.md)
2. [The Naive Assumption](02-naive-assumption.md)
3. [Gaussian Naive Bayes](03-gaussian-naive-bayes.md)
4. **📍 Multinomial Naive Bayes** ← You are here
5. [Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md)
6. [Text Classification with Naive Bayes](06-text-classification.md)
7. [Laplace Smoothing & Practical Considerations](07-laplace-smoothing.md)

---

> *"Multinomial Naive Bayes turns word counts into predictions — the workhorse of text classification since the 1990s."*

© 2025 AI/ML Study Notes. All rights reserved.
