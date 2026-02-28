# Chapter 1.5: Independent Events

[← Previous: Conditional Probability](04-conditional-probability.md) | [Back to README](../README.md) | [Next: Bayes' Theorem →](06-bayes-theorem.md)

---

## Overview

Two events are **independent** if the occurrence of one does not affect the probability of the other. Independence simplifies calculations dramatically and is a core assumption in many statistical models.

---

## 1. Definition of Independence

### Two Events

Events $A$ and $B$ are **independent** if and only if:

$$\boxed{P(A \cap B) = P(A) \cdot P(B)}$$

**Equivalent conditions** (any one implies the others):
1. $P(A \cap B) = P(A) \cdot P(B)$
2. $P(A \mid B) = P(A)$ (knowing $B$ doesn't change $A$)
3. $P(B \mid A) = P(B)$ (knowing $A$ doesn't change $B$)

### Dependent Events

Events are **dependent** if $P(A \cap B) \neq P(A) \cdot P(B)$.

```
  ┌──────────────────── INDEPENDENCE ─────────────────────┐
  │                                                        │
  │  A and B are INDEPENDENT                               │
  │                                                        │
  │    ⟺  P(A ∩ B) = P(A) · P(B)                          │
  │    ⟺  P(A | B) = P(A)                                  │
  │    ⟺  P(B | A) = P(B)                                  │
  │                                                        │
  │  "Knowing B happened tells us NOTHING about A"         │
  └────────────────────────────────────────────────────────┘
  
  ┌──────────────────── DEPENDENCE ────────────────────────┐
  │                                                        │
  │  A and B are DEPENDENT                                 │
  │                                                        │
  │    ⟺  P(A ∩ B) ≠ P(A) · P(B)                          │
  │    ⟺  P(A | B) ≠ P(A)                                  │
  │                                                        │
  │  "Knowing B happened CHANGES our belief about A"       │
  └────────────────────────────────────────────────────────┘
```

---

## 2. Independence vs. Mutual Exclusivity

This is a **very common source of confusion**. These are **completely different** concepts:

| Property | Mutually Exclusive | Independent |
|----------|-------------------|-------------|
| Definition | $A \cap B = \emptyset$ | $P(A \cap B) = P(A) \cdot P(B)$ |
| Meaning | Cannot happen together | Don't influence each other |
| $P(A \cap B)$ | $= 0$ | $= P(A) \cdot P(B)$ |
| Can both be true? | Only if $P(A) = 0$ or $P(B) = 0$ | — |
| Relationship | If ME and both have $P > 0$, then **dependent** | If independent and both have $P > 0$, then **not ME** |

### Key Insight

> If $A$ and $B$ are **mutually exclusive** (and both have positive probability), they are **dependent** — because knowing $A$ occurred guarantees $B$ did NOT occur.

```
  MUTUALLY EXCLUSIVE           INDEPENDENT
  (Cannot co-occur)            (Don't influence each other)
  
  ┌──────── S ────────┐        ┌──────── S ────────┐
  │ ┌───┐     ┌───┐   │        │   ┌──────┐        │
  │ │ A │     │ B │   │        │  /  A   /\         │
  │ │   │     │   │   │        │ │  ┌───┤  │        │
  │ └───┘     └───┘   │        │ │  │A∩B│  │ B      │
  │  No overlap        │        │ │  └───┤  │        │
  │  A ∩ B = ∅         │        │  \    /  \/        │
  │                    │        │   └──────┘         │
  │  Dependent!        │        │ P(A∩B) = P(A)·P(B) │
  └────────────────────┘        └────────────────────┘
```

---

## 3. Testing for Independence

To check if events $A$ and $B$ are independent:

**Step 1:** Compute $P(A)$, $P(B)$, and $P(A \cap B)$  
**Step 2:** Check if $P(A \cap B) = P(A) \cdot P(B)$

### Example 1: Die Roll

Roll a fair die. $A$ = "even" = $\{2,4,6\}$, $B$ = "≤ 3" = $\{1,2,3\}$.

- $P(A) = 3/6 = 1/2$
- $P(B) = 3/6 = 1/2$
- $A \cap B = \{2\}$ → $P(A \cap B) = 1/6$
- $P(A) \cdot P(B) = (1/2)(1/2) = 1/4 = 3/12$

Since $1/6 \neq 1/4$, $A$ and $B$ are **not independent** (dependent).

### Example 2: Two Coins

Toss two fair coins. $A$ = "1st coin is H", $B$ = "2nd coin is T".

- $S = \{HH, HT, TH, TT\}$
- $P(A) = 2/4 = 1/2$
- $P(B) = 2/4 = 1/2$
- $A \cap B = \{HT\}$ → $P(A \cap B) = 1/4$
- $P(A) \cdot P(B) = (1/2)(1/2) = 1/4$ ✓

Since $P(A \cap B) = P(A) \cdot P(B)$, $A$ and $B$ are **independent**.

---

## 4. Independence of Multiple Events

### Pairwise Independence

Events $A_1, A_2, \ldots, A_n$ are **pairwise independent** if:

$$P(A_i \cap A_j) = P(A_i) \cdot P(A_j) \quad \text{for all } i \neq j$$

### Mutual Independence

Events $A_1, A_2, \ldots, A_n$ are **mutually independent** if for **every** subset $\{i_1, i_2, \ldots, i_k\}$:

$$P(A_{i_1} \cap A_{i_2} \cap \cdots \cap A_{i_k}) = P(A_{i_1}) \cdot P(A_{i_2}) \cdots P(A_{i_k})$$

> **Warning:** Pairwise independence does **NOT** imply mutual independence!

### For Three Events ($A$, $B$, $C$)

Mutual independence requires ALL FOUR conditions:

1. $P(A \cap B) = P(A) \cdot P(B)$
2. $P(A \cap C) = P(A) \cdot P(C)$
3. $P(B \cap C) = P(B) \cdot P(C)$
4. $P(A \cap B \cap C) = P(A) \cdot P(B) \cdot P(C)$

Conditions 1–3 alone give only pairwise independence.

---

## 5. Counterexample: Pairwise ≠ Mutual

### The "Two Fair Coins" Example

Toss two fair coins. Define:
- $A$ = "1st coin is Head" = $\{HH, HT\}$
- $B$ = "2nd coin is Head" = $\{HH, TH\}$
- $C$ = "both coins show same face" = $\{HH, TT\}$

**Check pairwise:**
- $P(A) = P(B) = P(C) = 1/2$
- $P(A \cap B) = P(\{HH\}) = 1/4 = P(A)P(B)$ ✓
- $P(A \cap C) = P(\{HH\}) = 1/4 = P(A)P(C)$ ✓
- $P(B \cap C) = P(\{HH\}) = 1/4 = P(B)P(C)$ ✓

All pairs are independent!

**Check triple:**
- $P(A \cap B \cap C) = P(\{HH\}) = 1/4$
- $P(A) \cdot P(B) \cdot P(C) = (1/2)^3 = 1/8$

$1/4 \neq 1/8$ → **NOT mutually independent**, despite being pairwise independent!

---

## 6. Properties of Independent Events

If $A$ and $B$ are independent, then so are:

| Statement | Justification |
|-----------|--------------|
| $A$ and $B^c$ are independent | $P(A \cap B^c) = P(A) - P(A \cap B) = P(A) - P(A)P(B) = P(A)(1-P(B)) = P(A)P(B^c)$ |
| $A^c$ and $B$ are independent | By symmetry |
| $A^c$ and $B^c$ are independent | $P(A^c \cap B^c) = 1 - P(A \cup B) = 1 - P(A) - P(B) + P(A)P(B) = (1-P(A))(1-P(B))$ |

```
  Independence Preservation Under Complementation
  
  A ⊥ B  ⟹  A ⊥ Bᶜ  ⟹  Aᶜ ⊥ Bᶜ
             Aᶜ ⊥ B
  
  (⊥ means "independent of")
```

---

## 7. Applications of Independence

### Example 3: System Reliability

A system has 3 independent components, each with reliability 0.95.

**Series System** (all must work):
$$P(\text{system works}) = (0.95)^3 = 0.857$$

**Parallel System** (at least one must work):
$$P(\text{system works}) = 1 - (1 - 0.95)^3 = 1 - (0.05)^3 = 1 - 0.000125 = 0.9999$$

```
  Series (all needed):     Parallel (any one):
  
  ─[A]─[B]─[C]─           ─┬─[A]─┬─
                             ├─[B]─┤
  P = 0.95³ = 0.857         ├─[C]─┤
                             └─────┘
                           P = 1-(0.05)³ = 0.9999
```

### Example 4: Repeated Independent Trials

A fair coin is tossed 10 times. Find $P(\text{all heads})$.

Since tosses are independent:

$$P(\text{all H}) = \left(\frac{1}{2}\right)^{10} = \frac{1}{1024} \approx 0.000977$$

### Example 5: Password Security

A 4-digit PIN (0–9 each). What's $P(\text{guess correctly})$ on first try?

Digits are independent: $P = (1/10)^4 = 1/10000 = 0.0001$

---

## 8. Conditional Independence

Events $A$ and $B$ are **conditionally independent given $C$** if:

$$P(A \cap B \mid C) = P(A \mid C) \cdot P(B \mid C)$$

> **Important:** Independence and conditional independence are **separate** properties. Neither implies the other.

### Example

Students' test scores $A$ and $B$ might be independent in the general population but **dependent** given that they are in the same study group (event $C$).

---

## 9. Real-World Applications

| Application | Why Independence Matters |
|------------|------------------------|
| **Coin/Dice Games** | Each toss/roll is independent of previous ones |
| **Manufacturing** | Defects in separate items assumed independent |
| **Communication** | Bit errors in different channels assumed independent |
| **Clinical Trials** | Patient outcomes assumed independent |
| **Monte Carlo Simulation** | Random samples must be independent |
| **Machine Learning** | Naive Bayes assumes feature independence |

---

## Summary Table

| Concept | Key Fact |
|---------|---------|
| Independence (2 events) | $P(A \cap B) = P(A) \cdot P(B)$ |
| Equivalent condition | $P(A \mid B) = P(A)$ |
| Independent ≠ Mutually Exclusive | ME events (with $P>0$) are **dependent** |
| Pairwise independence | All pairs satisfy the product rule |
| Mutual independence | All subsets satisfy the product rule |
| Pairwise ⇏ Mutual | Counterexample exists (two-coin example) |
| Complement preservation | $A \perp B \implies A \perp B^c \implies A^c \perp B^c$ |
| $n$ independent trials | $P(\text{all}) = \prod P(A_i)$ |
| Conditional independence | $P(A \cap B \mid C) = P(A\mid C) \cdot P(B\mid C)$ |

---

## Quick Revision Questions

1. Events $A$ and $B$ satisfy $P(A) = 0.3$, $P(B) = 0.4$. If they are independent, find $P(A \cap B)$, $P(A \cup B)$, and $P(A \mid B)$.

2. **Explain** why mutually exclusive events (with positive probabilities) must be dependent. Give an intuitive argument and a mathematical proof.

3. A system has two components in parallel, each with reliability 0.8, feeding into a single component with reliability 0.9 (in series). Find the overall system reliability assuming independence.

4. Give an example where events are pairwise independent but not mutually independent. Why does the distinction matter?

5. A fair die is rolled twice. Let $A$ = "first roll > 3" and $B$ = "sum > 8". Are $A$ and $B$ independent?

6. A four-letter password uses uppercase letters (26 choices per position). If each position is chosen independently, what is the probability of guessing the password on the first try?

---

[← Previous: Conditional Probability](04-conditional-probability.md) | [Back to README](../README.md) | [Next: Bayes' Theorem →](06-bayes-theorem.md)
