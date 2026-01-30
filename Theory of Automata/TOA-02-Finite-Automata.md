# Chapter 2: Finite Automata

## 2.1 Introduction

**Finite Automata** (FA) are the simplest computational models that recognize patterns. They have a finite number of states and no additional memory beyond the current state.

### Real-World Applications
- Lexical analysis in compilers
- Text pattern matching (grep, search)
- Network protocol verification
- Vending machines
- Digital circuit design
- Video game AI (state machines)

---

## 2.2 Deterministic Finite Automata (DFA)

### Formal Definition

A **DFA** is a 5-tuple M = (Q, Σ, δ, q₀, F) where:

| Component | Meaning |
|-----------|---------|
| Q | Finite set of states |
| Σ | Input alphabet (finite set) |
| δ | Transition function: Q × Σ → Q |
| q₀ | Start state (q₀ ∈ Q) |
| F | Set of accepting/final states (F ⊆ Q) |

### Key Properties of DFA

1. **Deterministic**: Exactly ONE transition for each (state, symbol) pair
2. **No ε-transitions**: Every transition consumes exactly one input symbol
3. **Complete**: Must have a transition for every symbol from every state

### Example: DFA for strings ending with "01"

```
Language: L = {w ∈ {0,1}* | w ends with 01}

DFA:
         0           1           0
    →(q₀) ───→ (q₁) ───→ ((q₂))
       ↑   ↖         ↗
       │    \_______/
       │        0
       └────────────────────────┘
                 1

State Diagram:
         ┌───┐                     ┌───┐
         │ 0 │                     │ 1 │
         ↓   │                     ↓   │
    → ○─────────→ ○ ─────────→ ◎ ←────┤
      q₀    0     q₁     1      q₂    │
      ↑                         │     │
      └─────────── 0 ───────────┘     │
      └─────────── 1 ─────────────────┘

Formal Definition:
Q = {q₀, q₁, q₂}
Σ = {0, 1}
q₀ = q₀ (start state)
F = {q₂}

Transition Function δ:
┌─────┬─────┬─────┐
│  δ  │  0  │  1  │
├─────┼─────┼─────┤
│ q₀  │ q₁  │ q₀  │
│ q₁  │ q₁  │ q₂  │
│ q₂  │ q₁  │ q₀  │
└─────┴─────┴─────┘
```

### Processing a String

To determine if string w is accepted:
1. Start at q₀
2. Read symbols one by one
3. Follow transitions based on current state and input
4. After reading all input, check if current state ∈ F

**Example:** Process "1001" on above DFA
```
Input: 1 0 0 1
State: q₀ → q₀ → q₁ → q₁ → q₂
              ↑    ↑    ↑    ↑
              1    0    0    1

Final state q₂ ∈ F, so "1001" is ACCEPTED
```

**Example:** Process "100"
```
Input: 1 0 0
State: q₀ → q₀ → q₁ → q₁
              ↑    ↑    ↑
              1    0    0

Final state q₁ ∉ F, so "100" is REJECTED
```

### Extended Transition Function δ*

The **extended transition function** δ*: Q × Σ* → Q processes strings instead of single symbols.

```
Definition:
δ*(q, ε) = q
δ*(q, wa) = δ(δ*(q, w), a)   for w ∈ Σ*, a ∈ Σ

Example:
δ*(q₀, "01") = δ(δ*(q₀, "0"), 1)
              = δ(δ(δ*(q₀, ε), 0), 1)
              = δ(δ(q₀, 0), 1)
              = δ(q₁, 1)
              = q₂
```

### Language of a DFA

The language accepted by DFA M is:
```
L(M) = {w ∈ Σ* | δ*(q₀, w) ∈ F}
```

---

## 2.3 Designing DFAs

### Strategy for DFA Design

1. **Identify the pattern** to recognize
2. **Determine what to remember** (states represent "memory")
3. **Draw the state diagram** 
4. **Verify with examples**

### Common DFA Patterns

#### Pattern 1: Strings containing a substring

**DFA for strings containing "aba"**
```
States represent: How much of "aba" we've matched

     ┌─b─┐  ┌─a,b─┐
     ↓   │  ↓    │
→ ○ ──a──→ ○ ──b──→ ○ ──a──→ ◎
   q₀      q₁       q₂       q₃
   ↑       │
   └──b────┘

q₀: Haven't started matching
q₁: Matched "a"
q₂: Matched "ab"
q₃: Matched "aba" (accepting)
```

#### Pattern 2: Strings of specific length modulo n

**DFA for strings with length divisible by 3**
```
States = {q₀, q₁, q₂}
q_i = "length so far ≡ i (mod 3)"

     ┌──a,b──┐
     ↓       │
→ ◎ ──a,b──→ ○ ──a,b──→ ○
   q₀        q₁         q₂
   ↑                    │
   └─────── a,b ────────┘

Accept when length mod 3 = 0
```

