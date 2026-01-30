# Chapter 9: Quick Revision Guide

## 📚 Complete Theory of Automata Summary

This chapter provides a one-stop revision of all major concepts.

---

# UNIT 1: Mathematical Foundations

## Key Definitions

| Term | Definition |
|------|------------|
| **Alphabet (Σ)** | Finite, non-empty set of symbols |
| **String** | Finite sequence of symbols from Σ |
| **ε** | Empty string (length 0) |
| **Σ*** | All strings over Σ (including ε) |
| **Σ⁺** | All non-empty strings (Σ* − {ε}) |
| **Language** | Any subset of Σ* |

## Important Formulas

```
|w| = length of string w
|wⁿ| = n × |w|
|Σⁿ| = |Σ|ⁿ (strings of length exactly n)
|P(A)| = 2^|A| (power set size)
```

## Language Operations

| Operation | Definition |
|-----------|------------|
| L₁ ∪ L₂ | {w \| w ∈ L₁ or w ∈ L₂} |
| L₁ ∩ L₂ | {w \| w ∈ L₁ and w ∈ L₂} |
| L₁ · L₂ | {xy \| x ∈ L₁, y ∈ L₂} |
| L* | ∪ᵢ₌₀^∞ Lⁱ |
| L̄ | Σ* − L |

---

# UNIT 2: Finite Automata

## DFA (Deterministic Finite Automaton)

```
DFA = (Q, Σ, δ, q₀, F)

Q  = finite set of states
Σ  = input alphabet  
δ  = Q × Σ → Q (exactly one transition)
q₀ = start state
F  = accepting states
```

## NFA (Non-deterministic Finite Automaton)

```
NFA = (Q, Σ, δ, q₀, F)

δ = Q × Σ → P(Q) (set of states)
Can have multiple transitions or none
```

## ε-NFA

```
δ = Q × (Σ ∪ {ε}) → P(Q)

ε-closure(q) = states reachable via ε-transitions
```

## Key Conversions

| From | To | Method |
|------|-----|--------|
| NFA → DFA | Subset Construction | Each DFA state = set of NFA states |
| ε-NFA → NFA | ε-elimination | Use ε-closures |
| DFA → Minimal DFA | Table Filling | Merge equivalent states |

## DFA Minimization

```
1. Mark (p, q) if one accepting, one not
2. Mark (p, q) if δ(p,a) and δ(q,a) lead to marked pair
3. Repeat until no change
4. Merge unmarked pairs
```

---

# UNIT 3: Regular Languages

## Regular Expression Operators

| Operator | Meaning | Precedence |
|----------|---------|------------|
| * | Zero or more | Highest |
| · | Concatenation | Middle |
| + | Union | Lowest |

## RE Identities

```
r + ∅ = r           r · ε = r
r + r = r           r · ∅ = ∅
(r*)* = r*          ε* = ε
r* = ε + rr*        ∅* = ε
```

## Conversions

| From | To | Method |
|------|-----|--------|
| RE → NFA | Thompson's Construction | Build recursively |
| DFA → RE | State Elimination | Remove states, update labels |

## Pumping Lemma (Regular)

```
For regular L, ∃p (pumping length):

∀w ∈ L with |w| ≥ p:
  w = xyz where:
  1. |y| > 0
  2. |xy| ≤ p  
  3. xyⁱz ∈ L for all i ≥ 0
```

### Using Pumping Lemma

```
To prove L not regular:
1. Assume L regular with pumping length p
2. Choose w ∈ L with |w| ≥ p
3. Show for all valid xyz divisions, some xyⁱz ∉ L
4. Contradiction → L not regular
```

## Closure Properties

| Operation | Regular? |
|-----------|----------|
| Union | ✓ |
| Intersection | ✓ |
| Complement | ✓ |
| Concatenation | ✓ |
| Kleene Star | ✓ |
| Reversal | ✓ |

---

# UNIT 4: Context-Free Languages

## Context-Free Grammar

```
CFG = (V, Σ, P, S)

V = variables (non-terminals)
Σ = terminals
P = productions (A → α)
S = start symbol
```

## Derivations

```
Leftmost: Always expand leftmost variable
Rightmost: Always expand rightmost variable
```

## Ambiguity

```
Grammar is ambiguous if string has:
- Two different parse trees, OR
- Two different leftmost derivations
```

## Normal Forms

### Chomsky Normal Form (CNF)
```
A → BC  (two variables)
A → a   (one terminal)
S → ε   (only if ε ∈ L)
```

### Greibach Normal Form (GNF)
```
A → aα  (terminal first, then variables)
```

## CNF Conversion Steps

