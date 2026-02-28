# Chapter 6.1: Counting Principles

[← Previous: Growth of Functions](../05-Functions/05-growth-of-functions.md) | [Back to README](../README.md) | [Next: Permutations →](02-permutations.md)

---

## 📋 Chapter Overview

Combinatorics begins with two fundamental rules — the **sum rule** and the **product rule** — that form the foundation for all counting arguments.

---

## 1. The Sum Rule (Rule of Addition)

If task $T$ can be done in one of two mutually exclusive ways — $n_1$ ways or $n_2$ ways — then $T$ can be done in $n_1 + n_2$ ways.

**General form:** If $A_1, A_2, \ldots, A_k$ are **pairwise disjoint** sets:

$$|A_1 \cup A_2 \cup \cdots \cup A_k| = |A_1| + |A_2| + \cdots + |A_k|$$

```
  Sum Rule: Choose from Group A OR Group B
  
  Group A: {🍎 🍊 🍋}      3 choices
  Group B: {🚗 🚌}          2 choices
  
  Total choices = 3 + 2 = 5
  (choosing from A OR B, not both)
```

**Example:** A student can choose a CS elective (5 options) or a Math elective (4 options), but not both. Total choices = $5 + 4 = 9$.

---

## 2. The Product Rule (Rule of Multiplication)

If task $T$ consists of two sequential steps — $n_1$ ways for step 1 and $n_2$ ways for step 2 — then $T$ can be done in $n_1 \times n_2$ ways.

**General form:**

$$|A_1 \times A_2 \times \cdots \times A_k| = |A_1| \cdot |A_2| \cdots |A_k|$$

```
  Product Rule: Choose from Group A AND Group B
  
  Shirts: {Red, Blue, Green}     3 choices
  Pants:  {Black, White}          2 choices
  
  Outfits = 3 × 2 = 6:
  
  ┌──────────────────────────────────────┐
  │  Red-Black    Blue-Black   Green-Black│
  │  Red-White    Blue-White   Green-White│
  └──────────────────────────────────────┘
```

**Example:** License plates with 3 letters followed by 4 digits: $26^3 \times 10^4 = 175,760,000$.

---

## 3. Inclusion-Exclusion Principle

When sets **overlap**, the sum rule overcounts. Use:

$$|A \cup B| = |A| + |B| - |A \cap B|$$

For three sets:

$$|A \cup B \cup C| = |A| + |B| + |C| - |A \cap B| - |A \cap C| - |B \cap C| + |A \cap B \cap C|$$

```
  Inclusion-Exclusion for 2 sets:
  
      ┌──────────────────────────┐
      │    A    ┌────────┐       │
      │   ┌────┤ A ∩ B  ├────┐  │
      │   │    └────────┘    │  │
      │   │  A only  B only  │  │
      │   └──────────────────┘  │
      │           B              │
      └──────────────────────────┘
  
  |A ∪ B| = |A| + |B| − |A ∩ B|
  (subtract overlap to avoid double-counting)
```

**Example:** How many integers from 1 to 100 are divisible by 3 or 5?

- Divisible by 3: $\lfloor 100/3 \rfloor = 33$
- Divisible by 5: $\lfloor 100/5 \rfloor = 20$
- Divisible by 15: $\lfloor 100/15 \rfloor = 6$
- Answer: $33 + 20 - 6 = 47$

---

## 4. Complement Counting

Sometimes it's easier to count what you **don't** want:

$$|\text{desired}| = |\text{total}| - |\text{undesired}|$$

**Example:** How many 4-digit numbers have at least one repeated digit?

- Total 4-digit numbers: $9 \times 10^3 = 9000$
- No repeated digits: $9 \times 9 \times 8 \times 7 = 4536$
- At least one repeat: $9000 - 4536 = 4464$

---

## 5. Division Rule

If a task can be done $N$ ways, but each outcome is counted exactly $d$ times, then the number of distinct outcomes is:

$$\frac{N}{d}$$

**Example:** How many ways to seat 4 people around a circular table?

$$\frac{4!}{4} = \frac{24}{4} = 6$$

(Divide by 4 because rotations of the same arrangement are identical.)

---

## 6. Tree Diagrams

Visualize sequential choices:

```
  Coin flipped twice:
  
            Start
           /     \
          H       T
         / \     / \
        H   T   H   T
  
  Outcomes: HH, HT, TH, TT
  Count: 2 × 2 = 4
```

```
  Menu: 2 mains, 3 desserts
  
              Start
             /     \
         Pasta    Steak
         / | \    / | \
        C  I  P  C  I  P
  
  C=Cake, I=Ice cream, P=Pie
  Total: 2 × 3 = 6 meals
```

---

## 7. Comprehensive Example: Password Counting

**Passwords:** 6–8 characters, using lowercase letters (26) and digits (10).

| Length | Total passwords |
|:------:|:---:|
| 6 | $36^6 = 2,176,782,336$ |
| 7 | $36^7 = 78,364,164,096$ |
| 8 | $36^8 = 2,821,109,907,456$ |
| **Total** | $36^6 + 36^7 + 36^8$ |

**Passwords with at least one digit:**

Total − (all letters) = $36^k - 26^k$ for length $k$.

| Length $k$ | All letters | At least one digit |
|:---:|:---:|:---:|
| 6 | $26^6 = 308,915,776$ | $36^6 - 26^6 = 1,867,866,560$ |
| 8 | $26^8 = 208,827,064,576$ | $36^8 - 26^8 = 2,612,282,842,880$ |

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Counting Principles in Practice           │
  │                                                   │
  │  1. Password Strength Analysis                   │
  │     Product rule → total password space           │
  │     Larger space → harder to brute-force          │
  │                                                   │
  │  2. Network Routing                              │
  │     Count possible paths between nodes            │
  │                                                   │
  │  3. Probability                                   │
  │     P(event) = favorable / total outcomes         │
  │     Need counting for both!                       │
  │                                                   │
  │  4. Database Query Optimization                  │
  │     Estimate result sizes using counting rules    │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Principle | When to Use | Formula |
|-----------|------------|---------|
| Sum Rule | Choose one of several disjoint options | $n_1 + n_2 + \cdots$ |
| Product Rule | Sequential independent choices | $n_1 \times n_2 \times \cdots$ |
| Inclusion-Exclusion | Overlapping sets | $\|A \cup B\| = \|A\| + \|B\| - \|A \cap B\|$ |
| Complement | "At least one" problems | $\|S\| - \|\text{unwanted}\|$ |
| Division Rule | Overcounting by symmetry | $N / d$ |

---

## ❓ Quick Revision Questions

1. **A restaurant offers 5 appetizers, 8 entrees, and 4 desserts. How many different 3-course meals are possible?**

2. **How many bit strings of length 8 start with 1 or end with 00?**

3. **How many integers from 1 to 1000 are divisible by 2 or 3?**

4. **How many 3-letter "words" can be formed from the English alphabet with no repeated letters?**

5. **Use the complement method to find: how many 5-bit strings have at least one 1?**

6. **In how many ways can 5 people be seated at a round table?**

---

[← Previous: Growth of Functions](../05-Functions/05-growth-of-functions.md) | [Back to README](../README.md) | [Next: Permutations →](02-permutations.md)
