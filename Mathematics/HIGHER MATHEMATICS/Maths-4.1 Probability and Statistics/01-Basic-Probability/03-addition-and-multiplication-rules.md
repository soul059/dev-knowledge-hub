# Chapter 1.3: Addition and Multiplication Rules

[← Previous: Classical, Empirical, and Axiomatic Probability](02-classical-empirical-axiomatic-probability.md) | [Back to README](../README.md) | [Next: Conditional Probability →](04-conditional-probability.md)

---

## Overview

The **addition rule** tells us how to find the probability of the union of events ("A or B"), while the **multiplication rule** tells us how to find the probability of the intersection of events ("A and B"). These are the two most fundamental computational rules in probability.

---

## 1. Addition Rule (OR Rule)

### General Addition Rule

For **any** two events $A$ and $B$:

$$\boxed{P(A \cup B) = P(A) + P(B) - P(A \cap B)}$$

**Why subtract $P(A \cap B)$?** When we add $P(A)$ and $P(B)$, the outcomes in $A \cap B$ are counted **twice** — once in $P(A)$ and once in $P(B)$. We subtract once to correct for this double counting.

```
  Venn Diagram — Why We Subtract P(A ∩ B)
  
  ┌─────────────────────────── S ────────────────────────┐
  │                                                       │
  │       ┌──────────┐    ┌──────────┐                   │
  │      /            \  /            \                   │
  │     │  A only      ││   B only     │                  │
  │     │              ││              │                  │
  │     │    ┌─────────┤├──────────┐   │                  │
  │     │    │  A ∩ B   ││         │   │                  │
  │     │    │ (counted ││ twice)  │   │                  │
  │     │    └─────────┤├──────────┘   │                  │
  │      \            /  \            /                   │
  │       └──────────┘    └──────────┘                   │
  │                                                       │
  │  P(A∪B) = P(A) + P(B) - P(A∩B) ← Remove double count│
  └───────────────────────────────────────────────────────┘
```

### Special Case: Mutually Exclusive Events

If $A \cap B = \emptyset$ (mutually exclusive), then $P(A \cap B) = 0$:

$$\boxed{P(A \cup B) = P(A) + P(B) \quad \text{(mutually exclusive)}}$$

### Example 1: Drawing a Card

$A$ = "card is a Heart" → $P(A) = 13/52$  
$B$ = "card is a King" → $P(B) = 4/52$  
$A \cap B$ = "King of Hearts" → $P(A \cap B) = 1/52$  

$$P(A \cup B) = \frac{13}{52} + \frac{4}{52} - \frac{1}{52} = \frac{16}{52} = \frac{4}{13}$$

### Example 2: Mutually Exclusive

$A$ = "rolling a 2" → $P(A) = 1/6$  
$B$ = "rolling a 5" → $P(B) = 1/6$  
$A \cap B = \emptyset$ → Mutually exclusive

$$P(A \cup B) = \frac{1}{6} + \frac{1}{6} = \frac{2}{6} = \frac{1}{3}$$

---

## 2. Generalized Addition Rule (Three Events)

For **three** events $A$, $B$, and $C$:

$$P(A \cup B \cup C) = P(A) + P(B) + P(C) - P(A \cap B) - P(A \cap C) - P(B \cap C) + P(A \cap B \cap C)$$

### Inclusion-Exclusion Principle (General)

For $n$ events $A_1, A_2, \ldots, A_n$:

$$P\!\left(\bigcup_{i=1}^n A_i\right) = \sum_{i} P(A_i) - \sum_{i<j} P(A_i \cap A_j) + \sum_{i<j<k} P(A_i \cap A_j \cap A_k) - \cdots + (-1)^{n+1} P(A_1 \cap A_2 \cap \cdots \cap A_n)$$

```
  Inclusion-Exclusion for 3 Events — Visual
  
          ┌──A──┐
         /   ●   \          ● = A ∩ B ∩ C (add back)
        │  ┌──┼────┼──B──┐
        │  │  │    │      │
    ────┼──┼──●────┼──────┤────
        │  │  │    │      │
        │  └──┼────┼──────┘
         \   │   /
          └──┼──┘
             │
          ┌──C──┐
          └─────┘
  
  + Add singles: P(A) + P(B) + P(C)
  − Subtract pairs: P(A∩B) + P(A∩C) + P(B∩C)
  + Add back triple: P(A∩B∩C)
```

