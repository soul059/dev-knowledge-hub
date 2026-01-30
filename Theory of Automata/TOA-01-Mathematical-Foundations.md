# Chapter 1: Mathematical Foundations

## 1.1 Introduction

Before diving into automata theory, we need a solid foundation in mathematical concepts. This chapter covers the essential mathematical tools used throughout the subject.

---

## 1.2 Sets and Set Operations

### What is a Set?

A **set** is a collection of distinct objects, called **elements** or **members**.

**Notation:**
- Set: `A = {1, 2, 3, 4, 5}`
- Membership: `3 ∈ A` (3 is in A), `6 ∉ A` (6 is not in A)
- Empty set: `∅` or `{}`

### Important Sets

| Symbol | Set |
|--------|-----|
| **ℕ** | Natural numbers: {0, 1, 2, 3, ...} |
| **ℤ** | Integers: {..., -2, -1, 0, 1, 2, ...} |
| **ℚ** | Rational numbers |
| **ℝ** | Real numbers |
| **∅** | Empty set |

### Set Operations

```
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

Union (A ∪ B):        {1, 2, 3, 4, 5, 6}    - All elements in A OR B
Intersection (A ∩ B): {3, 4}                - Elements in A AND B
Difference (A - B):   {1, 2}                - Elements in A but NOT in B
Complement (A'):      Everything NOT in A (relative to universal set)
```

### Visual Representation

```
    Union (A ∪ B)              Intersection (A ∩ B)
    ┌─────────────────┐        ┌─────────────────┐
    │  ┌─────┬─────┐  │        │  ┌─────┬─────┐  │
    │  │█████│█████│  │        │  │     │█████│  │
    │  │█ A █│█ B █│  │        │  │  A  │ ∩B  │  │
    │  │█████│█████│  │        │  │     │█████│  │
    │  └─────┴─────┘  │        │  └─────┴─────┘  │
    └─────────────────┘        └─────────────────┘
```

### Power Set

The **power set** P(A) is the set of all subsets of A.

```
If A = {a, b}
P(A) = {∅, {a}, {b}, {a, b}}

|P(A)| = 2^|A|
If |A| = n, then |P(A)| = 2^n
```

### Cartesian Product

The **Cartesian product** A × B is the set of all ordered pairs.

```
A = {1, 2}
B = {a, b}
A × B = {(1, a), (1, b), (2, a), (2, b)}

|A × B| = |A| × |B|
```

---

## 1.3 Relations

### What is a Relation?

A **relation** R from set A to set B is a subset of A × B.

```
A = {1, 2, 3}
B = {a, b}
R = {(1, a), (2, b), (3, a)} ⊆ A × B
```

### Types of Relations (on a set A)

| Property | Definition | Example on A = {1, 2, 3} |
|----------|------------|---------------------------|
| **Reflexive** | ∀a ∈ A: (a, a) ∈ R | {(1,1), (2,2), (3,3), ...} |
| **Symmetric** | (a, b) ∈ R ⟹ (b, a) ∈ R | If (1,2) then (2,1) |
| **Transitive** | (a,b), (b,c) ∈ R ⟹ (a,c) ∈ R | If (1,2) and (2,3) then (1,3) |
| **Antisymmetric** | (a,b), (b,a) ∈ R ⟹ a = b | No mutual pairs except (a,a) |

### Equivalence Relation

A relation that is **reflexive**, **symmetric**, and **transitive**.

**Example:** "Same remainder when divided by 3" on integers
- 7 ≡ 4 (mod 3) because both give remainder 1
- Equivalence classes: {0, 3, 6, ...}, {1, 4, 7, ...}, {2, 5, 8, ...}

### Closure of Relations

| Closure | What it adds |
|---------|--------------|
| **Reflexive closure** | Add (a, a) for all a |
| **Symmetric closure** | Add (b, a) for every (a, b) |
| **Transitive closure** | Add (a, c) for every path a → b → c |

---

## 1.4 Functions

### What is a Function?

A **function** f: A → B assigns exactly one element of B to each element of A.

```
A = {1, 2, 3}    B = {a, b, c}
f(1) = a
f(2) = b
f(3) = a

Domain: A
Codomain: B
Range: {a, b} ⊆ B
```

### Types of Functions

```
ONE-TO-ONE (Injective)         ONTO (Surjective)          BIJECTIVE
Different inputs →             Every output is            Both one-to-one
different outputs              reached                    AND onto

A        B                     A        B                 A        B
┌───┐   ┌───┐                 ┌───┐   ┌───┐              ┌───┐   ┌───┐
│ 1 │───│ a │                 │ 1 │───│ a │              │ 1 │───│ a │
│ 2 │───│ b │                 │ 2 │╲  │   │              │ 2 │───│ b │
│ 3 │───│ c │                 │ 3 │─╲─│ b │              │ 3 │───│ c │
│   │   │ d │                 │   │   │   │              │   │   │   │
└───┘   └───┘                 └───┘   └───┘              └───┘   └───┘
```

### Important Properties

