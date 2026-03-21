# 🌲 FP-Growth Algorithm

> **Chapter 8.4 — Frequent Pattern Mining Without Candidate Generation**

---

## 📌 Overview

The **FP-Growth** (Frequent Pattern Growth) algorithm, introduced by **Han, Pei, and Yin (2000)**, is a significant improvement over Apriori for mining frequent itemsets. It avoids the expensive **candidate generation** step by compressing the transaction database into a compact data structure called the **FP-tree**, then extracting frequent patterns directly from the tree.

### Key Advantages Over Apriori

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  APRIORI                           FP-GROWTH                        │
│  ┌─────────────────────────┐      ┌─────────────────────────┐      │
│  │ Multiple database scans │      │ Only 2 database scans   │      │
│  │ Candidate generation    │      │ No candidate generation │      │
│  │ Large candidate sets    │      │ Compressed FP-tree      │      │
│  │ Slow on large data      │      │ Much faster             │      │
│  └─────────────────────────┘      └─────────────────────────┘      │
│                                                                      │
│  FP-Growth is typically 1-2 orders of magnitude FASTER               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🏗️ The FP-Tree Data Structure

### Structure

```
┌──────────────────────────────────────────────────────────────┐
│                    FP-TREE COMPONENTS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. ROOT NODE: Empty node (null) at the top                  │
│                                                              │
│  2. ITEM PREFIX PATHS: Each path from root to leaf           │
│     represents a transaction (or partial transaction)        │
│                                                              │
│  3. NODE COUNTS: Each node stores the count of               │
│     transactions sharing that prefix path                    │
│                                                              │
│  4. HEADER TABLE: Links to the first occurrence of each      │
│     item in the tree (for traversal)                        │
│                                                              │
│  5. NODE LINKS: Horizontal links connecting all nodes        │
│     of the same item across different paths                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📝 Step-by-Step FP-Tree Construction

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

N = 9,  min_support_count = 2
```

### Step 1: First Database Scan — Count Items

```
┌──────┬───────┬──────────┐
│ Item │ Count │ Frequent?│
├──────┼───────┼──────────┤
│  B   │   7   │ ✅ Yes   │
│  A   │   6   │ ✅ Yes   │
│  C   │   6   │ ✅ Yes   │
│  D   │   2   │ ✅ Yes   │
│  E   │   2   │ ✅ Yes   │
└──────┴───────┴──────────┘

Sort by frequency (descending): B(7) > A(6) = C(6) > D(2) = E(2)
Order: B, A, C, D, E
```

### Step 2: Reorder Transactions by Frequency

```
┌──────────────┬─────────────────┬──────────────────┐
│ Transaction  │ Original        │ Ordered (by freq)│
├──────────────┼─────────────────┼──────────────────┤
│     T1       │ A, B, E         │ B, A, E          │
│     T2       │ B, D            │ B, D             │
│     T3       │ B, C            │ B, C             │
│     T4       │ A, B, D         │ B, A, D          │
│     T5       │ A, C            │ A, C             │
│     T6       │ B, C            │ B, C             │
│     T7       │ A, C            │ A, C             │
│     T8       │ A, B, C, E      │ B, A, C, E       │
│     T9       │ A, B, C         │ B, A, C          │
└──────────────┴─────────────────┴──────────────────┘
```

### Step 3: Build FP-Tree (Insert Transactions One by One)

**Insert T1: B, A, E**

```
      (null)
        │
      B:1
        │
      A:1
        │
      E:1
```

**Insert T2: B, D** (share prefix B)

```
      (null)
        │
      B:2 ──────────┐
        │            │
      A:1          D:1
        │
      E:1
```

**Insert T3: B, C** (share prefix B)

```
      (null)
        │
      B:3 ──────────┬─────────┐
        │            │         │
      A:1          D:1       C:1
        │
      E:1
```

**Insert T4: B, A, D** (share prefix B→A)

```
      (null)
        │
      B:4 ──────────┬─────────┐
        │            │         │
      A:2          D:1       C:1
       ╱ ╲
     E:1  D:1
```

**Insert T5: A, C** (no shared prefix with B-paths, new branch)

```
         (null)
        ╱      ╲
      B:4      A:1
      │╲╲        │
     A:2 D:1 C:1  C:1
    ╱  ╲
  E:1  D:1
```

**Continue inserting T6(B,C), T7(A,C), T8(B,A,C,E), T9(B,A,C)...**

**Final FP-Tree:**