```
1. Eliminate ε-productions
2. Eliminate unit productions (A → B)
3. Eliminate useless symbols
4. Convert to proper form
```

## Closure Properties

| Operation | CFL? |
|-----------|------|
| Union | ✓ |
| Concatenation | ✓ |
| Kleene Star | ✓ |
| Intersection | ✗ |
| Complement | ✗ |
| Intersection with Regular | ✓ |

## Pumping Lemma (CFL)

```
For CFL L, ∃p:

∀w ∈ L with |w| ≥ p:
  w = uvxyz where:
  1. |vy| > 0
  2. |vxy| ≤ p
  3. uvⁱxyⁱz ∈ L for all i ≥ 0
```

---

# UNIT 5: Pushdown Automata

## PDA Definition

```
PDA = (Q, Σ, Γ, δ, q₀, Z₀, F)

Γ = stack alphabet
δ = Q × (Σ ∪ {ε}) × Γ → P(Q × Γ*)
Z₀ = initial stack symbol
```

## Transition Notation

```
δ(q, a, X) = {(p, γ)}

In state q, reading a, with X on stack:
→ Go to state p
→ Replace X with γ (pop X, push γ)
```

## Acceptance Modes

```
By Final State: (q₀, w, Z₀) ⊢* (qf, ε, γ), qf ∈ F
By Empty Stack: (q₀, w, Z₀) ⊢* (q, ε, ε)
```

## Equivalences

```
CFG ↔ PDA (equivalent power)
DPDA < NPDA (deterministic less powerful)
```

---

# UNIT 6: Turing Machines

## TM Definition

```
TM = (Q, Σ, Γ, δ, q₀, B, F)

Γ = tape alphabet (includes blank B)
δ = Q × Γ → Q × Γ × {L, R}
```

## TM Transition

```
δ(q, X) = (p, Y, D)

In state q, reading X:
→ Write Y
→ Move head in direction D
→ Go to state p
```

## TM Variants (All Equivalent!)

```
- Multi-tape TM
- Non-deterministic TM
- Two-way infinite tape
- Multi-track TM
```

## Church-Turing Thesis

```
Anything "computable" can be computed by TM.
TM = Algorithm = Computation
```

---

# UNIT 7: Decidability

## Language Classes

```
Decidable (Recursive):
  TM always halts, correct answer

RE (Recursively Enumerable):
  TM accepts members, may loop on non-members

L is Decidable ⟺ L and L̄ are both RE
```

## The Halting Problem

```
HALT = {⟨M, w⟩ | M halts on w}

HALT is RE but NOT Decidable!
```

## Undecidable Problems

| Problem | Status |
|---------|--------|
| Does TM halt on input? | Undecidable |
| Is L(TM) empty? | Undecidable |
| Is L(TM) = Σ*? | Undecidable |
| Are two TMs equivalent? | Undecidable |

## Rice's Theorem

```
Any non-trivial property of L(TM) is undecidable.

Non-trivial = Some TMs have it, some don't
```

## Reduction

```
A ≤_m B: If B decidable, then A decidable
         If A undecidable, then B undecidable
```

---

# UNIT 8: Complexity Theory

## Time Complexity Classes

```
P = Polynomial time (deterministic)
NP = Polynomial time (non-deterministic)
    = Polynomial-time verifiable

P ⊆ NP (P = NP is unknown!)
```

## NP-Complete

```
L is NP-Complete if:
1. L ∈ NP
2. ∀A ∈ NP: A ≤_p L (L is NP-Hard)
```

## Famous NP-Complete Problems

```
SAT, 3-SAT
CLIQUE, VERTEX COVER, INDEPENDENT SET
HAMILTONIAN PATH/CYCLE
SUBSET SUM, KNAPSACK
GRAPH COLORING
TSP (decision version)
```

## Proving NP-Completeness

```
1. Show L ∈ NP (give verifier)
2. Reduce known NP-Complete problem to L
```

## Space Classes

```
L ⊆ NL ⊆ P ⊆ NP ⊆ PSPACE ⊆ EXPTIME

PSPACE = NPSPACE (Savitch's theorem)
```

---

# Quick Reference Tables

## Chomsky Hierarchy

| Type | Language | Grammar | Recognizer |
|------|----------|---------|------------|
| 3 | Regular | A → aB, A → a | FA |
| 2 | Context-Free | A → α | PDA |
| 1 | Context-Sensitive | αAβ → αγβ | LBA |
| 0 | Recursively Enumerable | Any | TM |

## Closure Properties Summary

| Operation | Regular | CFL |
|-----------|---------|-----|
| Union | ✓ | ✓ |
| Intersection | ✓ | ✗ |
| Complement | ✓ | ✗ |
| Concatenation | ✓ | ✓ |
| Kleene Star | ✓ | ✓ |
| Reversal | ✓ | ✓ |
| ∩ with Regular | ✓ | ✓ |

