# Chapter 5.4: Recursive Functions

[← Previous: Pigeonhole Principle](03-pigeonhole-principle.md) | [Back to README](../README.md) | [Next: Growth of Functions →](05-growth-of-functions.md)

---

## 📋 Chapter Overview

A **recursively defined function** specifies its value at 0 (or 1) and gives a rule for computing $f(n)$ from previous values. Recursive definitions are fundamental to computer science and provide elegant ways to express complex computations.

---

## 1. What Is Recursion?

A **recursive definition** has two parts:

1. **Base case(s):** Define $f$ for initial value(s)
2. **Recursive step:** Define $f(n)$ in terms of $f(k)$ for $k < n$

```
  ┌─────────────────────────────────────────────┐
  │   Recursion = Base Case + Recursive Rule    │
  │                                              │
  │   f(0) = <initial value>       ← base      │
  │   f(n) = <expression using f(n-1), ...>     │
  │                                ← recursive  │
  │                                              │
  │   Like climbing stairs:                      │
  │   • Know how to stand on step 0             │
  │   • Know how to go from step n to step n+1  │
  └─────────────────────────────────────────────┘
```

---

## 2. Classic Examples

### 2.1 Factorial

$$n! = \begin{cases} 1 & n = 0 \\ n \cdot (n-1)! & n \geq 1 \end{cases}$$

```
  Computation trace:
  
  5! = 5 × 4!
     = 5 × 4 × 3!
     = 5 × 4 × 3 × 2!
     = 5 × 4 × 3 × 2 × 1!
     = 5 × 4 × 3 × 2 × 1 × 0!
     = 5 × 4 × 3 × 2 × 1 × 1
     = 120
```

### 2.2 Fibonacci Sequence

$$F(n) = \begin{cases} 0 & n = 0 \\ 1 & n = 1 \\ F(n-1) + F(n-2) & n \geq 2 \end{cases}$$

| $n$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|
| $F(n)$ | 0 | 1 | 1 | 2 | 3 | 5 | 8 | 13 | 21 | 34 | 55 |

```
  Fibonacci call tree for F(5):
  
                    F(5)
                   /    \
                F(4)    F(3)
               /   \    /   \
            F(3) F(2) F(2) F(1)
            / \   / \   / \
         F(2) F(1) F(1) F(0) F(1) F(0)
         / \
      F(1) F(0)
      
  Note the repeated computations!
  (This is why memoization/dynamic programming helps)
```

### 2.3 Power Function

$$a^n = \begin{cases} 1 & n = 0 \\ a \cdot a^{n-1} & n \geq 1 \end{cases}$$

**Efficient version (repeated squaring):**

$$a^n = \begin{cases} 1 & n = 0 \\ (a^{n/2})^2 & n \text{ even} \\ a \cdot a^{n-1} & n \text{ odd} \end{cases}$$

---

## 3. Recursively Defined Sequences

### Arithmetic Sequence

$$a_n = \begin{cases} a_0 & n = 0 \\ a_{n-1} + d & n \geq 1 \end{cases}$$

Closed form: $a_n = a_0 + nd$

### Geometric Sequence

$$a_n = \begin{cases} a_0 & n = 0 \\ r \cdot a_{n-1} & n \geq 1 \end{cases}$$

Closed form: $a_n = a_0 \cdot r^n$

### Tower of Hanoi

$$H(n) = \begin{cases} 0 & n = 0 \\ 2H(n-1) + 1 & n \geq 1 \end{cases}$$

Closed form: $H(n) = 2^n - 1$

| $n$ | 1 | 2 | 3 | 4 | 5 | 10 |
|:---:|:-:|:-:|:-:|:-:|:-:|:--:|
| $H(n)$ | 1 | 3 | 7 | 15 | 31 | 1023 |

---

## 4. Recursively Defined Sets

### Natural Numbers (Peano)

- Base: $0 \in \mathbb{N}$
- Recursive: If $n \in \mathbb{N}$, then $n + 1 \in \mathbb{N}$