```
                    (null)
                   ╱      ╲
                B:7       A:2
              ╱  │  ╲       │
           A:4  D:1  C:2   C:2
          ╱ │ ╲
        C:2 D:1 E:1
         │
        E:1

  Header Table:
  ┌──────┬───────┬────────────────┐
  │ Item │ Count │ Node Links     │
  ├──────┼───────┼────────────────┤
  │  B   │   7   │ B:7            │
  │  A   │   6   │ A:4 → A:2     │
  │  C   │   6   │ C:2→C:2→C:2  │
  │  D   │   2   │ D:1 → D:1     │
  │  E   │   2   │ E:1 → E:1     │
  └──────┴───────┴────────────────┘
```

---

## ⛏️ FP-Growth: Mining Frequent Patterns

### Algorithm

```
╔══════════════════════════════════════════════════════════════╗
║                  FP-GROWTH ALGORITHM                        ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  For each item (starting from least frequent):               ║
║                                                              ║
║  1. CONSTRUCT CONDITIONAL PATTERN BASE                       ║
║     • Follow header table links for the item                 ║
║     • Collect all prefix paths (with counts)                 ║
║                                                              ║
║  2. BUILD CONDITIONAL FP-TREE                                ║
║     • Create a new FP-tree from the pattern base             ║
║     • Only keep items meeting min_support                    ║
║                                                              ║
║  3. RECURSIVELY MINE                                         ║
║     • If conditional FP-tree is non-empty:                   ║
║       - Recursively apply FP-Growth on it                    ║
║     • If single path:                                        ║
║       - Enumerate all subsets as frequent patterns            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

### Mining Example: Extract Patterns for Item E

```
Step 1: Conditional Pattern Base for E
  Follow E node links:
    E:1 in path (null)→B→A→E  → prefix: {B,A} with count 1
    E:1 in path (null)→B→A→C→E → prefix: {B,A,C} with count 1

  Conditional Pattern Base for E:
    {B, A}: 1
    {B, A, C}: 1

Step 2: Conditional FP-Tree for E
  Item counts in conditional pattern base:
    B: 1+1 = 2 ✅ (≥ min_support)
    A: 1+1 = 2 ✅
    C: 0+1 = 1 ❌ (remove)

  Conditional FP-Tree for E:
      (null)
        │
      B:2
        │
      A:2

Step 3: Mine Conditional FP-Tree
  Single path → enumerate all subsets:
  
  Frequent patterns containing E:
    {E}: 2
    {B, E}: 2
    {A, E}: 2
    {B, A, E}: 2
```

### Mining for Item D

```
Step 1: Conditional Pattern Base for D
  D:1 in path (null)→B→D  → prefix: {B} with count 1
  D:1 in path (null)→B→A→D → prefix: {B,A} with count 1

  Conditional Pattern Base for D:
    {B}: 1
    {B, A}: 1

Step 2: Conditional FP-Tree for D
  Item counts:
    B: 1+1 = 2 ✅
    A: 0+1 = 1 ❌ (remove)

  Conditional FP-Tree for D:
      (null)
        │
      B:2

Step 3: Frequent patterns containing D:
    {D}: 2
    {B, D}: 2
```

### Summary of All Mined Patterns

```
┌──────────────────┬──────────┐
│ Frequent Itemset │ Support  │
├──────────────────┼──────────┤
│ {B}              │ 7        │
│ {A}              │ 6        │
│ {C}              │ 6        │
│ {D}              │ 2        │
│ {E}              │ 2        │
│ {B,A}            │ 4        │
│ {B,C}            │ 4        │
│ {A,C}            │ 4        │
│ {B,D}            │ 2        │
│ {B,E}            │ 2        │
│ {A,E}            │ 2        │
│ {B,A,C}          │ 2        │
│ {B,A,E}          │ 2        │
└──────────────────┴──────────┘

Same results as Apriori — but MUCH faster!
```

---

## ⚖️ FP-Growth vs Apriori

```
┌────────────────────────┬─────────────────┬──────────────────┐
│      Aspect            │    Apriori      │    FP-Growth     │
├────────────────────────┼─────────────────┼──────────────────┤
│ Database scans         │ k (one per      │ 2 (only!)        │
│                        │ level)          │                  │
│ Candidate generation   │ Yes (expensive) │ No               │
│ Data structure         │ Hash trees      │ FP-tree          │
│ Memory                 │ Moderate        │ Higher (FP-tree) │
│ Speed (small data)     │ Comparable      │ Comparable       │
│ Speed (large data)     │ Slow            │ Much faster      │
│ Speed (low min_sup)    │ Very slow       │ Fast             │
│ Implementation         │ Simpler         │ More complex     │
│ Parallelizable         │ Yes             │ Harder           │
└────────────────────────┴─────────────────┴──────────────────┘
```

```
  Performance Comparison (conceptual):

  Time
  ▲
  │
  │  Apriori ╱
  │        ╱
  │      ╱          FP-Growth
  │    ╱       ──────────────────
  │  ╱    ╱────
  │╱─────
  └───────────────────────────────► Data Size / 1/min_support

  FP-Growth advantage grows with dataset size and lower min_support
