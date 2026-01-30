# Chapter 8: Complexity Theory

## 8.1 Introduction

While decidability asks "CAN we solve this problem?", **complexity theory** asks "HOW EFFICIENTLY can we solve it?"

### The Core Questions

```
1. How much TIME does an algorithm need?
2. How much SPACE (memory) does it need?
3. Can we do better?
4. What problems are inherently hard?
```

---

## 8.2 Time Complexity

### Measuring Time

The **time complexity** of a TM M on input w is the number of steps before halting.

```
TIME_M(w) = number of moves M makes on input w
            (undefined if M doesn't halt)
```

### Time Complexity of a TM

```
T_M(n) = max{TIME_M(w) | |w| = n}

The worst-case time over all inputs of length n.
```

### Asymptotic Notation (Big-O)

```
O(f(n)):    Upper bound (at most)
Ω(f(n)):    Lower bound (at least)  
Θ(f(n)):    Tight bound (exactly)
o(f(n)):    Strictly less than
ω(f(n)):    Strictly greater than

Example:
3n² + 5n + 7 = O(n²) = Ω(n²) = Θ(n²)
```

### Common Complexity Classes by Time

```
        Fast
          ↑
     O(1)      │  Constant
     O(log n)  │  Logarithmic
     O(n)      │  Linear
     O(n log n)│  Linearithmic
     O(n²)     │  Quadratic
     O(n³)     │  Cubic
     O(2ⁿ)     │  Exponential
     O(n!)     │  Factorial
          ↓
        Slow

For n = 1000:
O(n)    = 1,000
O(n²)   = 1,000,000
O(n³)   = 1,000,000,000
O(2ⁿ)   = unimaginably large
```

---

## 8.3 Space Complexity

### Measuring Space

The **space complexity** is the number of tape cells used.

```
SPACE_M(w) = number of distinct tape cells used by M on w
```

### Space Complexity of a TM

```
S_M(n) = max{SPACE_M(w) | |w| = n}
```

### Time-Space Relationship

```
SPACE(n) ≤ TIME(n)     (can't use more cells than steps)
TIME(n) ≤ 2^O(SPACE(n)) (bounded configurations → bounded time)
```

---

## 8.4 The Class P (Polynomial Time)

### Definition

```
P = ⋃ TIME(nᵏ) for all k ≥ 1

P = class of languages decidable in polynomial time
    by a deterministic TM

P = {L | ∃ TM M, ∃ constant c:
     M decides L in time O(nᶜ)}
```

### Interpretation

```
P represents "efficiently solvable" problems.

Polynomial time is considered tractable because:
- O(n²) might be slow, but scales reasonably
- O(2ⁿ) becomes impractical very quickly
- The gap is enormous for large n
```

### Examples of Problems in P

| Problem | Time Complexity |
|---------|-----------------|
| Searching sorted array | O(log n) |
| Sorting | O(n log n) |
| Matrix multiplication | O(n³) (naïve) |
| Shortest path | O(n² log n) |
| Primality testing | O(log⁶ n) |
| Maximum flow | O(n³) |
| Linear programming | Polynomial |
| 2-SAT | O(n) |
| Context-free parsing | O(n³) |

### P is Robust

```
P is the same class for:
- Single-tape TM
- Multi-tape TM
- RAM model
- Any "reasonable" deterministic model

(May differ by polynomial factor, but stays polynomial)
```

---

## 8.5 The Class NP (Nondeterministic Polynomial Time)

### Definition

```
NP = ⋃ NTIME(nᵏ) for all k ≥ 1

NP = class of languages decidable in polynomial time
     by a NON-DETERMINISTIC TM
```

### Alternate Definition (Verifier)

```
NP = class of languages with polynomial-time VERIFIERS

L ∈ NP if there exists:
- A polynomial-time TM V (verifier)
- A polynomial p(n)

Such that:
L = {w | ∃ certificate c with |c| ≤ p(|w|)
         and V(w, c) accepts}
```

### Understanding NP

