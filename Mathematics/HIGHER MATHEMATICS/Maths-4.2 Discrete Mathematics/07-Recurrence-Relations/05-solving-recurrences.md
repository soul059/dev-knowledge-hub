# Chapter 7.5: Solving Recurrences — Divide-and-Conquer & Master Theorem

[← Previous: Generating Functions](04-generating-functions.md) | [Back to README](../README.md) | [Next: Graph Theory — Basic Terminology →](../08-Graph-Theory/01-basic-terminology.md)

---

## 📋 Chapter Overview

Many algorithms use **divide-and-conquer**: split a problem into subproblems, solve recursively, and combine. This produces recurrences of the form $T(n) = aT(n/b) + f(n)$. The **Master Theorem** gives a quick closed-form for such recurrences.

---

## 1. Divide-and-Conquer Recurrences

A divide-and-conquer algorithm on input of size $n$:
1. **Divides** into $a$ subproblems of size $n/b$
2. **Conquers** each subproblem recursively
3. **Combines** results in $f(n)$ time

General form:

$$T(n) = aT\left(\frac{n}{b}\right) + f(n), \quad T(1) = c$$

```
  Divide-and-Conquer Recursion Tree:
  
  Level 0:         T(n)                    → f(n)
                  / | \
  Level 1:    T(n/b) ... T(n/b)           → a · f(n/b)
              (a subproblems)
                / | \
  Level 2:  T(n/b²)... T(n/b²)           → a² · f(n/b²)
              (a² subproblems)
                ...
  Level k:  T(1) T(1) ... T(1)           → a^k · c
              (a^k subproblems)
  
  Total levels: log_b(n)
  Total leaves: a^(log_b n) = n^(log_b a)
```

---

## 2. Recursion Tree Method

Expand the recurrence into a tree and sum all levels.

### Example: Merge Sort

$T(n) = 2T(n/2) + n$

| Level | # Nodes | Work per node | Total work |
|:-----:|:-------:|:-------------:|:----------:|
| 0 | 1 | $n$ | $n$ |
| 1 | 2 | $n/2$ | $n$ |
| 2 | 4 | $n/4$ | $n$ |
| $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ |
| $k$ | $2^k$ | $n/2^k$ | $n$ |

Number of levels: $\log_2 n$. Each level contributes $n$.

$$T(n) = n \cdot \log_2 n = \Theta(n \log n)$$

---

## 3. The Master Theorem

For recurrences of the form:

$$T(n) = aT(n/b) + \Theta(n^d)$$

where $a \geq 1$, $b > 1$, $d \geq 0$:

$$T(n) = \begin{cases} \Theta(n^d) & \text{if } a < b^d \quad (\text{Case 1: combine dominates}) \\ \Theta(n^d \log n) & \text{if } a = b^d \quad (\text{Case 2: balanced}) \\ \Theta(n^{\log_b a}) & \text{if } a > b^d \quad (\text{Case 3: subproblems dominate}) \end{cases}$$

**Critical comparison:** $\log_b a$ vs $d$

```
  ┌─────────────────────────────────────────────┐
  │         Master Theorem Decision              │
  │                                              │
  │  Compute: log_b(a)  vs  d                   │
  │                                              │
  │  log_b(a) < d  →  T(n) = Θ(n^d)            │
  │  log_b(a) = d  →  T(n) = Θ(n^d log n)      │
  │  log_b(a) > d  →  T(n) = Θ(n^{log_b a})    │
  └─────────────────────────────────────────────┘
```

---

## 4. Master Theorem Examples

| Algorithm | Recurrence | $a$ | $b$ | $d$ | $\log_b a$ | Case | $T(n)$ |
|-----------|:----------:|:---:|:---:|:---:|:----------:|:----:|:------:|
| Binary Search | $T(n)=T(n/2)+1$ | 1 | 2 | 0 | 0 | 2 | $\Theta(\log n)$ |
| Merge Sort | $T(n)=2T(n/2)+n$ | 2 | 2 | 1 | 1 | 2 | $\Theta(n\log n)$ |
| Karatsuba | $T(n)=3T(n/2)+n$ | 3 | 2 | 1 | 1.58 | 3 | $\Theta(n^{1.58})$ |
| Strassen | $T(n)=7T(n/2)+n^2$ | 7 | 2 | 2 | 2.81 | 3 | $\Theta(n^{2.81})$ |
| Naive multiply | $T(n)=4T(n/2)+n$ | 4 | 2 | 1 | 2 | 3 | $\Theta(n^2)$ |
| Closest pair | $T(n)=2T(n/2)+n\log n$ | 2 | 2 | — | 1 | * | $\Theta(n\log^2 n)$ |

*The last one doesn't fit the basic Master Theorem ($f(n)$ not polynomial).

---

## 5. Extended Master Theorem

For $T(n) = aT(n/b) + f(n)$ with $f(n) = \Theta(n^d \log^p n)$:

| Condition | Result |
|-----------|--------|
| $\log_b a > d$ | $\Theta(n^{\log_b a})$ |
| $\log_b a = d, \; p > -1$ | $\Theta(n^d \log^{p+1} n)$ |
| $\log_b a = d, \; p = -1$ | $\Theta(n^d \log\log n)$ |
| $\log_b a = d, \; p < -1$ | $\Theta(n^d)$ |
| $\log_b a < d$ | $\Theta(f(n))$ |

---

## 6. Iteration (Unrolling) Method

Expand the recurrence step by step.

### Example: $T(n) = 3T(n/4) + n$

$$T(n) = n + 3T(n/4)$$
$$= n + 3(n/4 + 3T(n/16)) = n + \frac{3n}{4} + 9T(n/16)$$
$$= n + \frac{3n}{4} + \frac{9n}{16} + 27T(n/64)$$

After $k$ steps:

$$T(n) = n\sum_{i=0}^{k-1}\left(\frac{3}{4}\right)^i + 3^k T(n/4^k)$$

Base case when $n/4^k = 1$, i.e., $k = \log_4 n$:

$$T(n) = n \cdot \frac{1-(3/4)^{\log_4 n}}{1-3/4} + 3^{\log_4 n} \cdot T(1)$$

The geometric series converges to $\leq 4n$, and $3^{\log_4 n} = n^{\log_4 3} \approx n^{0.79}$.

$$T(n) = \Theta(n)$$

Matches Master Theorem: $a=3, b=4, d=1, \log_4 3 \approx 0.79 < 1$ → Case 1.

---

## 7. Substitution Method

**Guess** the answer, then **prove** by induction.

### Example: $T(n) = 2T(n/2) + n$

**Guess:** $T(n) = O(n \log n)$. We claim $T(n) \leq cn\log n$ for some $c > 0$.

**Inductive step:**

$$T(n) = 2T(n/2) + n \leq 2c(n/2)\log(n/2) + n$$
$$= cn(\log n - 1) + n = cn\log n - cn + n$$
$$= cn\log n - (c-1)n \leq cn\log n$$

The last inequality holds when $c \geq 1$. ✓

$$T(n) = O(n\log n)$$

---

## 8. Comparison of Methods

| Method | Best For | Difficulty |
|--------|----------|:----------:|
| Master Theorem | Standard divide-and-conquer | Easy |
| Recursion Tree | Visualizing and summing | Medium |
| Substitution | Proving bounds (after guessing) | Medium |
| Iteration | Small recurrences | Medium |
| Generating Functions | Complex linear recurrences | Hard |
| Characteristic Equation | Homogeneous, constant coeff. | Easy |

---

## 9. Common Algorithm Complexities

```
  ┌──────────────────────────────────────────────┐
  │      Common Recurrences & Solutions           │
  │                                               │
  │  T(n) = T(n/2) + 1       →  O(log n)        │
  │  T(n) = T(n-1) + 1       →  O(n)            │
  │  T(n) = T(n-1) + n       →  O(n²)           │
  │  T(n) = 2T(n/2) + n      →  O(n log n)      │
  │  T(n) = 2T(n/2) + 1      →  O(n)            │
  │  T(n) = 2T(n-1) + 1      →  O(2ⁿ)           │
  │  T(n) = T(n/2) + n       →  O(n)            │
  │  T(n) = 2T(n/2) + n²     →  O(n²)           │
  │  T(n) = 3T(n/2) + n      →  O(n^1.58)       │
  └──────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Divide-and-conquer | $T(n) = aT(n/b) + f(n)$ |
| Master Theorem | Compares $n^{\log_b a}$ with $f(n)$ → 3 cases |
| Case 1 ($a<b^d$) | $T(n) = \Theta(n^d)$ — combine dominates |
| Case 2 ($a=b^d$) | $T(n) = \Theta(n^d \log n)$ — balanced |
| Case 3 ($a>b^d$) | $T(n) = \Theta(n^{\log_b a})$ — subproblems dominate |
| Substitution | Guess + induction |
| Recursion tree | Visualize and sum all levels |

---

## ❓ Quick Revision Questions

1. **Use the Master Theorem to solve $T(n) = 4T(n/2) + n$.**

2. **What is the complexity of $T(n) = 2T(n/2) + n\log n$? Does the Master Theorem apply directly?**

3. **Draw the recursion tree for $T(n) = 3T(n/3) + n$ and determine $T(n)$.**

4. **Use substitution to prove $T(n) = T(n/2) + 1$ is $O(\log n)$.**

5. **An algorithm divides a problem of size $n$ into 8 subproblems of size $n/2$ and combines in $O(n^2)$ time. What is its complexity?**

6. **Compare the Master Theorem and generating function approaches. When would you prefer each?**

---

[← Previous: Generating Functions](04-generating-functions.md) | [Back to README](../README.md) | [Next: Graph Theory — Basic Terminology →](../08-Graph-Theory/01-basic-terminology.md)
