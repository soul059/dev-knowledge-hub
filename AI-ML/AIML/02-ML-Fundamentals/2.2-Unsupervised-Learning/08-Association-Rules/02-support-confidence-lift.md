# 📏 Support, Confidence, and Lift

> **Chapter 8.2 — Measuring the Strength of Association Rules**

---

## 📌 Overview

Association rules are evaluated using **interestingness measures** that quantify how strong, reliable, and useful a rule is. The three fundamental metrics are **Support**, **Confidence**, and **Lift**. Additional metrics like **Conviction** and **Leverage** provide further insight. Understanding these measures is essential for filtering meaningful rules from the thousands that mining algorithms typically generate.

---

## 📊 Reference Transaction Dataset

We'll use this dataset throughout all examples:

```
┌──────────────┬──────────────────────────────────┐
│ Transaction  │ Items                            │
├──────────────┼──────────────────────────────────┤
│     T1       │ Bread, Butter, Milk              │
│     T2       │ Bread, Diapers                   │
│     T3       │ Butter, Eggs, Milk               │
│     T4       │ Bread, Butter, Diapers, Milk     │
│     T5       │ Bread, Butter, Diapers, Eggs     │
│     T6       │ Bread, Milk                      │
│     T7       │ Eggs, Milk                       │
│     T8       │ Bread, Eggs                      │
│     T9       │ Bread, Butter, Milk, Eggs        │
│     T10      │ Bread, Butter, Milk              │
└──────────────┴──────────────────────────────────┘
  N = 10 transactions total
```

**Item counts (support counts):**

```
Bread:   8 (T1,T2,T4,T5,T6,T8,T9,T10)
Butter:  6 (T1,T3,T4,T5,T9,T10)
Milk:    8 (T1,T3,T4,T6,T7,T9,T10 — wait, let me recount)
```

Let me recount carefully:

```
Bread:   T1,T2,T4,T5,T6,T8,T9,T10     = 8 transactions
Butter:  T1,T3,T4,T5,T9,T10           = 6 transactions
Milk:    T1,T3,T4,T6,T7,T9,T10        = 7 transactions
Eggs:    T3,T5,T7,T8,T9               = 5 transactions
Diapers: T2,T4,T5                      = 3 transactions
```

---

## 📐 1. Support

### Definition

**Support** measures how frequently an itemset appears in the dataset:

```
                    Number of transactions containing X
Support(X)  =  ──────────────────────────────────────────
                    Total number of transactions (N)

                    freq(X)
Support(X)  =  ────────────
                      N
```

For a rule X → Y:

```
                       freq(X ∪ Y)
Support(X → Y)  =  ────────────────
                          N
```

### Intuition

```
┌──────────────────────────────────────────────────────────────┐
│ SUPPORT = "How common is this combination?"                  │
│                                                              │
│ High support → Frequent, broadly applicable                  │
│ Low support  → Rare, possibly niche or noise                 │
│                                                              │
│ Acts as a FILTER:                                            │
│ • Removes rare itemsets that aren't statistically reliable   │
│ • Reduces computational cost (fewer candidates to evaluate)  │
│ • Ensures rules have enough evidence                         │
└──────────────────────────────────────────────────────────────┘
```

### Worked Examples

```
Support(Bread) = 8/10 = 0.80  (80%)
Support(Butter) = 6/10 = 0.60  (60%)
Support(Milk) = 7/10 = 0.70  (70%)
Support(Eggs) = 5/10 = 0.50  (50%)
Support(Diapers) = 3/10 = 0.30  (30%)

Support({Bread, Butter}) = ?
  Transactions with BOTH Bread AND Butter:
  T1 ✓, T4 ✓, T5 ✓, T9 ✓, T10 ✓  → 5 transactions
  Support({Bread, Butter}) = 5/10 = 0.50  (50%)

Support({Bread, Milk}) = ?
  T1 ✓, T4 ✓, T6 ✓, T9 ✓, T10 ✓  → 5 transactions
  Support({Bread, Milk}) = 5/10 = 0.50  (50%)

Support({Bread, Butter, Milk}) = ?
  T1 ✓, T4 ✓, T9 ✓, T10 ✓  → 4 transactions
  Support({Bread, Butter, Milk}) = 4/10 = 0.40  (40%)
```

---

## 📐 2. Confidence

### Definition

**Confidence** measures how often the rule is correct — the probability of Y given X:

```
                         Support(X ∪ Y)       freq(X ∪ Y)
Confidence(X → Y)  =  ──────────────────  =  ──────────────
                          Support(X)            freq(X)
```

