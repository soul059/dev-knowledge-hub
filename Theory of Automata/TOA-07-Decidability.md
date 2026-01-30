# Chapter 7: Decidability and Undecidability

## 7.1 Introduction

One of the most profound discoveries in computer science is that **some problems cannot be solved by any algorithm**. This chapter explores the boundary between what is computable and what is not.

### The Big Question

```
Can every well-defined problem be solved by an algorithm?

Answer: NO!

There exist problems that are:
- Well-defined (clear yes/no answer exists)
- But UNDECIDABLE (no algorithm can always give the answer)
```

---

## 7.2 Decidable vs. Undecidable Languages

### Definitions

**Decidable (Recursive) Language:**
```
L is decidable if there exists a TM M such that:
- M halts on every input
- M accepts w if w ∈ L
- M rejects w if w ∉ L

Also called: Recursive, Computable, Solvable
```

**Recognizable (RE) Language:**
```
L is recursively enumerable if there exists a TM M such that:
- M accepts w if w ∈ L
- M may reject OR loop forever if w ∉ L

Also called: Turing-recognizable, Semi-decidable
```

**Unrecognizable Language:**
```
L is not RE if no TM recognizes L.
```

### Hierarchy of Languages

```
┌─────────────────────────────────────────────────────────────┐
│                      ALL LANGUAGES                          │
│                                                              │
│   Most languages are not even describable!                   │
│   (Uncountably many languages, countably many TMs)           │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              Not Recursively Enumerable                │  │
│  │              (No TM recognizes these)                  │  │
│  │                                                        │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │         Recursively Enumerable (RE)              │  │  │
│  │  │         (TM recognizes, may not halt)            │  │  │
│  │  │                                                  │  │  │
│  │  │  ┌────────────────────────────────────────────┐  │  │  │
│  │  │  │           Recursive (Decidable)           │  │  │  │
│  │  │  │           (TM always halts)               │  │  │  │
│  │  │  │                                           │  │  │  │
│  │  │  │        CFL ⊂ CSL ⊂ Recursive              │  │  │  │
│  │  │  └────────────────────────────────────────────┘  │  │  │
│  │  │                                                  │  │  │
│  │  │  RE but not Recursive (e.g., Halting Problem)   │  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  │                                                        │  │
│  │  co-RE but not RE (complement of Halting Problem)      │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 7.3 Decidable Problems

### Examples of Decidable Problems

**For DFAs/NFAs/Regular Expressions:**
| Problem | Description | Decidable? |
|---------|-------------|------------|
| Acceptance | Does DFA M accept w? | Yes |
| Emptiness | Is L(M) = ∅? | Yes |
| Finiteness | Is L(M) finite? | Yes |
| Equivalence | Is L(M₁) = L(M₂)? | Yes |
| Subset | Is L(M₁) ⊆ L(M₂)? | Yes |

**For CFGs/PDAs:**
| Problem | Description | Decidable? |
|---------|-------------|------------|
| Acceptance | Does PDA P accept w? | Yes |
| Emptiness | Is L(G) = ∅? | Yes |
| Finiteness | Is L(G) finite? | Yes |
| CYK membership | Is w ∈ L(G)? | Yes |

**For Turing Machines:**
| Problem | Description | Decidable? |
|---------|-------------|------------|
| Does TM have 5 states? | Check encoding | Yes |
| Does TM ever write symbol X? | Can verify | No! |

### Why These Are Decidable

```
DFA Acceptance:
- Simulate DFA on input w
- DFA always halts (finite, deterministic)
- Check final state

CFG Membership (CYK):
- Convert to CNF
- Dynamic programming: O(n³)
- Always terminates
```

---

## 7.4 The Halting Problem

### Statement

```
HALTING PROBLEM (H):

Given: A Turing Machine M and input w
Question: Does M halt on input w?

H = {⟨M, w⟩ | M halts on input w}
```

### Theorem: The Halting Problem is Undecidable

This is one of the most important theorems in computer science.

### Proof by Contradiction (Diagonalization)

```
Assume H is decidable. Then there exists TM HALT such that:
  HALT(⟨M, w⟩) = {
    ACCEPT if M halts on w
    REJECT if M loops on w
  }

