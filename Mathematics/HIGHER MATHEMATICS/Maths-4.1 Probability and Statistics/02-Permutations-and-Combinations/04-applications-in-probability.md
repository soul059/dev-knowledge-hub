# Chapter 2.4: Applications in Probability

[← Previous: Combinations](03-combinations.md) | [Back to README](../README.md) | [Next: Discrete Random Variables →](../03-Random-Variables/01-discrete-random-variables.md)

---

## Overview

This chapter connects **counting techniques** (permutations and combinations) with **probability calculations**. When all outcomes are equally likely, the probability of an event reduces to a counting problem:

$$P(A) = \frac{|A|}{|S|} = \frac{\text{favorable outcomes}}{\text{total outcomes}}$$

---

## 1. Counting-Based Probability Framework

```
  ┌───────────────────────────────────────────────┐
  │          PROBABILITY via COUNTING              │
  │                                                │
  │  1. Identify the experiment                    │
  │  2. Count total outcomes |S|                   │
  │  3. Count favorable outcomes |A|               │
  │  4. P(A) = |A| / |S|                          │
  │                                                │
  │  KEY: All outcomes must be EQUALLY LIKELY       │
  └───────────────────────────────────────────────┘
```

---

## 2. Card Problems

A standard deck has 52 cards: 4 suits (♠♣♥♦) × 13 ranks (A,2,...,10,J,Q,K).

### Example 1: 5-Card Poker Hand

Total hands: $\binom{52}{5} = 2{,}598{,}960$

#### (a) Royal Flush
10, J, Q, K, A of same suit.
- Choose suit: $\binom{4}{1} = 4$

$$P(\text{Royal Flush}) = \frac{4}{2{,}598{,}960} \approx 0.00000154$$

#### (b) Four of a Kind
4 cards of same rank + 1 other card.
- Choose rank for 4: $\binom{13}{1} = 13$
- Choose the 5th card: $\binom{48}{1} = 48$

$$P(\text{Four of a Kind}) = \frac{13 \times 48}{2{,}598{,}960} = \frac{624}{2{,}598{,}960} \approx 0.000240$$

#### (c) Full House
3 of one rank + 2 of another.
- Choose rank for triple: $\binom{13}{1} = 13$
- Choose 3 cards from that rank: $\binom{4}{3} = 4$
- Choose rank for pair: $\binom{12}{1} = 12$
- Choose 2 cards from that rank: $\binom{4}{2} = 6$

$$P(\text{Full House}) = \frac{13 \times 4 \times 12 \times 6}{2{,}598{,}960} = \frac{3{,}744}{2{,}598{,}960} \approx 0.00144$$

#### Poker Hand Rankings (Probability Order)

| Hand | Count | Probability |
|------|:-----:|:-----------:|
| Royal Flush | 4 | 0.00000154 |
| Straight Flush | 36 | 0.0000139 |
| Four of a Kind | 624 | 0.000240 |
| Full House | 3,744 | 0.00144 |
| Flush | 5,108 | 0.00197 |
| Straight | 10,200 | 0.00392 |
| Three of a Kind | 54,912 | 0.0211 |
| Two Pair | 123,552 | 0.0475 |
| One Pair | 1,098,240 | 0.423 |
| High Card | 1,302,540 | 0.501 |

---

## 3. Ball/Urn Problems

### Example 2: Drawing Without Replacement

An urn has 8 red, 5 blue, and 3 green balls (16 total). Draw 4 balls without replacement.

**(a) All 4 are red:**

$$P = \frac{\binom{8}{4}}{\binom{16}{4}} = \frac{70}{1820} = \frac{1}{26} \approx 0.0385$$

**(b) Exactly 2 red and 2 blue:**

$$P = \frac{\binom{8}{2} \times \binom{5}{2}}{\binom{16}{4}} = \frac{28 \times 10}{1820} = \frac{280}{1820} = \frac{2}{13} \approx 0.154$$

**(c) At least 1 green:**

$$P = 1 - P(\text{no green}) = 1 - \frac{\binom{13}{4}}{\binom{16}{4}} = 1 - \frac{715}{1820} = \frac{1105}{1820} \approx 0.607$$

---

## 4. Birthday Problem

**What is the probability that in a group of $n$ people, at least two share a birthday?** (Assume 365 equally likely birthdays.)

### Strategy: Use complement

$$P(\text{at least one match}) = 1 - P(\text{all different})$$

$$P(\text{all different}) = \frac{365 \times 364 \times 363 \times \cdots \times (365-n+1)}{365^n} = \frac{P(365, n)}{365^n}$$

### Results Table

| $n$ (people) | $P(\text{all different})$ | $P(\text{match})$ |
|:------------:|:-------------------------:|:------------------:|
| 5 | 0.973 | 0.027 |
| 10 | 0.883 | 0.117 |
| 20 | 0.589 | 0.411 |
| **23** | **0.493** | **0.507** ← exceeds 50%! |
| 30 | 0.294 | 0.706 |
| 40 | 0.109 | 0.891 |
| 50 | 0.030 | 0.970 |
| 57 | 0.010 | 0.990 |

```
  Birthday Problem — Probability of Match vs Group Size
  
  P(match)
  1.0 │                          ___________
      │                       __/
  0.9 │                     _/
      │                   _/
  0.8 │                  /
      │                /
  0.7 │              /
      │            /
  0.6 │           /
      │         /
  0.5 │─ ─ ─ ─/─ ─ ─ ─ ─ ─ ─ ─ ─  ← 50% at n = 23
      │       /
  0.4 │     /
      │    /
  0.3 │   /
      │  /
  0.2 │ /
      │/
  0.1 │
      │
  0.0 └──────────────────────────────────►
       5   10   15   20   25   30   35  n
```