```
For L ∈ NP:
- "Yes" instances have short proofs (certificates)
- The proof can be verified quickly (polynomial time)
- Finding the proof might be hard

Example: COMPOSITE
  - Input: number n
  - Certificate: factor p (1 < p < n, p divides n)
  - Verification: check if n mod p = 0 (easy!)
```

### Examples in NP

| Problem | Certificate |
|---------|-------------|
| SAT | Satisfying assignment |
| CLIQUE | Set of k vertices |
| HAMILTONIAN PATH | Path sequence |
| SUBSET SUM | Subset that sums to target |
| GRAPH COLORING | Color assignment |
| TSP (decision) | Tour of length ≤ k |

---

## 8.6 P vs NP

### The Relationship

```
P ⊆ NP

Every problem in P is also in NP.
(Deterministic TM is special case of non-deterministic)
```

### The Million Dollar Question

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                       P = NP ?                                   │
│                                                                  │
│  One of the greatest unsolved problems in mathematics            │
│  Clay Millennium Prize: $1,000,000                               │
│                                                                  │
│  Most experts believe: P ≠ NP                                    │
│  But no one has proven it!                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implications

```
If P = NP:
  - All NP problems solvable efficiently
  - Cryptography broken (RSA, etc.)
  - Optimization becomes easy
  - AI problems become tractable
  - Theorem proving becomes efficient

If P ≠ NP:
  - Some problems are inherently hard
  - Cryptography remains secure
  - Hard problems require approximation
  - Current beliefs confirmed
```

---

## 8.7 NP-Completeness

### Polynomial-Time Reductions

```
A ≤_p B (A polynomial-time reduces to B)

If there exists polynomial-time computable function f such that:
    w ∈ A ⟺ f(w) ∈ B

Meaning: If we can solve B in polynomial time,
         we can solve A in polynomial time.
```

### NP-Hard

```
L is NP-HARD if:
    For every A ∈ NP: A ≤_p L

"At least as hard as everything in NP"
```

### NP-Complete

```
L is NP-COMPLETE if:
    1. L ∈ NP
    2. L is NP-hard

"The hardest problems in NP"
```

### Visual Representation

```
If P ≠ NP:
          ┌─────────────────────────────────────┐
          │               NP                    │
          │  ┌───────────────────────────────┐  │
          │  │         NP-Complete           │  │
          │  │  ┌─────────────────────────┐  │  │
          │  │  │   SAT, CLIQUE, TSP...   │  │  │
          │  │  └─────────────────────────┘  │  │
          │  └───────────────────────────────┘  │
          │                                     │
          │         NP-Intermediate             │
          │     (if they exist: factoring?)     │
          │                                     │
          │  ┌───────────────────────────────┐  │
          │  │              P                │  │
          │  │  Sorting, Shortest Path...    │  │
          │  └───────────────────────────────┘  │
          └─────────────────────────────────────┘

Outside NP: NP-Hard but not NP-Complete
           (e.g., Halting Problem)
```

---

## 8.8 Cook-Levin Theorem

### SAT Problem

```
SATISFIABILITY (SAT):
  Input: Boolean formula φ
  Question: Is φ satisfiable?

Example:
  φ = (x₁ ∨ x₂) ∧ (¬x₁ ∨ x₃) ∧ (¬x₂ ∨ ¬x₃)
  
  Is there assignment to x₁, x₂, x₃ making φ true?
  
  Try: x₁ = T, x₂ = F, x₃ = T
       (T ∨ F) ∧ (F ∨ T) ∧ (T ∨ F)
       = T ∧ T ∧ T = T ✓
```

### The Theorem

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  COOK-LEVIN THEOREM (1971):                                      │
│                                                                  │
│  SAT is NP-Complete.                                             │
│                                                                  │
│  - SAT ∈ NP (verification is easy)                               │
│  - Every NP problem reduces to SAT                               │
│                                                                  │
│  First problem proven NP-Complete!                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Proof Idea