Construct a new TM D (the "diagonalizer"):
  D(⟨M⟩) = {
    Loop forever  if HALT(⟨M, ⟨M⟩⟩) = ACCEPT
    HALT         if HALT(⟨M, ⟨M⟩⟩) = REJECT
  }

Now run D on its own encoding ⟨D⟩:

Case 1: D(⟨D⟩) halts
  → HALT(⟨D, ⟨D⟩⟩) = REJECT  [by HALT's definition]
  → D doesn't halt on ⟨D⟩    [by HALT's correctness]
  → D(⟨D⟩) loops... CONTRADICTION!

Case 2: D(⟨D⟩) loops
  → HALT(⟨D, ⟨D⟩⟩) = ACCEPT  [by HALT's definition]
  → D halts on ⟨D⟩           [by HALT's correctness]
  → D(⟨D⟩) halts... CONTRADICTION!

Both cases lead to contradiction!
Therefore, HALT cannot exist.
The Halting Problem is UNDECIDABLE.
```

### Visual Representation

```
Consider table of TM behavior on TM encodings:

                 Input: ⟨M₁⟩  ⟨M₂⟩  ⟨M₃⟩  ⟨M₄⟩  ...
           ┌────────────────────────────────────────
    TM M₁  │    H      L      H      H     ...
    TM M₂  │    L      H      L      H     ...
    TM M₃  │    H      H      L      L     ...
    TM M₄  │    L      L      H      H     ...
     ...   │   ...    ...    ...    ...    ...

H = Halts, L = Loops

D is designed to differ on the diagonal:
- If Mᵢ halts on ⟨Mᵢ⟩, then D loops on ⟨Mᵢ⟩
- If Mᵢ loops on ⟨Mᵢ⟩, then D halts on ⟨Mᵢ⟩

D cannot be any Mᵢ in the list!
But D is a TM... so it should be in the list.
CONTRADICTION!
```

---

## 7.5 Consequences of the Halting Problem

### The Halting Problem is RE but not Recursive

```
L_halt is Recursively Enumerable:
- Simulate M on w
- If it halts, accept
- If it doesn't, we'll never know (loop)

L_halt is NOT Recursive:
- Cannot always determine if M loops
```

### The Complement is Not RE

```
L̄_halt = {⟨M, w⟩ | M does NOT halt on w}

L̄_halt is NOT recursively enumerable!

Proof: If both L and L̄ were RE:
- Run recognizers in parallel
- One must accept (input either in L or L̄)
- This would make L decidable
- But L_halt is not decidable... contradiction!
```

### Theorem: L is Recursive iff L and L̄ are both RE

```
L decidable ⟺ L ∈ RE and L̄ ∈ RE

       Recursive
      /         \
     ↙           ↘
   RE    ∩     co-RE
    \           /
     \         /
      → ... ←
```

---

## 7.6 More Undecidable Problems

### Undecidable Problems about TMs

| Problem | Description | Status |
|---------|-------------|--------|
| Halting | Does M halt on w? | Undecidable |
| Halting on ε | Does M halt on empty input? | Undecidable |
| Emptiness | Is L(M) = ∅? | Undecidable |
| Universality | Is L(M) = Σ*? | Undecidable |
| Equivalence | Is L(M₁) = L(M₂)? | Undecidable |
| Regularity | Is L(M) regular? | Undecidable |
| Finiteness | Is L(M) finite? | Undecidable |

### Programming Problems (Undecidable)

| Problem | Description |
|---------|-------------|
| Termination | Does program P terminate? |
| Correctness | Is program P correct? |
| Equivalence | Do P₁ and P₂ compute same function? |
| Dead code | Will statement S ever execute? |
| Optimization | Is this the optimal code? |

### Mathematical Problems (Undecidable)

| Problem | Description |
|---------|-------------|
| Hilbert's 10th | Does Diophantine equation have solution? |
| Post Correspondence | PCP (see below) |
| Wang Tiles | Can tiles cover the plane? |

---

## 7.7 Reduction

### What is a Reduction?

A **reduction** from problem A to problem B shows:
```
If we can solve B, we can solve A.

A ≤ B  means  "A reduces to B"

Contrapositive:
If A is undecidable, then B is undecidable.
```

### Mapping Reduction

```
A ≤_m B (A mapping-reduces to B) if:

There exists a computable function f such that:
    w ∈ A ⟺ f(w) ∈ B

    ┌───────────┐     f      ┌───────────┐
    │     A     │  ───────→  │     B     │
    │           │            │           │
    │  w ∈ A?   │            │ f(w) ∈ B? │
    └───────────┘            └───────────┘
```

### Using Reductions to Prove Undecidability

```
To prove B is undecidable:
1. Take a known undecidable problem A
2. Show A ≤_m B
3. If B were decidable, A would be decidable (contradiction)
4. Therefore, B is undecidable
```

### Example: Emptiness Problem is Undecidable

```
E_TM = {⟨M⟩ | L(M) = ∅}

Prove: E_TM is undecidable

Reduction from A_TM (acceptance problem):
A_TM = {⟨M, w⟩ | M accepts w}

Given ⟨M, w⟩, construct M':
  M'(x) = {
    Ignore x
    Simulate M on w
    If M accepts w, accept x
    If M rejects w, reject x
  }

Key insight:
- If M accepts w: M' accepts everything → L(M') = Σ* ≠ ∅
- If M doesn't accept w: M' accepts nothing → L(M') = ∅

So: ⟨M, w⟩ ∈ A_TM ⟺ ⟨M'⟩ ∉ E_TM

Since A_TM is undecidable, E_TM is undecidable.
(Actually, we showed complement of E_TM is undecidable,
 so E_TM is also undecidable)
```

---

## 7.8 Rice's Theorem

### Statement

```
RICE'S THEOREM:

Any non-trivial property of the language of a TM is undecidable.

Property P is "non-trivial" if:
- Some TMs have it (P is not always false)
- Some TMs don't have it (P is not always true)
```

### Formal Statement

```
Let P be a property of RE languages such that:
1. P ≠ ∅ (some RE language has property P)
2. P ≠ all RE languages (some RE language doesn't have P)

Then L_P = {⟨M⟩ | L(M) has property P} is undecidable.
```

### Examples Using Rice's Theorem

These are ALL undecidable (by Rice's Theorem):

```
1. Is L(M) finite?              (property of L(M))
2. Is L(M) regular?             (property of L(M))
3. Is L(M) context-free?        (property of L(M))
4. Is L(M) = Σ*?                (property of L(M))
5. Is L(M) = {0, 1}?            (property of L(M))
6. Does L(M) contain "hello"?   (property of L(M))
7. Is L(M) = L(M')?             (property of L(M))
```

### What Rice's Theorem Does NOT Apply To

Properties of the **machine** (not the language):

```
These may be decidable:
- Does M have exactly 5 states?        (property of M)
- Does M ever move left?               (check transitions)
- Does M have more states than symbols? (compare counts)
```

---

## 7.9 Post Correspondence Problem (PCP)

### Definition

```
Given: Two lists of strings over Σ:
  A = [α₁, α₂, ..., αₙ]
  B = [β₁, β₂, ..., βₙ]

Question: Is there a sequence of indices i₁, i₂, ..., iₖ such that:
  αᵢ₁ αᵢ₂ ... αᵢₖ = βᵢ₁ βᵢ₂ ... βᵢₖ
```

### Example

```
A = [a, ab, bba]
B = [baa, aa, bb]

Try: indices 3, 2, 3, 1
  A: bba + ab + bba + a = bbaabbba + a = bbaabbbaa
  B: bb + aa + bb + baa = bbaabbba + a = bbaabbbaa ✓

Solution exists!
```

### PCP is Undecidable

```
Theorem: PCP is undecidable.

Proof idea: Reduce the halting problem to PCP.
Encode TM computations as PCP instances.
```

### Modified PCP (MPCP)

Same as PCP but the first pair must be used first.

MPCP is also undecidable, and it's easier to reduce to.

---

## 7.10 Computable and Non-Computable Functions

### Computable Functions

A function f: Σ* → Σ* is **computable** if there exists a TM that:
- On input w, halts with f(w) on tape
- (If f(w) undefined, TM may not halt)

### Examples of Computable Functions

```
1. Arithmetic: add(n, m), multiply(n, m), factorial(n)
2. String operations: reverse(w), concatenate(u, v)
3. Sorting: sort(list)
4. Searching: find(pattern, text)
5. Parsing: parse(program)
```

### Non-Computable Functions

```
1. Busy Beaver function: BB(n) = maximum number of 1s
   a TM with n states can write before halting
   
2. Kolmogorov complexity: K(w) = length of shortest
   program that outputs w
```

### Busy Beaver Function

```
BB(n) = max {number of 1s written by halting TM with n states}

BB(1) = 1
BB(2) = 4
BB(3) = 6
BB(4) = 13
BB(5) ≥ 4098
BB(6) ≥ 10↑↑15 (tower of 15 powers of 10)

BB grows faster than any computable function!
BB is not computable.
```

---

## 7.11 Summary: The Landscape of Computability

### Classification of Languages

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  Recursive (Decidable)                                        │
│  ├── All regular languages                                    │
│  ├── All context-free languages                               │
│  ├── {aⁿbⁿcⁿ | n ≥ 0}                                         │
│  ├── Primality testing                                        │
│  └── Any problem with a guaranteed-halting algorithm         │
│                                                               │
│  RE but not Recursive                                         │
│  ├── A_TM = {⟨M, w⟩ | M accepts w}                           │
│  ├── HALT = {⟨M, w⟩ | M halts on w}                          │
│  └── Problems where "yes" can be verified but "no" cannot    │
│                                                               │
│  co-RE but not RE                                             │
│  ├── Complement of halting problem                            │
│  └── Problems where "no" can be verified but "yes" cannot    │
│                                                               │
│  Neither RE nor co-RE                                         │
│  └── Most languages (uncountably many)                        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Decision Chart

```
Given a language L, to determine its status:

1. Can you build a TM that always halts?
   YES → L is RECURSIVE (decidable)
   
2. Can you build a TM that accepts all w ∈ L?
   YES → L is RE
   
3. Can you build a TM that accepts all w ∉ L?
   YES → L̄ is RE (L is co-RE)
   
4. Both L and L̄ are RE?
   YES → L is RECURSIVE
   
5. Neither L nor L̄ is RE?
   → L is not RE and not co-RE
```

---

## 7.12 Practice Problems

### Problem 1: Classification

Classify each as Decidable, RE but not Decidable, or Not RE:

1. L = {⟨M⟩ | M is a TM with exactly 100 states}
2. L = {⟨M⟩ | M accepts at least one string}
3. L = {⟨M⟩ | L(M) is infinite}
4. L = {⟨M, w⟩ | M halts on w within 1000 steps}
5. L = {⟨M⟩ | M halts on every input}

### Problem 2: Reduction

Show that "Does M accept all strings of length 5?" is undecidable.

### Problem 3: Rice's Theorem

Which can be decided using properties of the machine (not applying Rice's Theorem)?
1. Does M have a transition that moves left?
2. Is L(M) empty?
3. Does M have more than 10 states?

### Solutions

**Problem 1:**
1. Decidable - just count states in encoding
2. RE but not decidable - can enumerate and simulate
3. Not decidable (Rice's Theorem), but RE
4. Decidable - simulate for 1000 steps
5. Not RE - co-RE (complement is RE)

**Problem 2:**
Reduce from A_TM. Given ⟨M, w⟩, create M' that:
- On strings of length ≠ 5: accept all
- On strings of length 5: simulate M on w, accept if M accepts

Then M accepts w iff M' accepts all strings of length 5.

**Problem 3:**
1. Decidable (property of machine, check transition function)
2. Not decidable (Rice's Theorem - property of L(M))
3. Decidable (property of machine, count states)

---

## 📌 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Decidable: TM always halts with correct answer              │
│  2. RE: TM accepts members, may loop on non-members             │
│  3. Halting Problem is undecidable (diagonal argument)          │
│  4. L decidable ⟺ L and L̄ both RE                               │
│  5. Reduction: A ≤ B means "A's difficulty ≤ B's difficulty"    │
│  6. Rice's Theorem: Non-trivial language properties undecidable │
│  7. Many practical problems are undecidable                     │
│  8. Undecidability ≠ unsolvable for specific inputs             │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Complexity Classes](TOA-08-Complexity-Theory.md)*
