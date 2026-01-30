# Chapter 6: Turing Machines

## 6.1 Introduction

The **Turing Machine (TM)** is the most powerful computational model. It captures the essence of what it means to "compute" - anything that can be computed by any algorithm can be computed by a Turing Machine.

### Historical Context

```
1936: Alan Turing published "On Computable Numbers"
      - Defined the Turing Machine
      - Proved limitations of computation
      - Foundation of computer science

The Turing Machine predates actual computers!
It's a theoretical model, not a physical machine.
```

### Why Study Turing Machines?

1. **Definition of computability**: What CAN be computed
2. **Limits of computation**: What CANNOT be computed
3. **Complexity theory**: How HARD is computation
4. **Foundation**: Theoretical basis for all of computer science

---

## 6.2 Intuitive Description

### Components of a Turing Machine

```
                    ┌─────────────────────────────────────┐
                    │         Infinite Tape                │
                    │ ┌───┬───┬───┬───┬───┬───┬───┬───┐   │
                    │ │ a │ b │ a │ ▷ │ b │ a │ B │ B │...│
                    │ └───┴───┴───┴───┴───┴───┴───┴───┘   │
                    │               ↑                      │
                    │          Read/Write Head             │
                    └─────────────────────────────────────┘
                                    │
                                    ↓
                           ┌───────────────┐
                           │    Finite     │
                           │   Control     │
                           │   (state q)   │
                           └───────────────┘

Key features:
- Infinite tape (in both directions or one direction)
- Read/Write head (can read, write, move left/right)
- Finite control (current state)
- Deterministic operation
```

### What Makes TM Powerful?

| Feature | FA | PDA | TM |
|---------|-----|-----|-----|
| Memory | Finite (state) | Stack (LIFO) | Tape (random access) |
| Read/Write | Read only | Stack: R/W, Tape: Read | Read and Write |
| Movement | Left to right | Stack: top only | Left and Right |
| Input processing | One direction | One direction | Bidirectional |

---

## 6.3 Formal Definition

### Turing Machine

A **TM** is a 7-tuple M = (Q, Σ, Γ, δ, q₀, B, F) where:

