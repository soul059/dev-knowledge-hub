# Chapter 6: Bayes' Theorem

> **Unit 3: Probability Theory** | **Topic**: Bayes' Rule, Prior, Likelihood, Posterior, Bayesian Updating

---

## 1. Overview

Bayes' theorem is arguably the single most important result in probability for machine learning. It
provides a principled framework for **updating beliefs** in light of new evidence. From spam filters
to medical diagnosis to the entire field of Bayesian ML, Bayes' theorem is the foundation.

---

## 2. Bayes' Theorem Formula

```
                    P(B|A) · P(A)
P(A|B) = ─────────────────────────
                    P(B)

Where:
  P(A|B)  = Posterior    — updated belief about A after observing B
  P(B|A)  = Likelihood   — probability of observing B if A is true
  P(A)    = Prior        — initial belief about A before seeing B
  P(B)    = Evidence     — total probability of observing B (normalizing constant)
```

### Expanded Form (using Law of Total Probability):

```
                    P(B|A) · P(A)
P(A|B) = ───────────────────────────────────
          P(B|A)·P(A) + P(B|A')·P(A')
```

---

## 3. Intuitive Explanation

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   PRIOR              EVIDENCE           POSTERIOR        │
│   "What I            "What I            "What I         │
│    believed           observed"          now believe"    │
│    before"                                              │
│                                                          │
│   P(A)        ×     P(B|A)        =    P(A|B)          │
│                    ─────────           (normalized)      │
│                      P(B)                               │
│                                                          │
│   Initial          How well the        Updated          │
│   belief           evidence fits       belief           │
│                    this hypothesis                      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**In plain English**: Start with what you believe (prior), see new data (likelihood), update
your belief (posterior). The evidence P(B) ensures probabilities sum to 1.

---

## 4. Step-by-Step Example 1: Medical Diagnosis

**Problem**: A disease affects 1% of the population. A test is 95% accurate for those with the
disease and 90% accurate for those without. If you test positive, what's the probability you
have the disease?

```
Step 1: Define
  D  = has disease       P(D)  = 0.01
  D' = no disease        P(D') = 0.99
  +  = test positive

  P(+|D)  = 0.95    (sensitivity)
  P(+|D') = 0.10    (false positive rate = 1 - specificity)

Step 2: Apply Bayes' Theorem
  P(D|+) = P(+|D) · P(D) / P(+)

Step 3: Compute P(+) using law of total probability
  P(+) = P(+|D)·P(D) + P(+|D')·P(D')
       = 0.95 × 0.01 + 0.10 × 0.99
       = 0.0095 + 0.0990
       = 0.1085

Step 4: Compute posterior
  P(D|+) = 0.0095 / 0.1085 = 0.0876 ≈ 8.8%

Answer: Only ~8.8% chance of having the disease despite a positive test!
```

### Why So Low?

```
Out of 10,000 people:
  ├── 100 have disease (1%)
  │     └── 95 test positive  (true positives)
  │
  └── 9,900 healthy (99%)
        └── 990 test positive (false positives!)

Total positive: 95 + 990 = 1,085
Truly sick among positive: 95 / 1,085 ≈ 8.8%

The huge number of false positives overwhelms the true positives!
```

---

## 5. Step-by-Step Example 2: Spam Filtering

**Problem**: 30% of emails are spam. A spam email contains the word "free" 80% of the time.
A legitimate email contains "free" 10% of the time. An email contains "free" — is it spam?

```
Step 1: Define
  S  = spam         P(S)  = 0.30
  S' = not spam     P(S') = 0.70
  F  = contains "free"

  P(F|S)  = 0.80
  P(F|S') = 0.10

Step 2: P(F) = P(F|S)·P(S) + P(F|S')·P(S')
             = 0.80 × 0.30 + 0.10 × 0.70
             = 0.24 + 0.07 = 0.31

Step 3: P(S|F) = P(F|S)·P(S) / P(F)
               = 0.24 / 0.31
               = 0.7742 ≈ 77.4%

Answer: 77.4% probability the email is spam.
```

---

## 6. Bayesian Updating

Bayes' theorem can be applied **sequentially** — the posterior from one observation becomes
the prior for the next:

```
Iteration 1: Prior₀ → observe data₁ → Posterior₁
Iteration 2: Prior₁ = Posterior₁ → observe data₂ → Posterior₂
Iteration 3: Prior₂ = Posterior₂ → observe data₃ → Posterior₃
...

Each new piece of evidence refines our belief.
```

### Example: Coin Fairness

Is a coin fair (p=0.5) or biased (p=0.8)?

```
Prior: P(fair) = P(biased) = 0.5

Flip 1: Heads
  P(H|fair)   = 0.5
  P(H|biased) = 0.8

  P(fair|H)   = (0.5 × 0.5) / (0.5×0.5 + 0.8×0.5)
               = 0.25 / 0.65 = 0.385

  P(biased|H) = 0.40 / 0.65 = 0.615

Flip 2: Heads again (using updated prior)
  P(fair|HH)   = (0.5 × 0.385) / (0.5×0.385 + 0.8×0.615)
                = 0.1925 / 0.6845 = 0.281

  P(biased|HH) = 0.4920 / 0.6845 = 0.719

Flip 3: Tails
  P(fair|HHT)   = (0.5 × 0.281) / (0.5×0.281 + 0.2×0.719)
                 = 0.1405 / 0.2843 = 0.494

  P(biased|HHT) = 0.1438 / 0.2843 = 0.506

After HHT: beliefs are nearly back to 50-50!
```

---

## 7. Bayes' Theorem for Continuous Variables

