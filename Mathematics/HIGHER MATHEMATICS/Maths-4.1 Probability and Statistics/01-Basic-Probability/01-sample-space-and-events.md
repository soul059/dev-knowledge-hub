# Chapter 1.1: Sample Space and Events

[← Back to README](../README.md) | [Next: Classical, Empirical, and Axiomatic Probability →](02-classical-empirical-axiomatic-probability.md)

---

## Overview

The foundation of probability theory rests on three key concepts: **experiments**, **sample spaces**, and **events**. Before we can calculate any probability, we must precisely define what outcomes are possible and which outcomes we care about.

---

## 1. Random Experiment

A **random experiment** (or stochastic experiment) is a process that:
1. Has a well-defined set of possible outcomes
2. Can be repeated under identical conditions
3. Has an outcome that **cannot be predicted with certainty** before the experiment is performed

### Examples of Random Experiments

| Experiment | Possible Outcomes |
|-----------|-------------------|
| Toss a coin | Head (H), Tail (T) |
| Roll a die | 1, 2, 3, 4, 5, 6 |
| Draw a card from a deck | Any of 52 cards |
| Measure lifetime of a bulb | Any $t \geq 0$ hours |
| Count cars passing a toll | 0, 1, 2, 3, ... |

### Deterministic vs. Random

```
┌──────────────────────────────────────────────────────────┐
│           EXPERIMENT CLASSIFICATION                      │
├──────────────────────┬───────────────────────────────────┤
│   DETERMINISTIC      │       RANDOM (STOCHASTIC)         │
│                      │                                   │
│  Outcome is certain  │  Outcome is uncertain             │
│  before experiment   │  before experiment                │
│                      │                                   │
│  Example:            │  Example:                         │
│  Water boils at      │  Tossing a fair coin              │
│  100°C at 1 atm      │  → H or T (unknown beforehand)   │
└──────────────────────┴───────────────────────────────────┘
```

---

## 2. Sample Space ($S$ or $\Omega$)

The **sample space** is the set of **all possible outcomes** of a random experiment.

### Notation
- Usually denoted by $S$, $\Omega$, or $U$
- Each individual outcome is called a **sample point** (denoted $\omega$ or $s$)

### Types of Sample Spaces

#### Finite Sample Space
Contains a countable, finite number of outcomes.

**Example — Rolling a die:**
$$S = \{1, 2, 3, 4, 5, 6\}$$

**Example — Tossing two coins:**
$$S = \{HH, HT, TH, TT\}$$

#### Countably Infinite Sample Space
Contains infinitely many but countable outcomes.

**Example — Toss a coin until the first Head appears:**
$$S = \{H, TH, TTH, TTTH, TTTTH, \ldots\}$$

#### Uncountable (Continuous) Sample Space
Contains uncountably many outcomes (typically intervals of real numbers).

**Example — Measuring the lifetime of a light bulb:**
$$S = \{t : t \geq 0\} = [0, \infty)$$

### Visualization of Sample Spaces

```
  FINITE                 COUNTABLY INFINITE         UNCOUNTABLE
                                                    
  ┌─────────┐            ┌─────────────────┐        ◄──────────────────►
  │ • 1      │            │ • H              │        0        ∞
  │ • 2      │            │ • TH             │        Every real number
  │ • 3      │            │ • TTH            │        in the interval
  │ • 4      │            │ • TTTH           │        
  │ • 5      │            │ • TTTTH          │        Example:
  │ • 6      │            │ • ...            │        Time (t ≥ 0)
  └─────────┘            └─────────────────┘        Temperature
  Die roll               Coin toss until H          Weight
```

---

## 3. Events

An **event** is any **subset** of the sample space $S$.

### Types of Events

| Type | Description | Example (Die Roll) |
|------|-------------|-------------------|
| **Simple (Elementary)** | Contains exactly one outcome | $\{3\}$ |
| **Compound** | Contains two or more outcomes | $\{2, 4, 6\}$ (even numbers) |
| **Sure/Certain Event** | The entire sample space $S$ | $\{1,2,3,4,5,6\}$ |
| **Impossible Event** | The empty set $\emptyset$ | Rolling a 7 |
| **Complementary Event** | All outcomes NOT in event $A$ | If $A = \{1,2\}$, then $A' = \{3,4,5,6\}$ |