This is equivalent to the **conditional probability P(Y|X)**.

### Intuition

```
┌──────────────────────────────────────────────────────────────┐
│ CONFIDENCE = "How reliable is this rule?"                    │
│                                                              │
│ Confidence(X → Y) = P(Y | X)                                │
│                                                              │
│ "Of all transactions containing X, what fraction also        │
│  contain Y?"                                                 │
│                                                              │
│ High confidence → Y almost always appears with X             │
│ Low confidence  → Y rarely appears with X (weak rule)        │
│                                                              │
│ ⚠️ CAUTION: High confidence alone can be MISLEADING!         │
│ If Y is already very common (high support), confidence       │
│ will be high regardless of any real association.              │
└──────────────────────────────────────────────────────────────┘
```

### Worked Examples

```
Rule: {Bread} → {Butter}
  Confidence = Support({Bread, Butter}) / Support({Bread})
             = (5/10) / (8/10)
             = 5/8 = 0.625  (62.5%)
  
  "Of the 8 transactions with Bread, 5 also have Butter"

Rule: {Butter} → {Bread}
  Confidence = Support({Bread, Butter}) / Support({Butter})
             = (5/10) / (6/10)
             = 5/6 = 0.833  (83.3%)
  
  "Of the 6 transactions with Butter, 5 also have Bread"

  ⚠️ NOTE: Confidence is NOT symmetric!
  Conf({Bread} → {Butter}) = 62.5%
  Conf({Butter} → {Bread}) = 83.3%

Rule: {Bread, Butter} → {Milk}
  Confidence = Support({Bread, Butter, Milk}) / Support({Bread, Butter})
             = (4/10) / (5/10)
             = 4/5 = 0.80  (80%)
```

---

## 📐 3. Lift

### Definition

**Lift** measures how much more likely X and Y are to occur together than if they were **independent**:

```
                       Confidence(X → Y)       Support(X ∪ Y)
Lift(X → Y)  =  ────────────────────────  =  ────────────────────
                      Support(Y)             Support(X) · Support(Y)
```

### Intuition

```
┌──────────────────────────────────────────────────────────────┐
│ LIFT = "Is this association REAL or just COINCIDENCE?"        │
│                                                              │
│ Lift = 1.0  → X and Y are INDEPENDENT                       │
│               (no association — just coincidence!)            │
│                                                              │
│ Lift > 1.0  → X and Y are POSITIVELY associated             │
│               (buying X increases probability of Y)          │
│               COMPLEMENTARY items                            │
│                                                              │
│ Lift < 1.0  → X and Y are NEGATIVELY associated             │
│               (buying X decreases probability of Y)          │
│               SUBSTITUTE items                               │
│                                                              │
│ Lift is SYMMETRIC: Lift(X → Y) = Lift(Y → X)                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Worked Examples

```
Rule: {Bread} → {Butter}
  Lift = Support({Bread, Butter}) / (Support(Bread) · Support(Butter))
       = 0.50 / (0.80 × 0.60)
       = 0.50 / 0.48
       = 1.042

  Lift ≈ 1.04 → Very slight positive association
  (nearly independent — Bread and Butter occur together 
   about as often as random chance would predict)

Rule: {Bread} → {Milk}
  Lift = Support({Bread, Milk}) / (Support(Bread) · Support(Milk))
       = 0.50 / (0.80 × 0.70)
       = 0.50 / 0.56
       = 0.893

  Lift ≈ 0.89 → Slight NEGATIVE association
  (Bread and Milk occur together LESS than expected!)

Rule: {Diapers} → {Bread}
  Support({Diapers, Bread}): T2 ✓, T4 ✓, T5 ✓ → 3/10 = 0.30
  Lift = 0.30 / (0.30 × 0.80) = 0.30 / 0.24 = 1.25

  Lift = 1.25 → Positive association (25% more likely than random)
```

### Visual Interpretation

```
  Lift Scale:
  
  ◄── Negative Association ──── No Association ──── Positive Association ──►
  0.0                          1.0                                     ∞
  │                             │                                      │
  │  Substitutes               │  Complements                         │
  │  (Coke vs Pepsi)           │  (Bread & Butter)                    │
  │                             │                                      │
  ├──────────┼──────────┼──────┼──────┼──────────┼──────────┤
  0.5       0.7        0.9    1.0    1.2        1.5        2.0+
  Strong    Moderate   Weak   None   Weak    Moderate    Strong
  negative  negative   neg    assoc  pos     positive    positive
