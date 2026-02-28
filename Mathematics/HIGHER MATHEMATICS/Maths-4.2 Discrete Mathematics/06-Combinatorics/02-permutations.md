# Chapter 6.2: Permutations

[← Previous: Counting Principles](01-counting-principles.md) | [Back to README](../README.md) | [Next: Combinations →](03-combinations.md)

---

## 📋 Chapter Overview

A **permutation** is an arrangement of objects where **order matters**. This chapter covers permutations with and without repetition, circular permutations, and permutations of multisets.

---

## 1. Permutations (Without Repetition)

An **$r$-permutation** of $n$ objects is an ordered arrangement of $r$ objects chosen from $n$ distinct objects.

$$P(n, r) = \frac{n!}{(n - r)!}$$

**Derivation (Product Rule):**
- 1st position: $n$ choices
- 2nd position: $n - 1$ choices
- $\vdots$
- $r$-th position: $n - r + 1$ choices

$$P(n, r) = n \cdot (n-1) \cdot (n-2) \cdots (n-r+1) = \frac{n!}{(n-r)!}$$

| $P(n,r)$ | Formula | Value |
|:--------:|---------|:-----:|
| $P(5,3)$ | $\frac{5!}{2!} = 5 \times 4 \times 3$ | $60$ |
| $P(10,2)$ | $\frac{10!}{8!} = 10 \times 9$ | $90$ |
| $P(n,n)$ | $n!$ | All objects arranged |
| $P(n,1)$ | $n$ | Just picking one |
| $P(n,0)$ | $1$ | Empty arrangement |

---

## 2. Permutations of All $n$ Objects

When arranging ALL $n$ distinct objects:

$$P(n, n) = n!$$

| $n$ | $n!$ |
|:---:|:----:|
| 1 | 1 |
| 2 | 2 |
| 3 | 6 |
| 4 | 24 |
| 5 | 120 |
| 6 | 720 |
| 10 | 3,628,800 |

**Example:** How many ways can 6 books be arranged on a shelf?

$$6! = 720$$

---

## 3. Permutations with Repetition

When choosing from $n$ types of objects WITH repetition allowed, the number of $r$-length sequences is:

$$n^r$$

**Example:** How many 4-digit PINs (digits 0–9)?

$$10^4 = 10{,}000$$

---

## 4. Permutations of Multisets (Identical Objects)

If $n$ objects have $n_1$ of type 1, $n_2$ of type 2, $\ldots$, $n_k$ of type $k$ (where $n_1 + n_2 + \cdots + n_k = n$):

$$\frac{n!}{n_1! \cdot n_2! \cdots n_k!}$$

**Example:** Arrangements of "MISSISSIPPI"

$$\text{Letters: M(1), I(4), S(4), P(2)} \quad n = 11$$

$$\frac{11!}{1! \cdot 4! \cdot 4! \cdot 2!} = \frac{39916800}{1 \cdot 24 \cdot 24 \cdot 2} = 34{,}650$$

```
  MISSISSIPPI has 11 letters:
  
  Letter │ Count
  ───────┼──────
    M    │  1
    I    │  4
    S    │  4
    P    │  2
  ───────┼──────
  Total  │ 11
  
  Arrangements = 11! / (1! · 4! · 4! · 2!) = 34,650
```

---

## 5. Circular Permutations

Arranging $n$ objects in a **circle** (rotations are the same):

$$\frac{n!}{n} = (n - 1)!$$

```
  Linear:  A B C  ≠  B C A  ≠  C A B  (3 different)
  
  Circular:
  
      A           B           C
     / \         / \         / \
    C   B  =   A   C  =   B   A   (same circular arrangement!)
  
  So divide by n = 3: (3-1)! = 2! = 2 circular arrangements
  
  Arrangement 1:    Arrangement 2:
       A                 A
      / \               / \
     C   B             B   C
```

**With reflections identical** (e.g., bracelet): $\frac{(n-1)!}{2}$

---

## 6. Derangements

A **derangement** is a permutation where **no element is in its original position**.

$$D_n = n! \sum_{k=0}^{n} \frac{(-1)^k}{k!} = n! \left(1 - 1 + \frac{1}{2!} - \frac{1}{3!} + \cdots + \frac{(-1)^n}{n!}\right)$$

| $n$ | $D_n$ | $n!$ | Fraction deranged |
|:---:|:-----:|:----:|:-:|
| 1 | 0 | 1 | 0% |
| 2 | 1 | 2 | 50% |
| 3 | 2 | 6 | 33.3% |
| 4 | 9 | 24 | 37.5% |
| 5 | 44 | 120 | 36.7% |
| 10 | 1,334,961 | 3,628,800 | 36.8% |

**Converges to $1/e \approx 36.8\%$**

**Example:** 4 letters in 4 envelopes — how many ways to put every letter in the wrong envelope?

$$D_4 = 4!\left(1 - 1 + \frac{1}{2} - \frac{1}{6} + \frac{1}{24}\right) = 24 \cdot \frac{3}{8} = 9$$

---

## 7. Worked Examples

### Example 1: Seating with Constraints

How many ways can 8 people sit in a row if persons A and B must sit together?

**Solution:** Treat AB as a single unit → 7 units to arrange → $7!$ ways. A and B can swap → $\times 2$.

$$7! \times 2 = 5040 \times 2 = 10{,}080$$

### Example 2: No Two Vowels Adjacent

Arrange "GARDEN" so that no two vowels are adjacent.

Consonants G, R, D, N → $4!$ arrangements
Consonants create 5 gaps: _ G _ R _ D _ N _
Place vowels A, E in 2 of these 5 gaps: $P(5, 2) = 20$

$$4! \times P(5,2) = 24 \times 20 = 480$$

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Permutations in Practice                  │
  │                                                   │
  │  1. Cryptography                                 │
  │     Substitution cipher = permutation of alphabet │
  │     26! ≈ 4 × 10²⁶ possible keys                │
  │                                                   │
  │  2. Tournament Scheduling                        │
  │     Round-robin pairings use permutation theory   │
  │                                                   │
  │  3. DNA Sequences                                │
  │     Arrangements of A, T, G, C with repeats      │
  │                                                   │
  │  4. Shuffling Algorithms                         │
  │     Fisher-Yates: generates random permutation   │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Type | Formula | Example |
|------|:-------:|---------|
| $r$ from $n$ (no repeat) | $P(n,r) = \frac{n!}{(n-r)!}$ | Arrange 3 of 5 books |
| All $n$ objects | $n!$ | Arrange 5 books |
| With repetition | $n^r$ | 4-digit PIN |
| Multiset | $\frac{n!}{n_1! n_2! \cdots}$ | "MISSISSIPPI" |
| Circular | $(n-1)!$ | Seating at round table |
| Derangement | $D_n = n! \sum \frac{(-1)^k}{k!}$ | All letters in wrong envelopes |

---

## ❓ Quick Revision Questions

1. **How many 3-letter arrangements can be formed from the letters A, B, C, D, E (no repeats)?**

2. **How many distinct arrangements are there of the word "ENGINEERING"?**

3. **In how many ways can 8 people be seated at a round table?**

4. **How many 5-character passwords can be formed using digits $\{0,...,9\}$ with repetition allowed?**

5. **Compute $D_5$ (the number of derangements of 5 elements).**

6. **How many ways can 10 people line up if two specific people refuse to stand next to each other?**

---

[← Previous: Counting Principles](01-counting-principles.md) | [Back to README](../README.md) | [Next: Combinations →](03-combinations.md)
