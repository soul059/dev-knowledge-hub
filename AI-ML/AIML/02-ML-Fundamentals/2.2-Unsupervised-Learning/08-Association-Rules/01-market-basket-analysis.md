# 🛒 Market Basket Analysis

> **Chapter 8.1 — Discovering Patterns in Transaction Data**

---

## 📌 Overview

**Market Basket Analysis (MBA)** is a data mining technique that discovers relationships between items purchased together in transactions. Originally developed for retail, it reveals hidden patterns like *"customers who buy bread also tend to buy butter"* — enabling better product placement, cross-selling, and recommendation strategies.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  TRANSACTION DATA:                 DISCOVERED RULES:                 │
│  ┌──────────────────────┐         ┌──────────────────────────┐      │
│  │ T1: Bread, Butter    │         │ {Bread} → {Butter}       │      │
│  │ T2: Beer, Diapers    │  ───►   │ {Beer}  → {Diapers}      │      │
│  │ T3: Bread, Milk      │         │ {Bread, Milk} → {Eggs}   │      │
│  │ T4: Bread, Butter,   │         │ ...                      │      │
│  │     Milk, Eggs       │         │                          │      │
│  └──────────────────────┘         └──────────────────────────┘      │
│                                                                      │
│  "What items are frequently purchased TOGETHER?"                     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Terminology

### Items and Itemsets

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ITEM:     A single product (e.g., "Bread")                 │
│                                                             │
│  ITEMSET:  A collection of one or more items                │
│            {Bread, Butter} is a 2-itemset                   │
│            {Bread, Milk, Eggs} is a 3-itemset               │
│                                                             │
│  TRANSACTION: A set of items bought together by a customer  │
│               T = {Bread, Butter, Milk}                     │
│                                                             │
│  TRANSACTION DATABASE: Collection of all transactions       │
│               D = {T₁, T₂, ..., Tₙ}                        │
│                                                             │
│  ASSOCIATION RULE:  X → Y                                   │
│  "If a customer buys X, they are likely to also buy Y"      │
│  (X is antecedent, Y is consequent)                         │
│                                                             │
│  X ∩ Y = ∅  (antecedent and consequent must not overlap)    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Transaction Database Example

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
```

---

## 📊 Binary Transaction Matrix

The transaction database can be represented as a **binary matrix** where rows are transactions and columns are items:

```
         Bread  Butter  Diapers  Eggs  Milk
   T1  │  1      1       0       0     1  │
   T2  │  1      0       1       0     0  │
   T3  │  0      1       0       1     1  │
   T4  │  1      1       1       0     1  │
   T5  │  1      1       1       1     0  │
   T6  │  1      0       0       0     1  │
   T7  │  0      0       0       1     1  │
   T8  │  1      0       0       1     0  │
   T9  │  1      1       0       1     1  │
   T10 │  1      1       0       0     1  │
```

```python
import pandas as pd

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

# Convert to binary matrix
from mlxtend.preprocessing import TransactionEncoder

te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)
print(df.astype(int))
```

---

## 🌍 Real-World Applications

### 1. Retail / E-Commerce

```
┌──────────────────────────────────────────────────────────────┐
│  PRODUCT PLACEMENT:                                          │
│  ┌──────────┐  ┌──────────┐                                  │
│  │  Bread   │  │  Butter  │  ← Place near each other        │
│  │  ▓▓▓▓▓▓  │  │  ▓▓▓▓▓▓  │                                  │
│  └──────────┘  └──────────┘                                  │
│                                                              │
│  CROSS-SELLING:                                              │
│  "Customers who bought this also bought..."                  │
│                                                              │
│  PROMOTIONS:                                                 │
│  Bundle frequently associated items for discounts            │
│                                                              │
│  INVENTORY MANAGEMENT:                                       │
│  Stock associated items together                             │
└──────────────────────────────────────────────────────────────┘
```

### 2. Web Mining

```
  Clickstream Analysis:
  
  Page A → Page B → Page C → Purchase
  
  Rules discovered:
  • {Product Page, Reviews Page} → {Add to Cart}
  • {Homepage, Category Page} → {Product Page}
  
  Applications:
  • Website layout optimization
  • Personalized navigation
  • Content recommendation
