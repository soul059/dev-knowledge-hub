# Chapter 2.1: Fundamental Counting Principle

[← Previous: Bayes' Theorem](../01-Basic-Probability/06-bayes-theorem.md) | [Back to README](../README.md) | [Next: Permutations →](02-permutations.md)

---

## Overview

The **Fundamental Counting Principle** (also called the Rule of Product) is the cornerstone of combinatorics. It provides a systematic way to count the total number of outcomes when a procedure consists of multiple steps.

---

## 1. The Principle

### Multiplication Principle

If a procedure can be broken into $k$ successive stages, where:
- Stage 1 can be done in $n_1$ ways
- Stage 2 can be done in $n_2$ ways
- $\vdots$
- Stage $k$ can be done in $n_k$ ways

Then the **total number** of ways to complete the entire procedure is:

$$\boxed{N = n_1 \times n_2 \times \cdots \times n_k}$$

### Condition

Each stage must be **independent** of the others — the number of choices at each stage does not depend on choices made in previous stages. (If it does, we use a modified counting approach — see tree diagrams.)

```
  Multiplication Principle — Visual
  
  Stage 1       Stage 2       Stage 3       Total
  (n₁ ways)     (n₂ ways)     (n₃ ways)     Outcomes
  
    ┌─ b₁ ──── c₁             a₁ b₁ c₁
    ├─ b₁ ──── c₂             a₁ b₁ c₂
  a₁┤    ...                     ...
    ├─ b₂ ──── c₁             a₁ b₂ c₁
    └─ b₂ ──── c₂             a₁ b₂ c₂
  
    ┌─ b₁ ──── c₁             a₂ b₁ c₁
  a₂┤    ...                     ...
    └─ b₂ ──── c₂             a₂ b₂ c₂
  
  Total = n₁ × n₂ × n₃
```

---

## 2. Addition Principle

If a task can be done **either** by method $A$ (in $m$ ways) **or** by method $B$ (in $n$ ways), and the two methods are **mutually exclusive**, then:

$$\boxed{N = m + n}$$

### When to Use Which

| Keyword | Principle | Operation |
|---------|-----------|-----------|
| **AND** / "then" / sequential | Multiplication | $\times$ |
| **OR** / "either...or" / alternative | Addition | $+$ |

---

## 3. Examples

### Example 1: Outfit Selection

A person has 4 shirts, 3 pants, and 2 pairs of shoes. How many different outfits are possible?

$$N = 4 \times 3 \times 2 = 24 \text{ outfits}$$

### Example 2: License Plates

A license plate has 3 letters followed by 4 digits. How many plates are possible (with repetition)?

$$N = 26 \times 26 \times 26 \times 10 \times 10 \times 10 \times 10 = 26^3 \times 10^4 = 17{,}576 \times 10{,}000 = 175{,}760{,}000$$

### Example 3: Routes Between Cities

```
  Cities A, B, C with routes:
  
  A ═══╦═══ B ═══╦═══ C
       ║         ║
  3 routes    2 routes
  A → B       B → C
  
  Total routes A → C = 3 × 2 = 6
```

### Example 4: PIN Codes

A 4-digit PIN where no digit repeats:

$$N = 10 \times 9 \times 8 \times 7 = 5{,}040$$

(First digit: 10 choices, second: 9, third: 8, fourth: 7)

### Example 5: Committee Travel

A committee member travels from City X to City Z, possibly via City Y.
- Direct routes X → Z: 2
- Routes X → Y: 3, Routes Y → Z: 4

$$N = 2 + (3 \times 4) = 2 + 12 = 14 \text{ total routes}$$

(Addition for "direct OR via Y"; multiplication for the "via Y" path)

---

## 4. Tree Diagrams for Counting

When choices at later stages depend on earlier ones, tree diagrams systematically enumerate all possibilities.

### Example 6: Seating Choice

2 people (A, B) arrange themselves in 2 seats:

```
  Person 1 choice:     Person 2 choice:     Arrangement:
  
       ┌── B ──────────  A, B
   A ──┤
       └── (only B left)
  
       ┌── A ──────────  B, A
   B ──┤
       └── (only A left)
  
  Total arrangements = 2 × 1 = 2! = 2
```

### Example 7: Coin Toss (3 times)

```
                        Start
                       /     \
                     H         T
                    / \       / \
                  H     T   H     T
                 / \   / \ / \   / \
                H   T H  TH  T H   T
  Outcomes:   HHH HHT HTH HTT THH THT TTH TTT
  
  Total = 2 × 2 × 2 = 2³ = 8
```

---

## 5. Advanced Applications

### Counting with Restrictions

**Example 8:** How many 3-digit even numbers can be formed using digits $\{1, 2, 3, 4, 5\}$ without repetition?

**Strategy:** Start with the most restrictive position first (the last digit must be even).

- Last digit (units): Must be even → $\{2, 4\}$ → 2 choices
- First digit (hundreds): Any of the remaining 4 digits → 4 choices
- Middle digit (tens): Any of the remaining 3 digits → 3 choices

$$N = 4 \times 3 \times 2 = 24$$

(Note: We fill the restricted position first, then work outward.)

### Counting with "At Least One" Constraint

**Example 9:** How many 3-digit numbers (using 0–9 with repetition) have at least one digit equal to 5?

**Strategy:** Total − (none have 5)

- Total 3-digit numbers: $9 \times 10 \times 10 = 900$ (first digit 1–9)
- 3-digit numbers with NO 5: first digit has 8 choices (1–9 except 5), other digits have 9 each
  - $8 \times 9 \times 9 = 648$
- At least one 5: $900 - 648 = 252$

---

## 6. The Pigeonhole Principle

If $n$ items are placed into $m$ containers and $n > m$, then at least one container has more than one item.

### Generalized Version

If $n$ items are placed into $m$ containers, at least one container has at least $\lceil n/m \rceil$ items.

### Example 10

In any group of 13 people, at least 2 share the same birth month.
- 13 people, 12 months → $\lceil 13/12 \rceil = 2$ → At least 2 share a month. ✓

---

## 7. Real-World Applications

| Application | Counting Principle Used |
|------------|------------------------|
| **Password strength** | Multiplication: characters × positions |
| **Menu combinations** | Multiplication: appetizer × entrée × dessert |
| **Network routing** | Addition + Multiplication for path counting |
| **Genetic codes** | $4^3 = 64$ codons (3 nucleotides, 4 bases) |
| **IP addresses (IPv4)** | $256^4 = 4.3 \times 10^9$ possible addresses |
| **Phone numbers** | Multiplication with area code restrictions |

---

## Summary Table

| Concept | Formula | When to Use |
|---------|---------|-------------|
| Multiplication Principle | $N = n_1 \times n_2 \times \cdots \times n_k$ | Sequential stages ("and") |
| Addition Principle | $N = n_1 + n_2 + \cdots$ | Alternative methods ("or") |
| With repetition | Each stage has full set of choices | Digits/letters can repeat |
| Without repetition | Choices decrease by 1 each stage | No element used twice |
| Complement counting | Total − (unwanted) | "At least one" problems |
| Pigeonhole Principle | $n > m \implies$ some container has $\geq 2$ | Existence proofs |

---

## Quick Revision Questions

1. A restaurant offers 5 appetizers, 8 main courses, and 4 desserts. How many different 3-course meals can you order?

2. How many 5-letter "words" (not necessarily meaningful) can be formed from the English alphabet (a) with repetition, (b) without repetition?

3. How many integers between 100 and 999 have all distinct digits?

4. A committee must travel from A to D. There are 3 routes A→B, 2 routes B→C, 4 routes C→D, and 2 direct routes A→D. How many total routes from A to D exist?

5. How many binary strings of length 8 have at least one '1'?

6. In a class of 367 students, prove that at least two students share the same birthday. Which principle applies?

---

[← Previous: Bayes' Theorem](../01-Basic-Probability/06-bayes-theorem.md) | [Back to README](../README.md) | [Next: Permutations →](02-permutations.md)
