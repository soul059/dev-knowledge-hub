# ⛏️ Apriori Algorithm

> **Chapter 8.3 — Level-Wise Search for Frequent Itemsets**

---

## 📌 Overview

The **Apriori algorithm** is a classic algorithm for mining frequent itemsets and generating association rules from transaction databases. Introduced by **Agrawal and Srikant (1994)**, it uses the **Apriori principle** (downward closure property) to efficiently prune the search space, making it feasible to analyze large datasets.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  APRIORI PRINCIPLE (Downward Closure Property):                      │
│                                                                      │
│  "If an itemset is INFREQUENT, then ALL its SUPERSETS must also      │
│   be infrequent."                                                    │
│                                                                      │
│  Equivalently:                                                       │
│  "ALL SUBSETS of a FREQUENT itemset must also be frequent."          │
│                                                                      │
│  Example:                                                            │
│  If {Beer} is infrequent → we don't need to check                   │
│  {Beer, Diapers}, {Beer, Bread}, {Beer, Diapers, Bread}, etc.       │
│                                                                      │
│  This PRUNES the search space dramatically!                          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 The Apriori Principle: Visual Proof

```
                    ITEMSET LATTICE
                    ────────────────
                    
  Level 0:                  {}
                    ╱    ╱    ╲    ╲
  Level 1:       {A}   {B}   {C}   {D}
                 ╱ ╲   ╱ ╲   ╱ ╲
  Level 2:    {A,B}{A,C}{A,D}{B,C}{B,D}{C,D}
               ╱    │    ╲ ╱    ╲
  Level 3:  {A,B,C} {A,B,D} {A,C,D} {B,C,D}
                  ╲     │    ╱
  Level 4:       {A,B,C,D}


  PRUNING EXAMPLE:
  ─────────────────

  If {B} is INFREQUENT (below min_support):
  
  Level 1:       {A}   {B}✗  {C}   {D}
                 ╱ ╲         ╱ ╲
  Level 2:    {A,B}✗{A,C}{A,D}{B,C}✗{B,D}✗{C,D}
                     │    ╲
  Level 3:  {A,B,C}✗{A,B,D}✗{A,C,D}{B,C,D}✗
  
  All supersets of {B} are pruned! → Huge savings!
```

### Why It Works (Proof)

```
Support is ANTI-MONOTONE:

  Support(X) ≥ Support(X ∪ {item})

  Why? Every transaction containing X ∪ {item} must also 
  contain X, but not vice versa.

  Therefore: if Support(X) < min_support,
  then Support(X ∪ Y) ≤ Support(X) < min_support
  for ANY Y.
```

---

## 📊 Algorithm Steps

```
╔══════════════════════════════════════════════════════════════════╗
║                    APRIORI ALGORITHM                            ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  INPUT: Transaction database D, minimum support threshold        ║
║                                                                  ║
║  STEP 1: FIND FREQUENT 1-ITEMSETS                                ║
║  ┌─────────────────────────────────────────────────┐            ║
║  │ • Count support for each individual item         │            ║
║  │ • Remove items below min_support                 │            ║
║  │ • Result: L₁ (frequent 1-itemsets)               │            ║
║  └─────────────────────────────────────────────────┘            ║
║                    │                                             ║
║                    ▼                                             ║
║  REPEAT for k = 2, 3, 4, ...:                                   ║
║  ┌─────────────────────────────────────────────────┐            ║
║  │ STEP 2: CANDIDATE GENERATION                     │            ║
║  │ • Join: Combine pairs from L_{k-1} that share    │            ║
║  │   (k-2) items to create k-itemset candidates     │            ║
║  │ • Result: Cₖ (candidate k-itemsets)               │            ║
║  └─────────────────────────────────────────────────┘            ║
║                    │                                             ║
║                    ▼                                             ║
║  ┌─────────────────────────────────────────────────┐            ║
║  │ STEP 3: PRUNING                                   │            ║
║  │ • Remove candidates whose ANY (k-1)-subset is    │            ║
║  │   NOT in L_{k-1}                                  │            ║
║  │ • (Apriori principle)                             │            ║
║  └─────────────────────────────────────────────────┘            ║
║                    │                                             ║
║                    ▼                                             ║
║  ┌─────────────────────────────────────────────────┐            ║
║  │ STEP 4: COUNTING                                  │            ║
║  │ • Scan database to count support for each         │            ║
║  │   remaining candidate                             │            ║
║  │ • Remove candidates below min_support             │            ║
║  │ • Result: Lₖ (frequent k-itemsets)                │            ║
║  └─────────────────────────────────────────────────┘            ║
║                    │                                             ║
║  UNTIL: Lₖ = ∅ (no more frequent itemsets found)                ║
║                                                                  ║
║  OUTPUT: L = L₁ ∪ L₂ ∪ ... ∪ Lₖ (all frequent itemsets)        ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 📝 Step-by-Step Worked Example

### Transaction Database

```
┌──────────────┬──────────────────┐
│ Transaction  │ Items            │
├──────────────┼──────────────────┤
│     T1       │ A, B, E          │
│     T2       │ B, D             │
│     T3       │ B, C             │
│     T4       │ A, B, D          │
│     T5       │ A, C             │
│     T6       │ B, C             │
│     T7       │ A, C             │
│     T8       │ A, B, C, E       │
│     T9       │ A, B, C          │
└──────────────┴──────────────────┘

