# Chapter 2.2: Permutations (With and Without Repetition)

[← Previous: Fundamental Counting Principle](01-fundamental-counting-principle.md) | [Back to README](../README.md) | [Next: Combinations →](03-combinations.md)

---

## Overview

A **permutation** is an arrangement of objects where **order matters**. "ABC" and "BAC" are different permutations. This chapter covers permutations without repetition, with repetition, and circular permutations.

---

## 1. Permutations Without Repetition

### All Objects

The number of ways to arrange $n$ **distinct** objects in a line is:

$$\boxed{P(n) = n! = n \times (n-1) \times (n-2) \times \cdots \times 2 \times 1}$$

**Convention:** $0! = 1$

| $n$ | $n!$ |
|-----|------|
| 0 | 1 |
| 1 | 1 |
| 2 | 2 |
| 3 | 6 |
| 4 | 24 |
| 5 | 120 |
| 6 | 720 |
| 7 | 5,040 |
| 8 | 40,320 |
| 10 | 3,628,800 |

### Selecting $r$ from $n$ (Without Repetition)

The number of ways to select and arrange $r$ objects from $n$ distinct objects:

$$\boxed{P(n, r) = \frac{n!}{(n-r)!} = n \times (n-1) \times \cdots \times (n-r+1)}$$

**Logic:** First position has $n$ choices, second has $n-1$, ..., $r$-th position has $n-r+1$ choices.

```
  Permutation P(n, r) — Selection Process
  
  Position:    1st      2nd      3rd      ...    r-th
  Choices:      n      (n-1)    (n-2)    ...   (n-r+1)
               ┃        ┃        ┃               ┃
               ▼        ▼        ▼               ▼
            ┌──────┐ ┌──────┐ ┌──────┐       ┌──────┐
            │ Pick │ │ Pick │ │ Pick │  ...  │ Pick │
            │  1st │ │  2nd │ │  3rd │       │  rth │
            └──────┘ └──────┘ └──────┘       └──────┘
  
  Total = n × (n-1) × (n-2) × ... × (n-r+1) = n!/(n-r)!
```

### Example 1

How many 3-letter "words" from {A, B, C, D, E} without repeating letters?

$$P(5, 3) = \frac{5!}{(5-3)!} = \frac{5!}{2!} = \frac{120}{2} = 60$$

### Example 2

8 runners in a race. How many ways can gold, silver, bronze be awarded?

$$P(8, 3) = 8 \times 7 \times 6 = 336$$

---

## 2. Permutations With Repetition

### All Distinct, Repetition Allowed

The number of ways to fill $r$ positions from $n$ distinct objects, where each object **can be used multiple times**:

$$\boxed{n^r}$$

### Example 3

How many 4-digit PINs (digits 0–9, repetition allowed)?

$$10^4 = 10{,}000$$

### Permutations of Multisets (Identical Objects)

If we have $n$ objects where there are $n_1$ identical objects of type 1, $n_2$ of type 2, ..., $n_k$ of type $k$ (with $n_1 + n_2 + \cdots + n_k = n$), then:

$$\boxed{\text{Permutations} = \frac{n!}{n_1! \cdot n_2! \cdots n_k!}}$$

### Example 4: Arranging "MISSISSIPPI"

Letters: M(1), I(4), S(4), P(2) → Total = 11 letters

$$\frac{11!}{1! \cdot 4! \cdot 4! \cdot 2!} = \frac{39{,}916{,}800}{1 \times 24 \times 24 \times 2} = \frac{39{,}916{,}800}{1{,}152} = 34{,}650$$

### Example 5: Distributing Identical Objects

How many ways to arrange 3 red, 2 blue, and 1 green ball in a row?

$$\frac{6!}{3! \cdot 2! \cdot 1!} = \frac{720}{6 \times 2 \times 1} = 60$$

---

## 3. Circular Permutations

When objects are arranged in a **circle**, one position is fixed as reference (to avoid counting rotations as different).

$$\boxed{P_{\text{circular}}(n) = (n-1)!}$$

**Why $(n-1)!$?** In a circle, rotating everyone by one position gives the same arrangement. There are $n$ such rotations, so we divide $n!$ by $n$.

```
  Linear vs. Circular Arrangement (3 people: A, B, C)
  
  LINEAR (3! = 6 arrangements):
  A B C    A C B    B A C    B C A    C A B    C B A
  
  CIRCULAR ((3-1)! = 2 arrangements):
  
      A                 A
     / \               / \
    C   B             B   C
    
  (Rotations of ABC, BCA, CAB    (Rotations of ACB, CBA, BAC
   are the SAME arrangement)      are the SAME arrangement)
```

### With Reflection (Necklace/Bracelet)

If arrangements that are **mirror images** are also considered identical (e.g., necklaces that can be flipped):

$$\boxed{P_{\text{necklace}}(n) = \frac{(n-1)!}{2}}$$

---

## 4. Restricted Permutations