```
              f(x|θ) · π(θ)
π(θ|x) = ─────────────────────
               f(x)

Where:
  π(θ)    = prior distribution over parameter θ
  f(x|θ)  = likelihood of data x given θ
  π(θ|x)  = posterior distribution over θ given data
  f(x)    = ∫ f(x|θ)·π(θ) dθ  (marginal likelihood / evidence)
```

---

## 8. Connection to Bayesian Machine Learning

```
┌─────────────────────────────────────────────────────────────┐
│                 Bayesian ML Framework                       │
│                                                             │
│   P(θ|D) ∝ P(D|θ) · P(θ)                                  │
│                                                             │
│   Posterior    Likelihood   Prior                            │
│   over model  of data      belief about                     │
│   parameters  given model  parameters                       │
│                                                             │
│   Training = Computing/Approximating the posterior          │
│   Prediction = ∫ P(y|x,θ) · P(θ|D) dθ                     │
│                (average over all plausible parameters)      │
│                                                             │
│   Frequentist ML: Find single best θ (MLE/MAP)             │
│   Bayesian ML:    Use entire distribution P(θ|D)           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

| ML Method           | Uses Bayes' Theorem How?                                  |
|---------------------|-----------------------------------------------------------|
| Naive Bayes         | Classifies using P(C\|features) ∝ P(features\|C)·P(C)   |
| Bayesian regression | Full posterior over weights: P(w\|data)                   |
| Gaussian processes  | Prior over functions → posterior over functions            |
| Bayesian neural nets| Prior over weights → approximate posterior via VI          |
| Thompson sampling   | Posterior over reward → exploration-exploitation           |

---

## 9. Odds Form of Bayes' Theorem

```
Posterior odds = Likelihood ratio × Prior odds

P(A|B)     P(B|A)     P(A)
────── = ──────── × ──────
P(A'|B)    P(B|A')    P(A')
```

**Useful for**: Quick mental calculations and sequential updating.

### Example (Medical test from above):

```
Prior odds     = 0.01 / 0.99 = 1/99
Likelihood ratio = 0.95 / 0.10 = 9.5
Posterior odds = 9.5 × (1/99) = 9.5/99 ≈ 0.096

P(D|+) = 0.096 / (1 + 0.096) = 0.0876 ≈ 8.8%  ✓ (matches!)
```

---

## 10. Python Implementation

```python
def bayes_theorem(prior, likelihood, false_positive_rate):
    """Compute posterior probability using Bayes' theorem."""
    p_evidence = likelihood * prior + false_positive_rate * (1 - prior)
    posterior = (likelihood * prior) / p_evidence
    return posterior

# Medical diagnosis
posterior = bayes_theorem(prior=0.01, likelihood=0.95, false_positive_rate=0.10)
print(f"P(Disease | Positive Test) = {posterior:.4f}")  # 0.0876

# Spam filter
posterior = bayes_theorem(prior=0.30, likelihood=0.80, false_positive_rate=0.10)
print(f"P(Spam | 'free') = {posterior:.4f}")  # 0.7742
```

```python
import numpy as np

# Bayesian updating simulation
def bayesian_coin_updating(flips, p_fair_head=0.5, p_biased_head=0.8):
    prior_fair = 0.5
    history = [prior_fair]
    
    for flip in flips:
        if flip == 'H':
            lk_fair = p_fair_head
            lk_biased = p_biased_head
        else:
            lk_fair = 1 - p_fair_head
            lk_biased = 1 - p_biased_head
        
        evidence = lk_fair * prior_fair + lk_biased * (1 - prior_fair)
        prior_fair = (lk_fair * prior_fair) / evidence
        history.append(prior_fair)
    
    return history

flips = "HHHTHHTHHH"
history = bayesian_coin_updating(flips)
for i, p in enumerate(history):
    flip_str = flips[i-1] if i > 0 else '-'
    print(f"After flip {i} ({flip_str}): P(fair) = {p:.4f}")
```

---

## 11. Common Pitfalls

| Pitfall                    | Description                                             |
|----------------------------|---------------------------------------------------------|
| Base rate neglect          | Ignoring P(A) — confusing P(B\|A) with P(A\|B)        |
| Prosecutor's fallacy       | P(evidence\|innocent) ≠ P(innocent\|evidence)          |
| Ignoring false positives   | High test accuracy ≠ high posterior if disease is rare  |
| Flat prior assumption      | "No prior" is itself a prior (uniform)                  |

---

## 12. Summary Table

| Component   | Symbol   | Meaning                            | Example (medical)          |
|-------------|----------|------------------------------------|----------------------------|
| Prior       | P(A)     | Belief before evidence             | P(Disease) = 0.01          |
| Likelihood  | P(B\|A)  | Prob. of evidence if A is true     | P(+\|Disease) = 0.95       |
| Evidence    | P(B)     | Total prob. of evidence            | P(+) = 0.1085              |
| Posterior   | P(A\|B)  | Updated belief after evidence      | P(Disease\|+) = 0.088      |

---

## 13. Quick Revision Questions

1. **State Bayes' theorem** and label each component.
2. **In the medical example,** what happens to P(D\|+) if the disease prevalence increases to 10%?
3. **Why is P(B\|A) ≠ P(A\|B)?** Give a concrete example.
4. **Explain Bayesian updating** in 2-3 sentences with an example.
5. **How does Naive Bayes classifier use Bayes' theorem?**
6. **What is the "base rate fallacy"?** Why does it cause errors in interpreting test results?

---

## 14. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 5: Joint & Conditional Probability](05-joint-and-conditional-probability.md) | [Chapter 7: Common Distributions (Deep Dive)](07-common-distributions.md) |

---

*Unit 3 · Probability Theory · Chapter 6 of 9*