N = 9 transactions
min_support = 2/9 ≈ 0.22 (i.e., min count = 2)
```

### Pass 1: Frequent 1-Itemsets

```
  Scan database, count each item:

  ┌────────┬───────┬──────────┐
  │ Item   │ Count │ Frequent?│
  ├────────┼───────┼──────────┤
  │ {A}    │   6   │ ✅ Yes   │
  │ {B}    │   7   │ ✅ Yes   │
  │ {C}    │   6   │ ✅ Yes   │
  │ {D}    │   2   │ ✅ Yes   │
  │ {E}    │   2   │ ✅ Yes   │
  └────────┴───────┴──────────┘

  L₁ = {{A}, {B}, {C}, {D}, {E}}    (all items are frequent)
```

### Pass 2: Frequent 2-Itemsets

**Generate Candidates C₂** (all pairs from L₁):

```
  C₂ = {{A,B}, {A,C}, {A,D}, {A,E}, {B,C}, {B,D}, {B,E}, {C,D}, {C,E}, {D,E}}
```

**Count Support** (scan database):

```
  ┌──────────┬───────┬──────────────────────────────────┬──────────┐
  │ Itemset  │ Count │ Transactions                     │ Frequent?│
  ├──────────┼───────┼──────────────────────────────────┼──────────┤
  │ {A,B}    │   4   │ T1, T4, T8, T9                   │ ✅ Yes   │
  │ {A,C}    │   4   │ T5, T7, T8, T9                   │ ✅ Yes   │
  │ {A,D}    │   1   │ T4                               │ ❌ No    │
  │ {A,E}    │   2   │ T1, T8                           │ ✅ Yes   │
  │ {B,C}    │   4   │ T3, T6, T8, T9                   │ ✅ Yes   │
  │ {B,D}    │   2   │ T2, T4                           │ ✅ Yes   │
  │ {B,E}    │   2   │ T1, T8                           │ ✅ Yes   │
  │ {C,D}    │   0   │ —                                │ ❌ No    │
  │ {C,E}    │   1   │ T8                               │ ❌ No    │
  │ {D,E}    │   0   │ —                                │ ❌ No    │
  └──────────┴───────┴──────────────────────────────────┴──────────┘

  L₂ = {{A,B}, {A,C}, {A,E}, {B,C}, {B,D}, {B,E}}
```

### Pass 3: Frequent 3-Itemsets

**Generate Candidates C₃** (join L₂ with itself):

```
  Join rule: Two itemsets in L₂ can be joined if they share 
  the first (k-2) = 1 item.

  {A,B} ∪ {A,C} → {A,B,C}  ← Check: {B,C} ∈ L₂? YES ✅
  {A,B} ∪ {A,E} → {A,B,E}  ← Check: {B,E} ∈ L₂? YES ✅
  {A,C} ∪ {A,E} → {A,C,E}  ← Check: {C,E} ∈ L₂? NO  ❌ PRUNE!
  {B,C} ∪ {B,D} → {B,C,D}  ← Check: {C,D} ∈ L₂? NO  ❌ PRUNE!
  {B,C} ∪ {B,E} → {B,C,E}  ← Check: {C,E} ∈ L₂? NO  ❌ PRUNE!
  {B,D} ∪ {B,E} → {B,D,E}  ← Check: {D,E} ∈ L₂? NO  ❌ PRUNE!

  After pruning: C₃ = {{A,B,C}, {A,B,E}}
```

**Count Support:**

```
  ┌──────────┬───────┬──────────────────┬──────────┐
  │ Itemset  │ Count │ Transactions     │ Frequent?│
  ├──────────┼───────┼──────────────────┼──────────┤
  │ {A,B,C}  │   2   │ T8, T9           │ ✅ Yes   │
  │ {A,B,E}  │   2   │ T1, T8           │ ✅ Yes   │
  └──────────┴───────┴──────────────────┴──────────┘

  L₃ = {{A,B,C}, {A,B,E}}