### Event Notation

- Events are denoted by capital letters: $A, B, C, \ldots$
- $A'$ or $\bar{A}$ or $A^c$ = complement of $A$
- $A \cup B$ = event A **or** B (or both) occurs
- $A \cap B$ = event A **and** B both occur
- $A - B$ or $A \setminus B$ = event A occurs but B does not

---

## 4. Set Operations on Events

Since events are sets, all set operations apply.

### Union ($A \cup B$) — "OR"

Event that **at least one** of $A$ or $B$ occurs.

$$A \cup B = \{\omega : \omega \in A \text{ or } \omega \in B\}$$

### Intersection ($A \cap B$) — "AND"

Event that **both** $A$ and $B$ occur.

$$A \cap B = \{\omega : \omega \in A \text{ and } \omega \in B\}$$

### Complement ($A^c$ or $A'$) — "NOT"

Event that $A$ does **not** occur.

$$A^c = \{\omega : \omega \in S, \omega \notin A\}$$

### Difference ($A - B$)

Event that $A$ occurs **but** $B$ does not.

$$A - B = A \cap B^c$$

### Venn Diagram Representation

```
    ┌───────────────────────────────────── S ─────────────────────────┐
    │                                                                 │
    │        ┌──────────────┐      ┌──────────────┐                  │
    │       /                \    /                \                  │
    │      │    A only        │  │     B only       │                 │
    │      │                  │  │                   │                 │
    │      │         ┌────────┼──┼───────┐          │                 │
    │      │         │ A ∩ B  │  │       │          │                 │
    │      │         │        │  │       │          │                 │
    │      │         └────────┼──┼───────┘          │                 │
    │       \                /    \                /                  │
    │        └──────────────┘      └──────────────┘                  │
    │                                                                 │
    │                    Neither A nor B = (A ∪ B)ᶜ                   │
    └─────────────────────────────────────────────────────────────────┘
```

### Key Relationships

| Property | Formula |
|----------|---------|
| Commutative | $A \cup B = B \cup A$; $A \cap B = B \cap A$ |
| Associative | $(A \cup B) \cup C = A \cup (B \cup C)$ |
| Distributive | $A \cap (B \cup C) = (A \cap B) \cup (A \cap C)$ |
| De Morgan's Laws | $(A \cup B)^c = A^c \cap B^c$; $(A \cap B)^c = A^c \cup B^c$ |
| Complement Laws | $A \cup A^c = S$; $A \cap A^c = \emptyset$ |
| Identity Laws | $A \cup \emptyset = A$; $A \cap S = A$ |

---

## 5. Mutually Exclusive (Disjoint) Events

Events $A$ and $B$ are **mutually exclusive** if they cannot occur simultaneously:

$$A \cap B = \emptyset$$

```
    ┌──────────────────────────────── S ──────────────────────────┐
    │                                                              │
    │     ┌──────────┐              ┌──────────┐                  │
    │     │    A     │              │    B     │                  │
    │     │          │              │          │                  │
    │     │          │    (gap)     │          │                  │
    │     │          │              │          │                  │
    │     └──────────┘              └──────────┘                  │
    │                                                              │
    │          No overlap → A ∩ B = ∅                              │
    └──────────────────────────────────────────────────────────────┘
```

**Example (Die Roll):**
- $A$ = "getting an even number" = $\{2, 4, 6\}$
- $B$ = "getting an odd number" = $\{1, 3, 5\}$
- $A \cap B = \emptyset$ → Mutually exclusive

---

## 6. Exhaustive Events

Events $A_1, A_2, \ldots, A_n$ are **exhaustive** if their union covers the entire sample space:

$$A_1 \cup A_2 \cup \cdots \cup A_n = S$$

If events are **both mutually exclusive and exhaustive**, they form a **partition** of $S$:

```
    ┌────────────────────────── S ────────────────────────────┐
    │          │            │           │           │          │
    │   A₁     │    A₂      │    A₃     │    A₄     │   A₅    │
    │          │            │           │           │          │
    └──────────┴────────────┴───────────┴───────────┴─────────┘
    
    Union = S,  All Aᵢ ∩ Aⱼ = ∅ for i ≠ j
    → This is a PARTITION of S
```

---