```

### 3. Medical / Healthcare

```
  Symptom-Disease Associations:
  
  {Fever, Cough, Fatigue} → {Flu Diagnosis}
  {Chest Pain, Shortness of Breath} → {Cardiac Event}
  
  Drug Interactions:
  {Drug A, Drug B} → {Adverse Reaction}
  
  Treatment Patterns:
  {Diagnosis X} → {Treatment Protocol Y}
```

### 4. Other Domains

| Domain | Application | Example Rule |
|--------|-------------|--------------|
| **Telecommunications** | Service bundling | {Internet} → {TV Package} |
| **Banking** | Product cross-selling | {Savings Account} → {Credit Card} |
| **Manufacturing** | Defect analysis | {Defect A, Defect B} → {Root Cause C} |
| **Bioinformatics** | Gene co-expression | {Gene1, Gene2} → {Gene3} |
| **Education** | Course enrollment | {Math 101, CS 101} → {Statistics 201} |
| **Fraud Detection** | Fraud patterns | {Large withdrawal, New device} → {Fraud} |

---

## 🔄 The Association Rule Mining Process

```
╔══════════════════════════════════════════════════════════════╗
║              ASSOCIATION RULE MINING PIPELINE                ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  Step 1: COLLECT transaction data                            ║
║          ↓                                                   ║
║  Step 2: PREPROCESS (clean, encode, create binary matrix)    ║
║          ↓                                                   ║
║  Step 3: FIND FREQUENT ITEMSETS                              ║
║          (Apriori, FP-Growth, or ECLAT)                      ║
║          ↓                                                   ║
║  Step 4: GENERATE ASSOCIATION RULES                          ║
║          from frequent itemsets                              ║
║          ↓                                                   ║
║  Step 5: EVALUATE rules using metrics                        ║
║          (Support, Confidence, Lift, etc.)                   ║
║          ↓                                                   ║
║  Step 6: INTERPRET and APPLY business rules                  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🐍 Python: Quick Start

```python
from mlxtend.frequent_patterns import apriori, association_rules
from mlxtend.preprocessing import TransactionEncoder
import pandas as pd

# Sample transactions
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

# Encode transactions
te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)

# Find frequent itemsets
frequent_itemsets = apriori(df, min_support=0.3, use_colnames=True)
print("Frequent Itemsets:")
print(frequent_itemsets.sort_values('support', ascending=False))

# Generate association rules
rules = association_rules(frequent_itemsets, metric="lift", min_threshold=1.0)
print("\nAssociation Rules:")
print(rules[['antecedents', 'consequents', 'support', 'confidence', 'lift']])
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Market Basket Analysis** | Discover items frequently bought together |
| **Transaction** | A set of items purchased together |
| **Itemset** | A collection of one or more items |
| **Association Rule** | X → Y ("if X then Y") |
| **Antecedent** | The "if" part (X) |
| **Consequent** | The "then" part (Y) |
| **Binary Matrix** | Rows = transactions, Columns = items, Values = 0/1 |
| **Key Algorithms** | Apriori, FP-Growth |
| **Key Metrics** | Support, Confidence, Lift |
| **Python Library** | `mlxtend` |

---

## ❓ Revision Questions

1. **What is market basket analysis and what type of patterns does it discover?**
   Define the concept and give three real-world examples from different domains.

2. **Define the following terms: item, itemset, transaction, transaction database, and association rule.**
   Provide concrete examples for each.

3. **Convert the following transactions into a binary transaction matrix:**
   T1: {A, B, C}, T2: {A, C}, T3: {B, D}, T4: {A, B, C, D}, T5: {B, C}

4. **Explain the difference between the antecedent and consequent of an association rule. Why must X ∩ Y = ∅?**
   Discuss the logical interpretation and what it would mean if they overlapped.

5. **Beyond retail, describe two real-world applications of association rule mining and the types of rules that would be useful.**
   Consider healthcare, web mining, or other domains.

6. **What are the main steps in the association rule mining pipeline?**
   Walk through from data collection to business application.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DR Comparison](../07-Other-Dimensionality-Reduction/05-comparison-of-methods.md) | [📂 Unsupervised Learning](../README.md) | [Support, Confidence, Lift →](./02-support-confidence-lift.md) |

---

> **Key Takeaway:** Market basket analysis transforms transaction data into actionable insights by discovering which items are frequently purchased together. The technique extends far beyond retail to healthcare, web analytics, telecommunications, and any domain where co-occurrence patterns matter.