| Type | Condition | Inverse exists? |
|------|-----------|-----------------|
| Injective (1-1) | f(a) = f(b) ⟹ a = b | Left inverse |
| Surjective (onto) | ∀b ∈ B, ∃a ∈ A: f(a) = b | Right inverse |
| Bijective | Both injective and surjective | Full inverse |

---

## 1.5 Alphabets, Strings, and Languages

### Alphabet (Σ)

An **alphabet** is a finite, non-empty set of symbols.

**Examples:**
```
Binary alphabet:    Σ = {0, 1}
English lowercase:  Σ = {a, b, c, ..., z}
DNA alphabet:       Σ = {A, C, G, T}
ASCII:              Σ = {all ASCII characters}
```

### String (Word)

A **string** is a finite sequence of symbols from an alphabet.

```
Σ = {a, b}
Strings: a, b, aa, ab, ba, bb, aaa, aab, ...
```

**Key Terminology:**
| Term | Symbol | Meaning | Example |
|------|--------|---------|---------|
| Empty string | ε or λ | String with no symbols | ε |
| Length | \|w\| | Number of symbols | \|abba\| = 4 |
| Concatenation | w₁w₂ | Joining strings | ab · ba = abba |
| Reverse | wᴿ | Symbols in reverse | (abba)ᴿ = abba |
| Prefix | - | Beginning portion | ab is prefix of abba |
| Suffix | - | Ending portion | ba is suffix of abba |

### Kleene Star and Plus

```
Σ = {a, b}

Σ* (Kleene Star) = All strings including ε
                 = {ε, a, b, aa, ab, ba, bb, aaa, ...}

Σ⁺ (Kleene Plus) = All strings excluding ε
                 = {a, b, aa, ab, ba, bb, aaa, ...}

Σ⁺ = Σ* - {ε}
Σ* = Σ⁺ ∪ {ε}
```

### Powers of Strings

```
w = ab
w⁰ = ε
w¹ = ab
w² = abab
w³ = ababab
wⁿ = w concatenated n times
```

### Language

A **language** L over alphabet Σ is any subset of Σ*.

```
Σ = {a, b}

L₁ = {a, aa, aaa}           - Finite language
L₂ = {aⁿ | n ≥ 0}           - Infinite language (all a's)
L₃ = {aⁿbⁿ | n ≥ 0}         - Equal a's followed by b's
L₄ = ∅                      - Empty language
L₅ = {ε}                    - Language with only empty string

Note: ∅ ≠ {ε}
      ∅ has no strings
      {ε} has one string (the empty string)
```

---

## 1.6 Operations on Languages

### Basic Operations

```
L₁ = {a, ab}
L₂ = {b, ba}

Union:         L₁ ∪ L₂ = {a, ab, b, ba}
Intersection:  L₁ ∩ L₂ = ∅
Difference:    L₁ - L₂ = {a, ab}
Complement:    L̄₁ = Σ* - L₁
```

### Concatenation

```
L₁ · L₂ = {xy | x ∈ L₁ and y ∈ L₂}

L₁ = {a, ab}
L₂ = {b, ba}

L₁ · L₂ = {ab, aba, abb, abba}
          (a·b, a·ba, ab·b, ab·ba)
```

### Kleene Closure

```
L* = L⁰ ∪ L¹ ∪ L² ∪ L³ ∪ ...

where:
L⁰ = {ε}
L¹ = L
L² = L · L
L³ = L · L · L

Example:
L = {a, b}
L* = {ε, a, b, aa, ab, ba, bb, aaa, ...} = Σ*
```

### Positive Closure

```
L⁺ = L¹ ∪ L² ∪ L³ ∪ ...
   = L* - {ε}  (only if ε ∉ L)
   = L · L*
```

### Reversal

```
Lᴿ = {wᴿ | w ∈ L}

L = {ab, ba, abc}
Lᴿ = {ba, ab, cba}
```

---

## 1.7 Proof Techniques

### 1. Direct Proof

Prove P → Q by assuming P and deriving Q.

**Example:** Prove that if n is even, then n² is even.
```
Assume n is even.
Then n = 2k for some integer k.
n² = (2k)² = 4k² = 2(2k²)
Since 2k² is an integer, n² is even.
```

### 2. Proof by Contradiction

Assume the negation, derive a contradiction.

**Example:** Prove √2 is irrational.
```
Assume √2 is rational.
Then √2 = p/q where p, q have no common factors.
2 = p²/q²
p² = 2q²
So p² is even, meaning p is even.
Let p = 2k.
(2k)² = 2q²
4k² = 2q²
q² = 2k²
So q² is even, meaning q is even.
But p and q both even contradicts "no common factors."
Therefore, √2 is irrational.
```

### 3. Proof by Induction

For proving properties of natural numbers.

**Structure:**
```
1. Base Case: Prove P(0) or P(1)
2. Inductive Step: Assume P(k), prove P(k+1)
3. Conclusion: P(n) holds for all n
```

**Example:** Prove 1 + 2 + 3 + ... + n = n(n+1)/2