## 7. Worked Examples

### Example 1: Two Dice

**Experiment:** Roll two fair dice.

**Sample Space:** $S = \{(i, j) : i, j \in \{1,2,3,4,5,6\}\}$

Total outcomes = $6 \times 6 = 36$

```
        Die 2 →   1      2      3      4      5      6
Die 1 ↓
  1          (1,1)  (1,2)  (1,3)  (1,4)  (1,5)  (1,6)
  2          (2,1)  (2,2)  (2,3)  (2,4)  (2,5)  (2,6)
  3          (3,1)  (3,2)  (3,3)  (3,4)  (3,5)  (3,6)
  4          (4,1)  (4,2)  (4,3)  (4,4)  (4,5)  (4,6)
  5          (5,1)  (5,2)  (5,3)  (5,4)  (5,5)  (5,6)
  6          (6,1)  (6,2)  (6,3)  (6,4)  (6,5)  (6,6)
```

**Define Events:**
- $A$ = "sum is 7" = $\{(1,6),(2,5),(3,4),(4,3),(5,2),(6,1)\}$ → $|A| = 6$
- $B$ = "both dice show same number" = $\{(1,1),(2,2),(3,3),(4,4),(5,5),(6,6)\}$ → $|B| = 6$
- $A \cap B = \emptyset$ (no pair sums to 7 with equal dice) → Mutually exclusive

### Example 2: Drawing Cards

**Experiment:** Draw one card from a standard 52-card deck.

- $A$ = "card is a Heart" → $|A| = 13$
- $B$ = "card is a King" → $|B| = 4$
- $A \cap B$ = "King of Hearts" → $|A \cap B| = 1$
- $A \cup B$ = "Heart or King" → $|A \cup B| = 13 + 4 - 1 = 16$

---

## 8. Real-World Applications

| Domain | Experiment | Sample Space |
|--------|-----------|-------------|
| **Quality Control** | Test if a product is defective | $\{D, D'\}$ (defective, not defective) |
| **Medicine** | Patient response to treatment | $\{+, -, \text{no change}\}$ |
| **Finance** | Daily stock price change | $(-\infty, +\infty)$ |
| **Networking** | Packets arriving per second | $\{0, 1, 2, \ldots\}$ |
| **Weather** | Tomorrow's temperature | $[-50°C, 60°C]$ |

---

## Summary Table

| Concept | Definition | Key Property |
|---------|-----------|-------------|
| Random Experiment | Process with uncertain outcome | Repeatable |
| Sample Space ($S$) | Set of all possible outcomes | Can be finite, countably infinite, or uncountable |
| Event | Any subset of $S$ | Can be simple or compound |
| Certain Event | $S$ itself | Always occurs |
| Impossible Event | $\emptyset$ | Never occurs |
| Complement $A^c$ | Outcomes not in $A$ | $A \cup A^c = S$ |
| Union $A \cup B$ | $A$ or $B$ occurs | "OR" operation |
| Intersection $A \cap B$ | Both $A$ and $B$ occur | "AND" operation |
| Mutually Exclusive | $A \cap B = \emptyset$ | Cannot co-occur |
| Exhaustive | $\bigcup A_i = S$ | Cover all outcomes |
| Partition | Mutually exclusive + exhaustive | Divides $S$ completely |

---

## Quick Revision Questions

1. **Define** a sample space. What are the three types based on the number of outcomes?

2. **List** the sample space when three coins are tossed simultaneously. How many sample points does it contain?

3. If $A = \{1, 2, 3\}$ and $B = \{3, 4, 5\}$ in $S = \{1,2,3,4,5,6\}$, find $A \cup B$, $A \cap B$, $A^c$, and $A - B$.

4. **State** De Morgan's Laws. Verify them with an example using a die roll.

5. Two events $A$ and $B$ are mutually exclusive with $A = \{1, 3\}$ and $B = \{2, 4\}$. Are they exhaustive for $S = \{1, 2, 3, 4, 5, 6\}$? Why or why not?

6. Give an example of a real-world experiment with (a) a finite sample space, (b) a countably infinite sample space, and (c) an uncountable sample space.

---

[← Back to README](../README.md) | [Next: Classical, Empirical, and Axiomatic Probability →](02-classical-empirical-axiomatic-probability.md)
