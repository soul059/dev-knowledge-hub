# 📘 Chapter 1: Bayes' Theorem Review

> **Unit 10: Naive Bayes** | **Section 1 of 7**
> A comprehensive review of Bayes' theorem — the probabilistic foundation of Naive Bayes classification. Master the concepts of prior, likelihood, posterior, and evidence through a medical diagnosis worked example and see how Bayesian thinking drives classification.

---

## 📑 Table of Contents

1. [Introduction: Why Bayes?](#1-introduction-why-bayes)
2. [Probability Refresher](#2-probability-refresher)
3. [Bayes' Theorem: The Formula](#3-bayes-theorem-the-formula)
4. [Components of Bayes' Theorem](#4-components-of-bayes-theorem)
5. [Worked Example: Medical Diagnosis](#5-worked-example-medical-diagnosis)
6. [Posterior Updating: Sequential Evidence](#6-posterior-updating-sequential-evidence)
7. [Bayesian Thinking for Classification](#7-bayesian-thinking-for-classification)
8. [Step-by-Step Classification Calculation](#8-step-by-step-classification-calculation)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Introduction: Why Bayes?

In machine learning, classification is fundamentally about answering: **"Given what I observe (features), what class does this belong to?"**

This is exactly what Bayes' theorem answers:

```
    Given observed data X, what is the probability of each class C?

              P(X | C) · P(C)
    P(C|X) = ─────────────────
                   P(X)

    "What is the probability of class C given that I see data X?"
```

### Why Bayes Matters for ML

```
    ┌─────────────────────────────────────────────────────────┐
    │              THE BAYESIAN CLASSIFICATION PIPELINE       │
    │                                                         │
    │   Prior Knowledge    +    Observed Data    =  Decision  │
    │      P(class)             P(data|class)       P(C|X)   │
    │                                                         │
    │   "How common is      "How likely is       "What class  │
    │    each class?"        this data for        is this?"   │
    │                        each class?"                     │
    └─────────────────────────────────────────────────────────┘
```

Bayes' theorem provides:
- A **principled framework** for combining prior knowledge with evidence
- A way to **update beliefs** as new data arrives
- The **mathematical foundation** for the entire Naive Bayes family of classifiers

---

## 2. Probability Refresher

### 2.1 Basic Probability Notation

| Symbol | Meaning | Example |
|--------|---------|---------|
| P(A) | Probability of event A | P(Spam) = 0.3 |
| P(A, B) | Joint probability of A and B | P(Spam, "free") = 0.2 |
| P(A\|B) | Conditional: probability of A given B | P(Spam\|"free") = 0.8 |
| P(A') or P(¬A) | Probability of NOT A | P(Not Spam) = 0.7 |

### 2.2 Fundamental Rules

**Sum Rule (Total Probability):**
```
    P(A) + P(¬A) = 1

    If A has k mutually exclusive outcomes:
    P(A₁) + P(A₂) + ... + P(Aₖ) = 1
```

**Product Rule (Joint Probability):**
```
    P(A, B) = P(A|B) · P(B)
            = P(B|A) · P(A)
```

**Law of Total Probability:**
```
    P(B) = Σ P(B|Aᵢ) · P(Aᵢ)    for all mutually exclusive Aᵢ
         = P(B|A) · P(A) + P(B|¬A) · P(¬A)
```

### 2.3 Conditional Probability

```
    Definition:
                   P(A ∩ B)     P(A, B)
    P(A | B)  =  ──────────  = ────────
                    P(B)          P(B)

    "Out of all the times B happens, how often does A also happen?"
```

**Visual: Conditional Probability as Filtering**

```
    ┌──────────────────────────────────────────┐
    │              All Emails (Ω)              │
    │                                          │
    │   ┌──────────────────────────┐           │
    │   │    Contains "free" (B)   │           │
    │   │                          │           │
    │   │   ┌──────────────┐       │           │
    │   │   │  Spam ∩ Free │       │           │
    │   │   │    (A ∩ B)   │       │           │
    │   │   └──────────────┘       │           │
    │   │                          │           │
    │   └──────────────────────────┘           │
    │                                          │
    └──────────────────────────────────────────┘

    P(Spam | "free") = P(Spam ∩ "free") / P("free")
                     = (Spam emails containing "free") / (All emails containing "free")
```

---

## 3. Bayes' Theorem: The Formula

### 3.1 Derivation

Starting from the product rule applied two ways:

```
    P(A, B) = P(A|B) · P(B)     ... (1)
    P(A, B) = P(B|A) · P(A)     ... (2)

    Setting (1) = (2):

    P(A|B) · P(B) = P(B|A) · P(A)

    Solving for P(A|B):

                    P(B|A) · P(A)
    P(A|B)  =  ─────────────────────
                       P(B)
```

### 3.2 Bayes' Theorem — The Full Formula

```
    ╔═══════════════════════════════════════════════════╗
    ║                                                   ║
    ║                P(B | A) · P(A)                    ║
    ║   P(A | B) = ─────────────────                    ║
    ║                    P(B)                            ║
    ║                                                   ║
    ║   Where:                                          ║
    ║     P(A|B) = Posterior   (what we want)           ║
    ║     P(B|A) = Likelihood  (how well data fits)     ║
    ║     P(A)   = Prior       (initial belief)         ║
    ║     P(B)   = Evidence    (normalizing constant)   ║
    ║                                                   ║
    ╚═══════════════════════════════════════════════════╝
```

### 3.3 The Proportionality Form

Since P(B) is the same for all classes, we often write:

```
    P(A | B) ∝ P(B | A) · P(A)

    Posterior ∝ Likelihood × Prior

    "The posterior is proportional to likelihood times prior"
```

This is sufficient for classification — we just pick the class with the highest value.

---

## 4. Components of Bayes' Theorem

### 4.1 Prior — P(A)

> **Definition:** The probability of class A *before* seeing any evidence.

```
    Prior = Your initial belief about the class distribution

    Examples:
    ┌────────────────────────────────────────────────┐
    │ Problem              │ Prior                   │
    ├────────────────────────────────────────────────┤
    │ Spam detection       │ P(Spam) = 0.30          │
    │ Disease diagnosis    │ P(Disease) = 0.001      │
    │ Sentiment analysis   │ P(Positive) = 0.50      │
    │ Fraud detection      │ P(Fraud) = 0.01         │
    └────────────────────────────────────────────────┘
```

**Estimation:** From training data:
```
    P(class = c) = (Number of samples in class c) / (Total number of samples)
```

### 4.2 Likelihood — P(B|A)

> **Definition:** The probability of observing data B *given* that the class is A.

```
    Likelihood = How well the observed data fits each class

    Examples:
    ┌──────────────────────────────────────────────────────────┐
    │ Problem           │ Likelihood                          │
    ├──────────────────────────────────────────────────────────┤
    │ Spam detection    │ P("free" appears | Spam) = 0.60     │
    │ Disease diagnosis │ P(Positive test | Disease) = 0.99   │
    │ Face recognition  │ P(pixel pattern | Person A) = 0.85  │
    └──────────────────────────────────────────────────────────┘
```

**Key insight:** Likelihood is NOT the probability of the class. It's the probability of the *data* assuming a specific class.

### 4.3 Posterior — P(A|B)

> **Definition:** The updated probability of class A *after* seeing evidence B.

```
    Posterior = Updated belief after seeing evidence

    Prior + Evidence → Posterior
    (before)   (data)   (after)
```

### 4.4 Evidence — P(B)

> **Definition:** The total probability of observing data B across all classes.

```
    P(B) = Σ P(B|Cᵢ) · P(Cᵢ)     over all classes Cᵢ

    Evidence = Normalizing constant that ensures probabilities sum to 1
```

**The Relationship Visualized:**

```
    ┌─────────┐    ┌────────────┐    ┌───────────┐
    │  PRIOR  │    │ LIKELIHOOD │    │ POSTERIOR  │
    │  P(C)   │ ×  │  P(X|C)   │ ÷  │  P(C|X)   │
    │         │    │            │    │           │
    │ Initial │    │ How well   │    │ Updated   │
    │ belief  │    │ data fits  │    │ belief    │
    │ about   │    │ under this │    │ after     │
    │ classes │    │ class      │    │ seeing    │
    │         │    │            │    │ data      │
    └────┬────┘    └─────┬──────┘    └───────────┘
         │               │               ▲
         └───────┬───────┘               │
                 │          ┌─────────┐  │
                 └──── ÷ ───│EVIDENCE │──┘
                            │  P(X)   │
                            │ (norm.) │
                            └─────────┘
```

---

## 5. Worked Example: Medical Diagnosis

### Problem Setup

A rare disease affects **1 in 1,000** people. A diagnostic test has:
- **Sensitivity** (true positive rate): 99% — P(Test+ | Disease) = 0.99
- **Specificity** (true negative rate): 95% — P(Test− | No Disease) = 0.95

**Question:** If a person tests positive, what is the probability they have the disease?

### Step 1: Define the Terms

```
    Event D = person has the disease
    Event T⁺ = test result is positive

    Given:
        P(D)    = 0.001          (prior — disease prevalence)
        P(¬D)   = 0.999          (prior — no disease)
        P(T⁺|D) = 0.99           (likelihood — sensitivity)
        P(T⁺|¬D)= 0.05           (false positive rate = 1 - specificity)
```

### Step 2: Compute the Evidence P(T⁺)

Using the law of total probability:

```
    P(T⁺) = P(T⁺|D) · P(D) + P(T⁺|¬D) · P(¬D)
           = 0.99 × 0.001  +  0.05 × 0.999
           = 0.00099  +  0.04995
           = 0.05094
```

### Step 3: Apply Bayes' Theorem

```
                   P(T⁺|D) · P(D)
    P(D | T⁺) = ───────────────────
                      P(T⁺)

                   0.99 × 0.001
               = ─────────────────
                     0.05094

                   0.00099
               = ───────────
                   0.05094

               ≈  0.01943  ≈  1.94%
```

### Step 4: Interpret the Result

```
    ╔════════════════════════════════════════════════════════════╗
    ║  SURPRISING RESULT:                                       ║
    ║                                                           ║
    ║  Despite a 99% accurate test, a positive result only      ║
    ║  means a 1.94% chance of actually having the disease!     ║
    ║                                                           ║
    ║  Why? Because the disease is so rare (prior = 0.001)      ║
    ║  that most positive results are FALSE POSITIVES.          ║
    ╚════════════════════════════════════════════════════════════╝
```

### Visual: Where Do the Positives Come From?

```
    Out of 100,000 people tested:
    ┌─────────────────────────────────────────────────────┐
    │                                                     │
    │  100 have disease              99,900 don't         │
    │  ┌──────────────┐              ┌──────────────┐     │
    │  │ Test+: 99    │              │ Test+: 4,995 │     │
    │  │ Test−: 1     │              │ Test−: 94,905│     │
    │  └──────────────┘              └──────────────┘     │
    │                                                     │
    │  Total positive tests: 99 + 4,995 = 5,094          │
    │  True positives among them: 99 / 5,094 = 1.94%     │
    │                                                     │
    └─────────────────────────────────────────────────────┘
```

### The Base Rate Fallacy

This example illustrates why **priors matter enormously**:

| Disease Prevalence (Prior) | P(Disease \| Test+) |
|----------------------------|---------------------|
| 1 in 1,000,000 (0.0001%) | 0.002% |
| 1 in 10,000 (0.01%) | 0.20% |
| 1 in 1,000 (0.1%) | **1.94%** |
| 1 in 100 (1%) | 16.7% |
| 1 in 10 (10%) | 68.8% |
| 1 in 2 (50%) | 95.2% |

---

## 6. Posterior Updating: Sequential Evidence

### The Power of Bayesian Updating

Bayes' theorem allows **sequential updating** — each new piece of evidence refines our belief:

```
    Round 1: Prior₁ ──[Evidence₁]──→ Posterior₁
    Round 2: Posterior₁ becomes Prior₂ ──[Evidence₂]──→ Posterior₂
    Round 3: Posterior₂ becomes Prior₃ ──[Evidence₃]──→ Posterior₃
    ...
```

### Worked Example: Second Test

Continuing the medical example — the person takes a **second independent test** and it's also positive.

```
    Now our prior for round 2 is the posterior from round 1:
        P(D) = 0.01943 (updated from 0.001)

    Apply Bayes again with the same test characteristics:

    P(T₂⁺) = P(T₂⁺|D) · P(D) + P(T₂⁺|¬D) · P(¬D)
            = 0.99 × 0.01943  +  0.05 × 0.98057
            = 0.01923  +  0.04903
            = 0.06826

                     0.99 × 0.01943
    P(D | T₂⁺) = ─────────────────── = 0.01923 / 0.06826 ≈ 0.2818
                       0.06826

    After TWO positive tests: P(Disease) ≈ 28.2%
```

### Tracking the Belief Update

```
    Probability of Disease
    1.0 ┤
        │
    0.8 ┤
        │
    0.6 ┤
        │
    0.4 ┤
        │                                          ● After 3rd test
    0.3 ┤                            ● After 2nd   (≈ 87.8%)
        │                             test (28.2%)
    0.2 ┤
        │
    0.1 ┤
        │         ● After 1st test (1.94%)
    0.0 ┤● Prior (0.1%)
        └──────┬────────┬────────┬────────┬─────
              Prior   Test 1   Test 2   Test 3

    Each positive test dramatically increases confidence!
```

### Key Insight: Evidence Accumulation

```
    ┌───────────────────────────────────────────────────────┐
    │  With enough evidence, the prior becomes irrelevant.  │
    │                                                       │
    │  Strong evidence eventually overwhelms any prior.     │
    │  Weak evidence requires more iterations to converge.  │
    └───────────────────────────────────────────────────────┘
```

---

## 7. Bayesian Thinking for Classification

### 7.1 From Bayes' Theorem to Classification

For a classification problem with classes C₁, C₂, ..., Cₖ and feature vector **x**:

```
    For each class Cₖ, compute:

                      P(x | Cₖ) · P(Cₖ)
    P(Cₖ | x) = ──────────────────────────
                          P(x)

    Classify as: ŷ = argmax  P(Cₖ | x)
                       k

    Since P(x) is constant across classes:

    ŷ = argmax  P(x | Cₖ) · P(Cₖ)
          k
```

### 7.2 The Classification Framework

```
    ┌─────────────────────────────────────────────────────────────┐
    │                   BAYESIAN CLASSIFIER                       │
    │                                                             │
    │  Input: Feature vector x = (x₁, x₂, ..., xₙ)             │
    │                                                             │
    │  For each class c ∈ {C₁, C₂, ..., Cₖ}:                    │
    │                                                             │
    │      score(c) = P(x₁, x₂, ..., xₙ | c) · P(c)            │
    │                  └──────────┬──────────┘   └─┬─┘           │
    │                    Likelihood              Prior            │
    │                    (complex!)            (easy!)            │
    │                                                             │
    │  Output: ŷ = class with highest score                      │
    │                                                             │
    │  Challenge: Computing P(x₁, x₂, ..., xₙ | c)              │
    │  requires estimating joint distribution over ALL features!  │
    │  → This is where the "Naive" assumption helps (Chapter 2)  │
    └─────────────────────────────────────────────────────────────┘
```

### 7.3 Generative vs Discriminative

Bayes' theorem leads to a **generative** classifier:

```
    ┌──────────────────────────────┬────────────────────────────────┐
    │    GENERATIVE CLASSIFIERS    │   DISCRIMINATIVE CLASSIFIERS   │
    ├──────────────────────────────┼────────────────────────────────┤
    │ Model P(x|C) and P(C)       │ Model P(C|x) directly         │
    │ Then use Bayes to get P(C|x)│ No need for Bayes' theorem    │
    │                              │                                │
    │ Examples:                    │ Examples:                      │
    │ • Naive Bayes                │ • Logistic Regression          │
    │ • Gaussian Mixture Models    │ • SVM                          │
    │ • Hidden Markov Models       │ • Neural Networks              │
    │                              │ • Decision Trees               │
    │ Can generate new samples     │ Cannot generate samples        │
    │ Works with missing features  │ Needs all features present     │
    │ Needs fewer training samples │ Often higher accuracy          │
    └──────────────────────────────┴────────────────────────────────┘
```

---

## 8. Step-by-Step Classification Calculation

### Problem: Fruit Classification

Classify a fruit as **Apple** or **Orange** based on two features:
- x₁ = Color (Red=1, Orange=0)
- x₂ = Size (Small=1, Large=0)

**Training Data:**

| Fruit | Color | Size | Count |
|-------|-------|------|-------|
| Apple | Red | Small | 3 |
| Apple | Red | Large | 2 |
| Apple | Orange | Small | 1 |
| Apple | Orange | Large | 0 |
| Orange | Red | Small | 0 |
| Orange | Red | Large | 1 |
| Orange | Orange | Small | 1 |
| Orange | Orange | Large | 2 |

**Classify: x = (Red, Large) — Color=1, Size=0**

### Step 1: Compute Priors

```
    Total: 10 samples (6 Apple, 4 Orange)

    P(Apple)  = 6/10 = 0.60
    P(Orange) = 4/10 = 0.40
```

### Step 2: Compute Likelihoods

```
    P(Red, Large | Apple) = P(Color=Red, Size=Large | Apple)
                          = Count(Apple, Red, Large) / Count(Apple)
                          = 2 / 6 = 0.3333

    P(Red, Large | Orange) = P(Color=Red, Size=Large | Orange)
                           = Count(Orange, Red, Large) / Count(Orange)
                           = 1 / 4 = 0.2500
```

### Step 3: Compute Unnormalized Posteriors

```
    score(Apple)  = P(Red, Large | Apple) × P(Apple)
                  = 0.3333 × 0.60 = 0.2000

    score(Orange) = P(Red, Large | Orange) × P(Orange)
                  = 0.2500 × 0.40 = 0.1000
```

### Step 4: Normalize (Optional — for Probabilities)

```
    P(x) = 0.2000 + 0.1000 = 0.3000

    P(Apple | Red, Large) = 0.2000 / 0.3000 = 0.6667 (66.7%)
    P(Orange | Red, Large) = 0.1000 / 0.3000 = 0.3333 (33.3%)
```

### Step 5: Classify

```
    ŷ = argmax { P(Apple|x), P(Orange|x) }
      = argmax { 0.6667, 0.3333 }
      = Apple ✓

    Confidence: 66.7%
```

---

## 9. Python Implementation

### 9.1 Bayes' Theorem from Scratch

```python
def bayes_theorem(prior, likelihood, evidence):
    """
    Compute posterior probability using Bayes' theorem.
    
    P(A|B) = P(B|A) * P(A) / P(B)
    """
    posterior = (likelihood * prior) / evidence
    return posterior


def compute_evidence(likelihoods, priors):
    """
    Compute evidence (marginal likelihood) using law of total probability.
    
    P(B) = Σ P(B|Aᵢ) · P(Aᵢ)
    """
    return sum(l * p for l, p in zip(likelihoods, priors))


# ─── Medical Diagnosis Example ─────────────────────────────
prior_disease = 0.001
prior_healthy = 0.999

likelihood_positive_given_disease = 0.99   # sensitivity
likelihood_positive_given_healthy = 0.05   # false positive rate

# Evidence: P(Test+)
evidence = compute_evidence(
    likelihoods=[likelihood_positive_given_disease, likelihood_positive_given_healthy],
    priors=[prior_disease, prior_healthy]
)

# Posterior: P(Disease | Test+)
posterior_disease = bayes_theorem(
    prior=prior_disease,
    likelihood=likelihood_positive_given_disease,
    evidence=evidence
)

print(f"P(Test+)           = {evidence:.5f}")
print(f"P(Disease | Test+) = {posterior_disease:.5f} ({posterior_disease*100:.2f}%)")
print(f"P(Healthy | Test+) = {1 - posterior_disease:.5f} ({(1-posterior_disease)*100:.2f}%)")
```

**Output:**
```
P(Test+)           = 0.05094
P(Disease | Test+) = 0.01943 (1.94%)
P(Healthy | Test+) = 0.98057 (98.06%)
```

### 9.2 Sequential Bayesian Updating

```python
def bayesian_update(prior, likelihood_positive, likelihood_negative_class):
    """
    Perform one round of Bayesian updating.
    
    Returns updated posterior after observing positive evidence.
    """
    evidence = likelihood_positive * prior + likelihood_negative_class * (1 - prior)
    posterior = (likelihood_positive * prior) / evidence
    return posterior


# Sequential testing — multiple positive test results
prior = 0.001
sensitivity = 0.99
false_positive_rate = 0.05

print("Sequential Bayesian Updating (all tests positive):")
print(f"{'Round':<8} {'Prior':>10} {'Posterior':>12}")
print("-" * 32)

for i in range(1, 7):
    posterior = bayesian_update(prior, sensitivity, false_positive_rate)
    print(f"Test {i:<3} {prior:>10.6f} {posterior:>12.6f}")
    prior = posterior  # posterior becomes next prior

print(f"\nAfter 6 positive tests: {posterior*100:.2f}% chance of disease")
```

**Output:**
```
Sequential Bayesian Updating (all tests positive):
Round       Prior     Posterior
--------------------------------
Test 1    0.001000     0.019434
Test 2    0.019434     0.281815
Test 3    0.281815     0.886121
Test 4    0.886121     0.993434
Test 5    0.993434     0.999670
Test 6    0.999670     0.999983

After 6 positive tests: 99.998% chance of disease
```

### 9.3 Bayesian Classifier (Full Example)

```python
import numpy as np
from collections import Counter

class SimpleBayesClassifier:
    """Simple Bayesian classifier using exact joint probabilities."""
    
    def fit(self, X, y):
        self.classes = np.unique(y)
        self.n_samples = len(y)
        
        # Compute priors
        class_counts = Counter(y)
        self.priors = {c: count / self.n_samples 
                       for c, count in class_counts.items()}
        
        # Store training data grouped by class
        self.class_data = {}
        for c in self.classes:
            mask = (y == c)
            self.class_data[c] = X[mask]
        
        return self
    
    def predict(self, X):
        return np.array([self._classify(x) for x in X])
    
    def _classify(self, x):
        scores = {}
        for c in self.classes:
            likelihood = self._compute_likelihood(x, c)
            scores[c] = likelihood * self.priors[c]
        return max(scores, key=scores.get)
    
    def _compute_likelihood(self, x, c):
        """Count exact matches in class data."""
        class_samples = self.class_data[c]
        n_class = len(class_samples)
        if n_class == 0:
            return 0
        matches = np.sum(np.all(class_samples == x, axis=1))
        return (matches + 1) / (n_class + len(self.classes))  # with smoothing


# ─── Fruit Classification Example ──────────────────────────
X_train = np.array([
    [1, 1], [1, 1], [1, 1],  # Apple, Red, Small (×3)
    [1, 0], [1, 0],          # Apple, Red, Large (×2)
    [0, 1],                   # Apple, Orange, Small (×1)
    [1, 0],                   # Orange, Red, Large (×1)
    [0, 1],                   # Orange, Orange, Small (×1)
    [0, 0], [0, 0],          # Orange, Orange, Large (×2)
])
y_train = np.array([
    'Apple', 'Apple', 'Apple',
    'Apple', 'Apple',
    'Apple',
    'Orange',
    'Orange',
    'Orange', 'Orange'
])

clf = SimpleBayesClassifier()
clf.fit(X_train, y_train)

# Classify: Red, Large → [1, 0]
test = np.array([[1, 0]])
prediction = clf.predict(test)
print(f"Red, Large fruit → Predicted: {prediction[0]}")
print(f"Priors: {clf.priors}")
```

---

## 10. Summary Table

| Concept | Formula / Value | Key Insight |
|---------|----------------|-------------|
| **Bayes' Theorem** | P(A\|B) = P(B\|A)·P(A) / P(B) | Inverts conditional probability |
| **Prior P(A)** | Estimated from class frequencies | Initial belief before evidence |
| **Likelihood P(B\|A)** | P(data \| class) | How well data fits each class |
| **Posterior P(A\|B)** | Updated belief | What we want for classification |
| **Evidence P(B)** | Σ P(B\|Cᵢ)·P(Cᵢ) | Normalizing constant |
| **Proportionality** | P(C\|x) ∝ P(x\|C)·P(C) | Skip evidence for argmax |
| **Posterior updating** | Posterior₁ → Prior₂ | Sequential evidence integration |
| **Base rate fallacy** | Ignoring P(A) leads to errors | Priors matter enormously |
| **Generative model** | Models P(x\|C) and P(C) | Naive Bayes is generative |
| **Classification rule** | ŷ = argmax P(x\|C)·P(C) | Pick class with highest score |

---

## 11. Revision Questions

### Q1: Apply Bayes' Theorem
**Q:** A factory has two machines. Machine A produces 60% of items and has a 2% defect rate. Machine B produces 40% of items and has a 5% defect rate. If an item is found to be defective, what is the probability it came from Machine A?

**A:**
- P(A) = 0.60, P(B) = 0.40
- P(Defect|A) = 0.02, P(Defect|B) = 0.05
- P(Defect) = 0.02 × 0.60 + 0.05 × 0.40 = 0.012 + 0.020 = 0.032
- P(A|Defect) = (0.02 × 0.60) / 0.032 = 0.012 / 0.032 = **0.375 (37.5%)**
- Despite Machine A producing most items, its lower defect rate means a defective item is more likely from Machine B (62.5%).

---

### Q2: The Role of the Prior
**Q:** Why does a 99% accurate disease test yield only ~2% posterior probability when the disease prevalence is 0.1%? What prevalence would make the posterior exceed 50%?

**A:** With prevalence 0.1% and false positive rate 5%, the vast majority of the population (99.9%) doesn't have the disease, but 5% of them still test positive. This flood of false positives (~4,995 per 100,000) dwarfs the true positives (~99). For posterior > 50%:
- P(D|T⁺) > 0.5 requires P(D) > 0.05/(0.05 + 0.99) ≈ 0.0481 or **≈4.8% prevalence**.

---

### Q3: Posterior Updating
**Q:** Starting with a prior P(Spam) = 0.3 and observing the word "free" (P("free"|Spam) = 0.6, P("free"|Ham) = 0.1), compute the posterior. Then update again upon observing "click" (P("click"|Spam) = 0.4, P("click"|Ham) = 0.05).

**A:**
- **After "free":** P(Spam|"free") = (0.6 × 0.3) / (0.6 × 0.3 + 0.1 × 0.7) = 0.18 / 0.25 = **0.72**
- **After "click":** Prior is now 0.72. P(Spam|"click") = (0.4 × 0.72) / (0.4 × 0.72 + 0.05 × 0.28) = 0.288 / 0.302 = **0.9536 (95.4%)**

---

### Q4: Evidence Computation
**Q:** In a 3-class problem with P(C₁)=0.5, P(C₂)=0.3, P(C₃)=0.2 and likelihoods P(x|C₁)=0.1, P(x|C₂)=0.4, P(x|C₃)=0.3, compute all three posterior probabilities and verify they sum to 1.

**A:**
- P(x) = 0.1×0.5 + 0.4×0.3 + 0.3×0.2 = 0.05 + 0.12 + 0.06 = **0.23**
- P(C₁|x) = 0.05/0.23 = **0.2174**
- P(C₂|x) = 0.12/0.23 = **0.5217**
- P(C₃|x) = 0.06/0.23 = **0.2609**
- Sum: 0.2174 + 0.5217 + 0.2609 = **1.0** ✓
- Classification: **C₂** (highest posterior despite not highest prior)

---

### Q5: Generative vs Discriminative
**Q:** Explain why Naive Bayes is called a "generative" classifier. What advantage does this give over discriminative classifiers like logistic regression?

**A:** Naive Bayes is generative because it models the **data generation process**: it estimates P(x|C) (how data is generated for each class) and P(C) (how frequently each class occurs), then uses Bayes' theorem to derive P(C|x). Advantages: (1) Can **generate synthetic samples** by sampling from P(x|C); (2) Works well with **small training sets** since it estimates simpler distributions; (3) Handles **missing features** naturally by simply omitting them from the likelihood product; (4) Easily incorporates **prior knowledge** via the prior term.

---

### Q6: Why Not Exact Bayes?
**Q:** For a dataset with 20 binary features and 3 classes, how many parameters would an exact (non-naive) Bayesian classifier need to estimate the class-conditional distributions? Why is this problematic?

**A:** With 20 binary features, there are 2²⁰ = **1,048,576** possible feature combinations. For each class, we need a probability for each combination (minus 1 for the constraint they sum to 1), so: 3 × (2²⁰ - 1) = **3,145,725 parameters**. This is problematic because: (1) Most feature combinations will never appear in training data; (2) We'd need millions of training samples per class; (3) Severe overfitting is guaranteed. This **curse of dimensionality** is exactly why the naive independence assumption (Chapter 2) is necessary.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| — | **Chapter 1: Bayes' Theorem Review** | [Chapter 2: The Naive Assumption →](02-naive-assumption.md) |

### Unit 10 Chapters
1. **📍 Bayes' Theorem Review** ← You are here
2. [The Naive Assumption](02-naive-assumption.md)
3. [Gaussian Naive Bayes](03-gaussian-naive-bayes.md)
4. [Multinomial Naive Bayes](04-multinomial-naive-bayes.md)
5. [Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md)
6. [Text Classification with Naive Bayes](06-text-classification.md)
7. [Laplace Smoothing & Practical Considerations](07-laplace-smoothing.md)

---

> *"Bayes' theorem is not just a formula — it is a way of thinking about how evidence should change your beliefs."*

© 2025 AI/ML Study Notes. All rights reserved.