```

### Pass 4: Frequent 4-Itemsets

```
  Join: {A,B,C} ∪ {A,B,E} → {A,B,C,E}
  Check all 3-subsets:
    {A,B,C} ∈ L₃? YES ✅
    {A,B,E} ∈ L₃? YES ✅
    {A,C,E} ∈ L₃? NO  ❌ PRUNE!
    {B,C,E} ∈ L₃? NO  ❌ PRUNE!

  C₄ = {} (pruned!)

  L₄ = {} → STOP
```

### Final Result

```
  ALL FREQUENT ITEMSETS:
  ┌──────────────────┬──────────┐
  │ Frequent Itemset │ Support  │
  ├──────────────────┼──────────┤
  │ {A}              │ 6/9=0.67 │
  │ {B}              │ 7/9=0.78 │
  │ {C}              │ 6/9=0.67 │
  │ {D}              │ 2/9=0.22 │
  │ {E}              │ 2/9=0.22 │
  │ {A,B}            │ 4/9=0.44 │
  │ {A,C}            │ 4/9=0.44 │
  │ {A,E}            │ 2/9=0.22 │
  │ {B,C}            │ 4/9=0.44 │
  │ {B,D}            │ 2/9=0.22 │
  │ {B,E}            │ 2/9=0.22 │
  │ {A,B,C}          │ 2/9=0.22 │
  │ {A,B,E}          │ 2/9=0.22 │
  └──────────────────┴──────────┘
```

---

## 📐 Generating Association Rules from Frequent Itemsets

Once we have frequent itemsets, we generate **all non-empty proper subsets** as rules:

```
For frequent itemset {A,B,C}:

  Possible rules:
  ┌────────────────────────────┬────────────┐
  │ Rule                       │ Confidence │
  ├────────────────────────────┼────────────┤
  │ {A,B} → {C}               │ 2/4 = 0.50 │
  │ {A,C} → {B}               │ 2/4 = 0.50 │
  │ {B,C} → {A}               │ 2/4 = 0.50 │
  │ {A}   → {B,C}             │ 2/6 = 0.33 │
  │ {B}   → {A,C}             │ 2/7 = 0.29 │
  │ {C}   → {A,B}             │ 2/6 = 0.33 │
  └────────────────────────────┴────────────┘

  With min_confidence = 0.50:
  Rules {A,B}→{C}, {A,C}→{B}, {B,C}→{A} survive
```

---

## ⏱️ Computational Complexity

```
┌──────────────────────────────────────────────────────────────┐
│                COMPLEXITY ANALYSIS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Let: N = number of transactions                             │
│       d = number of unique items                             │
│       w = average transaction width                          │
│       k = maximum itemset size                               │
│                                                              │
│  Without pruning:                                            │
│  • Total possible itemsets = 2^d - 1                         │
│  • For 100 items: 2^100 ≈ 10^30 (IMPOSSIBLE!)               │
│                                                              │
│  With Apriori pruning:                                       │
│  • Dramatically reduced, but still:                          │
│  • Multiple database scans (one per level k)                 │
│  • Candidate generation can be expensive                     │
│  • Worst case still exponential in d                         │
│                                                              │
│  BOTTLENECKS:                                                │
│  1. Multiple database scans (I/O intensive)                  │
│  2. Large candidate sets at intermediate levels              │
│  3. Support counting for each candidate                      │
│                                                              │
│  → FP-Growth solves these (see next chapter)                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### Using mlxtend

```python
import pandas as pd
from mlxtend.frequent_patterns import apriori, association_rules
from mlxtend.preprocessing import TransactionEncoder

# Transaction database
transactions = [
    ['A', 'B', 'E'],
    ['B', 'D'],
    ['B', 'C'],
    ['A', 'B', 'D'],
    ['A', 'C'],
    ['B', 'C'],
    ['A', 'C'],
    ['A', 'B', 'C', 'E'],
    ['A', 'B', 'C'],
]

# Encode transactions as binary matrix
te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)
print("Binary Transaction Matrix:")
print(df.astype(int))

# Run Apriori algorithm
min_sup = 2/9  # minimum support threshold
frequent_itemsets = apriori(df, min_support=min_sup, use_colnames=True)
frequent_itemsets['length'] = frequent_itemsets['itemsets'].apply(len)

print(f"\nFrequent Itemsets (min_support = {min_sup:.3f}):")
print(frequent_itemsets.sort_values(['length', 'support'], 
                                     ascending=[True, False]).to_string())

# Generate association rules
rules = association_rules(frequent_itemsets, metric="confidence", 
                          min_threshold=0.5, num_itemsets=len(df))
print(f"\nAssociation Rules (min_confidence = 0.5):")
cols = ['antecedents', 'consequents', 'support', 'confidence', 'lift']
print(rules[cols].sort_values('lift', ascending=False).to_string())
```

### Apriori from Scratch