| Component | Meaning |
|-----------|---------|
| Q | Finite set of states |
| Σ | Input alphabet (⊆ Γ, doesn't include blank) |
| Γ | Tape alphabet (Σ ⊆ Γ) |
| δ | Transition function: Q × Γ → Q × Γ × {L, R} |
| q₀ | Start state (q₀ ∈ Q) |
| B | Blank symbol (B ∈ Γ - Σ) |
| F | Set of accepting/final states (F ⊆ Q) |

### Transition Function

```
δ(q, X) = (p, Y, D)

Meaning:
- In state q, reading symbol X on tape
- Write symbol Y (replace X)
- Move head in direction D (L = Left, R = Right)
- Go to state p
```

### Transition Notation

```
        X/Y, D
    q ──────────→ p

or

    δ(q, X) = (p, Y, R)   [write Y, move right]
```

---

## 6.4 Configurations and Computation

### Instantaneous Description (ID)

A **configuration** captures the complete machine state:

```
α q β

where:
- q is current state
- αβ is tape contents (non-blank portion)
- Head is on first symbol of β

Example: aab q₁ bba
- State: q₁
- Tape: aabbba
- Head on: first 'b' after q₁
```

### Move Relation (⊢)

**Move Right:**
```
If δ(q, X) = (p, Y, R):
    α q Xβ ⊢ αY p β
```

**Move Left:**
```
If δ(q, X) = (p, Y, L):
    αZ q Xβ ⊢ α p ZY β
```

### Computation

A **computation** is a sequence of moves:
```
ID₀ ⊢ ID₁ ⊢ ID₂ ⊢ ... ⊢ IDₙ

- Halting: No more moves possible
- Accepting: Halt in accepting state
- Rejecting: Halt in non-accepting state
- Looping: Never halts (possible with TM!)
```

---

## 6.5 TM Examples

### Example 1: Accept {0ⁿ1ⁿ | n ≥ 1}

**Strategy:**
1. Mark first 0 as X
2. Move right to find first unmarked 1, mark as Y
3. Move left to find next unmarked 0
4. Repeat until all matched
5. Accept if all matched perfectly

```
States: q₀ (start), q₁ (moving right), q₂ (found 1, going left),
        q₃ (checking complete), q₄ (accept)

δ(q₀, 0) = (q₁, X, R)    [mark 0, go right]
δ(q₁, 0) = (q₁, 0, R)    [skip 0s]
δ(q₁, Y) = (q₁, Y, R)    [skip Ys]
δ(q₁, 1) = (q₂, Y, L)    [mark 1, go left]
δ(q₂, Y) = (q₂, Y, L)    [skip Ys]
δ(q₂, 0) = (q₂, 0, L)    [skip 0s]
δ(q₂, X) = (q₀, X, R)    [found marked 0, continue]
δ(q₀, Y) = (q₃, Y, R)    [no more 0s, verify]
δ(q₃, Y) = (q₃, Y, R)    [skip Ys]
δ(q₃, B) = (q₄, B, R)    [all done, accept]

Trace for "0011":
q₀ 0011B ⊢ Xq₁ 011B ⊢ X0q₁ 11B ⊢ X0q₂ Y1B ⊢ Xq₂ 0Y1B
         ⊢ q₂ X0Y1B ⊢ Xq₀ 0Y1B ⊢ XXq₁ Y1B ⊢ XXYq₁ 1B
         ⊢ XXq₂ YYB ⊢ Xq₂ XYYB ⊢ XXq₀ YYB ⊢ XXYq₃ YB
         ⊢ XXYYq₃ B ⊢ XXYYBq₄ [ACCEPT]
```

### Example 2: Binary Addition

```
Input: two binary numbers separated by +
Example: 101+011 (5+3)
Output: 1000 (8)

Strategy:
1. Find the rightmost digit of each number
2. Add with carry
3. Write result
4. Repeat

(Complex implementation - multiple states for carry handling)
```

### Example 3: Accept Palindromes

```
Strategy:
1. Remember first symbol, blank it
2. Move to end, check if matches
3. Blank last symbol if matches
4. Move to beginning
5. Repeat
6. Accept if all matched

This is non-deterministic for general alphabet
```

---

## 6.6 Language of a Turing Machine

### Accepted Language

```
L(M) = {w ∈ Σ* | q₀ w ⊢* α qf β for some qf ∈ F}
```

The language of strings that cause M to halt in an accepting state.

### Three Possible Outcomes

```
Input w to TM M:

1. ACCEPT: M halts in accepting state
   → w ∈ L(M)

2. REJECT: M halts in non-accepting state
   → w ∉ L(M)

3. LOOP: M never halts
   → w ∉ L(M) (but we can't tell by running M)
```

---

## 6.7 Variants of Turing Machines

### Key Theorem

All reasonable variants of TM are equivalent in power!

```
Standard TM = Multi-tape TM = Non-deterministic TM
            = Two-way infinite tape = etc.

They all recognize exactly the same class of languages.
```

### Multi-tape TM

```
┌─────────────────────┐
│ Tape 1: [a|b|a|B|..│  ← Input
│         ↑           │
├─────────────────────┤
│ Tape 2: [x|y|B|B|..│  ← Work tape
│            ↑        │
├─────────────────────┤
│ Tape 3: [1|0|1|B|..│  ← Output
│              ↑      │
└─────────────────────┘
        │
   Finite Control

Transition: δ(q, a₁, a₂, a₃) = (p, b₁, b₂, b₃, D₁, D₂, D₃)
```

**Equivalence:** Any k-tape TM can be simulated by single-tape TM.

### Non-deterministic TM

```
δ: Q × Γ → P(Q × Γ × {L, R})

Multiple possible moves at each step.
Machine accepts if ANY computational path accepts.
```

**Equivalence:** NTM can be simulated by DTM (using BFS on computation tree).

### Two-Way Infinite Tape

Tape extends infinitely in both directions.

**Equivalence:** Can be simulated by one-way infinite tape (interleave tracks).

### Multi-head TM

Multiple read/write heads on single tape.

**Equivalence:** Can be simulated by standard TM.

### Off-line TM

Separate read-only input tape and read/write work tape.

**Equivalence:** Same as standard TM.

---

## 6.8 The Church-Turing Thesis

### Statement

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CHURCH-TURING THESIS:                                          │
│                                                                  │
│  Everything that can be "computed" by any reasonable            │
│  algorithmic process can be computed by a Turing Machine.       │
│                                                                  │
│  TM = Algorithm = Computation                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Important Notes

1. This is a **thesis** (claim), not a theorem
2. It cannot be proven (informal concept of "algorithm")
3. Universally accepted by computer scientists
4. Never contradicted despite decades of attempts

### Evidence Supporting the Thesis

1. All reasonable computational models are equivalent to TM
2. Every algorithm ever devised can be implemented on a TM
3. Lambda calculus (Church) = TM (Turing) = Recursive functions (Gödel)

---

## 6.9 Universal Turing Machine

### Concept

A **Universal Turing Machine (UTM)** can simulate any other Turing Machine!

```
Input to UTM:
- Description of TM M (encoded as string)
- Input w for M

Output: 
- Whatever M would output on w

UTM(⟨M⟩, w) = M(w)
```

### Significance

1. **Programmable computer**: UTM is like a computer running a program
2. **Stored program concept**: The program (TM description) is data
3. **Foundation of modern computers**: Von Neumann architecture

### Encoding Turing Machines

A TM M can be encoded as a string ⟨M⟩:
```
⟨M⟩ = encoding of (Q, Σ, Γ, δ, q₀, B, F)

Example encoding:
- States: q₁ = 1, q₂ = 11, q₃ = 111, ...
- Symbols: a = 1, b = 11, B = 111, ...
- Directions: L = 1, R = 11
- Transitions separated by 00
- Complete encoding separated by 000
```

---

## 6.10 Turing Machine as Language Recognizers

### Recursively Enumerable Languages (RE)

A language L is **recursively enumerable** (RE) if there exists a TM M such that:
```
L = L(M)

For w ∈ L: M accepts (halts in accepting state)
For w ∉ L: M rejects OR loops forever
```

### Recursive Languages (Decidable)

A language L is **recursive** (decidable) if there exists a TM M such that:
```
L = L(M) AND M halts on every input

For w ∈ L: M accepts
For w ∉ L: M rejects (halts, doesn't loop)
```

### Relationship

```
┌─────────────────────────────────────────────────────────┐
│                 All Languages                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │          Recursively Enumerable (RE)              │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │              Recursive                      │  │  │
│  │  │  ┌───────────────────────────────────────┐  │  │  │
│  │  │  │       Context-Free (CFL)             │  │  │  │
│  │  │  │  ┌─────────────────────────────────┐  │  │  │  │
│  │  │  │  │        Regular                  │  │  │  │  │
│  │  │  │  └─────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│  Languages NOT RE exist! (more on this in next chapter)  │
└─────────────────────────────────────────────────────────┘
```

---

## 6.11 Turing Machine as Computers

### TM as Function Computer

A TM can compute functions f: Σ* → Σ*

```
Input: w on tape
Computation: TM runs
Output: f(w) on tape when TM halts

If TM doesn't halt, f(w) is undefined.
```

### Computable Functions

A function f is **computable** (or Turing-computable) if there exists a TM that computes it.

**Examples:**
- Addition: f(n, m) = n + m ✓
- Multiplication: f(n, m) = n × m ✓
- Primality test: f(n) = 1 if n is prime, 0 otherwise ✓

---

## 6.12 Restricted Turing Machines

### Linear Bounded Automata (LBA)

TM with tape limited to input length:

```
Tape: [▷|input|◁]

Only the cells containing input can be used.
```

**Languages recognized:** Context-Sensitive Languages (CSL)

```
Regular ⊂ CFL ⊂ CSL ⊂ Recursive ⊂ RE
```

### Two-Counter Machine

Two counters that can be incremented, decremented, and tested for zero.

**Equivalent to TM!** (Can simulate tape using two counters)

---

## 6.13 TM Design Techniques

### Technique 1: Storage in State

Use the finite state to remember a fixed amount of information.

```
Instead of one state q, use states like:
q_0, q_1, q_a, q_b (remembering what we've seen)
```

### Technique 2: Multiple Tracks

Divide tape into multiple tracks:

```
One tape cell:
┌─────────┐
│ Track 1 │  ← Original input
├─────────┤
│ Track 2 │  ← Markers
├─────────┤
│ Track 3 │  ← Work area
└─────────┘

Tape alphabet becomes Γ₁ × Γ₂ × Γ₃
```

### Technique 3: Subroutines

Design modular TMs and combine them:

```
TM_main:
  1. Call TM_findEnd
  2. Call TM_copy
  3. Call TM_compare
  4. Accept/Reject
```

### Technique 4: Marking

Use special symbols to mark positions:

```
Original: a a b b
Marked:   X X Y Y

Use different alphabet for marked/unmarked
```

---

## 6.14 Practice Problems

### Problem 1: Design TMs

1. Accept {aⁿbⁿcⁿ | n ≥ 1}
2. Accept {ww | w ∈ {a,b}*}
3. Compute f(n) = 2n (unary representation)

### Problem 2: Trace Execution

Trace the TM for 0ⁿ1ⁿ on input "000111"

### Problem 3: Analysis

For each language, state if it's Regular, CFL, Recursive, or RE:
1. {aⁿbⁿ}
2. {aⁿbⁿcⁿ}
3. {w | w is a valid C program}
4. {w | w is a C program that halts}

### Solutions

**Problem 3:**
1. {aⁿbⁿ} - CFL (can be recognized by PDA)
2. {aⁿbⁿcⁿ} - Recursive (not CFL, but decidable by TM)
3. Valid C programs - Recursive (compiler can check syntax)
4. C programs that halt - NOT EVEN RE (undecidable, by reduction to Halting Problem)

---

## 📌 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  1. TM = Finite Control + Infinite Tape + Read/Write Head      │
│  2. TM can: read, write, move left/right, halt or loop         │
│  3. All TM variants are equivalent (multi-tape, NTM, etc.)     │
│  4. Church-Turing Thesis: TM = Algorithm                        │
│  5. Universal TM can simulate any other TM                      │
│  6. Recursive = TM always halts (decidable)                     │
│  7. RE = TM may loop on non-members                             │
│  8. Some languages are not even RE (limits of computation)      │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Decidability and Undecidability](TOA-07-Decidability.md)*
