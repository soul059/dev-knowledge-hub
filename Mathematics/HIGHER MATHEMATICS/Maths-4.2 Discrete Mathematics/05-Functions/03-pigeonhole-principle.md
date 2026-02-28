# Chapter 5.3: Pigeonhole Principle

[← Previous: Composition and Inverse](02-composition-and-inverse.md) | [Back to README](../README.md) | [Next: Recursive Functions →](04-recursive-functions.md)

---

## 📋 Chapter Overview

The **Pigeonhole Principle** is deceptively simple yet extraordinarily powerful: if you stuff more items into fewer boxes, at least one box must hold multiple items. This principle proves existence results that seem almost magical.

---

## 1. Basic Pigeonhole Principle

**Theorem:** If $n + 1$ or more objects are placed into $n$ boxes, then at least one box contains **two or more** objects.

```
  5 pigeons, 4 holes:
  
  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
  │ 🐦  │ │ 🐦  │ │ 🐦  │ │🐦🐦 │ ← at least one hole
  │     │ │     │ │     │ │     │    has ≥ 2 pigeons!
  └─────┘ └─────┘ └─────┘ └─────┘
  Hole 1   Hole 2  Hole 3  Hole 4
```

**Formal:** If $f: A \to B$ and $|A| > |B|$, then $f$ is **not injective**.

---

## 2. Generalized Pigeonhole Principle

**Theorem:** If $N$ objects are placed into $k$ boxes, then at least one box contains:

$$\left\lceil \frac{N}{k} \right\rceil \text{ or more objects}$$

**Examples:**

| Objects ($N$) | Boxes ($k$) | At least one box has $\geq$ |
|:---:|:---:|:---:|
| 100 students | 12 months | $\lceil 100/12 \rceil = 9$ born in same month |
| 27 letters | 26 alphabet | $\lceil 27/26 \rceil = 2$ starting with same letter |
| 367 people | 366 days | $\lceil 367/366 \rceil = 2$ share a birthday |

---

## 3. Classic Applications

### 3.1 Birthday Problem (Guaranteed Match)

Among **367 people**, at least two share a birthday (accounting for leap years).

Pigeons = 367 people, Holes = 366 possible birthdays.

### 3.2 Handshake Theorem

Among $n \geq 2$ people at a party, at least two have shaken the same number of hands.

**Proof:** Each person shakes 0 to $n-1$ hands. But 0 and $n-1$ can't both occur (if someone shook everyone's hand, no one has 0 handshakes). So there are at most $n-1$ possible values for $n$ people.

### 3.3 Socks in the Dark

A drawer has 10 blue and 10 black socks. Grabbing **3 socks** guarantees a matching pair.

Pigeons = 3 socks, Holes = 2 colors → $\lceil 3/2 \rceil = 2$.

```
  3 socks, 2 colors:
  
  Blue pile    Black pile
  ┌────────┐  ┌────────┐
  │  🧦    │  │  🧦🧦  │  At least one pile
  │        │  │        │  has ≥ 2 socks
  └────────┘  └────────┘
  
  (or the other way around — either way, a match!)
```

---

## 4. Number Theory Applications

### 4.1 Consecutive Integers

Among any $n + 1$ integers chosen from $\{1, 2, \ldots, 2n\}$, at least two are consecutive.

**Proof:** Pair the integers: $\{1,2\}, \{3,4\}, \ldots, \{2n-1, 2n\}$. There are $n$ pairs and $n+1$ chosen integers. By PHP, two fall in the same pair → they're consecutive.

### 4.2 Divisibility

Among any $n + 1$ integers, at least two have the same remainder mod $n$.

**Proof:** There are only $n$ possible remainders ($0, 1, \ldots, n-1$).

### 4.3 Subset Sum

Any set of 5 integers contains two whose difference is divisible by 4.

**Proof:** Remainders mod 4 are $\{0, 1, 2, 3\}$ (4 pigeonholes, 5 pigeons).

---

## 5. Geometric Application

**Theorem:** If 5 points are placed inside a $2 \times 2$ square, at least two are within distance $\sqrt{2}$ of each other.

**Proof:** Divide the square into 4 unit squares ($1 \times 1$). By PHP, at least two points fall in the same unit square. The maximum distance within a unit square is $\sqrt{1^2 + 1^2} = \sqrt{2}$.

```
  ┌────────┬────────┐
  │  •     │     •  │
  │        │        │   5 points in 4 cells
  ├────────┼────────┤   → at least 2 in same cell
  │   •    │  • •   │   → distance ≤ √2
  │        │        │
  └────────┴────────┘
```

---

## 6. Graph Theory Application

**Theorem:** In any group of 6 people, either 3 mutually know each other, or 3 mutually don't know each other.

**Proof sketch:** Consider person $A$. They have 5 relationships. By PHP, at least $\lceil 5/2 \rceil = 3$ are either "know" or "don't know." Among those 3, analyze their mutual relationships...

This leads to **Ramsey Theory** — a whole branch of combinatorics!

---

## 7. Strengthened Pigeonhole Principle

**Theorem:** If $q_1 + q_2 + \cdots + q_n - n + 1$ objects are placed in $n$ boxes, then box $i$ contains at least $q_i$ objects for some $i$.

**Special case:** $q_1 = q_2 = \cdots = q_n = r$ gives: $(r-1)n + 1$ objects, $n$ boxes → some box has $\geq r$ items.

---

## 8. More Challenging Examples

### Example: Subsequence Problem

**Theorem:** Any sequence of $n^2 + 1$ distinct numbers contains either:
- An increasing subsequence of length $n + 1$, or
- A decreasing subsequence of length $n + 1$

**Proof:** For each element $a_k$, let $i_k$ = length of longest increasing subsequence ending at $a_k$, and $d_k$ = longest decreasing subsequence ending at $a_k$.

If neither exceeds $n$, then $(i_k, d_k) \in \{1,\ldots,n\}^2$ which has $n^2$ possible values. By PHP, two elements have the same $(i_k, d_k)$ → contradiction!

---

## 📝 Summary Table

| Concept | Statement |
|---------|-----------|
| Basic PHP | $n+1$ items in $n$ boxes → some box has $\geq 2$ |
| Generalized PHP | $N$ items in $k$ boxes → some box has $\geq \lceil N/k \rceil$ |
| Strategy | Identify items (pigeons) and categories (holes) |
| Key insight | Proves **existence**, not construction |
| Applications | Birthdays, number theory, geometry, graph theory |

---

## ❓ Quick Revision Questions

1. **In a class of 30 students, show that at least two have last names starting with the same letter.**

2. **How many cards must be drawn from a standard 52-card deck to guarantee at least 3 cards of the same suit?**

3. **Prove: Among any 52 integers, there exist two whose difference is divisible by 51.**

4. **If 10 points are placed inside an equilateral triangle of side 4, show that at least two points are within distance $\frac{4}{3}\sqrt{3}$ of each other.**

5. **Show that in any set of 10 two-digit natural numbers, there are two disjoint subsets with the same sum.**

6. **State and prove the Generalized Pigeonhole Principle.**

---

[← Previous: Composition and Inverse](02-composition-and-inverse.md) | [Back to README](../README.md) | [Next: Recursive Functions →](04-recursive-functions.md)
