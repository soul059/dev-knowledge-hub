# 📘 Chapter 7: Laplace Smoothing & Practical Considerations

> **Unit 10: Naive Bayes** | **Section 7 of 7**
> Solving the zero frequency problem with additive smoothing, understanding the mathematics of Laplace and Lidstone smoothing, preventing numerical underflow with log probabilities, choosing the optimal smoothing parameter, and mastering practical deployment considerations.

---

## 📑 Table of Contents

1. [The Zero Frequency Problem](#1-the-zero-frequency-problem)
2. [Additive (Laplace) Smoothing](#2-additive-laplace-smoothing)
3. [Lidstone Smoothing (0 < α < 1)](#3-lidstone-smoothing-0--α--1)
4. [Effect on Probability Estimates](#4-effect-on-probability-estimates)
5. [Choosing α: Hyperparameter Tuning](#5-choosing-α-hyperparameter-tuning)
6. [sklearn Alpha Parameter](#6-sklearn-alpha-parameter)
7. [Underflow Prevention: Log Probabilities](#7-underflow-prevention-log-probabilities)
8. [Practical Importance & Deployment](#8-practical-importance--deployment)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. The Zero Frequency Problem

### 1.1 What Goes Wrong Without Smoothing

When a word (or feature value) never appears in the training data for a particular class, its estimated probability is **zero**:

```
    Training data for class "Spam":
    ─────────────────────────────────
    "buy free click"
    "free offer deal"
    "buy deal special"
    
    Word "meeting" never appears in Spam training data.
    
    Without smoothing:
    P("meeting" | Spam) = 0/9 = 0.000
    
    Now classify: "meeting free today"
    
    score(Spam) = P(Spam) × P("meeting"|Spam) × P("free"|Spam) × P("today"|Spam)
                = 0.5    ×       0.000        ×     0.333      ×     ???
                = 0     ← ZERO! All other evidence destroyed!
```

### 1.2 Why This Is Catastrophic

```
    ╔═══════════════════════════════════════════════════════════════╗
    ║                                                               ║
    ║  THE ZERO FREQUENCY PROBLEM:                                 ║
    ║                                                               ║
    ║  A single unseen feature ZEROES OUT the entire probability.  ║
    ║                                                               ║
    ║  P(x|c) = P(x₁|c) × P(x₂|c) × ... × P(xₙ|c)             ║
    ║         = 0.8 × 0.6 × 0.0 × 0.9 × 0.7                     ║
    ║         = 0.000                                              ║
    ║         ↑                                                     ║
    ║     One zero kills everything!                                ║
    ║                                                               ║
    ║  In log space: log(0) = -∞                                   ║
    ║  The log score becomes -∞ regardless of other evidence.      ║
    ║                                                               ║
    ║  This is NOT a rare problem:                                 ║
    ║  • With 10,000 word vocabulary, many words are class-unseen  ║
    ║  • Small training sets make this worse                       ║
    ║  • New words in test data (OOV) hit this too                ║
    ║                                                               ║
    ╚═══════════════════════════════════════════════════════════════╝
```

### 1.3 Visual: The Problem

```
    Without Smoothing:
    
    P(word|class)
    1.0 ┤
        │  ■
    0.8 ┤  ■
        │  ■    ■
    0.6 ┤  ■    ■
        │  ■    ■    ■
    0.4 ┤  ■    ■    ■
        │  ■    ■    ■    ■
    0.2 ┤  ■    ■    ■    ■
        │  ■    ■    ■    ■
    0.0 ┤──■────■────■────■────────────────
        └──┬────┬────┬────┬────┬────┬────┬──
          buy  free deal offer meet report today
                                    ↑
                              P = 0 (NEVER seen!)
                              
    One zero → entire product = 0 → classification fails
```

---

## 2. Additive (Laplace) Smoothing

### 2.1 The Fix: Add Pseudo-Counts

The solution is beautifully simple: **add a small count to every feature**, even unseen ones:

```
    ╔══════════════════════════════════════════════════════════════════╗
    ║                                                                  ║
    ║  LAPLACE SMOOTHING (α = 1):                                     ║
    ║                                                                  ║
    ║  For Multinomial NB:                                            ║
    ║                                                                  ║
    ║              count(wⱼ, c) + α                                   ║
    ║  P(wⱼ|c) = ─────────────────────                                ║
    ║              Σₖ count(wₖ, c) + α · |V|                          ║
    ║                                                                  ║
    ║  For Bernoulli NB:                                               ║
    ║                                                                  ║
    ║              docs_with(wⱼ, c) + α                               ║
    ║  P(wⱼ|c) = ──────────────────────────                            ║
    ║                   Nc + 2α                                        ║
    ║                                                                  ║
    ║  Where:                                                          ║
    ║    α = smoothing parameter (Laplace: α = 1)                     ║
    ║    |V| = vocabulary size (number of distinct features)          ║
    ║    Nc = total count in class c (or docs in class c)             ║
    ║                                                                  ║
    ╚══════════════════════════════════════════════════════════════════╝
```

### 2.2 How It Works

```
    Before smoothing (raw counts):
    
    Word    │ Spam count │ P(word|Spam) │ Problem?
    ────────┼────────────┼──────────────┼──────────
    buy     │     3      │   3/9 = 0.333│ OK
    free    │     3      │   3/9 = 0.333│ OK
    click   │     1      │   1/9 = 0.111│ OK
    deal    │     2      │   2/9 = 0.222│ OK
    meeting │     0      │   0/9 = 0.000│ ⚠️ ZERO!
    report  │     0      │   0/9 = 0.000│ ⚠️ ZERO!
    total   │     9      │              │
    
    |V| = 6 (vocabulary size)
    
    ─────────────────────────────────────────────────
    
    After Laplace smoothing (α = 1):
    
    Word    │ Count + α │ P(word|Spam)          │ Problem?
    ────────┼───────────┼───────────────────────┼──────────
    buy     │   3 + 1=4 │   4/(9+6) = 4/15=0.267│ ✓ Reduced slightly
    free    │   3 + 1=4 │   4/15 = 0.267        │ ✓ Reduced slightly
    click   │   1 + 1=2 │   2/15 = 0.133        │ ✓ Slight change
    deal    │   2 + 1=3 │   3/15 = 0.200        │ ✓ Slight change
    meeting │   0 + 1=1 │   1/15 = 0.067        │ ✓ Non-zero!
    report  │   0 + 1=1 │   1/15 = 0.067        │ ✓ Non-zero!
    total   │    15     │   15/15 = 1.000        │ ✓ Sums to 1!
```

### 2.3 Intuition: The Virtual Observations

```
    ┌──────────────────────────────────────────────────────────────┐
    │  Laplace smoothing acts as if we've seen each word          │
    │  at least once in every class.                               │
    │                                                              │
    │  Real observations:                                          │
    │    Spam: "buy free click buy free deal buy deal free"        │
    │          buy=3, free=3, click=1, deal=2, meeting=0, report=0│
    │                                                              │
    │  Virtual (imagined) observations:                            │
    │    + "buy free click deal meeting report"  (one of each)    │
    │                                                              │
    │  Combined:                                                   │
    │    buy=4, free=4, click=2, deal=3, meeting=1, report=1     │
    │                                                              │
    │  This is a BAYESIAN interpretation: Laplace smoothing is    │
    │  equivalent to a uniform Dirichlet prior Dir(α, α, ..., α) │
    └──────────────────────────────────────────────────────────────┘
```

---

## 3. Lidstone Smoothing (0 < α < 1)

### 3.1 Generalized Additive Smoothing

Laplace smoothing (α=1) can be **too aggressive** — it adds a full pseudo-count which significantly affects rare words. **Lidstone smoothing** uses a smaller α:

```
    ╔════════════════════════════════════════════════════════════╗
    ║                                                            ║
    ║  LIDSTONE SMOOTHING:                                      ║
    ║                                                            ║
    ║              count(wⱼ, c) + α                             ║
    ║  P(wⱼ|c) = ─────────────────────     where 0 < α < 1    ║
    ║              Σₖ count(wₖ, c) + α·|V|                     ║
    ║                                                            ║
    ║  Special cases:                                            ║
    ║    α = 1   → Laplace smoothing (add-one)                 ║
    ║    α = 0.5 → Jeffreys' prior (√Dirichlet)               ║
    ║    α → 0   → Maximum Likelihood (no smoothing)           ║
    ║    α = 0.1 → Common choice in practice                   ║
    ║                                                            ║
    ╚════════════════════════════════════════════════════════════╝
```

### 3.2 Comparison of α Values

```
    Word "buy" (count=3 in Spam, total Spam words=9, |V|=6):
    
    α = 0      → P = 3/9 = 0.3333          (MLE, no smoothing)
    α = 0.01   → P = 3.01/9.06 = 0.3322    (barely changed)
    α = 0.1    → P = 3.1/9.6 = 0.3229      (slightly reduced)
    α = 0.5    → P = 3.5/12 = 0.2917       (noticeable reduction)
    α = 1.0    → P = 4/15 = 0.2667         (significant reduction)
    α = 10.0   → P = 13/69 = 0.1884        (heavily smoothed → uniform)
    
    Word "meeting" (count=0 in Spam):
    
    α = 0      → P = 0/9 = 0.0000          ⚠️ ZERO
    α = 0.01   → P = 0.01/9.06 = 0.0011    (tiny but non-zero)
    α = 0.1    → P = 0.1/9.6 = 0.0104      (small but non-zero)
    α = 0.5    → P = 0.5/12 = 0.0417       (moderate)
    α = 1.0    → P = 1/15 = 0.0667         (significant)
    α = 10.0   → P = 10/69 = 0.1449        (close to uniform=0.1667)
```

### 3.3 Visual: Effect of α

```
    P(word|Spam) as α increases:
    
    0.35 ┤                                        ← buy (common)
         │  ●
    0.30 ┤    ●
         │      ●
    0.25 ┤        ●
         │          ●
    0.20 ┤            ●                           ← All converge to
         │     uniform = 1/|V| = 0.1667              uniform (1/6)
    0.15 ┤                  ●   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
         │              ●
    0.10 ┤          ●                             ← meeting (unseen)
         │      ●
    0.05 ┤  ●
         │●
    0.00 ┤
         └──┬───┬───┬───┬───┬───┬───┬───┬──
            0  0.1 0.5  1   2   5  10  20
                        α →
    
    Small α: Estimates close to MLE (data-driven)
    Large α: Estimates close to uniform (prior-driven)
```

---

## 4. Effect on Probability Estimates

### 4.1 How Smoothing Changes the Distribution

```
    ┌──────────────────────────────────────────────────────────┐
    │  EFFECT OF SMOOTHING:                                    │
    │                                                          │
    │  1. UNSEEN features: 0 → small positive value           │
    │     (prevents the zero-frequency catastrophe)            │
    │                                                          │
    │  2. COMMON features: slightly reduced probability       │
    │     (probability mass "stolen" and given to unseen)      │
    │                                                          │
    │  3. RARE features: proportionally more affected          │
    │     (adding 1 to count of 2 is bigger change than 100)  │
    │                                                          │
    │  4. OVERALL: distribution shifts toward UNIFORM          │
    │     (more α → more uniform → less discriminative)        │
    │                                                          │
    │  5. TOTAL probability still sums to 1                    │
    │     (it's a proper probability distribution)             │
    └──────────────────────────────────────────────────────────┘
```

### 4.2 The Bias-Variance Perspective

```
    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  NO SMOOTHING (α = 0):                                    │
    │    • Bias: Zero (estimates are pure MLE)                  │
    │    • Variance: High (zero counts, unstable estimates)     │
    │    • Risk: Catastrophic failure on unseen features        │
    │                                                            │
    │  SMALL α (0.01 - 0.1):                                    │
    │    • Bias: Very low                                       │
    │    • Variance: Slightly reduced                           │
    │    • Good for: Large training sets, large vocabularies    │
    │                                                            │
    │  MODERATE α (0.5 - 1.0):                                  │
    │    • Bias: Moderate                                       │
    │    • Variance: Noticeably reduced                         │
    │    • Good for: Medium training sets, balanced choice      │
    │                                                            │
    │  LARGE α (> 1):                                           │
    │    • Bias: High (estimates → uniform)                     │
    │    • Variance: Low                                        │
    │    • Risk: Loss of discriminative power                   │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
    
    Optimal α minimizes total error (bias² + variance):
    
    Error │╲
          │ ╲ Variance
          │  ╲          ╱
          │   ╲   ╱───── Bias²
          │    ╲╱
          │     ● ← optimal α
          │
          └────────────────────→ α
          0    0.1   1    10
```

### 4.3 Numerical Example: Impact on Classification

```
    Vocabulary: {a, b, c}, |V| = 3
    
    Class 0 counts: a=10, b=5, c=0 (total=15)
    Class 1 counts: a=0, b=5, c=10 (total=15)
    Equal priors: P(C₀) = P(C₁) = 0.5
    
    Classify document: x = {a=1, b=0, c=1}
    
    ─── Without smoothing (α=0) ────────────────────────
    P(a|C₀) = 10/15, P(c|C₀) = 0/15 = 0
    score(C₀) = 0.5 × (10/15) × (0/15) = 0    ← WRONG! Zero kills it
    score(C₁) = 0.5 × (0/15) × (10/15) = 0    ← Also zero!
    Result: TIE (random guess) or error
    
    ─── With α=1 (Laplace) ─────────────────────────────
    P(a|C₀) = 11/18 = 0.611, P(c|C₀) = 1/18 = 0.056
    P(a|C₁) = 1/18 = 0.056,  P(c|C₁) = 11/18 = 0.611
    score(C₀) = 0.5 × 0.611 × 0.056 = 0.0171
    score(C₁) = 0.5 × 0.056 × 0.611 = 0.0171
    Result: TIE (symmetric case — makes sense!)
    
    ─── With α=0.1 (Lidstone) ──────────────────────────
    P(a|C₀) = 10.1/15.3 = 0.660, P(c|C₀) = 0.1/15.3 = 0.00654
    P(a|C₁) = 0.1/15.3 = 0.00654, P(c|C₁) = 10.1/15.3 = 0.660
    score(C₀) = 0.5 × 0.660 × 0.00654 = 0.00216
    score(C₁) = 0.5 × 0.00654 × 0.660 = 0.00216
    Result: TIE (correct — symmetric data should be ambiguous)
```

---

## 5. Choosing α: Hyperparameter Tuning

### 5.1 General Guidelines

```
    ┌──────────────────────────────────────────────────────────────┐
    │  CHOOSING α — RULES OF THUMB:                               │
    │                                                              │
    │  1. START with α = 1.0 (Laplace) — safe default            │
    │                                                              │
    │  2. TUNE over: [0.001, 0.01, 0.1, 0.5, 1.0, 2.0, 10.0]   │
    │                                                              │
    │  3. CONTEXT MATTERS:                                         │
    │     • Large training data → smaller α (trust data more)    │
    │     • Small training data → larger α (need more smoothing) │
    │     • Large vocabulary → smaller α (too much redistribution)│
    │     • Small vocabulary → larger α is fine                   │
    │                                                              │
    │  4. For TEXT classification:                                  │
    │     • CountVectorizer: α ∈ [0.1, 1.0] typically best       │
    │     • TfidfVectorizer: α ∈ [0.01, 0.1] typically best     │
    │     • (TF-IDF already smooths via weighting)               │
    │                                                              │
    │  5. ALWAYS use cross-validation to select α                 │
    └──────────────────────────────────────────────────────────────┘
```

### 5.2 Grid Search for α

```python
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB

pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(stop_words='english', ngram_range=(1, 2))),
    ('clf', MultinomialNB()),
])

# Search over alpha values
param_grid = {'clf__alpha': [0.001, 0.01, 0.05, 0.1, 0.5, 1.0, 2.0, 5.0, 10.0]}

grid_search = GridSearchCV(
    pipeline, param_grid, cv=5, scoring='f1_weighted', n_jobs=-1
)
grid_search.fit(train_texts, train_labels)

print(f"Best alpha: {grid_search.best_params_['clf__alpha']}")
print(f"Best F1:    {grid_search.best_score_:.4f}")

# Show all results
for params, mean_score, std_score in zip(
    grid_search.cv_results_['params'],
    grid_search.cv_results_['mean_test_score'],
    grid_search.cv_results_['std_test_score'],
):
    print(f"  α = {params['clf__alpha']:<6}: F1 = {mean_score:.4f} ± {std_score:.4f}")
```

---

## 6. sklearn Alpha Parameter

### 6.1 How sklearn Implements Smoothing

All sklearn Naive Bayes classifiers use the `alpha` parameter:

```python
from sklearn.naive_bayes import MultinomialNB, BernoulliNB, GaussianNB

# MultinomialNB: additive smoothing on word counts
mnb = MultinomialNB(alpha=1.0)  # Laplace smoothing
# P(wⱼ|c) = (count(wⱼ,c) + alpha) / (total(c) + alpha * n_features)

# BernoulliNB: additive smoothing on binary features  
bnb = BernoulliNB(alpha=1.0)
# P(wⱼ|c) = (docs_with(wⱼ,c) + alpha) / (Nc + 2*alpha)

# GaussianNB: uses var_smoothing (different concept!)
gnb = GaussianNB(var_smoothing=1e-9)
# Adds var_smoothing * max(variance) to all variances
# Prevents division by zero, not additive smoothing
```

### 6.2 Alpha Values and Their Effects

| `alpha` | Name | Effect | When to Use |
|---------|------|--------|-------------|
| 0.0 | No smoothing | Pure MLE | Never (causes zero probs) |
| 0.001 | Very light | Minimal perturbation | Very large datasets |
| 0.01 | Light | Small smoothing | Large datasets + TF-IDF |
| 0.1 | Moderate | Common choice | Most text classification |
| 0.5 | Jeffreys | Half-count | Medium-sized datasets |
| 1.0 | Laplace | Full count (default) | Safe default, small data |
| 10.0 | Heavy | Strong regularization | Very small datasets |

### 6.3 Special Cases in sklearn

```python
# Alpha = 0: Will raise error or give -inf in log space
# Alpha < 0: Not allowed (must be >= 0)
# Alpha = very large: All probabilities → 1/|V| (uniform)

# For GaussianNB, smoothing is different:
gnb = GaussianNB(var_smoothing=1e-9)
# var_smoothing is a fraction of the largest variance
# Added to ALL variances for numerical stability
# NOT related to Laplace/Lidstone smoothing
```

---

## 7. Underflow Prevention: Log Probabilities

### 7.1 The Underflow Problem

```
    ╔════════════════════════════════════════════════════════════════╗
    ║  THE UNDERFLOW PROBLEM:                                       ║
    ║                                                               ║
    ║  P(x|c) = P(x₁|c) × P(x₂|c) × ... × P(xₙ|c)              ║
    ║                                                               ║
    ║  With 1000 features, each probability ≈ 0.01:                ║
    ║  P(x|c) = 0.01¹⁰⁰⁰ = 10⁻²⁰⁰⁰                             ║
    ║                                                               ║
    ║  Float64 minimum: ≈ 5 × 10⁻³²⁴                              ║
    ║                                                               ║
    ║  10⁻²⁰⁰⁰ < 10⁻³²⁴ → UNDERFLOW TO ZERO!                    ║
    ║                                                               ║
    ║  Both classes underflow to 0 → division by zero,             ║
    ║  NaN results, or arbitrary classification.                    ║
    ║                                                               ║
    ╚════════════════════════════════════════════════════════════════╝
```

### 7.2 The Solution: Log Space

```
    Instead of:
        score(c) = P(c) × Π P(xᵢ|c)         ← underflows!
    
    Compute:
        log_score(c) = log P(c) + Σ log P(xᵢ|c)    ← stable!
    
    ─────────────────────────────────────────────────────────
    
    Properties of log:
    1. log(a × b) = log(a) + log(b)   ← products → sums
    2. log is monotonically increasing ← argmax preserved
    3. log values are manageable       ← log(0.01) = -4.6
    
    Example:
    Product form: 0.01¹⁰⁰⁰ = 10⁻²⁰⁰⁰ → underflow!
    Log form: 1000 × log(0.01) = 1000 × (-4.605) = -4605 → perfectly fine!
```

### 7.3 Log Probability Formulas

For each variant of Naive Bayes:

```
    ╔══════════════════════════════════════════════════════════════╗
    ║  MULTINOMIAL NB (log space):                                ║
    ║                                                              ║
    ║  log_score(c) = log P(c) + Σⱼ xⱼ · log P(wⱼ|c)           ║
    ║                                                              ║
    ╠══════════════════════════════════════════════════════════════╣
    ║  BERNOULLI NB (log space):                                  ║
    ║                                                              ║
    ║  log_score(c) = log P(c) +                                  ║
    ║    Σᵢ [xᵢ · log(pᵢc) + (1-xᵢ) · log(1-pᵢc)]             ║
    ║                                                              ║
    ╠══════════════════════════════════════════════════════════════╣
    ║  GAUSSIAN NB (log space):                                   ║
    ║                                                              ║
    ║  log_score(c) = log P(c) +                                  ║
    ║    Σᵢ [-½ log(2πσ²ᵢc) - (xᵢ - μᵢc)² / (2σ²ᵢc)]         ║
    ║                                                              ║
    ╚══════════════════════════════════════════════════════════════╝
```

### 7.4 Converting Log Scores to Probabilities

When you need actual probabilities (not just the argmax), use the **log-sum-exp trick**:

```
    Problem: Convert log scores to probabilities
    
    log_scores = [-4605, -4610, -4620]    (3 classes)
    
    Naive approach:
    scores = [exp(-4605), exp(-4610), exp(-4620)]  ← ALL underflow to 0!
    
    Log-Sum-Exp trick:
    1. Find max: M = max(log_scores) = -4605
    2. Subtract max: shifted = [-4605+4605, -4610+4605, -4620+4605]
                             = [0, -5, -15]
    3. Exponentiate: exp_shifted = [1.0, 0.00674, 3.06e-7]
    4. Normalize: sum = 1.00674
                  probs = [0.9933, 0.00669, 3.04e-7]
    
    ─────────────────────────────────────────────────
    
    In code:
    def log_sum_exp(log_scores):
        max_score = np.max(log_scores)
        shifted = log_scores - max_score
        return max_score + np.log(np.sum(np.exp(shifted)))
    
    log_evidence = log_sum_exp(log_scores)
    log_posteriors = log_scores - log_evidence
    posteriors = np.exp(log_posteriors)
```

### 7.5 How sklearn Handles This

```python
# sklearn Naive Bayes classifiers work in log space internally
from sklearn.naive_bayes import MultinomialNB
import numpy as np

mnb = MultinomialNB(alpha=1.0)
mnb.fit(X_train, y_train)

# Internally stored as log probabilities:
print("Log priors:", mnb.class_log_prior_)          # log P(c)
print("Log feature probs:", mnb.feature_log_prob_)   # log P(wⱼ|c)

# predict() uses log space for argmax
# predict_proba() uses log-sum-exp for probabilities
# predict_log_proba() returns raw log posteriors

y_pred = mnb.predict(X_test)              # argmax of log scores
proba = mnb.predict_proba(X_test)          # proper probabilities (via LSE)
log_proba = mnb.predict_log_proba(X_test)  # log posteriors
```

---

## 8. Practical Importance & Deployment

### 8.1 When Smoothing Makes or Breaks the Model

```
    ┌──────────────────────────────────────────────────────────────┐
    │  SCENARIO                     │ SMOOTHING IMPORTANCE         │
    ├───────────────────────────────┼──────────────────────────────┤
    │ Large vocab, small training   │ CRITICAL — many unseen words│
    │ Large vocab, large training   │ Important — some unseen     │
    │ Small vocab, large training   │ Minor — most values seen    │
    │ New test data has novel words │ CRITICAL — OOV words        │
    │ Streaming/online learning     │ CRITICAL — data evolves     │
    │ Multi-class (many classes)    │ Very important              │
    └──────────────────────────────┴──────────────────────────────┘
```

### 8.2 Deployment Checklist

```
    ┌──────────────────────────────────────────────────────────┐
    │  NAIVE BAYES DEPLOYMENT CHECKLIST                       │
    │                                                          │
    │  ☐ Preprocessing pipeline is reproducible               │
    │  ☐ Vectorizer fitted on training data only              │
    │  ☐ Alpha tuned via cross-validation                     │
    │  ☐ Model serialized with Pipeline (vectorizer + clf)    │
    │  ☐ Log probabilities used internally (no underflow)     │
    │  ☐ OOV handling tested (new words in production)        │
    │  ☐ Probabilities calibrated if used for ranking         │
    │  ☐ Monitoring for concept drift                         │
    │  ☐ Incremental learning setup if data streams           │
    │  ☐ Model interpretability: top features per class       │
    └──────────────────────────────────────────────────────────┘
```

### 8.3 Incremental Learning with `partial_fit`

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import HashingVectorizer

# HashingVectorizer doesn't need fitting — perfect for streaming
vectorizer = HashingVectorizer(
    n_features=2**16,
    stop_words='english',
    alternate_sign=False,    # non-negative for MultinomialNB
)

mnb = MultinomialNB(alpha=0.1)

# Simulate streaming data
batches = [
    (["Free money click here", "Win prize now"], [1, 1]),
    (["Meeting tomorrow 3pm", "Project update"], [0, 0]),
    (["Buy cheap pills online", "Team lunch friday"], [1, 0]),
]

classes = [0, 1]  # must specify all classes upfront

for batch_texts, batch_labels in batches:
    X_batch = vectorizer.transform(batch_texts)
    mnb.partial_fit(X_batch, batch_labels, classes=classes)
    print(f"Trained on {len(batch_labels)} samples, "
          f"total feature log probs shape: {mnb.feature_log_prob_.shape}")
```

---

## 9. Python Implementation

### 9.1 Smoothing Effect Visualization

```python
import numpy as np

def demonstrate_smoothing():
    """Show effect of different alpha values on probability estimates."""
    
    # Simulated word counts for Spam class
    word_counts = {
        'free': 50, 'buy': 30, 'click': 20, 'offer': 15,
        'win': 10, 'urgent': 5, 'deal': 3, 'limited': 1,
        'meeting': 0, 'report': 0, 'project': 0, 'review': 0,
    }
    
    total = sum(word_counts.values())  # 134
    vocab_size = len(word_counts)       # 12
    
    alphas = [0, 0.01, 0.1, 0.5, 1.0, 5.0]
    
    print(f"Total words: {total}, Vocabulary: {vocab_size}")
    print(f"\n{'Word':<10} {'Count':>5}", end="")
    for a in alphas:
        print(f" {'α='+str(a):>8}", end="")
    print()
    print("-" * (18 + 9 * len(alphas)))
    
    for word, count in word_counts.items():
        print(f"{word:<10} {count:>5}", end="")
        for a in alphas:
            if a == 0 and count == 0:
                print(f" {'0.0000':>8}", end="")
            else:
                smoothed = (count + a) / (total + a * vocab_size)
                print(f" {smoothed:>8.4f}", end="")
        print()
    
    # Verify probabilities sum to 1
    print(f"\n{'SUM':<10} {'':>5}", end="")
    for a in alphas:
        if a == 0:
            total_prob = sum(c / total for c in word_counts.values() if c > 0)
        else:
            total_prob = sum(
                (c + a) / (total + a * vocab_size) 
                for c in word_counts.values()
            )
        print(f" {total_prob:>8.4f}", end="")
    print("  ← Should all be 1.0 (except α=0 with zeros)")

demonstrate_smoothing()
```

### 9.2 Log Probability Implementation

```python
import numpy as np

def classify_with_log_probs(x, class_word_counts, class_priors, 
                            vocab_size, alpha=1.0):
    """
    Classify using log probabilities to prevent underflow.
    
    x: dict of word → count for the document
    class_word_counts: dict of class → dict of word → count
    class_priors: dict of class → prior probability
    vocab_size: int, number of unique words
    alpha: float, smoothing parameter
    """
    log_scores = {}
    
    for cls in class_priors:
        # Start with log prior
        log_score = np.log(class_priors[cls])
        
        # Total words in this class
        total_words = sum(class_word_counts[cls].values())
        
        # Add log likelihood for each word in document
        for word, count in x.items():
            word_count_in_class = class_word_counts[cls].get(word, 0)
            
            # Smoothed probability
            smoothed_prob = (word_count_in_class + alpha) / \
                          (total_words + alpha * vocab_size)
            
            # Add log probability (× word count in document)
            log_score += count * np.log(smoothed_prob)
        
        log_scores[cls] = log_score
    
    # Classification
    predicted_class = max(log_scores, key=log_scores.get)
    
    # Convert to probabilities using log-sum-exp
    max_log = max(log_scores.values())
    exp_scores = {c: np.exp(s - max_log) for c, s in log_scores.items()}
    total_exp = sum(exp_scores.values())
    probabilities = {c: s / total_exp for c, s in exp_scores.items()}
    
    return predicted_class, probabilities, log_scores


# ─── Example ────────────────────────────────────────────────
class_word_counts = {
    'Spam': {'free': 50, 'buy': 30, 'click': 20, 'win': 10, 'meeting': 0},
    'Ham':  {'free': 5,  'buy': 3,  'click': 2,  'win': 1,  'meeting': 40},
}

class_priors = {'Spam': 0.3, 'Ham': 0.7}
vocab_size = 5  # simplified

# Classify: "free buy meeting"
document = {'free': 1, 'buy': 1, 'meeting': 1}

for alpha in [0.01, 0.1, 1.0]:
    pred, probs, logs = classify_with_log_probs(
        document, class_word_counts, class_priors, vocab_size, alpha
    )
    print(f"α={alpha:<5}: {pred} | P(Spam)={probs['Spam']:.4f}, "
          f"P(Ham)={probs['Ham']:.4f} | "
          f"log: Spam={logs['Spam']:.2f}, Ham={logs['Ham']:.2f}")
```

### 9.3 Complete Alpha Tuning Pipeline

```python
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score
import warnings
warnings.filterwarnings('ignore')

# Sample dataset
texts = [
    "free money win prize click now offer deal",
    "limited offer buy now free shipping discount",
    "urgent action required claim your reward today",
    "congratulations you won lottery prize free",
    "buy cheap products online free delivery fast",
    "team meeting tomorrow quarterly review agenda",
    "project update report deadline friday review",
    "please review attached proposal document",
    "schedule conference call discuss client needs",
    "monthly newsletter company updates attached",
    "code review pull request merge branch ready",
    "budget approval needed for marketing campaign",
] * 5  # Repeat for more data
labels = ([1]*5 + [0]*7) * 5

# Test different alpha values
alphas = [0.001, 0.005, 0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.0, 5.0, 10.0]

print(f"{'Alpha':<10} {'Count+MNB':>12} {'TFIDF+MNB':>12}")
print("-" * 36)

best_count_alpha, best_count_score = None, 0
best_tfidf_alpha, best_tfidf_score = None, 0

for alpha in alphas:
    # Count + MNB
    pipe_count = Pipeline([
        ('vec', CountVectorizer(stop_words='english')),
        ('clf', MultinomialNB(alpha=alpha)),
    ])
    score_count = cross_val_score(pipe_count, texts, labels, cv=3).mean()
    
    # TF-IDF + MNB
    pipe_tfidf = Pipeline([
        ('vec', TfidfVectorizer(stop_words='english')),
        ('clf', MultinomialNB(alpha=alpha)),
    ])
    score_tfidf = cross_val_score(pipe_tfidf, texts, labels, cv=3).mean()
    
    marker_c = " ←" if score_count > best_count_score else ""
    marker_t = " ←" if score_tfidf > best_tfidf_score else ""
    
    if score_count > best_count_score:
        best_count_score = score_count
        best_count_alpha = alpha
    if score_tfidf > best_tfidf_score:
        best_tfidf_score = score_tfidf
        best_tfidf_alpha = alpha
    
    print(f"{alpha:<10} {score_count:>10.4f}{marker_c:>2} {score_tfidf:>10.4f}{marker_t:>2}")

print(f"\nBest Count+MNB:  α={best_count_alpha} (accuracy={best_count_score:.4f})")
print(f"Best TFIDF+MNB:  α={best_tfidf_alpha} (accuracy={best_tfidf_score:.4f})")
```

---

## 10. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **Zero frequency** | P(w\|c) = 0 → kills entire product | Single zero = catastrophic failure |
| **Laplace smoothing** | (count + 1) / (total + \|V\|) | Add 1 pseudo-count (α=1) |
| **Lidstone smoothing** | (count + α) / (total + α·\|V\|) | Fractional pseudo-counts (0<α<1) |
| **α = 0** | Pure MLE | No smoothing — dangerous |
| **α = 1** | Laplace | Safe default, sometimes too strong |
| **α → ∞** | Uniform distribution | All features equally probable |
| **Bias-variance** | More α → more bias, less variance | Optimal α balances both |
| **Log probabilities** | log P(c) + Σ log P(xᵢ\|c) | Products → sums, prevents underflow |
| **Log-sum-exp** | max + log Σ exp(shifted) | Converts log scores to probabilities |
| **sklearn alpha** | `MultinomialNB(alpha=1.0)` | Tune via GridSearchCV |
| **var_smoothing** | GaussianNB only, adds to σ² | Different from additive smoothing |
| **Tuning guideline** | Count: α∈[0.1,1.0]; TF-IDF: α∈[0.01,0.1] | Smaller α with TF-IDF |

---

## 11. Revision Questions

### Q1: Why Smoothing Is Necessary
**Q:** A Multinomial NB classifier trained on 1000 emails has a vocabulary of 50,000 words. Why is smoothing absolutely essential, not optional? What specific failure would occur without it?

**A:** With 50,000 vocabulary words and only 1000 emails, many words will appear only in one class (or not at all in some classes). Without smoothing, P(word|class) = 0 for these unseen words. When a test email contains even ONE word with P=0 for a class, the entire product P(x|c) = ... × 0 × ... = **0**, making that class impossible regardless of all other evidence. In log space, log(0) = **-∞**, which makes the score -∞. With 50,000 words, virtually every test email will contain at least one word unseen in at least one class, so the classifier would fail on **nearly every prediction**. Smoothing ensures all probabilities are > 0.

---

### Q2: Choosing Alpha
**Q:** You're comparing α=0.01 and α=1.0 for a text classifier with 100,000 training documents and 30,000 vocabulary words. Which would you expect to work better, and why?

**A:** **α=0.01** would likely work better. With 100,000 documents, most words have been seen many times, so the MLE estimates are already reliable. Adding 1 full pseudo-count (α=1) per word adds 30,000 pseudo-observations to each class denominator, which significantly dilutes the observed counts. α=0.01 adds only 300 pseudo-observations, barely perturbing the estimates while still preventing zero probabilities. With large training data, we should **trust the data more** (small α = less bias). With small training data, we need more smoothing (large α) because the observed counts are less reliable.

---

### Q3: Log Probability Computation
**Q:** Compute log_score(Spam) for a document with words: free=3, buy=1, meeting=0, using P(Spam)=0.4, P(free|Spam)=0.3, P(buy|Spam)=0.2, P(meeting|Spam)=0.05. Show why using log space is essential if we had 500 features instead of 3.

**A:**
- log_score(Spam) = log(0.4) + 3·log(0.3) + 1·log(0.2) + 0·log(0.05)
- = -0.9163 + 3·(-1.2040) + (-1.6094) + 0
- = -0.9163 + (-3.6120) + (-1.6094)
- = **-6.1377**
- Product form: 0.4 × 0.3³ × 0.2 = 0.4 × 0.027 × 0.2 = 0.00216 (same as exp(-6.1377))
- With 500 features averaging P≈0.1: product = 0.1⁵⁰⁰ = 10⁻⁵⁰⁰, far below float64 minimum (10⁻³²⁴) → **underflow to zero**. Log form: 500 × log(0.1) = 500 × (-2.3026) = -1151.3, a perfectly representable number. Log space handles any number of features.

---

### Q4: Smoothing and Distribution Shape
**Q:** A class has word counts: [100, 50, 20, 10, 5, 0, 0, 0, 0, 0] (10 words, total=185). Show how α=1 and α=10 change the distribution. Which is more uniform? What is the limiting distribution as α→∞?

**A:**
- **α=1:** denominator = 185+10=195. Probs: [101/195, 51/195, 21/195, 11/195, 6/195, 1/195, ...] = [0.518, 0.262, 0.108, 0.056, 0.031, 0.005, ...]
- **α=10:** denominator = 185+100=285. Probs: [110/285, 60/285, 30/285, 20/285, 15/285, 10/285, ...] = [0.386, 0.211, 0.105, 0.070, 0.053, 0.035, ...]
- **α=10 is more uniform** — range goes from [0.518, 0.005] to [0.386, 0.035], i.e., the max dropped and min increased
- **As α→∞:** every P → (0+α)/(0+10α) = 1/10 = **0.10** (perfectly uniform). The data is completely overwhelmed by the prior, and all words become equally probable.

---

### Q5: Practical Debugging
**Q:** Your Multinomial NB classifier achieves 95% accuracy on the test set but returns P(class|x) = 0.9999 or 0.0001 for every prediction (never moderate probabilities). What causes this and how do you fix it?

**A:** This is caused by **overconfident probabilities**, a known Naive Bayes issue from:
1. **Multiplying many (correlated) feature probabilities** — the product amplifies the signal, pushing posteriors toward 0 or 1
2. **Small alpha** — insufficient smoothing lets rare features dominate
3. **Naive independence assumption** — treats correlated features as independent, effectively double-counting evidence

**Fixes:**
- **Calibrate:** Use `CalibratedClassifierCV(nb, method='isotonic', cv=5)` to post-hoc calibrate
- **Increase alpha:** More smoothing moderates extreme probabilities
- **Reduce features:** Use `max_features` to limit vocabulary, reducing redundant features
- **Use `predict_log_proba()`** for ranking instead of probabilities
- Note: classification **accuracy** can still be excellent despite poor calibration

---

### Q6: Bernoulli vs Multinomial Smoothing
**Q:** Explain why BernoulliNB uses (count + α)/(Nc + 2α) while MultinomialNB uses (count + α)/(total + α·|V|). Why the difference in denominators?

**A:** The difference comes from the **number of possible outcomes per feature**:
- **BernoulliNB:** Each feature is binary (present/absent), so there are **2 outcomes**. Adding α pseudo-counts to each outcome means adding 2α total pseudo-observations. Denominator: Nc + 2α.
- **MultinomialNB:** Each "trial" (word position) can produce any word from the vocabulary, so there are **|V| outcomes**. Adding α pseudo-counts to each word means adding α·|V| total pseudo-words. Denominator: total_words + α·|V|.

The general rule is: denominator adds α × (number of possible outcomes for that feature type). Bernoulli has 2 outcomes per feature, Multinomial has |V| outcomes per word position.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 6: Text Classification](06-text-classification.md) | **Chapter 7: Laplace Smoothing** | — |

### Unit 10 Chapters
1. [Bayes' Theorem Review](01-bayes-theorem-review.md)
2. [The Naive Assumption](02-naive-assumption.md)
3. [Gaussian Naive Bayes](03-gaussian-naive-bayes.md)
4. [Multinomial Naive Bayes](04-multinomial-naive-bayes.md)
5. [Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md)
6. [Text Classification with Naive Bayes](06-text-classification.md)
7. **📍 Laplace Smoothing & Practical Considerations** ← You are here

---

> *"Smoothing is the art of gracefully admitting that we haven't seen everything — and that what we haven't seen might still happen."*

© 2025 AI/ML Study Notes. All rights reserved.