```
Base Case (n = 1):
  Left: 1
  Right: 1(2)/2 = 1 ✓

Inductive Step:
  Assume true for n = k: 1 + 2 + ... + k = k(k+1)/2
  
  Prove for n = k + 1:
  1 + 2 + ... + k + (k+1)
  = k(k+1)/2 + (k+1)         [by inductive hypothesis]
  = (k+1)(k/2 + 1)
  = (k+1)(k+2)/2
  = (k+1)((k+1)+1)/2 ✓
```

### 4. Structural Induction

For proving properties of recursively defined structures.

**Example:** Prove property P for all strings
```
Base Case: Prove P(ε)
Inductive Step: If P(w), prove P(wa) for all a ∈ Σ
```

### 5. Proof by Counterexample

Disprove by finding one counterexample.

**Example:** Disprove "All prime numbers are odd"
```
Counterexample: 2 is prime and even.
```

### 6. Pigeonhole Principle

If n+1 objects are placed in n boxes, at least one box has 2+ objects.

**Application in Automata:** Used in proving pumping lemma.
```
If a DFA has n states and accepts a string of length ≥ n,
some state must be visited twice (creating a loop).
```

---

## 1.8 Recursive Definitions

### Defining Sets Recursively

**Example: Natural Numbers**
```
Base: 0 ∈ ℕ
Recursive: If n ∈ ℕ, then n + 1 ∈ ℕ
Closure: Nothing else is in ℕ
```

**Example: Σ* (All strings over Σ)**
```
Base: ε ∈ Σ*
Recursive: If w ∈ Σ* and a ∈ Σ, then wa ∈ Σ*
Closure: Nothing else is in Σ*
```

### Recursive Function Definitions

**String Length:**
```
|ε| = 0
|wa| = |w| + 1   (for w ∈ Σ*, a ∈ Σ)
```

**String Reversal:**
```
εᴿ = ε
(wa)ᴿ = a(wᴿ)   (for w ∈ Σ*, a ∈ Σ)
```

**Concatenation:**
```
w · ε = w
w · (xa) = (w · x)a
```

---

## 1.9 Graphs and Trees

### Graphs

A **graph** G = (V, E) consists of:
- V: set of vertices (nodes)
- E: set of edges (connections)

```
Undirected Graph:        Directed Graph (Digraph):
    1 ─── 2                  1 ──→ 2
    │     │                  ↑     ↓
    │     │                  │     │
    4 ─── 3                  4 ←── 3
```

### Graph Terminology

| Term | Meaning |
|------|---------|
| **Path** | Sequence of vertices connected by edges |
| **Cycle** | Path that starts and ends at same vertex |
| **Connected** | Path exists between every pair of vertices |
| **Degree** | Number of edges incident to a vertex |
| **DAG** | Directed Acyclic Graph |

### Trees

A **tree** is a connected acyclic graph.

```
         Root
          │
    ┌─────┼─────┐
    │     │     │
   ○─┐   ○─┐   ○─┐
   │ │   │ │   │ │
   ○ ○   ○ ○   ○ ○
        Leaves
```

**Properties:**
- n vertices, n-1 edges
- Unique path between any two vertices
- Removing any edge disconnects the tree

---

## 1.10 Practice Problems

### Problem Set

1. **Sets:** If A = {1, 2, 3} and B = {2, 3, 4}, find:
   - A ∪ B
   - A ∩ B
   - A - B
   - P(A ∩ B)

2. **Strings:** For Σ = {0, 1} and w = 0110:
   - Find |w|
   - Find wᴿ
   - Find w²
   - List all prefixes of w

3. **Languages:** For L = {a, ab}:
   - Find L²
   - Find first 5 elements of L*

4. **Proof:** Prove using induction that |wⁿ| = n|w| for any string w.

5. **Functions:** Is f: ℤ → ℤ defined by f(x) = x² injective? Surjective?

### Solutions

1. **Sets:**
   - A ∪ B = {1, 2, 3, 4}
   - A ∩ B = {2, 3}
   - A - B = {1}
   - P(A ∩ B) = {∅, {2}, {3}, {2,3}}

2. **Strings:**
   - |w| = 4
   - wᴿ = 0110
   - w² = 01100110
   - Prefixes: ε, 0, 01, 011, 0110

3. **Languages:**
   - L² = {aa, aab, aba, abab}
   - L* starts with: {ε, a, ab, aa, aab, aba, ...}

---

## 📌 Key Takeaways

```
┌──────────────────────────────────────────────────────────┐
│  1. Alphabet (Σ) = finite set of symbols                │
│  2. String = sequence of symbols from Σ                 │
│  3. Language = subset of Σ*                             │
│  4. Σ* = all possible strings (including ε)             │
│  5. Kleene closure L* = L⁰ ∪ L¹ ∪ L² ∪ ...              │
│  6. Proof by induction: Base + Inductive Step           │
│  7. ∅ ≠ {ε} (empty language vs. language with ε)        │
└──────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Finite Automata - DFA and NFA](TOA-02-Finite-Automata.md)*