```
Any NP problem A has a polynomial-time verifier V.

V is a TM, which can be encoded.
The computation of V(w, c) can be expressed as Boolean formula:
  - Variables for each tape cell at each step
  - Clauses encoding valid transitions
  - Formula satisfiable iff V accepts

This encodes arbitrary NP computation as SAT!
```

---

## 8.9 Important NP-Complete Problems

### Proving NP-Completeness

```
To prove L is NP-Complete:
1. Show L ∈ NP (give verifier or NTM)
2. Reduce known NP-Complete problem to L
   (Find L' ∈ NP-Complete such that L' ≤_p L)
```

### Classic NP-Complete Problems

**SAT and Variants:**
```
SAT:    Any Boolean formula
3-SAT:  Formula in CNF with 3 literals per clause
        (x₁ ∨ x₂ ∨ ¬x₃) ∧ (¬x₁ ∨ x₄ ∨ x₂) ∧ ...

Note: 2-SAT is in P!
```

**Graph Problems:**
```
CLIQUE: Is there a clique of size k in graph G?
VERTEX COVER: Can k vertices cover all edges?
INDEPENDENT SET: Is there independent set of size k?
GRAPH COLORING: Can G be colored with k colors?
HAMILTONIAN PATH: Is there path visiting all vertices once?
HAMILTONIAN CYCLE: Is there cycle visiting all vertices once?
```

**Number Problems:**
```
SUBSET SUM: Is there subset summing to target?
KNAPSACK: Can items fit with value ≥ target?
PARTITION: Can set be partitioned into equal halves?
```

**Scheduling:**
```
JOB SCHEDULING: Can jobs be scheduled on machines?
```

### Reduction Chain

```
SAT
 │
 ├──→ 3-SAT
 │      │
 │      ├──→ CLIQUE ←──→ INDEPENDENT SET ←──→ VERTEX COVER
 │      │      │
 │      │      └──→ HAMILTONIAN CYCLE ──→ TSP
 │      │
 │      └──→ 3-COLORING
 │
 └──→ SUBSET SUM ──→ KNAPSACK ──→ PARTITION
```

---

## 8.10 Example Reductions

### 3-SAT ≤_p CLIQUE

```
Given: 3-CNF formula φ with k clauses

Create graph G:
- Nodes: one for each literal in each clause
- Edges: between compatible literals from different clauses
  (compatible = not negations of each other)

Claim: φ satisfiable ⟺ G has clique of size k

Why? A k-clique corresponds to selecting one true
     literal from each clause (consistency guaranteed
     by edge requirement)
```

### CLIQUE ≤_p VERTEX COVER

```
G has clique of size k ⟺ Ḡ has vertex cover of size n-k

(Ḡ is complement graph)
```

### CLIQUE ≤_p INDEPENDENT SET

```
G has clique of size k ⟺ Ḡ has independent set of size k
```

---

## 8.11 Dealing with NP-Completeness

### Strategies for Hard Problems

```
When facing NP-Complete problem:

1. EXACT ALGORITHMS
   - Might be okay for small inputs
   - Exponential, but optimized (backtracking, branch & bound)

2. APPROXIMATION
   - Find solution within factor of optimal
   - Vertex Cover: 2-approximation in O(n²)
   - TSP (with triangle inequality): 1.5-approximation

3. SPECIAL CASES
   - Identify tractable restrictions
   - 2-SAT is P, 3-SAT is NP-Complete
   - Trees often make problems easier

4. HEURISTICS
   - Genetic algorithms
   - Simulated annealing
   - No guarantee, but often works in practice

5. RANDOMIZATION
   - Sometimes randomness helps
   - Expected polynomial time

6. PARAMETERIZED COMPLEXITY
   - Exponential in parameter, polynomial in input
   - O(2ᵏ · n) might be okay if k is small
```

---

## 8.12 Other Complexity Classes

### Space Classes

```
L (LOGSPACE): Decidable in O(log n) space
NL: Decidable in O(log n) space by NTM
PSPACE: Decidable in polynomial space
NPSPACE: Polynomial space by NTM

Note: PSPACE = NPSPACE (Savitch's theorem)
```