---

## 3. Multiplication Rule (AND Rule)

### General Multiplication Rule

For any two events $A$ and $B$:

$$\boxed{P(A \cap B) = P(A) \cdot P(B \mid A) = P(B) \cdot P(A \mid B)}$$

where $P(B \mid A)$ is the **conditional probability** of $B$ given $A$ (covered in detail in the next chapter).

### Special Case: Independent Events

If $A$ and $B$ are **independent** (occurrence of one doesn't affect the other):

$$\boxed{P(A \cap B) = P(A) \cdot P(B) \quad \text{(independent events)}}$$

### Extended Multiplication Rule

For three events:

$$P(A \cap B \cap C) = P(A) \cdot P(B \mid A) \cdot P(C \mid A \cap B)$$

For $n$ events:

$$P(A_1 \cap A_2 \cap \cdots \cap A_n) = P(A_1) \cdot P(A_2 \mid A_1) \cdot P(A_3 \mid A_1 \cap A_2) \cdots P(A_n \mid A_1 \cap \cdots \cap A_{n-1})$$

---

## 4. Decision Framework: Which Rule to Use

```
  ┌─────────────────────────────────────────────┐
  │  What are you asked to find?                │
  └──────────────────────┬──────────────────────┘
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
  ┌───────────────┐            ┌───────────────┐
  │  P(A OR B)?   │            │  P(A AND B)?  │
  │  P(A ∪ B)     │            │  P(A ∩ B)     │
  └───────┬───────┘            └───────┬───────┘
          │                            │
          ▼                            ▼
  ┌───────────────────┐        ┌───────────────────┐
  │ ADDITION RULE     │        │ MULTIPLICATION     │
  │                   │        │ RULE               │
  │ P(A) + P(B)       │        │                   │
  │ - P(A∩B)          │        │ P(A) · P(B|A)     │
  │                   │        │                   │
  │ If disjoint:      │        │ If independent:   │
  │ P(A) + P(B)       │        │ P(A) · P(B)       │
  └───────────────────┘        └───────────────────┘
```

---

## 5. Worked Examples

### Example 3: Defective Products

A factory produces items with two types of defects. Let:
- $P(A)$ = probability of defect A = 0.05
- $P(B)$ = probability of defect B = 0.03
- $P(A \cap B)$ = probability of both defects = 0.01

**Find:** Probability that an item has **at least one** defect.

$$P(A \cup B) = P(A) + P(B) - P(A \cap B) = 0.05 + 0.03 - 0.01 = 0.07$$

**Find:** Probability that an item has **no defect**.

$$P(\text{no defect}) = 1 - P(A \cup B) = 1 - 0.07 = 0.93$$

### Example 4: Sequential Drawing (Without Replacement)

A bag has 6 red and 4 blue balls. Two balls are drawn **without replacement**. Find $P(\text{both red})$.

**Step 1:** $P(\text{1st red}) = \frac{6}{10}$

**Step 2:** Given 1st is red, remaining = 5 red + 4 blue = 9 balls

$P(\text{2nd red} \mid \text{1st red}) = \frac{5}{9}$

**Result:**

$$P(\text{both red}) = P(R_1) \cdot P(R_2 \mid R_1) = \frac{6}{10} \cdot \frac{5}{9} = \frac{30}{90} = \frac{1}{3}$$

### Example 5: Sequential Drawing (With Replacement)

Same bag. Two balls drawn **with replacement**. Find $P(\text{both red})$.

Since drawing is with replacement, events are **independent**:

$$P(\text{both red}) = P(R_1) \cdot P(R_2) = \frac{6}{10} \cdot \frac{6}{10} = \frac{36}{100} = 0.36$$

### Example 6: System Reliability

```
  Series System (both must work):
  
  ──►[ Component A ]──►[ Component B ]──►
      P(A works)=0.9     P(B works)=0.95
  
  P(System works) = P(A) × P(B) = 0.9 × 0.95 = 0.855
  
  
  Parallel System (at least one must work):
  
       ┌──►[ Component A ]──┐
  ──►──┤    P(A works)=0.9   ├──►
       └──►[ Component B ]──┘
            P(B works)=0.95
  
  P(System works) = 1 - P(both fail)
                  = 1 - P(A') × P(B')
                  = 1 - (0.1)(0.05) = 1 - 0.005 = 0.995
```

---

## 6. Common Probability Relationships Table

| Relationship | Formula |
|-------------|---------|
| $P(A \cup B)$ | $P(A) + P(B) - P(A \cap B)$ |
| $P(A \cup B)$ (disjoint) | $P(A) + P(B)$ |
| $P(A \cap B)$ | $P(A) \cdot P(B \mid A)$ |
| $P(A \cap B)$ (independent) | $P(A) \cdot P(B)$ |
| $P(A^c)$ | $1 - P(A)$ |
| $P(\text{at least one})$ | $1 - P(\text{none})$ |
| $P(A \cup B \cup C)$ | $\sum P - \sum P(\text{pairs}) + P(\text{triple})$ |
| $P(A \cap B \cap C)$ (indep.) | $P(A) \cdot P(B) \cdot P(C)$ |

---

## 7. "At Least One" Problems

A very common pattern: finding $P(\text{at least one event occurs})$.

**Strategy:** Use the complement.

$$P(\text{at least one}) = 1 - P(\text{none occurs})$$

### Example 7

Three independent systems have failure probabilities 0.1, 0.2, and 0.15. Find the probability that **at least one** fails.

$$P(\text{at least one fails}) = 1 - P(\text{none fails})$$

$$= 1 - (1 - 0.1)(1 - 0.2)(1 - 0.15)$$

$$= 1 - (0.9)(0.8)(0.85)$$

$$= 1 - 0.612 = 0.388$$

---

## 8. Real-World Applications

| Application | Rule Used | Example |
|------------|-----------|---------|
| **System Reliability** | Multiplication (series), Addition (parallel) | Will the satellite system work? |
| **Medical Diagnosis** | Addition | P(patient has disease A or B) |
| **Insurance** | Both | P(at least one claim in a year) |
| **Quality Control** | Multiplication | P(3 consecutive items pass inspection) |
| **Genetics** | Multiplication | P(inheriting both recessive alleles) |
| **Network Security** | "At least one" | P(at least one firewall breached) |

---

## Summary Table

| Concept | Formula | Condition |
|---------|---------|-----------|
| Addition Rule (General) | $P(A \cup B) = P(A) + P(B) - P(A \cap B)$ | Any events |
| Addition Rule (ME) | $P(A \cup B) = P(A) + P(B)$ | $A \cap B = \emptyset$ |
| Multiplication Rule (General) | $P(A \cap B) = P(A) \cdot P(B \mid A)$ | Any events |
| Multiplication Rule (Indep.) | $P(A \cap B) = P(A) \cdot P(B)$ | Independent events |
| Complement Shortcut | $P(\text{at least one}) = 1 - P(\text{none})$ | Any scenario |
| Inclusion-Exclusion (3) | $\sum P - \sum P(\text{pairs}) + P(\text{triple})$ | Three events |
| Series System | $P = \prod P_i$ | Independent, all needed |
| Parallel System | $P = 1 - \prod (1 - P_i)$ | Independent, any one needed |

---

## Quick Revision Questions

1. A student applies to two colleges. $P(\text{accepted at A}) = 0.6$, $P(\text{accepted at B}) = 0.7$, and $P(\text{both}) = 0.45$. Find $P(\text{at least one acceptance})$.

2. **Derive** the inclusion-exclusion formula for three events starting from two-event addition rule.

3. A series system has 4 independent components with reliabilities 0.99, 0.98, 0.97, and 0.95. Find the system reliability. Why is it less than any individual component's reliability?

4. Three fair coins are tossed. Using the multiplication rule, find $P(\text{all heads})$ and $P(\text{at least one tail})$.

5. Events A and B satisfy $P(A) = 0.4$, $P(B) = 0.5$. Find $P(A \cup B)$ if (a) they are mutually exclusive, (b) they are independent.

6. In a parallel-series system below, find the overall reliability if each component has reliability 0.9. (*Hint: break into subsystems*)
```
       ┌──[ A ]──[ B ]──┐
  ──►──┤                  ├──►
       └──[ C ]──[ D ]──┘
```

---

[← Previous: Classical, Empirical, and Axiomatic Probability](02-classical-empirical-axiomatic-probability.md) | [Back to README](../README.md) | [Next: Conditional Probability →](04-conditional-probability.md)