```

---

## 📐 4. Additional Metrics

### Conviction

Measures the degree to which X and ¬Y are **independent**:

```
                          1 - Support(Y)
Conviction(X → Y)  =  ─────────────────────
                       1 - Confidence(X → Y)
```

```
Conviction = 1    → X and Y are independent
Conviction > 1    → Positive association
Conviction = ∞    → Y always appears with X (Confidence = 1)
```

### Leverage

Measures the difference between observed and expected co-occurrence:

```
Leverage(X → Y) = Support(X ∪ Y) - Support(X) · Support(Y)
```

```
Leverage = 0    → Independent
Leverage > 0    → More co-occurrence than expected
Leverage < 0    → Less co-occurrence than expected
```

### Zhang's Metric

```
                    Confidence(X → Y) - Support(Y)
Zhang(X → Y) = ─────────────────────────────────────────
                max(Confidence(X → Y), Support(Y)) · (1 - Support(Y))
```

Range: [-1, 1]. Symmetric measure that handles edge cases better.

---

## 📊 Complete Worked Example

Let's compute all metrics for a rule from our dataset:

### Rule: {Butter} → {Milk}

```
Step 1: Gather counts
  freq(Butter) = 6  →  Support(Butter) = 0.60
  freq(Milk) = 7    →  Support(Milk) = 0.70
  freq({Butter, Milk}) = ?
    T1: Butter ✓, Milk ✓ → ✓
    T3: Butter ✓, Milk ✓ → ✓
    T4: Butter ✓, Milk ✓ → ✓
    T9: Butter ✓, Milk ✓ → ✓
    T10: Butter ✓, Milk ✓ → ✓
    → freq({Butter, Milk}) = 5  →  Support({Butter, Milk}) = 0.50

Step 2: Compute metrics

  SUPPORT:
  Support({Butter} → {Milk}) = 0.50  (50%)
  "Butter and Milk appear together in 50% of transactions"

  CONFIDENCE:
  Confidence = 0.50 / 0.60 = 0.833  (83.3%)
  "Of transactions with Butter, 83.3% also have Milk"

  LIFT:
  Lift = 0.50 / (0.60 × 0.70) = 0.50 / 0.42 = 1.190
  "Butter and Milk occur together 19% MORE than expected"

  CONVICTION:
  Conviction = (1 - 0.70) / (1 - 0.833) = 0.30 / 0.167 = 1.80

  LEVERAGE:
  Leverage = 0.50 - (0.60 × 0.70) = 0.50 - 0.42 = 0.08
```

```
┌──────────────────────────────────────────────────────────────┐
│  Rule: {Butter} → {Milk}                                    │
│                                                              │
│  Support    = 0.50  ✅ Common enough (> 0.3 threshold)       │
│  Confidence = 0.833 ✅ Reliable (> 0.7 threshold)            │
│  Lift       = 1.19  ✅ Positive association (> 1.0)          │
│  Conviction = 1.80  ✅ Strong directional association         │
│  Leverage   = 0.08  ✅ Occurs more than expected             │
│                                                              │
│  VERDICT: ✅ This is a MEANINGFUL rule!                       │
└──────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Minimum Thresholds

### Setting Thresholds

```
┌──────────────────────────────────────────────────────────────┐
│              TYPICAL THRESHOLD GUIDELINES                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Minimum Support:                                            │
│  • High-frequency retail: 1-5%                               │
│  • General purpose: 5-10%                                    │
│  • Small datasets: 20-50%                                    │
│                                                              │
│  Minimum Confidence:                                         │
│  • Typical: 50-80%                                           │
│  • Strict: 80%+                                              │
│  • ⚠️ Use with Lift — confidence alone is misleading          │
│                                                              │
│  Minimum Lift:                                               │
│  • Positive association: > 1.0                               │
│  • Meaningful association: > 1.2                             │
│  • Strong association: > 2.0                                 │
│                                                              │
│  RECOMMENDATION: Filter by Support first, then Lift,         │
│  then Confidence for the most reliable rules.                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

```python
import pandas as pd
from mlxtend.frequent_patterns import apriori, association_rules
from mlxtend.preprocessing import TransactionEncoder

# Transaction data
transactions = [
    ['Bread', 'Butter', 'Milk'],
    ['Bread', 'Diapers'],
    ['Butter', 'Eggs', 'Milk'],
    ['Bread', 'Butter', 'Diapers', 'Milk'],
    ['Bread', 'Butter', 'Diapers', 'Eggs'],
    ['Bread', 'Milk'],
    ['Eggs', 'Milk'],
    ['Bread', 'Eggs'],
    ['Bread', 'Butter', 'Milk', 'Eggs'],
    ['Bread', 'Butter', 'Milk'],
]