### Class Relationships

```
L ⊆ NL ⊆ P ⊆ NP ⊆ PSPACE ⊆ EXPTIME

Known: L ≠ PSPACE, P ≠ EXPTIME
Unknown: All other distinctions
```

### PSPACE-Complete

```
PSPACE-Complete problems:
- TQBF (True Quantified Boolean Formulas)
- Generalized chess, checkers, Go
- Regular expression equivalence with ∩ and complement
```

### Beyond NP: The Polynomial Hierarchy

```
PH = Σ₀ ∪ Σ₁ ∪ Σ₂ ∪ ...

Σ₀ = P
Σ₁ = NP
Π₁ = co-NP
Σ₂ = NP^NP (NP with NP oracle)
...

If P = NP, then PH collapses to P.
```

---

## 8.13 co-NP

### Definition

```
co-NP = {L | L̄ ∈ NP}

Languages whose complements are in NP.
```

### Example

```
TAUTOLOGY = {φ | φ is true for all assignments}

- TAUTOLOGY ∈ co-NP
- φ ∈ TAUTOLOGY ⟺ ¬φ ∉ SAT
- A "no" instance has short proof (falsifying assignment)
```

### P, NP, and co-NP

```
P ⊆ NP ∩ co-NP

If L ∈ P, then L ∈ NP and L ∈ co-NP.

Unknown: Is NP = co-NP?
         Is NP ∩ co-NP = P?
```

---

## 8.14 Summary: Complexity Landscape

```
┌─────────────────────────────────────────────────────────────────┐
│                         EXPTIME                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      PSPACE                                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                 NP     co-NP                        │  │  │
│  │  │              ╱           ╲                          │  │  │
│  │  │   NP-Complete   P   co-NP-Complete                 │  │  │
│  │  │              ╲           ╱                          │  │  │
│  │  │                NP ∩ co-NP                           │  │  │
│  │  │                   │                                 │  │  │
│  │  │               L ⊆ NL ⊆ P                            │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  EXPTIME-Complete: Chess, Go (generalized)                       │
│  PSPACE-Complete: TQBF, games                                    │
│  NP-Complete: SAT, CLIQUE, TSP                                   │
│  P: Sorting, shortest path, primality                            │
│  L: Undirected reachability                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8.15 Practice Problems

### Problem 1: Classify

1. Sorting an array
2. Determining if a graph is 3-colorable
3. Computing optimal TSP tour
4. Checking if a number is prime
5. Solving a Sudoku puzzle (general n×n²)

### Problem 2: Reduction

Show that INDEPENDENT SET ≤_p CLIQUE

### Problem 3: Verification

What is a polynomial-time verifier for SUBSET SUM?

### Solutions

**Problem 1:**
1. P (O(n log n))
2. NP-Complete
3. NP-Hard (optimization version), NP-Complete (decision)
4. P (AKS algorithm)
5. NP-Complete

**Problem 2:**
```
Given G, k for INDEPENDENT SET:
Create Ḡ (complement graph)
G has independent set of size k ⟺ Ḡ has clique of size k
```

**Problem 3:**
```
Verifier V(S, target, certificate):
  certificate = subset of indices
  sum = 0
  for each index i in certificate:
    sum += S[i]
  return (sum == target)

Time: O(n) to verify
```

---

## 📌 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  1. P = polynomial-time decidable (tractable)                   │
│  2. NP = polynomial-time verifiable                             │
│  3. P ⊆ NP, but P = NP is unknown ($1M problem!)                │
│  4. NP-Complete = hardest problems in NP                        │
│  5. Cook-Levin: SAT is NP-Complete (first proof)                │
│  6. NP-Completeness proven by reduction                         │
│  7. For NP-Complete: use approximation, heuristics, etc.        │
│  8. Space: L ⊆ NL ⊆ P ⊆ NP ⊆ PSPACE                             │
│  9. co-NP: complements of NP languages                          │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Quick Revision Guide](TOA-09-Quick-Revision.md)*
