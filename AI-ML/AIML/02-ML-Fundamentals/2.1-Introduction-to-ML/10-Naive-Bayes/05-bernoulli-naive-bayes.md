# 📘 Chapter 5: Bernoulli Naive Bayes

> **Unit 10: Naive Bayes** | **Section 5 of 7**
> Naive Bayes for binary features using the Bernoulli distribution. Understand how BernoulliNB differs from MultinomialNB by explicitly penalizing absent features, master the math of binary term presence/absence, and learn when each model is the better choice.

---

## 📑 Table of Contents

1. [Introduction: Binary Features](#1-introduction-binary-features)
2. [The Bernoulli Distribution](#2-the-bernoulli-distribution)
3. [Bernoulli Naive Bayes Formula](#3-bernoulli-naive-bayes-formula)
4. [Key Difference: Penalizing Absent Features](#4-key-difference-penalizing-absent-features)
5. [Parameter Estimation](#5-parameter-estimation)
6. [Worked Example: Document Classification](#6-worked-example-document-classification)
7. [BernoulliNB in scikit-learn](#7-bernoullinb-in-scikit-learn)
8. [Comparison: BernoulliNB vs MultinomialNB](#8-comparison-bernoullinb-vs-multinomialnb)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Introduction: Binary Features

### 1.1 When Features are Binary (0/1)

Some classification problems naturally have **binary features** — each feature is either present (1) or absent (0):

```
    ┌─────────────────────────────────────────────────────────────┐
    │  BINARY FEATURE EXAMPLES:                                  │
    │                                                             │
    │  • Word presence/absence in a document (not counts!)       │
    │  • Has feature X: yes/no (medical symptoms)                │
    │  • User clicked ad: yes/no                                 │
    │  • Product attribute present: yes/no                       │
    │  • Pixel above threshold: yes/no (binarized image)        │
    │  • Categorical features after one-hot encoding             │
    │                                                             │
    │  Key distinction from Multinomial NB:                       │
    │    Multinomial: "How MANY times does word w appear?"       │
    │    Bernoulli:   "DOES word w appear? (yes/no)"            │
    └─────────────────────────────────────────────────────────────┘
```

### 1.2 Binary Document Representation

```
    Document: "the cat sat on the mat"
    
    Multinomial (count) vector:
    the=2, cat=1, sat=1, on=1, mat=1
    → [2, 1, 1, 1, 1]
    
    Bernoulli (binary) vector:
    the=1, cat=1, sat=1, on=1, mat=1    (just presence/absence)
    → [1, 1, 1, 1, 1]
    
    Word "the" appeared twice, but Bernoulli only records "present"
```

---

## 2. The Bernoulli Distribution

### 2.1 Definition

The Bernoulli distribution models a single binary trial:

```
    ╔═══════════════════════════════════════════════════════════╗
    ║                                                           ║
    ║  X ~ Bernoulli(p)                                        ║
    ║                                                           ║
    ║  P(X = 1) = p          (feature present)                 ║
    ║  P(X = 0) = 1 - p      (feature absent)                 ║
    ║                                                           ║
    ║  Compact form:                                            ║
    ║  P(X = x) = p^x · (1-p)^(1-x)     where x ∈ {0, 1}    ║
    ║                                                           ║
    ║  When x=1:  p¹ · (1-p)⁰ = p        ✓                   ║
    ║  When x=0:  p⁰ · (1-p)¹ = 1-p      ✓                   ║
    ║                                                           ║
    ╚═══════════════════════════════════════════════════════════╝
```

### 2.2 Per-Class Bernoulli Parameters

For each feature i and each class c, we have a parameter pᵢc:

```
    Feature i, Class c:
    
    P(xᵢ = 1 | y = c) = pᵢc       ← probability feature i is present in class c
    P(xᵢ = 0 | y = c) = 1 - pᵢc   ← probability feature i is absent in class c
    
    Example (spam detection):
    
    Feature         P(present|Spam)   P(present|Ham)
    ─────────       ───────────────   ──────────────
    "free"              0.80              0.10
    "meeting"           0.05              0.60
    "click"             0.70              0.08
    "report"            0.02              0.45
```

---

## 3. Bernoulli Naive Bayes Formula

### 3.1 The Likelihood

For a document represented as a binary vector x = (x₁, x₂, ..., xₘ):

```
    ╔══════════════════════════════════════════════════════════════════╗
    ║                                                                  ║
    ║  P(x | y = c) = Π  [ pᵢc^xᵢ · (1-pᵢc)^(1-xᵢ) ]              ║
    ║                i=1                                               ║
    ║                                                                  ║
    ║  Expanded:                                                       ║
    ║                                                                  ║
    ║  P(x|c) =   Π  pᵢc    ×    Π  (1-pᵢc)                         ║
    ║           i: xᵢ=1        i: xᵢ=0                               ║
    ║                                                                  ║
    ║  = (prob of present features) × (prob of absent features)       ║
    ║                                                                  ║
    ╚══════════════════════════════════════════════════════════════════╝
```

### 3.2 The Full Classification Rule

```
    ŷ = argmax  P(c) · Π  [ pᵢc^xᵢ · (1-pᵢc)^(1-xᵢ) ]
          c           i=1

    In log space:

    log_score(c) = log P(c) + Σ [ xᵢ · log(pᵢc) + (1-xᵢ) · log(1-pᵢc) ]
                                i

    Equivalently:

    log_score(c) = log P(c) + Σ log(1-pᵢc) + Σ  xᵢ · log(pᵢc / (1-pᵢc))
                               i              i
                   \_________/  \____________/   \__________________________/
                    prior      baseline (all    adjustment for present features
                               features absent)  (log-odds contribution)
```

---

## 4. Key Difference: Penalizing Absent Features

### 4.1 The Critical Distinction

This is the **most important difference** between Bernoulli and Multinomial NB:

```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │  MULTINOMIAL NB:                                              │
    │    Only considers words that ARE PRESENT in the document      │
    │    Absent words contribute NOTHING to the score               │
    │    P(x|c) = Π P(wⱼ|c)^xⱼ   (only non-zero xⱼ matter)     │
    │                                                                │
    │  BERNOULLI NB:                                                │
    │    Considers ALL words — both present AND absent              │
    │    Absent words ACTIVELY contribute (1-pᵢc) to the score    │
    │    A word's ABSENCE is evidence too!                          │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### 4.2 Why Absence Matters

```
    Example: Classifying an email as Spam or Ham
    
    Vocabulary: {"free", "click", "meeting", "report"}
    
    Email: "quarterly report"  →  x = [0, 0, 0, 1]
    
    ─────────────────────────────────────────────────────────────
    
    MULTINOMIAL NB only sees:
      "report" is present → uses P("report"|c)
      "free", "click", "meeting" absent → IGNORES them
    
    BERNOULLI NB sees:
      "report" present    → uses P("report" present | c)
      "free" absent       → uses P("free" absent | c) = 1 - P("free"|Spam)
      "click" absent      → uses P("click" absent | c) = 1 - P("click"|Spam)
      "meeting" absent    → uses P("meeting" absent | c)
    
    ─────────────────────────────────────────────────────────────
    
    The ABSENCE of "free" and "click" is EVIDENCE against spam!
    
    If P("free"|Spam) = 0.80, then:
      P("free" absent | Spam) = 0.20    ← unlikely for spam!
      P("free" absent | Ham) = 0.90     ← normal for ham
    
    Bernoulli uses this signal; Multinomial does not.
```

### 4.3 Visual Comparison

```
    Document: "report project" (only 2 words present out of 6 in vocab)
    
    MULTINOMIAL NB scoring:
    ┌─────────────────────────────────────────────┐
    │ ■ report: uses P("report"|c)               │
    │ ■ project: uses P("project"|c)             │
    │ □ free: ─── (ignored)                      │
    │ □ click: ─── (ignored)                     │
    │ □ buy: ─── (ignored)                       │
    │ □ meeting: ─── (ignored)                   │
    └─────────────────────────────────────────────┘
    Score based on 2 features only
    
    BERNOULLI NB scoring:
    ┌─────────────────────────────────────────────┐
    │ ■ report = 1:  uses pᵢc                    │
    │ ■ project = 1: uses pᵢc                    │
    │ □ free = 0:    uses (1-pᵢc)   ← ACTIVE    │
    │ □ click = 0:   uses (1-pᵢc)   ← ACTIVE    │
    │ □ buy = 0:     uses (1-pᵢc)   ← ACTIVE    │
    │ □ meeting = 0: uses (1-pᵢc)   ← ACTIVE    │
    └─────────────────────────────────────────────┘
    Score based on ALL 6 features
```

---

## 5. Parameter Estimation

### 5.1 MLE with Smoothing

```
    ╔══════════════════════════════════════════════════════════════╗
    ║                                                              ║
    ║  Without smoothing:                                         ║
    ║                                                              ║
    ║              # docs in class c that contain word wᵢ         ║
    ║  p̂ᵢc  = ─────────────────────────────────────────           ║
    ║              # docs in class c                               ║
    ║                                                              ║
    ║  With Laplace smoothing (α = 1):                            ║
    ║                                                              ║
    ║              # docs in class c containing wᵢ  +  α          ║
    ║  p̂ᵢc  = ──────────────────────────────────────────          ║
    ║              # docs in class c  +  2α                        ║
    ║                                                              ║
    ║  Note: denominator adds 2α (not α·|V|) because             ║
    ║  each feature has 2 outcomes: present or absent              ║
    ║                                                              ║
    ╚══════════════════════════════════════════════════════════════╝
```

### 5.2 Key Difference in Counting

```
    MULTINOMIAL counts:                BERNOULLI counts:
    
    Total word occurrences             Number of documents
    in class c.                        in class c containing
                                       the word.
    
    Doc1 (Spam): "free free buy"       Doc1 (Spam): "free free buy"
    Doc2 (Spam): "free click"          Doc2 (Spam): "free click"
    
    count("free", Spam) = 3            docs_with("free", Spam) = 2
    (total occurrences)                (documents containing "free")
    
    Total words in Spam = 5            Total Spam docs = 2
    
    P("free"|Spam) = 3/5 = 0.60       P("free"|Spam) = 2/2 = 1.00
    (Multinomial estimate)             (Bernoulli estimate, before smoothing)
```

---

## 6. Worked Example: Document Classification

### Problem Setup

Classify a new document as **Sports** or **Politics**.

**Training Data:**

| Doc | Text | Class |
|-----|------|-------|
| D1 | "team wins game score" | Sports |
| D2 | "game score record" | Sports |
| D3 | "team wins championship" | Sports |
| D4 | "election vote policy" | Politics |
| D5 | "vote policy reform" | Politics |
| D6 | "election reform law" | Politics |

**Vocabulary:** {team, wins, game, score, record, championship, election, vote, policy, reform, law}
|V| = 11

**New Document:** "team wins vote" → x = [1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0]

### Step 1: Compute Priors

```
    P(Sports) = 3/6 = 0.50
    P(Politics) = 3/6 = 0.50
```

### Step 2: Compute Bernoulli Parameters (with α=1)

For each word, count how many documents in each class contain it:

```
    Word          │ Sports docs │ Politics docs │ p̂(w|Sports) │ p̂(w|Politics)
                  │ containing  │ containing    │ (n+1)/(N+2) │ (n+1)/(N+2)
    ──────────────┼─────────────┼───────────────┼──────────────┼──────────────
    team          │     2       │      0        │ (2+1)/5=0.60│ (0+1)/5=0.20
    wins          │     2       │      0        │ (2+1)/5=0.60│ (0+1)/5=0.20
    game          │     2       │      0        │ (2+1)/5=0.60│ (0+1)/5=0.20
    score         │     2       │      0        │ (2+1)/5=0.60│ (0+1)/5=0.20
    record        │     1       │      0        │ (1+1)/5=0.40│ (0+1)/5=0.20
    championship  │     1       │      0        │ (1+1)/5=0.40│ (0+1)/5=0.20
    election      │     0       │      2        │ (0+1)/5=0.20│ (2+1)/5=0.60
    vote          │     0       │      2        │ (0+1)/5=0.20│ (2+1)/5=0.60
    policy        │     0       │      2        │ (0+1)/5=0.20│ (2+1)/5=0.60
    reform        │     0       │      2        │ (0+1)/5=0.20│ (2+1)/5=0.60
    law           │     0       │      1        │ (0+1)/5=0.20│ (1+1)/5=0.40
    
    N_Sports = 3 docs, N_Politics = 3 docs
    Denominator = N_c + 2α = 3 + 2 = 5
```

### Step 3: Compute Log Scores

**Document: x = [1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0]**

```
    log_score(Sports):
    = log(0.50)
      + 1·log(0.60) + 1·log(0.60)         ← team=1, wins=1 (present)
      + 0... → log(1-0.60) = log(0.40)    ← game=0 (absent)
      + log(0.40) + log(0.60) + log(0.60)  ← score=0, record=0, championship=0
      + log(0.80) + 1·log(0.20)            ← election=0, vote=1 (present!)
      + log(0.80) + log(0.80) + log(0.80)  ← policy=0, reform=0, law=0

    Let me compute term by term:
    
    Word          xᵢ   p     Term = xᵢ·log(p) + (1-xᵢ)·log(1-p)
    ──────────    ──   ────  ─────────────────────────────────────
    team          1    0.60   log(0.60) = -0.5108
    wins          1    0.60   log(0.60) = -0.5108
    game          0    0.60   log(0.40) = -0.9163
    score         0    0.60   log(0.40) = -0.9163
    record        0    0.40   log(0.60) = -0.5108
    championship  0    0.40   log(0.60) = -0.5108
    election      0    0.20   log(0.80) = -0.2231
    vote          1    0.20   log(0.20) = -1.6094
    policy        0    0.20   log(0.80) = -0.2231
    reform        0    0.20   log(0.80) = -0.2231
    law           0    0.20   log(0.80) = -0.2231
                                         ────────
    Sum of feature terms:                -6.3777
    + log P(Sports) = log(0.50):         -0.6931
                                         ────────
    log_score(Sports):                   -7.0708
```

```
    log_score(Politics):
    
    Word          xᵢ   p     Term
    ──────────    ──   ────  ─────
    team          1    0.20   log(0.20) = -1.6094
    wins          1    0.20   log(0.20) = -1.6094
    game          0    0.20   log(0.80) = -0.2231
    score         0    0.20   log(0.80) = -0.2231
    record        0    0.20   log(0.80) = -0.2231
    championship  0    0.20   log(0.80) = -0.2231
    election      0    0.60   log(0.40) = -0.9163
    vote          1    0.60   log(0.60) = -0.5108
    policy        0    0.60   log(0.40) = -0.9163
    reform        0    0.60   log(0.40) = -0.9163
    law           0    0.40   log(0.60) = -0.5108
                                         ────────
    Sum of feature terms:                -7.8817
    + log P(Politics):                   -0.6931
                                         ────────
    log_score(Politics):                 -8.5748
```

### Step 4: Classify

```
    log_score(Sports)  = -7.0708
    log_score(Politics) = -8.5748
    
    -7.0708 > -8.5748  →  Sports wins!
    
    Converting to probabilities:
    P(Sports|x)  = exp(-7.0708) / [exp(-7.0708) + exp(-8.5748)]
                 = 0.000849 / (0.000849 + 0.000189)
                 = 0.000849 / 0.001038
                 ≈ 0.818 (81.8%)
    
    ╔════════════════════════════════════════════════╗
    ║  "team wins vote" → SPORTS (81.8%)           ║
    ║                                                ║
    ║  Despite "vote" being a strong politics word, ║
    ║  "team" and "wins" plus the ABSENCE of        ║
    ║  "election", "policy", "reform" pushed the    ║
    ║  classification toward Sports.                ║
    ╚════════════════════════════════════════════════╝
```

---

## 7. BernoulliNB in scikit-learn

### 7.1 Basic Usage

```python
from sklearn.naive_bayes import BernoulliNB
from sklearn.feature_extraction.text import CountVectorizer
import numpy as np

# Sample documents
documents = [
    "team wins game score",
    "game score record",
    "team wins championship",
    "election vote policy",
    "vote policy reform",
    "election reform law",
]
labels = [0, 0, 0, 1, 1, 1]  # 0=Sports, 1=Politics

# Binarize: CountVectorizer with binary=True
vectorizer = CountVectorizer(binary=True)
X = vectorizer.fit_transform(documents)

# Train BernoulliNB
bnb = BernoulliNB(alpha=1.0)
bnb.fit(X, labels)

# Classify new document
new_doc = ["team wins vote"]
X_new = vectorizer.transform(new_doc)
prediction = bnb.predict(X_new)
proba = bnb.predict_proba(X_new)

class_names = ['Sports', 'Politics']
print(f"Document: '{new_doc[0]}'")
print(f"Prediction: {class_names[prediction[0]]}")
print(f"P(Sports|x) = {proba[0, 0]:.4f}")
print(f"P(Politics|x) = {proba[0, 1]:.4f}")
```

### 7.2 Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `alpha` | 1.0 | Additive smoothing (Laplace) |
| `binarize` | 0.0 | Threshold for binarizing input features |
| `fit_prior` | True | Whether to learn class priors |
| `class_prior` | None | Custom prior probabilities |

### 7.3 The `binarize` Parameter

```python
# BernoulliNB can auto-binarize continuous or count features!

# If input has counts, binarize threshold converts to binary:
bnb = BernoulliNB(binarize=0.0)  # any value > 0 becomes 1
# Equivalent to: X = (X > 0).astype(int)

bnb = BernoulliNB(binarize=0.5)  # values > 0.5 become 1
# Useful for TF-IDF or other continuous inputs

bnb = BernoulliNB(binarize=None)  # no binarization (expects binary input)
```

---

## 8. Comparison: BernoulliNB vs MultinomialNB

### 8.1 Feature Comparison Table

| Aspect | BernoulliNB | MultinomialNB |
|--------|------------|---------------|
| **Input type** | Binary (0/1) | Non-negative counts |
| **Feature meaning** | Word present/absent | Word frequency |
| **Absent features** | **Actively penalized** | Ignored |
| **Document "free free free"** | free=1 (same as "free") | free=3 (triple weight) |
| **Word repetition** | No effect | Increases importance |
| **Better for** | Short documents, binary features | Long documents, frequency matters |
| **Vocabulary effect** | Large vocab helps (more absence signals) | Large vocab can hurt (noise) |
| **Typical use** | Short text, binary attributes | News articles, reviews |
| **sklearn class** | `BernoulliNB` | `MultinomialNB` |

### 8.2 When to Choose Which

```
    ┌────────────────────────────────────────────────────────────┐
    │  CHOOSE BernoulliNB when:                                 │
    │  • Features are naturally binary (yes/no, present/absent) │
    │  • Short documents (tweets, SMS, subject lines)           │
    │  • Word presence matters more than frequency              │
    │  • Feature absence is informative                         │
    │  • Binary attributes (medical symptoms, product features) │
    │                                                            │
    │  CHOOSE MultinomialNB when:                               │
    │  • Word frequency matters (longer documents)              │
    │  • Document length varies significantly                    │
    │  • Features are naturally count-based                      │
    │  • The number of times a word appears is informative      │
    │  • Standard text classification tasks                      │
    └────────────────────────────────────────────────────────────┘
```

### 8.3 Empirical Comparison

```
    Document length     →   Short (< 50 words)    Long (> 200 words)
    ─────────────────       ──────────────────     ──────────────────
    BernoulliNB             Often better ✓         Often worse ✗
    MultinomialNB           Often worse ✗          Often better ✓
    
    Research finding (McCallum & Nigam, 1998):
    "Bernoulli tends to work better with small vocabularies and 
     short documents; Multinomial tends to work better with large 
     vocabularies and longer documents."
```

---

## 9. Python Implementation

### 9.1 BernoulliNB from Scratch

```python
import numpy as np

class BernoulliNBFromScratch:
    """Bernoulli Naive Bayes classifier from scratch."""
    
    def __init__(self, alpha=1.0):
        self.alpha = alpha
    
    def fit(self, X, y):
        """
        X: binary feature matrix (n_samples × n_features), values in {0, 1}
        y: class labels
        """
        if hasattr(X, 'toarray'):
            X = X.toarray()
        X = np.array(X)
        
        self.classes = np.unique(y)
        n_classes = len(self.classes)
        n_samples, n_features = X.shape
        
        self.class_log_prior = np.zeros(n_classes)
        self.feature_prob = np.zeros((n_classes, n_features))
        
        for idx, c in enumerate(self.classes):
            X_c = X[y == c]
            n_c = len(X_c)
            
            # Prior
            self.class_log_prior[idx] = np.log(n_c / n_samples)
            
            # P(xᵢ=1|c) = (docs_with_feature + α) / (n_c + 2α)
            docs_with_feature = X_c.sum(axis=0)
            self.feature_prob[idx] = (
                (docs_with_feature + self.alpha) / (n_c + 2 * self.alpha)
            )
        
        return self
    
    def predict_log_proba(self, X):
        """
        Compute log P(c|x) for each class.
        
        log P(c|x) ∝ log P(c) + Σ [xᵢ·log(pᵢc) + (1-xᵢ)·log(1-pᵢc)]
        """
        if hasattr(X, 'toarray'):
            X = X.toarray()
        X = np.array(X, dtype=float)
        
        log_proba = np.zeros((len(X), len(self.classes)))
        
        for idx in range(len(self.classes)):
            p = self.feature_prob[idx]
            log_p = np.log(p)
            log_1_minus_p = np.log(1 - p)
            
            # xᵢ·log(pᵢc) + (1-xᵢ)·log(1-pᵢc) for all features
            log_likelihood = X @ log_p + (1 - X) @ log_1_minus_p
            log_proba[:, idx] = self.class_log_prior[idx] + log_likelihood
        
        return log_proba
    
    def predict(self, X):
        log_proba = self.predict_log_proba(X)
        return self.classes[np.argmax(log_proba, axis=1)]
    
    def predict_proba(self, X):
        log_proba = self.predict_log_proba(X)
        max_log = np.max(log_proba, axis=1, keepdims=True)
        log_proba_shifted = log_proba - max_log
        proba = np.exp(log_proba_shifted)
        proba /= proba.sum(axis=1, keepdims=True)
        return proba


# ─── Test: Compare with sklearn ─────────────────────────────
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import BernoulliNB

documents = [
    "team wins game score",
    "game score record",
    "team wins championship",
    "election vote policy",
    "vote policy reform",
    "election reform law",
]
labels = np.array([0, 0, 0, 1, 1, 1])

vectorizer = CountVectorizer(binary=True)
X = vectorizer.fit_transform(documents)

# Our implementation
bnb_scratch = BernoulliNBFromScratch(alpha=1.0)
bnb_scratch.fit(X, labels)

# sklearn
bnb_sklearn = BernoulliNB(alpha=1.0, binarize=None)
bnb_sklearn.fit(X, labels)

# Test document
new_doc = vectorizer.transform(["team wins vote"])

pred_scratch = bnb_scratch.predict(new_doc)
pred_sklearn = bnb_sklearn.predict(new_doc)
proba_scratch = bnb_scratch.predict_proba(new_doc)
proba_sklearn = bnb_sklearn.predict_proba(new_doc)

print(f"Scratch prediction: {pred_scratch}, proba: {proba_scratch[0]}")
print(f"Sklearn prediction: {pred_sklearn}, proba: {proba_sklearn[0]}")
print(f"Match: {np.allclose(proba_scratch, proba_sklearn, atol=1e-4)}")
```

### 9.2 BernoulliNB vs MultinomialNB Comparison

```python
from sklearn.naive_bayes import BernoulliNB, MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.model_selection import cross_val_score
import numpy as np

# Dataset with short documents (BernoulliNB should shine)
short_docs = [
    "win game", "score goal", "team play", "match win",
    "championship trophy", "league finals", "player score",
    "vote election", "policy reform", "government law",
    "political debate", "campaign rally", "senate bill",
    "congress vote",
]
labels_short = [0]*7 + [1]*7

# Dataset with longer documents (MultinomialNB should be better)
long_docs = [
    "team wins game score game score team play game",
    "game score record championship game team wins play game",
    "team wins championship game play score team wins game",
    "election vote policy vote election reform vote policy",
    "vote policy reform election law vote policy government",
    "election reform law government policy vote election",
]
labels_long = [0]*3 + [1]*3

for docs, labels, desc in [
    (short_docs, labels_short, "Short docs"),
    (long_docs, labels_long, "Long docs (with repetition)")
]:
    vec_count = CountVectorizer()
    X_count = vec_count.fit_transform(docs)
    
    vec_binary = CountVectorizer(binary=True)
    X_binary = vec_binary.fit_transform(docs)
    
    mnb = MultinomialNB(alpha=1.0)
    bnb = BernoulliNB(alpha=1.0, binarize=None)
    
    mnb.fit(X_count, labels)
    bnb.fit(X_binary, labels)
    
    mnb_acc = mnb.score(X_count, labels)
    bnb_acc = bnb.score(X_binary, labels)
    
    print(f"{desc}:")
    print(f"  MultinomialNB: {mnb_acc:.4f}")
    print(f"  BernoulliNB:   {bnb_acc:.4f}")
```

### 9.3 Practical Example: SMS Spam Detection

```python
from sklearn.naive_bayes import BernoulliNB, MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score
import numpy as np

# SMS messages (short text — good for BernoulliNB)
messages = [
    "Free entry in a competition! txt CLAIM to 8818",
    "WINNER!! Claim your prize now call 09061",
    "Free ringtone reply STOP to cancel",
    "Win cash prizes! Call 0900 for details",
    "Congratulations ur awarded $500 click here",
    "Hey, are we still meeting for lunch?",
    "Can you pick up milk on your way home?",
    "The meeting has been moved to 3pm",
    "Happy birthday! Hope you have a great day",
    "I'll be late, stuck in traffic",
    "Don't forget mom's birthday next week",
    "See you at the gym tomorrow morning",
    "URGENT: your account needs verification click link",
    "Thanks for the report, looks good",
    "Buy one get one free limited time only",
    "Project deadline extended to next Monday",
]
labels = [1,1,1,1,1, 0,0,0,0,0,0,0, 1,0,1,0]

# Compare both approaches
for name, pipeline in [
    ("MultinomialNB", Pipeline([
        ('vec', CountVectorizer(stop_words='english')),
        ('clf', MultinomialNB(alpha=1.0)),
    ])),
    ("BernoulliNB", Pipeline([
        ('vec', CountVectorizer(binary=True, stop_words='english')),
        ('clf', BernoulliNB(alpha=1.0, binarize=None)),
    ])),
]:
    pipeline.fit(messages, labels)
    
    # Test on new messages
    test_msgs = [
        "Free iPhone! Click to claim now",
        "Can we reschedule tomorrow's meeting?",
    ]
    preds = pipeline.predict(test_msgs)
    probas = pipeline.predict_proba(test_msgs)
    
    print(f"\n{name}:")
    for msg, pred, proba in zip(test_msgs, preds, probas):
        label = "SPAM" if pred == 1 else "HAM"
        conf = max(proba) * 100
        print(f"  '{msg[:45]}' → {label} ({conf:.1f}%)")
```

---

## 10. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **Bernoulli likelihood** | P(xᵢ\|c) = pᵢc^xᵢ · (1-pᵢc)^(1-xᵢ) | Models binary features |
| **Present feature** | Contributes pᵢc | Word appears → use pᵢc |
| **Absent feature** | Contributes (1-pᵢc) | Word missing → use (1-pᵢc) |
| **Key difference** | Penalizes absent features | Multinomial ignores absence |
| **Parameter** | p̂ᵢc = (docs_with + α)/(N_c + 2α) | Document-level counting |
| **vs Multinomial counting** | Counts documents | Multinomial counts occurrences |
| **Repetition** | "free free free" = "free" | Binary ignores frequency |
| **Best for** | Short docs, binary attributes | Absence carries information |
| **binarize param** | Threshold for auto-binarization | Can convert counts to binary |
| **sklearn class** | `BernoulliNB` | alpha, binarize, fit_prior |
| **Research** | Better for small vocabularies | McCallum & Nigam (1998) |

---

## 11. Revision Questions

### Q1: Bernoulli vs Multinomial Scoring
**Q:** Document D contains words {"free", "win"} from vocabulary {"free", "win", "meeting", "report"}. Show how BernoulliNB and MultinomialNB score this differently for a single class. Assume P(free|c)=0.8, P(win|c)=0.6, P(meeting|c)=0.1, P(report|c)=0.1.

**A:**
- **MultinomialNB:** score = P(c) × P(free|c) × P(win|c) = P(c) × 0.8 × 0.6 = P(c) × 0.48. Only uses present words.
- **BernoulliNB:** score = P(c) × P(free present) × P(win present) × P(meeting absent) × P(report absent) = P(c) × 0.8 × 0.6 × (1-0.1) × (1-0.1) = P(c) × 0.8 × 0.6 × 0.9 × 0.9 = P(c) × **0.3888**
- BernoulliNB's score is **lower** because it also includes the (1-p) terms for absent features. The absent words "meeting" and "report" contribute 0.9 × 0.9 = 0.81, reducing the overall score.

---

### Q2: Why Absence Matters
**Q:** An email does NOT contain the word "free". P("free" present|Spam) = 0.9, P("free" present|Ham) = 0.05. Show how BernoulliNB uses this absence to distinguish Spam from Ham. Why can't MultinomialNB use this signal?

**A:**
- **BernoulliNB:** P("free" absent|Spam) = 1-0.9 = 0.10 (very unlikely for spam). P("free" absent|Ham) = 1-0.05 = 0.95 (very normal for ham). The ratio 0.95/0.10 = 9.5× in favor of Ham — strong evidence!
- **MultinomialNB:** When "free" has count=0, it contributes P(free|c)⁰ = 1.0 for both classes. Zero count means the word is simply ignored — no contribution to the score whatsoever. MultinomialNB literally cannot use absence as evidence.

---

### Q3: Binarization Effect
**Q:** Document: "free free free buy" (3 occurrences of "free"). How does BernoulliNB treat this differently from MultinomialNB? Which model gives "free" more influence here?

**A:**
- **MultinomialNB:** x_free = 3, so contribution = P(free|c)³. If P(free|Spam)=0.3, contribution = 0.3³ = 0.027. The triple occurrence gives "free" triple weight in log space (3 × log P).
- **BernoulliNB:** After binarization, x_free = 1 (present), so contribution = P(free|c)¹ = 0.3. The repetition is completely ignored.
- **MultinomialNB** gives "free" much more influence (3× in log space). For spam detection where word repetition is a signal ("free free free" is spammier than "free"), MultinomialNB captures this better.

---

### Q4: Parameter Estimation
**Q:** With 4 Spam documents and vocabulary size 5, compute P("buy"|Spam) using Bernoulli estimation with α=1 if "buy" appears in 3 out of the 4 Spam documents. Compare with the Multinomial estimate if "buy" has 7 total occurrences across those 4 Spam documents (total words in Spam = 20).

**A:**
- **Bernoulli:** P("buy"|Spam) = (3+1)/(4+2) = 4/6 = **0.6667**
  - Denominator: N_c + 2α = 4 + 2 = 6 (binary: 2 outcomes per feature)
- **Multinomial:** P("buy"|Spam) = (7+1)/(20+5) = 8/25 = **0.3200**
  - Denominator: total_words + α·|V| = 20 + 5 = 25
- Bernoulli gives a higher estimate because it counts at the document level (3/4 docs contain "buy"), while Multinomial counts at the word level (7/20 total words are "buy").

---

### Q5: Vocabulary Size Effect
**Q:** Why does a larger vocabulary tend to help BernoulliNB more than MultinomialNB? Consider a vocabulary with 10,000 words and a document containing only 20 unique words.

**A:** With 10,000 vocabulary words and only 20 present, BernoulliNB uses **9,980 absence signals** in addition to the 20 presence signals. Each absent word contributes (1-pᵢc), which differs between classes. These absence signals can be very informative (e.g., if a word is common in one class but absent in the document). MultinomialNB ignores all 9,980 absent words — it only uses the 20 present words. So BernoulliNB gets 10,000 features of evidence vs MultinomialNB's 20. However, this can also be a disadvantage if the large vocabulary includes many irrelevant words, as their absence signals add noise.

---

### Q6: When BernoulliNB Fails
**Q:** Give a concrete scenario where BernoulliNB would perform poorly compared to MultinomialNB, and explain why.

**A:** **Sentiment analysis of product reviews:** A review saying "good good good, very very good, excellent product" vs "the product is good." Both have good=1 in BernoulliNB. MultinomialNB sees good=4 vs good=1, capturing the intensity difference. The repeated "good" strongly signals positive sentiment. BernoulliNB treats both documents identically regarding "good," losing the intensity signal. Generally, for longer documents where **word frequency correlates with class** (reviews, articles, essays), MultinomialNB preserves crucial frequency information that BernoulliNB discards through binarization.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 4: Multinomial Naive Bayes](04-multinomial-naive-bayes.md) | **Chapter 5: Bernoulli Naive Bayes** | [Chapter 6: Text Classification →](06-text-classification.md) |

### Unit 10 Chapters
1. [Bayes' Theorem Review](01-bayes-theorem-review.md)
2. [The Naive Assumption](02-naive-assumption.md)
3. [Gaussian Naive Bayes](03-gaussian-naive-bayes.md)
4. [Multinomial Naive Bayes](04-multinomial-naive-bayes.md)
5. **📍 Bernoulli Naive Bayes** ← You are here
6. [Text Classification with Naive Bayes](06-text-classification.md)
7. [Laplace Smoothing & Practical Considerations](07-laplace-smoothing.md)

---

> *"BernoulliNB teaches us that sometimes what's missing is just as important as what's present."*

© 2025 AI/ML Study Notes. All rights reserved.