# Encode
te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)

# Frequent itemsets
freq_items = apriori(df, min_support=0.3, use_colnames=True)

# Generate rules with ALL metrics
rules = association_rules(
    freq_items, 
    metric="confidence", 
    min_threshold=0.5,
    num_itemsets=len(df)
)

# Display key columns
display_cols = ['antecedents', 'consequents', 'support', 
                'confidence', 'lift', 'conviction', 'leverage']
print(rules[display_cols].sort_values('lift', ascending=False).to_string())

# Filter for strong rules
strong_rules = rules[(rules['lift'] > 1.0) & 
                     (rules['confidence'] > 0.7) &
                     (rules['support'] > 0.3)]
print(f"\nStrong rules: {len(strong_rules)}")
print(strong_rules[display_cols].to_string())
```

### Manual Computation

```python
import numpy as np

# Manual computation for verification
N = len(transactions)

def support(itemset, transactions):
    """Compute support for an itemset."""
    count = sum(1 for t in transactions if itemset.issubset(set(t)))
    return count / len(transactions)

def confidence(X, Y, transactions):
    """Compute confidence for rule X → Y."""
    return support(X | Y, transactions) / support(X, transactions)

def lift(X, Y, transactions):
    """Compute lift for rule X → Y."""
    return support(X | Y, transactions) / (support(X, transactions) * support(Y, transactions))

def conviction_metric(X, Y, transactions):
    """Compute conviction for rule X → Y."""
    conf = confidence(X, Y, transactions)
    if conf == 1:
        return float('inf')
    return (1 - support(Y, transactions)) / (1 - conf)

# Test
X = {'Butter'}
Y = {'Milk'}

print(f"Support(Butter ∪ Milk) = {support(X | Y, transactions):.3f}")
print(f"Confidence(Butter → Milk) = {confidence(X, Y, transactions):.3f}")
print(f"Lift(Butter → Milk) = {lift(X, Y, transactions):.3f}")
print(f"Conviction(Butter → Milk) = {conviction_metric(X, Y, transactions):.3f}")
```

---

## 📋 Metrics Summary Table

| Metric | Formula | Range | Interpretation |
|--------|---------|-------|----------------|
| **Support** | freq(X∪Y)/N | [0, 1] | How frequent is the rule? |
| **Confidence** | Support(X∪Y)/Support(X) | [0, 1] | How reliable is the rule? |
| **Lift** | Confidence(X→Y)/Support(Y) | [0, ∞) | Is association real? (1 = independent) |
| **Conviction** | (1−Support(Y))/(1−Confidence) | [0, ∞) | Directional dependency strength |
| **Leverage** | Support(X∪Y)−Support(X)·Support(Y) | [−1, 1] | Deviation from independence |

---

## ❓ Revision Questions

1. **Given the rule {A, B} → {C} with Support = 0.4, Confidence = 0.8, and Support(C) = 0.5, compute the Lift. Is this a meaningful rule?**
   Show all calculations and interpret the result.

2. **Why can high confidence be misleading without considering Lift? Give a concrete example.**
   Construct a scenario where confidence is high but the items are independent.

3. **Prove that Lift is symmetric: Lift(X → Y) = Lift(Y → X). Is this also true for Confidence?**
   Use the formulas to show the mathematical relationship.

4. **A supermarket sets min_support = 0.01, min_confidence = 0.5, and min_lift = 1.2. Explain the reasoning behind each threshold.**
   Discuss what each threshold filters out and why these values are practical.

5. **Compute Support, Confidence, and Lift for ALL possible 2-item rules from this dataset: T1={A,B,C}, T2={A,B}, T3={A,C}, T4={B,C}, T5={A,B,C}.**
   Create a complete table of all rules and their metrics.

6. **What is Conviction and when is it more useful than Lift?**
   Discuss the directional nature of conviction vs. the symmetry of lift.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Market Basket Analysis](./01-market-basket-analysis.md) | [📂 Unsupervised Learning](../README.md) | [Apriori Algorithm →](./03-apriori-algorithm.md) |

---

> **Key Takeaway:** Support filters for frequency, Confidence measures reliability, and Lift reveals whether the association is genuine. Always use Lift (or Conviction) alongside Confidence — high confidence alone can be misleading when the consequent is already common. A rule is truly interesting when Support ≥ threshold AND Lift > 1.