#### Pattern 3: Counting occurrences modulo n

**DFA for strings with even number of 1s**
```
     ┌──0──┐      ┌──0──┐
     ↓    │      ↓    │
→ ◎ ─────────────→ ○ ─────┐
   q₀     1        q₁     │
   ↑                      │
   └──────── 1 ───────────┘

q₀: Seen even number of 1s (including 0)
q₁: Seen odd number of 1s
```

#### Pattern 4: Last n characters

**DFA for strings where last 2 chars are different**
```
Remember last 2 characters:
States: Start, Saw0, Saw1, Saw00, Saw01, Saw10, Saw11

Accepting: Saw01, Saw10
```

### Product Construction

To build DFA for L₁ ∩ L₂ (intersection):

```
Given: DFA₁ = (Q₁, Σ, δ₁, q₁, F₁)
       DFA₂ = (Q₂, Σ, δ₂, q₂, F₂)

Product DFA:
Q = Q₁ × Q₂
δ((p,q), a) = (δ₁(p,a), δ₂(q,a))
Start = (q₁, q₂)
F = F₁ × F₂ (for intersection)
F = (F₁ × Q₂) ∪ (Q₁ × F₂) (for union)
```

---

## 2.4 Non-deterministic Finite Automata (NFA)

### What is Non-determinism?

In an NFA:
- Multiple transitions possible for same (state, symbol)
- Some transitions may be missing (implicit rejection)
- Machine "guesses" the correct path

### Formal Definition

An **NFA** is a 5-tuple M = (Q, Σ, δ, q₀, F) where:

| Component | Meaning |
|-----------|---------|
| Q | Finite set of states |
| Σ | Input alphabet |
| δ | Transition function: Q × Σ → **P(Q)** |
| q₀ | Start state |
| F | Set of accepting states |

**Key difference:** δ returns a SET of states (including possibly empty set)

### NFA Example: Strings ending with "01"

```
NFA (simpler than DFA!):

→ ○ ─────0────→ ○ ─────1────→ ◎
   q₀            q₁            q₂
   │
   └──0,1──┐
           │
           ↓
          (loop back to q₀)

More precisely:
         ┌──0,1──┐
         ↓       │
    → ○ ─────────┴────0────→ ○ ─────1────→ ◎
       q₀                      q₁            q₂

Transition Table:
┌─────┬─────────┬─────────┐
│  δ  │    0    │    1    │
├─────┼─────────┼─────────┤
│ q₀  │ {q₀,q₁} │  {q₀}   │
│ q₁  │   ∅     │  {q₂}   │
│ q₂  │   ∅     │   ∅     │
└─────┴─────────┴─────────┘
```

### Processing in NFA

The NFA accepts if **any** path leads to accepting state.

Think of it as:
- Exploring all possible paths simultaneously
- Accept if at least one path ends in F

**Example:** Process "101" on above NFA
```
Step 0: Start in {q₀}
Step 1: Read '1': from q₀ on 1 → {q₀}
Step 2: Read '0': from q₀ on 0 → {q₀, q₁}
Step 3: Read '1': from q₀ on 1 → {q₀}
                  from q₁ on 1 → {q₂}
        Combined: {q₀, q₂}

Final states include q₂ ∈ F, so ACCEPTED

Visualization:
              ┌── q₀ ── q₀ ── q₀ ── q₀
              │    
Start ── q₀ ──┤
              │
              └── q₀ ── q₀ ── q₁ ── q₂ ✓
                        ↑
                       (guess to start matching)
```

### Extended Transition Function for NFA

```
δ*(q, ε) = {q}
δ*(q, wa) = ⋃ δ(p, a)   for all p ∈ δ*(q, w)

For a set of states:
δ*(S, w) = ⋃ δ*(q, w)   for all q ∈ S
```

---

## 2.5 NFA with ε-transitions (ε-NFA)

### What are ε-transitions?

**ε-transitions** allow state changes WITHOUT consuming any input symbol.

### Formal Definition

An **ε-NFA** has transition function:
```
δ: Q × (Σ ∪ {ε}) → P(Q)
```

### Example: ε-NFA

```
Language: (a|b)*abb

→ ○ ──ε──→ ○ ──a,b──→ ○ ──a──→ ○ ──b──→ ○ ──b──→ ◎
   q₀       q₁         q₁        q₂       q₃       q₄
            ↑           │
            └───────────┘

Simplified view with ε:
    → ○ ──ε──→ ○ ════╗
       q₀      q₁    ║ a,b (self-loop)
                     ↓
               ○ ──a──→ ○ ──b──→ ○ ──b──→ ◎
               q₁       q₂       q₃       q₄
```

