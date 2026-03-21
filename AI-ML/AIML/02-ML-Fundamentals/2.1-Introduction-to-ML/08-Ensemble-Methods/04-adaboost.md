# Chapter 8.4: AdaBoost (Adaptive Boosting)

> **Unit 8: Ensemble Methods** — File 4 of 8

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [AdaBoost Algorithm — Step by Step](#2-adaboost-algorithm--step-by-step)
3. [Sample Weight Updating](#3-sample-weight-updating)
4. [Learner Weight Calculation (αₜ)](#4-learner-weight-calculation-αt)
5. [Weighted Voting](#5-weighted-voting)
6. [Mathematical Derivation](#6-mathematical-derivation)
7. [Complete Worked Example with Numbers](#7-complete-worked-example-with-numbers)
8. [Exponential Loss Function](#8-exponential-loss-function)
9. [AdaBoostClassifier in Scikit-Learn](#9-adaboostclassifier-in-scikit-learn)
10. [Sensitivity to Noise and Outliers](#10-sensitivity-to-noise-and-outliers)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Introduction

**AdaBoost** (Adaptive Boosting), introduced by Yoav Freund and Robert Schapire in 1995, was the first practical boosting algorithm. It won the prestigious Gödel Prize in 2003.

### Core Idea

AdaBoost **adaptively re-weights** training samples: misclassified samples get higher weights so that subsequent weak learners focus on the "hard" examples.

```
┌─────────────────────────────────────────────────────────┐
│                    AdaBoost in One Sentence:             │
│                                                         │
│  "Pay more attention to the examples you got wrong."    │
│                                                         │
│  Round 1: Equal weights → Train stump → Find errors     │
│  Round 2: ↑ weight on errors → Train stump → Find errors│
│  Round 3: ↑ weight on errors → Train stump → Find errors│
│  ...                                                    │
│  Final: Weighted vote of all stumps                     │
└─────────────────────────────────────────────────────────┘
```

---

## 2. AdaBoost Algorithm — Step by Step

### Algorithm: AdaBoost (Binary Classification, y ∈ {-1, +1})

```
INPUT:  Training data: {(x₁,y₁), ..., (xₙ,yₙ)},  yᵢ ∈ {-1, +1}
        Number of rounds: T
        Weak learner: H (e.g., Decision Stump)

STEP 0: Initialize sample weights
        wᵢ⁽¹⁾ = 1/n  for all i = 1, ..., n

FOR t = 1, 2, ..., T:

    STEP 1: Train weak learner hₜ on weighted data
            hₜ = H(D, w⁽ᵗ⁾)
            (find the stump that minimizes weighted error)

    STEP 2: Compute weighted error
                    n
            εₜ = Σ wᵢ⁽ᵗ⁾ · I(hₜ(xᵢ) ≠ yᵢ)
                   i=1

            (If εₜ > 0.5, flip the learner or stop)
            (If εₜ = 0, perfect — assign large αₜ)

    STEP 3: Compute learner weight
            αₜ = ½ · ln((1 - εₜ) / εₜ)

    STEP 4: Update sample weights
                                                    ⎧ wᵢ⁽ᵗ⁾ · e^(αₜ)    if hₜ(xᵢ) ≠ yᵢ  (wrong)
            w̃ᵢ⁽ᵗ⁺¹⁾ = wᵢ⁽ᵗ⁾ · exp(-αₜ · yᵢ · hₜ(xᵢ)) = ⎨
                                                    ⎩ wᵢ⁽ᵗ⁾ · e^(-αₜ)   if hₜ(xᵢ) = yᵢ   (correct)

    STEP 5: Normalize weights
            wᵢ⁽ᵗ⁺¹⁾ = w̃ᵢ⁽ᵗ⁺¹⁾ / Σⱼ w̃ⱼ⁽ᵗ⁺¹⁾

END FOR

OUTPUT: Final classifier
                    T
        H(x) = sign( Σ αₜ · hₜ(x) )
                   t=1
```

### Visual Flow

```
    ┌──────────────────────────────────────────────────┐
    │             Initialize: w = [1/n, ..., 1/n]      │
    └──────────────────────┬───────────────────────────┘
                           │
                           ▼
    ┌──────────────────────────────────────────────────┐
    │  Round t:                                        │
    │                                                  │
    │  1. Train weak learner hₜ with weights w⁽ᵗ⁾     │
    │                    │                             │
    │                    ▼                             │
    │  2. Compute error: εₜ = Σ wᵢ · I(hₜ(xᵢ)≠yᵢ)   │
    │                    │                             │
    │                    ▼                             │
    │  3. Learner weight: αₜ = ½ ln((1-εₜ)/εₜ)       │
    │                    │                             │
    │                    ▼                             │
    │  4. Update weights:                              │
    │     Misclassified: wᵢ × e^(αₜ)   ← INCREASE    │
    │     Correct:       wᵢ × e^(-αₜ)  ← DECREASE    │
    │                    │                             │
    │                    ▼                             │
    │  5. Normalize weights → w⁽ᵗ⁺¹⁾                  │
    └──────────────────────┬───────────────────────────┘
                           │
                           ▼ (repeat T times)
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │  Final: H(x) = sign(α₁h₁(x) + α₂h₂(x) + ...)  │
    │                                                  │
    └──────────────────────────────────────────────────┘
```

---

## 3. Sample Weight Updating

### Intuition Behind Weight Updates

```
Before Round t:
  Sample weights:  [0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1]
                    ○    ○    ○    ●    ●    ●    ●    ○    ○    ●

  Stump hₜ predicts: ○ for samples 1-5, ● for samples 6-10

  Correct:       ✓    ✓    ✓    ✗    ✗    ✓    ✓    ✗    ✗    ✓
                 ○    ○    ○    ●    ●    ●    ●    ○    ○    ●
  
  Misclassified: samples 4, 5, 8, 9  (4 out of 10 → εₜ = 0.4)

After weight update:
  Correct samples:      weight × e^(-αₜ) = 0.1 × 0.756 ≈ 0.076  ↓
  Misclassified samples: weight × e^(+αₜ) = 0.1 × 1.322 ≈ 0.132  ↑

After normalization:
  [0.076, 0.076, 0.076, 0.132, 0.132, 0.076, 0.076, 0.132, 0.132, 0.076]
   ──────────────────── ────────────── ──────────────────── ──────────────
   correct (decreased)  wrong (incr.)  correct (decreased)  wrong (incr.)
```

### The Weight Update Formula Decoded

```
wᵢ⁽ᵗ⁺¹⁾ ∝ wᵢ⁽ᵗ⁾ · exp(-αₜ · yᵢ · hₜ(xᵢ))

Since yᵢ, hₜ(xᵢ) ∈ {-1, +1}:

  If CORRECT:   yᵢ · hₜ(xᵢ) = +1  →  exp(-αₜ · 1) = e^(-αₜ) < 1   (weight decreases)
  If WRONG:     yᵢ · hₜ(xᵢ) = -1  →  exp(-αₜ ·-1) = e^(+αₜ) > 1   (weight increases)

The RATIO of increase to decrease:

  e^(+αₜ) / e^(-αₜ) = e^(2αₜ) = (1-εₜ)/εₜ

  Example: εₜ = 0.3 → ratio = 0.7/0.3 = 2.33
           Misclassified samples become 2.33× heavier than correct ones
```

---

## 4. Learner Weight Calculation (αₜ)

### The Formula

```
αₜ = ½ · ln((1 - εₜ) / εₜ)
```

### Behavior of αₜ as a Function of Error Rate

```
  αₜ
  3 │ *
    │
  2 │   *
    │
  1 │      *
    │          *
  0 │─────────────────*──── εₜ = 0.5 (random guessing → zero weight)
    │                    *
 -1 │                       *
    │                          *
 -2 │                             *
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──► εₜ
      0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8

  Key observations:
  • εₜ → 0:   αₜ → +∞  (perfect classifier gets infinite weight)
  • εₜ = 0.5: αₜ = 0    (random guessing gets zero weight)
  • εₜ > 0.5: αₜ < 0    (worse than random → negative weight, flip predictions)
```

### Numerical Examples

| Error Rate (εₜ) | (1-εₜ)/εₜ | αₜ = ½ ln(...) | Interpretation |
|------------------|------------|-----------------|----------------|
| 0.01 | 99.0 | 2.298 | Near-perfect → very high weight |
| 0.10 | 9.0 | 1.099 | Strong learner → high weight |
| 0.20 | 4.0 | 0.693 | Decent learner → moderate weight |
| 0.30 | 2.33 | 0.424 | Weak learner → lower weight |
| 0.40 | 1.50 | 0.203 | Barely useful → small weight |
| 0.50 | 1.00 | 0.000 | Random guessing → zero weight |

---

## 5. Weighted Voting

### How the Final Prediction Is Made

```
Final classifier: H(x) = sign(Σ αₜ · hₜ(x))

Example with T = 3 learners:

  h₁(x) = +1,  α₁ = 0.42
  h₂(x) = -1,  α₂ = 0.65
  h₃(x) = +1,  α₃ = 0.89

  Score = α₁·h₁ + α₂·h₂ + α₃·h₃
        = 0.42·(+1) + 0.65·(-1) + 0.89·(+1)
        = 0.42 - 0.65 + 0.89
        = 0.66

  H(x) = sign(0.66) = +1

Note: The learner with the lowest error (h₃, α₃ = 0.89)
      has the MOST influence on the final prediction.
```

### Comparison: Equal vs Weighted Voting

```
Equal Voting (Bagging):
  h₁ = +1,  h₂ = -1,  h₃ = +1  →  Majority: +1 (2 vs 1)

Weighted Voting (AdaBoost):
  α₁·(+1) + α₂·(-1) + α₃·(+1)
  
  Scenario A: α = [0.8, 0.1, 0.1]
    0.8(+1) + 0.1(-1) + 0.1(+1) = 0.8 → +1 (h₁ dominates)
  
  Scenario B: α = [0.1, 0.8, 0.1]
    0.1(+1) + 0.8(-1) + 0.1(+1) = -0.6 → -1 (h₂ dominates!)
    
  In weighted voting, a single highly accurate learner
  can override the majority of weaker learners.
```

---

## 6. Mathematical Derivation

### AdaBoost Minimizes Exponential Loss

AdaBoost can be shown to perform **forward stagewise additive modeling** with the **exponential loss function**:

```
L(y, F(x)) = exp(-y · F(x))
```

### Derivation of αₜ

At round t, we want to minimize:

```
                 n
L(αₜ, hₜ) = Σ wᵢ⁽ᵗ⁾ · exp(-αₜ · yᵢ · hₜ(xᵢ))
               i=1

Split into correct and incorrect predictions:

= Σ      wᵢ⁽ᵗ⁾ · e^(-αₜ)  +  Σ      wᵢ⁽ᵗ⁾ · e^(+αₜ)
  correct                     incorrect

= (1 - εₜ) · e^(-αₜ) + εₜ · e^(+αₜ)

Where εₜ = Σ wᵢ⁽ᵗ⁾ · I(hₜ(xᵢ) ≠ yᵢ)  (weighted error)

Take derivative w.r.t. αₜ and set to zero:

∂L/∂αₜ = -(1 - εₜ) · e^(-αₜ) + εₜ · e^(+αₜ) = 0

εₜ · e^(αₜ) = (1 - εₜ) · e^(-αₜ)

e^(2αₜ) = (1 - εₜ) / εₜ

2αₜ = ln((1 - εₜ) / εₜ)

┌─────────────────────────────────────┐
│  αₜ = ½ · ln((1 - εₜ) / εₜ)   ✓   │
└─────────────────────────────────────┘
```

### Derivation of Weight Update

The additive model after round t:

```
Fₜ(x) = Fₜ₋₁(x) + αₜ · hₜ(x)

The exponential loss for sample i after round t:

exp(-yᵢ · Fₜ(xᵢ)) = exp(-yᵢ · Fₜ₋₁(xᵢ)) · exp(-yᵢ · αₜ · hₜ(xᵢ))
                     = [old loss for sample i] · exp(-αₜ · yᵢ · hₜ(xᵢ))

This means the "weight" for sample i in round t+1 is proportional to:

wᵢ⁽ᵗ⁺¹⁾ ∝ wᵢ⁽ᵗ⁾ · exp(-αₜ · yᵢ · hₜ(xᵢ))

┌──────────────────────────────────────────────────┐
│  Correct:   wᵢ⁽ᵗ⁺¹⁾ ∝ wᵢ⁽ᵗ⁾ · e^(-αₜ)  ↓     │
│  Incorrect: wᵢ⁽ᵗ⁺¹⁾ ∝ wᵢ⁽ᵗ⁾ · e^(+αₜ)  ↑     │
└──────────────────────────────────────────────────┘
```

### Training Error Bound

```
Training error of AdaBoost after T rounds:

                T
error_train ≤ Π 2√(εₜ(1-εₜ))
              t=1

              T
           = Π √(1 - 4γₜ²)    where γₜ = ½ - εₜ (margin)
             t=1

           ≤ exp(-2 Σ γₜ²)
                    t=1

If each γₜ ≥ γ > 0:

error_train ≤ exp(-2Tγ²)  →  converges to 0 exponentially!
```

---

## 7. Complete Worked Example with Numbers

### Problem Setup

10 data points, binary classification (y ∈ {-1, +1}), 2 features.

```
┌─────┬────┬────┬────┐
│  i  │ x₁ │ x₂ │  y │
├─────┼────┼────┼────┤
│  1  │  1 │  2 │ +1 │
│  2  │  2 │  1 │ +1 │
│  3  │  3 │  3 │ +1 │
│  4  │  4 │  2 │ -1 │
│  5  │  5 │  4 │ -1 │
│  6  │  6 │  1 │ -1 │
│  7  │  7 │  3 │ -1 │
│  8  │  2 │  4 │ +1 │
│  9  │  5 │  1 │ +1 │
│ 10  │  8 │  2 │ -1 │
└─────┴────┴────┴────┘
```

### Round 1

**Step 0: Initialize weights**
```
w⁽¹⁾ = [0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1]
```

**Step 1: Train best decision stump**

Evaluate all possible stumps. Best stump: **x₁ ≤ 3.5**
```
h₁(x): if x₁ ≤ 3.5 → +1, else → -1

Predictions:  [+1, +1, +1, -1, -1, -1, -1, +1, -1, -1]
Actual:       [+1, +1, +1, -1, -1, -1, -1, +1, +1, -1]
                                           ✗        ✗
Correct:      [✓,  ✓,  ✓,  ✓,  ✓,  ✓,  ✓,  ✗,  ✗,  ✓]
```

**Step 2: Compute weighted error**
```
ε₁ = w₈ + w₉ = 0.1 + 0.1 = 0.2
```

**Step 3: Compute learner weight**
```
α₁ = ½ · ln((1-0.2)/0.2) = ½ · ln(4) = ½ · 1.386 = 0.693
```

**Step 4: Update sample weights**
```
For correct samples (i = 1,2,3,4,5,6,7,10):
  w̃ᵢ = 0.1 × e^(-0.693) = 0.1 × 0.500 = 0.050

For incorrect samples (i = 8, 9):
  w̃ᵢ = 0.1 × e^(+0.693) = 0.1 × 2.000 = 0.200

Unnormalized: [0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.20, 0.20, 0.05]
Sum = 8×0.05 + 2×0.20 = 0.40 + 0.40 = 0.80
```

**Step 5: Normalize**
```
w⁽²⁾ = [0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.25, 0.25, 0.0625]

Misclassified samples 8 and 9 now have 4× the weight of correct samples!
```

### Round 2

**Step 1: Train best stump on re-weighted data**

With the increased weights on samples 8 (x₁=2, y=+1) and 9 (x₁=5, y=+1), the algorithm needs to classify these correctly.

Best stump: **x₂ ≤ 1.5**
```
h₂(x): if x₂ ≤ 1.5 → -1, else → +1

Predictions:  [+1, -1, +1, +1, +1, -1, +1, +1, -1, +1]
Actual:       [+1, +1, +1, -1, -1, -1, -1, +1, +1, -1]
               ✓   ✗   ✓   ✗   ✗   ✓   ✗   ✓   ✗   ✗
```

**Step 2: Compute weighted error**
```
Misclassified: i = 2, 4, 5, 7, 9, 10
ε₂ = 0.0625 + 0.0625 + 0.0625 + 0.0625 + 0.25 + 0.0625 = 0.5625
```

Since ε₂ > 0.5, this stump is worse than random. Let's try another stump.

Best alternative stump: **x₁ ≤ 5.5 AND considering weights**

Let's try: **x₂ ≤ 2.5**
```
h₂(x): if x₂ ≤ 2.5 → -1, else → +1

Predictions:  [-1, -1, +1, -1, +1, -1, +1, +1, -1, -1]
Actual:       [+1, +1, +1, -1, -1, -1, -1, +1, +1, -1]
               ✗   ✗   ✓   ✓   ✗   ✓   ✗   ✓   ✗   ✓

Misclassified: i = 1, 2, 5, 7, 9
ε₂ = 0.0625 + 0.0625 + 0.0625 + 0.0625 + 0.25 = 0.5
```

Still at 0.5. Let's try: **x₁ ≤ 2.5**
```
h₂(x): if x₁ ≤ 2.5 → +1, else → -1

Predictions:  [+1, +1, +1, -1, -1, -1, -1, +1, -1, -1]

Wait — this is the same issue as h₁. Let's use a stump that correctly 
classifies the high-weight samples 8 and 9:

h₂(x): if x₁ ≤ 5.5 → +1, else → -1

Predictions:  [+1, +1, +1, +1, +1, -1, -1, +1, +1, -1]
Actual:       [+1, +1, +1, -1, -1, -1, -1, +1, +1, -1]
               ✓   ✓   ✓   ✗   ✗   ✓   ✓   ✓   ✓   ✓

Misclassified: i = 4, 5
ε₂ = 0.0625 + 0.0625 = 0.125
```

**Step 3: Compute learner weight**
```
α₂ = ½ · ln((1-0.125)/0.125) = ½ · ln(7) = ½ · 1.946 = 0.973
```

**Step 4 & 5: Update and normalize weights**
```
Correct (i=1,2,3,6,7,8,9,10):    w̃ᵢ = wᵢ × e^(-0.973) = wᵢ × 0.378
Incorrect (i=4,5):                w̃ᵢ = wᵢ × e^(+0.973) = wᵢ × 2.646

w̃₁ = 0.0625 × 0.378 = 0.0236     w̃₆ = 0.0625 × 0.378 = 0.0236
w̃₂ = 0.0625 × 0.378 = 0.0236     w̃₇ = 0.0625 × 0.378 = 0.0236
w̃₃ = 0.0625 × 0.378 = 0.0236     w̃₈ = 0.2500 × 0.378 = 0.0946
w̃₄ = 0.0625 × 2.646 = 0.1654     w̃₉ = 0.2500 × 0.378 = 0.0946
w̃₅ = 0.0625 × 2.646 = 0.1654     w̃₁₀= 0.0625 × 0.378 = 0.0236

Sum = 6×0.0236 + 2×0.1654 + 2×0.0946 = 0.1418 + 0.3308 + 0.1892 = 0.6618

Normalized w⁽³⁾:
w₁ = 0.0357, w₂ = 0.0357, w₃ = 0.0357
w₄ = 0.2500, w₅ = 0.2500  ← Previously misclassified, now HIGH weight
w₆ = 0.0357, w₇ = 0.0357
w₈ = 0.1429, w₉ = 0.1429  ← Still elevated from Round 1
w₁₀ = 0.0357
```

### Final Ensemble (after 2 rounds)

```
H(x) = sign(α₁·h₁(x) + α₂·h₂(x))
     = sign(0.693·h₁(x) + 0.973·h₂(x))

For each sample:
┌─────┬──────────┬──────────┬──────────────────────────┬───────┬────────┐
│  i  │ h₁(xᵢ)  │ h₂(xᵢ)  │ 0.693·h₁ + 0.973·h₂     │ H(xᵢ)│Correct?│
├─────┼──────────┼──────────┼──────────────────────────┼───────┼────────┤
│  1  │   +1     │   +1     │ +0.693 + 0.973 = +1.666  │  +1   │   ✓    │
│  2  │   +1     │   +1     │ +0.693 + 0.973 = +1.666  │  +1   │   ✓    │
│  3  │   +1     │   +1     │ +0.693 + 0.973 = +1.666  │  +1   │   ✓    │
│  4  │   -1     │   +1     │ -0.693 + 0.973 = +0.280  │  +1   │   ✗    │
│  5  │   -1     │   +1     │ -0.693 + 0.973 = +0.280  │  +1   │   ✗    │
│  6  │   -1     │   -1     │ -0.693 - 0.973 = -1.666  │  -1   │   ✓    │
│  7  │   -1     │   -1     │ -0.693 - 0.973 = -1.666  │  -1   │   ✓    │
│  8  │   +1     │   +1     │ +0.693 + 0.973 = +1.666  │  +1   │   ✓    │
│  9  │   -1     │   +1     │ -0.693 + 0.973 = +0.280  │  +1   │   ✓    │
│ 10  │   -1     │   -1     │ -0.693 - 0.973 = -1.666  │  -1   │   ✓    │
└─────┴──────────┴──────────┴──────────────────────────┴───────┴────────┘

After 2 rounds: 8/10 correct (80%)
After 1 round:  8/10 correct (80%)
Sample 9 (previously wrong) is now CORRECT! ✓
But samples 4, 5 are now wrong — Round 3 would focus on them.
```

---

## 8. Exponential Loss Function

### Definition

```
L(y, F(x)) = exp(-y · F(x))

Where:
  y ∈ {-1, +1}     (true label)
  F(x) = Σ αₜhₜ(x) (ensemble score, before sign)
```

### Properties

```
  Loss
    │
  4 │ *
    │  *
  3 │   *
    │    *
  2 │     *
    │       *
  1 │─────────*──────── = e⁰ = 1 (at y·F = 0)
    │           *
    │             *  *  *  *  *  → approaches 0
  0 └──┬──┬──┬──┬──┬──┬──┬──┬──► y·F(x)  (margin)
      -2 -1.5-1-0.5 0 0.5 1 1.5 2

  y·F(x) < 0: WRONG prediction → HIGH loss (exponentially growing)
  y·F(x) > 0: CORRECT prediction → LOW loss (exponentially decaying)
  y·F(x) = 0: Decision boundary → loss = 1
```

### Comparison with Other Losses

```
  Loss
    │
  4 │\                    
    │ \           Exponential: e^(-yF)
  3 │  \          Hinge (SVM): max(0, 1-yF)
    │   \         Logistic: log(1 + e^(-yF))
  2 │    \        0-1 Loss: I(yF < 0)
    │     \ \
    │      \.\
  1 │──┐    \.\ ────
    │  │    .  \ ....
    │  │   .    \\   ......
    │  │  .      ─────────........
  0 │──┘.´
    └───────────────────────────── yF (margin)
       -2  -1   0   1   2

Exponential loss heavily penalizes negative margins,
making AdaBoost very sensitive to outliers.
```

### Why Exponential Loss Causes Outlier Sensitivity

```
Example: A correctly classified point with large margin:
  y·F(x) = 3  →  Loss = e⁻³ = 0.050  (tiny)

Example: A misclassified point (outlier/noise):
  y·F(x) = -3 →  Loss = e³  = 20.086  (HUGE!)

This ONE outlier contributes ~400× more to the total loss!
AdaBoost will distort the model trying to fix this single point.
```

---

## 9. AdaBoostClassifier in Scikit-Learn

### Basic Usage

```python
from sklearn.ensemble import AdaBoostClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# Generate data
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    random_state=42
)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# --- AdaBoost with Decision Stumps (default) ---
ada_stump = AdaBoostClassifier(
    estimator=DecisionTreeClassifier(max_depth=1),  # stump
    n_estimators=200,
    learning_rate=1.0,
    algorithm='SAMME',
    random_state=42
)
ada_stump.fit(X_train, y_train)
print(f"Stump AdaBoost Accuracy: {ada_stump.score(X_test, y_test):.4f}")

# --- AdaBoost with Deeper Trees ---
ada_deep = AdaBoostClassifier(
    estimator=DecisionTreeClassifier(max_depth=3),  # depth-3 tree
    n_estimators=100,
    learning_rate=0.5,
    random_state=42
)
ada_deep.fit(X_train, y_train)
print(f"Deep AdaBoost Accuracy:  {ada_deep.score(X_test, y_test):.4f}")

print(f"\nClassification Report (Deep):")
print(classification_report(y_test, ada_deep.predict(X_test)))
```

### Key Parameters

```python
AdaBoostClassifier(
    estimator=None,          # Base learner (default: DecisionTreeClassifier(max_depth=1))
    n_estimators=50,         # Number of boosting rounds
    learning_rate=1.0,       # Shrinkage: scales each learner's contribution
    algorithm='SAMME',       # 'SAMME' (discrete) or 'SAMME.R' (real, uses probabilities)
    random_state=None        # Reproducibility
)
```

### Tracking Error vs Boosting Rounds

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.ensemble import AdaBoostClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

ada = AdaBoostClassifier(
    estimator=DecisionTreeClassifier(max_depth=1),
    n_estimators=300,
    learning_rate=1.0,
    random_state=42
)
ada.fit(X_train, y_train)

# Staged predictions (prediction at each round)
train_errors = []
test_errors = []

for y_pred_train in ada.staged_predict(X_train):
    train_errors.append(1 - accuracy_score(y_train, y_pred_train))

for y_pred_test in ada.staged_predict(X_test):
    test_errors.append(1 - accuracy_score(y_test, y_pred_test))

plt.figure(figsize=(10, 6))
plt.plot(range(1, 301), train_errors, 'b-', label='Training Error', alpha=0.7)
plt.plot(range(1, 301), test_errors, 'r-', label='Test Error', alpha=0.7)
plt.xlabel('Number of Boosting Rounds')
plt.ylabel('Error Rate')
plt.title('AdaBoost: Training & Test Error vs Rounds')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### Learning Rate & n_estimators Trade-off

```python
# Learning rate controls the contribution of each learner
# Lower learning rate → need more estimators → better generalization

learning_rates = [0.01, 0.1, 0.5, 1.0]
n_est_range = range(10, 501, 10)

plt.figure(figsize=(12, 6))
for lr in learning_rates:
    ada = AdaBoostClassifier(
        estimator=DecisionTreeClassifier(max_depth=1),
        n_estimators=500, learning_rate=lr, random_state=42
    )
    ada.fit(X_train, y_train)
    
    test_scores = [accuracy_score(y_test, y_pred)
                   for y_pred in ada.staged_predict(X_test)]
    plt.plot(range(1, 501), test_scores, label=f'lr={lr}')

plt.xlabel('Number of Estimators')
plt.ylabel('Test Accuracy')
plt.title('AdaBoost: Learning Rate vs Estimators')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## 10. Sensitivity to Noise and Outliers

### The Problem

```
Clean Data:
  AdaBoost performs excellently — converges quickly, high accuracy

Noisy Data (mislabeled points):
  Round 1: Misses noisy points → increases their weight
  Round 2: Still misses them → weight increases MORE
  Round 3: Weight even HIGHER → model starts distorting
  ...
  Round T: Model is dominated by trying to fit NOISE

  Noisy point weight grows EXPONENTIALLY across rounds!

Weight of a consistently misclassified point after T rounds:
  w ∝ Π e^(αₜ) = e^(Σαₜ)  → can become ENORMOUS
      t=1
```

### Demonstration: AdaBoost on Noisy Data

```python
import numpy as np
from sklearn.ensemble import AdaBoostClassifier, RandomForestClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score

X, y = make_classification(n_samples=500, n_features=10, random_state=42)

# Add label noise (flip 15% of labels)
np.random.seed(42)
noise_mask = np.random.random(len(y)) < 0.15
y_noisy = y.copy()
y_noisy[noise_mask] = 1 - y_noisy[noise_mask]
print(f"Flipped {noise_mask.sum()} labels ({noise_mask.mean()*100:.1f}%)")

# Compare on clean data
ada_clean = cross_val_score(
    AdaBoostClassifier(n_estimators=100, random_state=42),
    X, y, cv=5
)
rf_clean = cross_val_score(
    RandomForestClassifier(n_estimators=100, random_state=42),
    X, y, cv=5
)

# Compare on noisy data
ada_noisy = cross_val_score(
    AdaBoostClassifier(n_estimators=100, random_state=42),
    X, y_noisy, cv=5
)
rf_noisy = cross_val_score(
    RandomForestClassifier(n_estimators=100, random_state=42),
    X, y_noisy, cv=5
)

print(f"\nClean Data:")
print(f"  AdaBoost:      {ada_clean.mean():.4f} ± {ada_clean.std():.4f}")
print(f"  Random Forest: {rf_clean.mean():.4f} ± {rf_clean.std():.4f}")
print(f"\nNoisy Data (15% label noise):")
print(f"  AdaBoost:      {ada_noisy.mean():.4f} ± {ada_noisy.std():.4f}")
print(f"  Random Forest: {rf_noisy.mean():.4f} ± {rf_noisy.std():.4f}")
print(f"\nDrop in accuracy:")
print(f"  AdaBoost:      {(ada_clean.mean() - ada_noisy.mean())*100:.2f}%")
print(f"  Random Forest: {(rf_clean.mean() - rf_noisy.mean())*100:.2f}%")
```

### Mitigation Strategies

| Strategy | How It Helps |
|----------|--------------|
| **Limit n_estimators** | Prevents too many rounds of weight escalation |
| **Lower learning_rate** | Slows weight growth; each round has less impact |
| **Use deeper base learners** | Stronger learners converge faster with fewer rounds |
| **Data cleaning** | Remove obvious outliers before training |
| **Use Gradient Boosting instead** | Different loss functions (Huber) are more robust |
| **Weight capping** | Limit maximum sample weight (custom implementation) |

---

## 11. Applications

| Domain | Application | Notes |
|--------|-------------|-------|
| **Face Detection** | Viola-Jones face detector | Real-time face detection using cascaded AdaBoost |
| **Physics** | Particle classification at CERN | Used in Higgs boson discovery |
| **Medical Imaging** | Tumor detection in mammograms | Cascade of weak classifiers for efficiency |
| **Text Classification** | Spam filtering, sentiment analysis | AdaBoost with text features |
| **Computer Vision** | Pedestrian detection | Cascaded boosting for real-time detection |
| **Bioinformatics** | Protein structure prediction | Handles high-dimensional feature spaces |

### The Viola-Jones Face Detector (Landmark Application)

```
┌─────────────────────────────────────────────────────┐
│  Viola-Jones (2001): AdaBoost's Most Famous Use     │
│                                                     │
│  • Uses Haar-like features (simple rectangular      │
│    patterns) as weak learners                       │
│  • AdaBoost selects most discriminative features    │
│  • Cascade structure: quick rejection of non-faces  │
│                                                     │
│  Image → Stage 1 → Stage 2 → ... → Stage N → FACE  │
│            │          │                     │       │
│            ▼          ▼                     ▼       │
│         Not Face   Not Face            Not Face     │
│                                                     │
│  99%+ detection rate with very few false positives  │
│  Runs in real-time on 2001 hardware!                │
└─────────────────────────────────────────────────────┘
```

---

## 12. Summary Table

| Aspect | Details |
|--------|---------|
| **Full Name** | Adaptive Boosting |
| **Inventors** | Yoav Freund & Robert Schapire (1995) |
| **Core Idea** | Re-weight samples; focus on mistakes |
| **Labels** | y ∈ {-1, +1} |
| **Default Weak Learner** | Decision stump (depth=1 tree) |
| **Learner Weight** | αₜ = ½ ln((1-εₜ)/εₜ) |
| **Weight Update** | Misclassified: × e^(αₜ); Correct: × e^(-αₜ) |
| **Final Prediction** | sign(Σ αₜ hₜ(x)) |
| **Loss Function** | Exponential: L = exp(-y·F(x)) |
| **Error Bound** | ≤ exp(-2Tγ²) where γ = ½ - ε |
| **Reduces** | Bias (primarily) |
| **Outlier Sensitivity** | HIGH (exponential loss amplifies outlier influence) |
| **Key Parameters** | n_estimators, learning_rate, base estimator depth |
| **Award** | Gödel Prize 2003 |

---

## 13. Revision Questions

**Q1.** Walk through one complete round of AdaBoost: given 5 samples with weights [0.2, 0.2, 0.2, 0.2, 0.2] and a stump that misclassifies samples 2 and 4, compute εₜ, αₜ, and the new weights.

<details>
<summary>Answer</summary>

**Step 1:** Weighted error
εₜ = w₂ + w₄ = 0.2 + 0.2 = 0.4

**Step 2:** Learner weight
αₜ = ½ × ln((1-0.4)/0.4) = ½ × ln(1.5) = ½ × 0.405 = 0.203

**Step 3:** Weight update
- Correct (i=1,3,5): w̃ᵢ = 0.2 × e^(-0.203) = 0.2 × 0.816 = 0.1633
- Wrong (i=2,4): w̃ᵢ = 0.2 × e^(0.203) = 0.2 × 1.225 = 0.2449

**Step 4:** Normalize
Sum = 3 × 0.1633 + 2 × 0.2449 = 0.4898 + 0.4898 = 0.9797
- w₁ = w₃ = w₅ = 0.1633/0.9797 = 0.1667
- w₂ = w₄ = 0.2449/0.9797 = 0.2500

Final weights: [0.167, 0.250, 0.167, 0.250, 0.167]
Misclassified samples gained 50% more weight (0.250 vs 0.167).
</details>

**Q2.** Prove that if εₜ = 0 (perfect weak learner), the learner weight αₜ → ∞. What does this mean practically?

<details>
<summary>Answer</summary>

αₜ = ½ × ln((1 - εₜ)/εₜ)

As εₜ → 0: (1-εₜ)/εₜ → 1/0 → ∞, so ln(...) → ∞, thus αₜ → ∞.

**Practical meaning:** A perfect weak learner would receive infinite weight in the ensemble, effectively making the final prediction depend entirely on this single learner. This makes sense — if one learner is perfect, why consider any others?

In practice, εₜ = 0 rarely occurs with weak learners. If it does:
1. The algorithm can terminate early (problem solved)
2. Implementation may cap αₜ at a large finite value to avoid numerical issues
3. scikit-learn handles this edge case by assigning a very large but finite weight
</details>

**Q3.** Why does AdaBoost use exponential loss instead of 0-1 loss? What are the trade-offs?

<details>
<summary>Answer</summary>

**Why not 0-1 loss:** The 0-1 loss I(y ≠ ŷ) is non-differentiable and non-convex, making optimization intractable. There's no closed-form solution for optimal αₜ.

**Why exponential loss:**
1. **Convex:** Has a unique minimum, making optimization tractable
2. **Differentiable:** Allows closed-form solutions for αₜ and weight updates
3. **Upper bound on 0-1 loss:** exp(-yF) ≥ I(yF < 0), so minimizing exponential loss also pushes down classification error
4. **Elegant math:** Leads to the clean AdaBoost algorithm with simple formulas

**Trade-offs:**
- **Pro:** Computationally efficient, theoretically elegant
- **Con:** Heavily penalizes large negative margins → sensitive to outliers
- **Con:** Not calibrated for probability estimation (unlike logistic loss)
- **Alternative:** LogitBoost uses logistic loss for more robustness
</details>

**Q4.** Compare SAMME and SAMME.R algorithms in scikit-learn's AdaBoost. When would you use each?

<details>
<summary>Answer</summary>

**SAMME (Stagewise Additive Modeling using a Multi-class Exponential loss):**
- Uses discrete class labels from each weak learner
- Standard AdaBoost extended to multi-class
- Works with any classifier that outputs class labels
- The classic algorithm

**SAMME.R (SAMME Real):**
- Uses class probability estimates from each weak learner
- Typically converges faster (needs fewer boosting rounds)
- Requires base estimators that support predict_proba()
- Generally achieves better accuracy with fewer iterations

**When to use each:**
- Use SAMME when the base learner doesn't support predict_proba (e.g., SVM without calibration)
- Use SAMME.R (default before sklearn 1.2) when the base learner supports probability estimates — it's usually more accurate and converges faster
- SAMME.R is preferred in most practical cases
</details>

**Q5.** AdaBoost achieves 0% training error after 50 rounds, but test error is 15%. After 200 rounds, training error is still 0% but test error drops to 10%. Explain this seemingly paradoxical behavior.

<details>
<summary>Answer</summary>

This is the famous **"AdaBoost doesn't overfit" phenomenon** observed by Breiman and others.

**Explanation via margin theory:** Even after reaching 0% training error, additional boosting rounds continue to **increase the margin** (confidence) of predictions. A larger margin means the model is more robust — predictions are further from the decision boundary.

Think of it like this:
- After 50 rounds: All training points are correctly classified, but some are barely correct (small margin)
- After 200 rounds: All training points are still correct, and the margins have increased
- Larger margins → better generalization → lower test error

**Mathematically:** The bound on generalization error depends on the margin distribution, not just training error. As margins increase, the generalization bound tightens.

**However:** This resistance to overfitting is not absolute. With very noisy data, AdaBoost can eventually overfit as it tries to increase margins on mislabeled points. The effect is most pronounced with clean data.
</details>

**Q6.** Design an AdaBoost cascade (like Viola-Jones) for a spam email detector. Describe the stages, weak learners at each stage, and the early rejection mechanism.

<details>
<summary>Answer</summary>

**Cascade Design for Spam Detection:**

**Stage 1 (Quick Filter):** 1-2 stumps
- Features: presence of common spam words ("free", "winner", "click")
- Very fast check; rejects ~50% of non-spam immediately
- High recall (catches 99.9% of spam), low precision (some false positives)
- Non-spam passes → email delivered; possible spam → Stage 2

**Stage 2 (Content Analysis):** 5-10 stumps
- Features: word frequencies, caps ratio, exclamation count, URL count
- More thorough analysis for emails that passed Stage 1
- Rejects another ~30% of non-spam
- Possible spam → Stage 3

**Stage 3 (Deep Analysis):** 20-50 stumps
- Features: sender reputation, header analysis, link analysis, TF-IDF scores
- Detailed analysis for the hardest cases
- Final classification

**Early Rejection Mechanism:**
- Each stage has a very low false negative rate (99%+ spam detection)
- The cascade quickly rejects obvious non-spam in early stages
- Only ambiguous emails reach later (more expensive) stages
- Result: Most emails classified in microseconds, only ~5% need full analysis
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.3 Boosting Concepts](./03-boosting-concepts.md) | [Unit 8: Ensemble Methods](../README.md) | [8.5 Gradient Boosting →](./05-gradient-boosting.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 4: AdaBoost.*