```python
from itertools import combinations
from collections import defaultdict

def apriori_from_scratch(transactions, min_support_count):
    """
    Apriori algorithm implementation from scratch.
    
    Parameters:
        transactions: list of sets, each set contains item names
        min_support_count: minimum number of transactions for frequent
    
    Returns:
        frequent_itemsets: dict mapping itemset (frozenset) to count
    """
    # Convert to list of frozensets
    trans = [frozenset(t) for t in transactions]
    N = len(trans)
    
    # Step 1: Find frequent 1-itemsets
    item_counts = defaultdict(int)
    for t in trans:
        for item in t:
            item_counts[frozenset([item])] += 1
    
    L_prev = {itemset: count for itemset, count in item_counts.items() 
              if count >= min_support_count}
    
    all_frequent = dict(L_prev)
    k = 2
    
    while L_prev:
        # Candidate generation: join L_{k-1} with itself
        prev_list = list(L_prev.keys())
        candidates = set()
        
        for i in range(len(prev_list)):
            for j in range(i + 1, len(prev_list)):
                union = prev_list[i] | prev_list[j]
                if len(union) == k:
                    # Pruning: check all (k-1)-subsets are in L_prev
                    all_subsets_frequent = True
                    for item in union:
                        subset = union - frozenset([item])
                        if subset not in L_prev:
                            all_subsets_frequent = False
                            break
                    if all_subsets_frequent:
                        candidates.add(union)
        
        # Count support for candidates
        candidate_counts = defaultdict(int)
        for t in trans:
            for candidate in candidates:
                if candidate.issubset(t):
                    candidate_counts[candidate] += 1
        
        # Filter by min support
        L_current = {itemset: count for itemset, count in candidate_counts.items()
                     if count >= min_support_count}
        
        all_frequent.update(L_current)
        L_prev = L_current
        k += 1
        
        print(f"Level {k-1}: {len(L_current)} frequent {k-1}-itemsets found")
    
    return all_frequent

# Run
transactions = [
    {'A', 'B', 'E'}, {'B', 'D'}, {'B', 'C'},
    {'A', 'B', 'D'}, {'A', 'C'}, {'B', 'C'},
    {'A', 'C'}, {'A', 'B', 'C', 'E'}, {'A', 'B', 'C'}
]

frequent = apriori_from_scratch(transactions, min_support_count=2)

print("\nAll Frequent Itemsets:")
for itemset, count in sorted(frequent.items(), key=lambda x: (len(x[0]), -x[1])):
    print(f"  {set(itemset):20s}  count={count}  support={count/9:.3f}")
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Algorithm** | Apriori (Agrawal & Srikant, 1994) |
| **Input** | Transaction database, min_support |
| **Output** | All frequent itemsets |
| **Key Principle** | Downward closure (anti-monotonicity of support) |
| **Strategy** | Level-wise, bottom-up search |
| **Candidate Gen.** | Join L_{k-1} with itself |
| **Pruning** | Remove if any (k-1)-subset is infrequent |
| **DB Scans** | k scans (one per level) |
| **Weakness** | Multiple scans, large candidate sets |
| **Improvement** | FP-Growth (no candidate generation) |
| **Python** | `mlxtend.frequent_patterns.apriori` |

---

## ❓ Revision Questions

1. **State the Apriori principle and prove why it holds using the anti-monotonicity of support.**
   Give a formal argument based on the subset relationship.

2. **Given transactions {A,B,C}, {A,B}, {A,C}, {B,C}, {A,B,C}, with min_support = 40%, trace through the Apriori algorithm step by step.**
   Show L₁, C₂, L₂, C₃, L₃, including all pruning decisions.

3. **Why does Apriori require multiple database scans? How does this impact performance on large datasets?**
   Discuss I/O costs and compare with FP-Growth.

4. **The Apriori algorithm generates {A,B,C} as a candidate 3-itemset. What conditions must be met for it NOT to be pruned?**
   List all required subsets and their frequency requirements.

5. **After finding frequent itemsets, how are association rules generated? For the itemset {A,B,C,D}, how many possible rules exist?**
   Count all non-empty proper subset combinations (2^k - 2 rules for k-itemset).

6. **Compare the Apriori algorithm's worst-case and practical complexity. What factors most affect its running time?**
   Discuss number of items, transaction width, and minimum support threshold.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Support, Confidence, Lift](./02-support-confidence-lift.md) | [📂 Unsupervised Learning](../README.md) | [FP-Growth →](./04-fp-growth.md) |

---

> **Key Takeaway:** The Apriori algorithm's power lies in the downward closure property — if an itemset is infrequent, all its supersets are also infrequent. This simple principle prunes the exponential search space dramatically, making association rule mining tractable. However, multiple database scans and candidate generation remain bottlenecks, motivating FP-Growth.