### Certain Objects Together

If specific objects must be **adjacent**, treat them as a single "super-object," then arrange internally.

**Example 6:** Arrange A, B, C, D, E such that A and B are always together.

- Treat (AB) as one unit → 4 units: (AB), C, D, E
- Arrange 4 units: $4! = 24$
- Internal arrangement of AB: $2! = 2$ (AB or BA)
- Total: $24 \times 2 = 48$

### Certain Objects NOT Together

Use complement: Total − (cases where they are together).

**Example 7:** Arrange A, B, C, D, E such that A and B are NOT adjacent.

- Total arrangements: $5! = 120$
- A and B together: $48$ (from above)
- A and B NOT together: $120 - 48 = 72$

### Specific Position Constraints

**Example 8:** Arrange 5 people in a row. Person X must NOT be at either end.

- Total without constraint: $5! = 120$
- X at left end: $4! = 24$
- X at right end: $4! = 24$
- X at either end: $24 + 24 = 48$
- X NOT at end: $120 - 48 = 72$

**Alternative:** Place X first (3 middle positions), then arrange 4 others: $3 \times 4! = 72$ ✓

---

## 5. Derangements

A **derangement** is a permutation where **no element** appears in its original position.

$$\boxed{D_n = n! \sum_{k=0}^{n} \frac{(-1)^k}{k!} = n!\left(1 - 1 + \frac{1}{2!} - \frac{1}{3!} + \cdots + \frac{(-1)^n}{n!}\right)}$$

| $n$ | $D_n$ | $n!$ | $D_n/n!$ |
|-----|-------|------|-----------|
| 1 | 0 | 1 | 0 |
| 2 | 1 | 2 | 0.500 |
| 3 | 2 | 6 | 0.333 |
| 4 | 9 | 24 | 0.375 |
| 5 | 44 | 120 | 0.367 |
| 6 | 265 | 720 | 0.368 |
| $\infty$ | — | — | $1/e \approx 0.368$ |

As $n \to \infty$, $D_n / n! \to 1/e$.

---

## 6. Comparison Table of Permutation Types

| Type | Formula | Order Matters? | Repetition? | Example |
|------|---------|:--------------:|:-----------:|---------|
| All $n$ objects | $n!$ | Yes | No | Arrange 5 books |
| $r$ from $n$ (no rep.) | $\frac{n!}{(n-r)!}$ | Yes | No | Award gold/silver/bronze |
| $r$ from $n$ (with rep.) | $n^r$ | Yes | Yes | 4-digit PIN |
| Multiset | $\frac{n!}{n_1! n_2! \cdots}$ | Yes | Has duplicates | Arrange MISSISSIPPI |
| Circular | $(n-1)!$ | Yes (circular) | No | Seat people at round table |
| Necklace | $\frac{(n-1)!}{2}$ | Yes (circular, flip) | No | Arrange beads on necklace |
| Derangement | $D_n$ | Yes | No | Secret Santa (no self-gift) |

---

## 7. Real-World Applications

| Application | Type |
|------------|------|
| **Scheduling** | Arranging $n$ tasks in a specific order |
| **Cryptography** | Number of possible key arrangements |
| **DNA sequencing** | Permutations of nucleotide bases |
| **Playlist ordering** | Shuffling $n$ songs |
| **Tournament brackets** | Ordering match-ups |
| **Secret Santa** | Derangements (no one draws themselves) |

---

## Summary Table

| Formula | Expression | Condition |
|---------|-----------|-----------|
| $n!$ | $n \times (n-1) \times \cdots \times 1$ | All $n$ distinct objects |
| $P(n,r)$ | $\frac{n!}{(n-r)!}$ | $r$ from $n$, no repetition |
| $n^r$ | — | $r$ positions, $n$ choices each, repetition OK |
| Multiset | $\frac{n!}{n_1! n_2! \cdots n_k!}$ | Objects with duplicates |
| Circular | $(n-1)!$ | Round table arrangement |
| Necklace | $\frac{(n-1)!}{2}$ | Circular with flip symmetry |
| Derangement | $n! \sum_{k=0}^{n} \frac{(-1)^k}{k!}$ | No fixed points |

---

## Quick Revision Questions

1. How many ways can 7 students stand in a line for a photo? What if only 4 of them are in the photo?

2. Find the number of distinct arrangements of the letters in "STATISTICS."

3. How many ways can 6 people sit around a circular table? What if the table has a "head" position (making it linear)?

4. Arrange 5 boys and 3 girls in a row such that all girls sit together. How many arrangements?

5. A professor has 8 different books but only 5 slots on a shelf. How many ways can she arrange them? Does it matter if books can be used more than once?

6. Find $D_4$ (derangements of 4 elements). List all derangements of $\{1,2,3,4\}$ explicitly.

---

[← Previous: Fundamental Counting Principle](01-fundamental-counting-principle.md) | [Back to README](../README.md) | [Next: Combinations →](03-combinations.md)