## Decision Problems

| Problem | Regular | CFL | RE |
|---------|---------|-----|-----|
| Membership | ✓ | ✓ | Semi |
| Emptiness | ✓ | ✓ | ✗ |
| Finiteness | ✓ | ✓ | ✗ |
| Equivalence | ✓ | ✗ | ✗ |
| Subset | ✓ | ✗ | ✗ |

---

# Common Exam Questions

## 1. Design Problems

**Design DFA for:**
- Strings with even number of 0s and 1s
- Strings not containing "101"
- Binary numbers divisible by 3

**Design CFG for:**
- {aⁿbⁿ | n ≥ 0}
- Balanced parentheses
- {aⁿbᵐ | n ≠ m}

**Design PDA for:**
- {aⁿbⁿ}
- {ww^R}
- {aⁿbᵐcⁿ⁺ᵐ}

## 2. Conversion Problems

- NFA to DFA (subset construction)
- RE to NFA (Thompson's)
- CFG to CNF
- CFG to PDA

## 3. Pumping Lemma Proofs

Show not regular:
- {aⁿbⁿ}
- {ww}
- {aⁿ² | n ≥ 0}

Show not CFL:
- {aⁿbⁿcⁿ}
- {ww}

## 4. Reduction Problems

Prove undecidable using reduction from halting problem.

## 5. Complexity

- Is problem in P?
- Is problem in NP? What's the certificate?
- Show NP-Complete via reduction

---

# Key Formulas Cheat Sheet

```
Strings:
|Σⁿ| = |Σ|ⁿ
|wⁿ| = n|w|
|P(A)| = 2^|A|

Regular:
NFA states → DFA states: up to 2ⁿ
Minimum DFA states = Myhill-Nerode equivalence classes

CFL:
CNF parse tree height for |w| = n: at most 2n-1
CYK algorithm: O(n³)

TM:
Space ≤ Time
Time ≤ 2^O(Space)

Complexity:
P ⊆ NP ⊆ PSPACE ⊆ EXPTIME
PSPACE = NPSPACE
```

---

# Memory Tricks

## Hierarchy (Inner to Outer)

```
"Regular → Context-Free → Context-Sensitive → RE"
"FA → PDA → LBA → TM"

Think: More memory = More power
FA: finite memory (states only)
PDA: stack memory (LIFO)
TM: tape memory (unlimited)
```

## Pumping Lemma

```
Regular: xyz (pump y)
CFL: uvxyz (pump v and y together)

"Regular pumps one, CFL pumps two"
```

## Decidability

```
"If it's about the LANGUAGE (not machine), 
 it's probably undecidable for TMs" (Rice's Theorem)
```

## NP-Completeness

```
"NP = Need Proof"
Certificate is the proof that verifier checks quickly.
```

---

# Practice Problem Solutions Template

## DFA Design Template

```
1. Identify what to "remember" (this becomes states)
2. Determine accepting condition
3. Draw start state
4. For each state, add transitions for all input symbols
5. Mark accepting states
6. Verify with test strings
```

## Pumping Lemma Template

```
1. "Assume L is regular/CFL with pumping length p"
2. "Choose w = [specific string in L with |w| ≥ p]"
3. "By pumping lemma, w = xyz/uvxyz with conditions..."
4. "Consider all possible divisions..."
5. "For i = [some value], xy^i z / uv^i xy^i z ∉ L because..."
6. "Contradiction! Therefore L is not regular/CFL."
```

## NP-Completeness Template

```
1. "Show L ∈ NP:"
   - "Certificate: [what the proof/witness is]"
   - "Verifier: [how to check in polynomial time]"
   
2. "Show L is NP-Hard via reduction from [known NP-Complete problem]:"
   - "Given instance of [problem], construct instance of L..."
   - "Show: original yes ⟺ constructed yes"
   - "Show: reduction is polynomial time"
```

---

# Final Exam Preparation Checklist

□ Can convert NFA to DFA (subset construction)
□ Can minimize DFA (table filling)
□ Can convert RE ↔ FA
□ Can apply pumping lemma (Regular and CFL)
□ Can convert CFG to CNF
□ Can design PDA for given CFL
□ Can trace TM computation
□ Understand decidability hierarchy
□ Know common undecidable problems
□ Can prove NP-Completeness via reduction
□ Know P, NP, NP-Complete definitions
□ Know closure properties for each language class

---

## Good Luck! 🎓

*Remember: Understanding the concepts is more important than memorizing. If you understand WHY something works, you can derive the HOW.*