### ε-closure

The **ε-closure** of a state q is the set of all states reachable from q using only ε-transitions.

```
ε-closure(q) = {q} ∪ {p | p reachable from q via ε-transitions}

Example:
    → ○ ──ε──→ ○ ──ε──→ ○
       q₀       q₁       q₂

ε-closure(q₀) = {q₀, q₁, q₂}
ε-closure(q₁) = {q₁, q₂}
ε-closure(q₂) = {q₂}
```

**For a set of states:**
```
ε-closure(S) = ⋃ ε-closure(q)  for all q ∈ S
```

### Extended Transition for ε-NFA

```
δ*(q, ε) = ε-closure(q)
δ*(q, wa) = ε-closure( ⋃ δ(p, a) )  for all p ∈ δ*(q, w)
```

---

## 2.6 Equivalence of DFA, NFA, and ε-NFA

### Theorem: All Three Models are Equivalent

```
DFA ≡ NFA ≡ ε-NFA

They recognize exactly the same class of languages:
REGULAR LANGUAGES

    ┌─────────────────────────────────┐
    │         Regular Languages        │
    │                                  │
    │   DFA ←──→ NFA ←──→ ε-NFA       │
    │                                  │
    └─────────────────────────────────┘
```

### Conversion: NFA → DFA (Subset Construction)

**Algorithm:**
1. Each DFA state = set of NFA states
2. Start state = {q₀}
3. Transitions: δ_DFA(S, a) = ⋃ δ_NFA(q, a) for q ∈ S
4. Accepting states = sets containing any NFA accepting state

**Example:** Convert NFA for "ends with 01"

```
NFA:
δ(q₀, 0) = {q₀, q₁}
δ(q₀, 1) = {q₀}
δ(q₁, 1) = {q₂}

Subset Construction:
Start: {q₀}

From {q₀}:
  on 0: δ(q₀, 0) = {q₀, q₁}
  on 1: δ(q₀, 1) = {q₀}

From {q₀, q₁}:
  on 0: δ(q₀, 0) ∪ δ(q₁, 0) = {q₀, q₁} ∪ ∅ = {q₀, q₁}
  on 1: δ(q₀, 1) ∪ δ(q₁, 1) = {q₀} ∪ {q₂} = {q₀, q₂}

From {q₀, q₂}:
  on 0: δ(q₀, 0) ∪ δ(q₂, 0) = {q₀, q₁} ∪ ∅ = {q₀, q₁}
  on 1: δ(q₀, 1) ∪ δ(q₂, 1) = {q₀} ∪ ∅ = {q₀}

Resulting DFA:
┌───────────┬───────────┬───────────┐
│    State  │     0     │     1     │
├───────────┼───────────┼───────────┤
│  → {q₀}   │ {q₀,q₁}   │   {q₀}    │
│  {q₀,q₁}  │ {q₀,q₁}   │ {q₀,q₂}*  │
│ *{q₀,q₂}  │ {q₀,q₁}   │   {q₀}    │
└───────────┴───────────┴───────────┘
* = accepting (contains q₂)
```

### Conversion: ε-NFA → NFA

**Algorithm:**
1. Compute ε-closure for each state
2. New transitions: δ'(q, a) = ε-closure(δ(ε-closure(q), a))
3. New accepting states: q ∈ F' if ε-closure(q) ∩ F ≠ ∅

### Conversion: DFA → NFA

Trivial: Every DFA is already an NFA where each transition set has exactly one state.

---

## 2.7 DFA Minimization

### Why Minimize?

- Reduce memory usage
- Faster processing
- Canonical form for comparison

### Equivalent States

Two states p and q are **equivalent** if:
```
For all strings w: δ*(p, w) ∈ F ⟺ δ*(q, w) ∈ F
```

### Minimization Algorithm (Table Filling)

**Step 1:** Create table for all pairs of states
**Step 2:** Mark pairs where one is accepting, one is not
**Step 3:** Iteratively mark pairs (p, q) if δ(p, a) and δ(q, a) lead to already-marked pair
**Step 4:** Merge unmarked (equivalent) pairs

**Example:**

```
DFA with states: q₀, q₁, q₂, q₃, q₄
F = {q₂, q₄}

Initial marking (accepting vs non-accepting):
     q₀   q₁   q₂   q₃   q₄
q₁   -
q₂   X    X
q₃   -    -    X
q₄   X    X    -    X

After propagation:
     q₀   q₁   q₂   q₃   q₄
q₁   X
q₂   X    X
q₃   X    X    X
q₄   X    X    -    X

Result: q₂ ≡ q₄ (can be merged)
```

### Myhill-Nerode Theorem