```

---

## 🐍 Python Implementation

### Using mlxtend

```python
import pandas as pd
from mlxtend.frequent_patterns import fpgrowth, association_rules
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

# Encode
te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)

# FP-Growth (same interface as apriori!)
frequent_itemsets = fpgrowth(df, min_support=2/9, use_colnames=True)
frequent_itemsets['length'] = frequent_itemsets['itemsets'].apply(len)

print("Frequent Itemsets (FP-Growth):")
print(frequent_itemsets.sort_values(['length', 'support'], 
                                     ascending=[True, False]).to_string())

# Generate rules (same as before)
rules = association_rules(frequent_itemsets, metric="lift", 
                          min_threshold=1.0, num_itemsets=len(df))
print(f"\nAssociation Rules (min_lift = 1.0):")
cols = ['antecedents', 'consequents', 'support', 'confidence', 'lift']
print(rules[cols].sort_values('lift', ascending=False).to_string())
```

### Speed Comparison: Apriori vs FP-Growth

```python
import time
from mlxtend.frequent_patterns import apriori, fpgrowth

# Generate larger dataset
import numpy as np
np.random.seed(42)
n_transactions = 10000
n_items = 50
items = [f'Item_{i}' for i in range(n_items)]

large_transactions = []
for _ in range(n_transactions):
    n = np.random.randint(2, 8)
    trans = list(np.random.choice(items, size=n, replace=False))
    large_transactions.append(trans)

te = TransactionEncoder()
te_array = te.fit(large_transactions).transform(large_transactions)
df_large = pd.DataFrame(te_array, columns=te.columns_)

# Time Apriori
start = time.time()
freq_apriori = apriori(df_large, min_support=0.01, use_colnames=True)
time_apriori = time.time() - start

# Time FP-Growth
start = time.time()
freq_fpgrowth = fpgrowth(df_large, min_support=0.01, use_colnames=True)
time_fpgrowth = time.time() - start

print(f"Apriori:   {time_apriori:.3f}s  ({len(freq_apriori)} itemsets)")
print(f"FP-Growth: {time_fpgrowth:.3f}s  ({len(freq_fpgrowth)} itemsets)")
print(f"Speedup:   {time_apriori/time_fpgrowth:.1f}x")
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Algorithm** | FP-Growth (Han, Pei, Yin, 2000) |
| **Strategy** | Compress database → mine patterns from tree |
| **Data Structure** | FP-tree (compact representation of transactions) |
| **Database Scans** | Only 2 (vs. k for Apriori) |
| **Candidate Gen.** | None (major advantage over Apriori) |
| **Key Concept** | Conditional pattern bases and conditional FP-trees |
| **Speed** | 1-2 orders of magnitude faster than Apriori |
| **Memory** | Higher (stores FP-tree in memory) |
| **Python** | `mlxtend.frequent_patterns.fpgrowth` |
| **When to Use** | Large datasets, low minimum support |

---

## ❓ Revision Questions

1. **Explain the FP-tree data structure. What are its components, and how does it compress the transaction database?**
   Describe root, item prefix paths, node counts, header table, and node links.

2. **Given these transactions with min_support = 2: {A,B,C}, {A,B}, {B,C,D}, {A,B,C,D}, {B,C}, construct the FP-tree step by step.**
   Show the tree after each insertion, including item ordering by frequency.

3. **What is a conditional pattern base? Compute the conditional pattern base for item C from your FP-tree in Q2.**
   Trace the header links and collect prefix paths with counts.

4. **Why does FP-Growth only need 2 database scans while Apriori needs k? What are those 2 scans used for?**
   Explain the first scan (item counting) and second scan (tree construction).

5. **Compare the memory usage of Apriori and FP-Growth. In what scenario might Apriori use less memory?**
   Consider sparse vs. dense datasets and the FP-tree compression ratio.

6. **FP-Growth is not easily parallelizable. Why? What modifications exist for distributed FP-Growth?**
   Discuss the recursive nature and mention PFP (Parallel FP-Growth).

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Apriori Algorithm](./03-apriori-algorithm.md) | [📂 Unsupervised Learning](../README.md) | [Applications →](./05-applications.md) |

---

> **Key Takeaway:** FP-Growth eliminates Apriori's two main bottlenecks — multiple database scans and candidate generation — by compressing the database into an FP-tree and mining patterns directly from it. Use FP-Growth for large datasets and low minimum support thresholds where Apriori becomes impractical.