### Well-Formed Formulas (Propositional Logic)

- Base: Propositional variables $p, q, r, \ldots$ are WFFs
- Recursive: If $\alpha, \beta$ are WFFs, then $(\neg \alpha)$, $(\alpha \land \beta)$, $(\alpha \lor \beta)$, $(\alpha \to \beta)$ are WFFs

### Strings over Alphabet $\Sigma$

- Base: Empty string $\epsilon \in \Sigma^*$
- Recursive: If $w \in \Sigma^*$ and $a \in \Sigma$, then $wa \in \Sigma^*$

---

## 5. Structural Induction

To prove properties of recursively defined objects, use **structural induction**:

1. **Base case:** Prove for base elements
2. **Inductive step:** Assuming true for components, prove for constructed objects

**Example:** Prove every WFF has balanced parentheses.

- Base: Variables have 0 parentheses — balanced ✓
- Inductive: If $\alpha$ and $\beta$ are balanced, then $(\alpha \land \beta)$ adds one "(" and one ")" — still balanced ✓

---

## 6. Recursion vs. Iteration

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| Structure | Function calls itself | Loop repeats |
| Memory | Stack frames (can overflow) | Constant (usually) |
| Readability | Often more elegant | Sometimes simpler |
| Performance | Can be slow (repeated work) | Generally faster |
| Conversion | Can always be converted | Via explicit stack |

```
  Factorial - Recursive:       Factorial - Iterative:
  
  fact(n):                     fact(n):
    if n == 0: return 1          result = 1
    return n * fact(n-1)         for i = 1 to n:
                                     result *= i
                                 return result
```

**Tail recursion** (last action is recursive call) can be automatically converted to iteration by compilers.

---

## 7. Mutual Recursion

Two or more functions defined in terms of each other:

$$E(n) = \begin{cases} 1 & n = 0 \\ O(n-1) & n > 0 \end{cases} \qquad O(n) = \begin{cases} 0 & n = 0 \\ E(n-1) & n > 0 \end{cases}$$

$E(n) = 1$ iff $n$ is even; $O(n) = 1$ iff $n$ is odd.

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Recursive Functions in Practice           │
  │                                                   │
  │  1. File System Traversal                        │
  │     List all files: process directory, then       │
  │     recursively process each subdirectory         │
  │                                                   │
  │  2. Parsing (Compilers)                          │
  │     Recursive descent parsers follow grammar      │
  │     rules that are themselves recursive           │
  │                                                   │
  │  3. Divide and Conquer Algorithms                │
  │     Merge Sort, Quick Sort, Binary Search         │
  │                                                   │
  │  4. Fractal Generation                           │
  │     Koch snowflake, Sierpinski triangle            │
  │     Self-similar at every scale                   │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Base case | Starting value(s) for the recursion |
| Recursive step | Rule to compute $f(n)$ from $f(k), k < n$ |
| Factorial | $n! = n \cdot (n-1)!$, $0! = 1$ |
| Fibonacci | $F(n) = F(n-1) + F(n-2)$ |
| Tower of Hanoi | $H(n) = 2H(n-1) + 1 = 2^n - 1$ |
| Structural induction | Proof technique for recursive structures |
| Tail recursion | Last action is recursive call (optimizable) |

---

## ❓ Quick Revision Questions

1. **Give a recursive definition for the sum $S(n) = 1 + 2 + \cdots + n$.**

2. **Compute $F(8)$ using the Fibonacci recurrence.**

3. **Write a recursive definition for the GCD using Euclid's algorithm: $\gcd(a, b)$.**

4. **Prove by structural induction that every well-formed formula in propositional logic has an even number of parentheses.**

5. **Convert the recursive Tower of Hanoi function to verify $H(4) = 15$.**

6. **What is the difference between direct and mutual recursion?**

---

[← Previous: Pigeonhole Principle](03-pigeonhole-principle.md) | [Back to README](../README.md) | [Next: Growth of Functions →](05-growth-of-functions.md)