A language L is regular if and only if it has a **finite number of equivalence classes** under the relation:
```
x ≡_L y ⟺ (∀z: xz ∈ L ⟺ yz ∈ L)
```

The minimum DFA has exactly as many states as there are equivalence classes.

---

## 2.8 Dead States and Trap States

### Dead State (Trap State)

A **dead state** is a non-accepting state from which no accepting state is reachable.

```
→ ○ ──a──→ ○ ──a──→ ◎
   q₀       q₁       q₂
   │        │
   b        b
   ↓        ↓
   ○ ←─a,b──┘
   qd (dead state)
   ↺ a,b

All transitions from qd lead back to qd
qd is never accepting
```

**Note:** Dead states can be omitted in informal diagrams (implicit rejection).

---

## 2.9 Comparison: DFA vs NFA

| Feature | DFA | NFA |
|---------|-----|-----|
| Transitions | Exactly one per symbol | Zero, one, or more |
| ε-transitions | Not allowed | Allowed (in ε-NFA) |
| Determinism | Yes | No |
| States needed | May need up to 2ⁿ | Can be fewer |
| Processing | Single path | Multiple paths |
| Implementation | Easier | Harder |
| Design | Often harder | Often easier |
| Execution speed | Faster (single path) | Slower (track all paths) |

---

## 2.10 Finite Automata with Output

### Moore Machine

Output depends on **current state only**.

```
M = (Q, Σ, Δ, δ, λ, q₀)

Δ = Output alphabet
λ: Q → Δ (output function)

Output is associated with states, not transitions.
```

### Mealy Machine

Output depends on **current state AND input symbol**.

```
M = (Q, Σ, Δ, δ, λ, q₀)

Δ = Output alphabet
λ: Q × Σ → Δ (output function)

Output is associated with transitions.
```

### Moore vs Mealy

```
Moore Machine:               Mealy Machine:

   ○/0 ──a──→ ○/1             ○ ───a/1──→ ○
   q₀         q₁              q₀          q₁

Output in states             Output on transitions
One more state typically     More compact
Output after reading         Output while reading
```

### Equivalence

Every Moore machine can be converted to equivalent Mealy machine and vice versa.

---

## 2.11 Two-Way Finite Automata

Standard FA read input left-to-right. **Two-Way Finite Automata** can move the read head left or right.

```
Transition: δ(q, a) = (q', D)
where D ∈ {L, R} (Left or Right)

Despite extra power, 2DFA = DFA in terms of languages accepted!
```

---

## 2.12 Practice Problems

### Problem 1: Design DFAs

Design DFAs for:
1. Strings over {a, b} containing exactly two a's
2. Strings over {0, 1} where every 0 is followed by at least one 1
3. Binary numbers divisible by 3

### Problem 2: NFA to DFA

Convert to DFA:
```
NFA:
δ(q₀, a) = {q₀, q₁}
δ(q₀, b) = {q₁}
δ(q₁, a) = {q₀}
δ(q₁, b) = {q₀, q₁}
Start: q₀, Accept: {q₁}
```

### Problem 3: Minimization

Minimize the following DFA:
```
States: A, B, C, D, E
Start: A, Accept: {C, E}
δ(A,0)=B, δ(A,1)=C
δ(B,0)=D, δ(B,1)=E
δ(C,0)=D, δ(C,1)=E
δ(D,0)=B, δ(D,1)=C
δ(E,0)=B, δ(E,1)=C
```

### Solutions

**Problem 1.1: Exactly two a's**
```
States track count: 0, 1, 2, 3+ (dead)
→ ○ ──a──→ ○ ──a──→ ◎ ──a──→ ○
   q₀       q₁       q₂       q₃ (dead)
   ↺b       ↺b       ↺b       ↺a,b
```

**Problem 2 Solution:**
```
Subset Construction:
{q₀}: on a→{q₀,q₁}, on b→{q₁}
{q₁}: on a→{q₀}, on b→{q₀,q₁}
{q₀,q₁}: on a→{q₀,q₁}, on b→{q₀,q₁}

DFA States: {q₀}, {q₁}*, {q₀,q₁}*
```

---

## 📌 Key Takeaways

```
┌──────────────────────────────────────────────────────────────┐
│  1. DFA: Exactly one transition per (state, symbol)          │
│  2. NFA: Multiple transitions allowed (power set)            │
│  3. DFA ≡ NFA ≡ ε-NFA (all recognize Regular Languages)      │
│  4. NFA → DFA: Subset construction (may exponentially grow)  │
│  5. DFA minimization: Merge equivalent states                │
│  6. States represent "what needs to be remembered"           │
│  7. L(M) = {w | δ*(q₀, w) ∈ F}                               │
└──────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Regular Expressions and Languages](TOA-03-Regular-Expressions.md)*