**Surprise:** Only 23 people needed for a 50% chance of a birthday match!

---

## 5. Geometric Probability

When the sample space is continuous (e.g., a region), probability is calculated using **measures** (length, area, volume):

$$P(A) = \frac{\text{measure of favorable region}}{\text{measure of total region}}$$

### Example 3: Needle on a Ruler

A point is chosen uniformly at random on $[0, 10]$. What is $P(3 \leq x \leq 7)$?

$$P = \frac{7 - 3}{10 - 0} = \frac{4}{10} = 0.4$$

### Example 4: Meeting Problem

Two friends agree to meet at a café between 12:00 and 1:00 PM. Each arrives at a random time (uniform) and waits 15 minutes. What is the probability they meet?

Let $X$ and $Y$ be arrival times (in minutes past noon), uniformly distributed on $[0, 60]$.

They meet if $|X - Y| \leq 15$.

```
  Y
  60 ┌──────────────────────────┐
     │\  ////////////////////// │
     │ \  //////////////////// /│
     │  \  ////////////////// / │
     │   \  //////////////// /  │
     │    \  ////////////// /   │
  15 │     \  //////////// /    │
     │      \  ////////// /     │
     │       \  //////// /      │
     │        \ /////// /       │
     │         \ ///   /        │
     │          \     /         │
   0 └──────────────────────────┘ X
     0         15               60
     
     Shaded = |X - Y| ≤ 15 (they meet)
```

$$P(\text{meet}) = 1 - P(\text{don't meet}) = 1 - \frac{2 \times \frac{1}{2}(45)^2}{60^2} = 1 - \frac{2025}{3600} = 1 - \frac{45^2}{60^2} = 1 - \left(\frac{3}{4}\right)^2 = 1 - \frac{9}{16} = \frac{7}{16} \approx 0.4375$$

---

## 6. Occupancy Problems

**Distributing $r$ objects into $n$ cells.**

| Objects | Cells | Formula | Name |
|---------|-------|---------|------|
| Distinguishable | Distinguishable | $n^r$ | Arrangements |
| Indistinguishable | Distinguishable | $\binom{n+r-1}{r}$ | Stars & Bars |
| Distinguishable | Indistinguishable | Stirling numbers | Set partitions |
| Indistinguishable | Indistinguishable | Partition function $p(r,n)$ | Integer partitions |

### Example 5

Distribute 3 distinct balls into 2 distinct boxes:

$$N = 2^3 = 8$$

```
  Ball 1 → Box A or B  (2 choices)
  Ball 2 → Box A or B  (2 choices)
  Ball 3 → Box A or B  (2 choices)
  
  Outcomes:
  Box A    Box B
  {1,2,3}  {}        
  {1,2}    {3}       
  {1,3}    {2}       
  {2,3}    {1}       
  {1}      {2,3}     
  {2}      {1,3}     
  {3}      {1,2}     
  {}       {1,2,3}   
  
  Total = 8 = 2³ ✓
```

---

## 7. The Matching Problem (Hatcheck Problem)

$n$ people leave their hats. Hats are returned randomly. What is $P(\text{at least one person gets their own hat})$?

Using inclusion-exclusion:

$$P(\text{at least one match}) = 1 - \frac{D_n}{n!} = 1 - \sum_{k=0}^{n} \frac{(-1)^k}{k!} \approx 1 - \frac{1}{e} \approx 0.632$$

For large $n$, this converges to $1 - 1/e \approx 63.2\%$ regardless of $n$.

---

## 8. Real-World Applications

| Problem Type | Application |
|-------------|------------|
| **Card counting** | Poker odds, blackjack strategy |
| **Birthday problem** | Hash collision probability, cryptographic security |
| **Geometric probability** | Arrival time coordination, random sampling |
| **Occupancy problems** | Load balancing, hash table analysis |
| **Matching problems** | Random assignment, genetics |
| **Lottery odds** | Game design, expected winnings |

---

## Summary Table

| Problem Type | Key Formula |
|-------------|-------------|
| Card probability | $P = \frac{\text{favorable hands}}{\binom{52}{5}}$ |
| Urn without replacement | $P = \frac{\binom{a}{k}\binom{b}{n-k}}{\binom{a+b}{n}}$ |
| Birthday problem ($n$ people) | $P(\text{match}) = 1 - \frac{P(365,n)}{365^n}$ |
| Geometric probability | $P = \frac{\text{favorable area/length}}{\text{total area/length}}$ |
| Occupancy (distinct→distinct) | $n^r$ |
| Occupancy (identical→distinct) | $\binom{n+r-1}{r}$ |
| Matching problem | $P \approx 1 - 1/e \approx 63.2\%$ |

---

## Quick Revision Questions

1. From a deck of 52 cards, 5 are drawn. Find the probability of getting exactly 3 hearts.

2. An urn has 10 white and 6 black balls. Four are drawn. Find $P(\text{exactly 2 white and 2 black})$.

3. In a group of 30 people, what is the approximate probability that at least two share a birthday? Why is this sometimes called the "birthday paradox"?

4. Two numbers are chosen independently and uniformly from $[0, 1]$. Find the probability their sum is less than 1. (*Use geometric probability.*)

5. 5 letters are placed randomly into 5 addressed envelopes. What is the probability (a) all letters are in the correct envelope, (b) no letter is in the correct envelope?

6. A 4-digit number is formed using digits 1–9 (repetition allowed). What is the probability that all 4 digits are different?

---

[← Previous: Combinations](03-combinations.md) | [Back to README](../README.md) | [Next: Discrete Random Variables →](../03-Random-Variables/01-discrete-random-variables.md)
