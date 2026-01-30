# Chapter 5: Pushdown Automata

## 5.1 Introduction

**Pushdown Automata (PDA)** are finite automata enhanced with a **stack** - an unlimited LIFO (Last-In-First-Out) memory structure. This extra memory allows PDAs to recognize context-free languages.

### The Power of a Stack

```
Regular Languages    Context-Free Languages
      ↓                      ↓
Finite Automata         Pushdown Automata
(finite memory)         (stack memory)

FA cannot count:  aⁿbⁿ requires remembering n
PDA can count:    Push a's onto stack, pop for each b
```

---

## 5.2 Intuitive Understanding

### How PDA Works

```
┌─────────────────────────────────────┐
│            Input Tape               │
│   ┌───┬───┬───┬───┬───┬───┬───┐    │
│   │ a │ a │ b │ b │ c │ c │   │    │
│   └───┴───┴───┴───┴───┴───┴───┘    │
│           ↑                         │
│      Read Head                      │
└─────────────────────────────────────┘
                │
                ↓
        ┌──────────────┐
        │  Finite      │
        │  Control     │
        │   (q₀)       │
        └──────────────┘
                │
                ↓
┌─────────────────────────────────────┐
│             Stack                   │
│   ┌───┐                             │
│   │ B │ ← Top                       │
│   ├───┤                             │
│   │ A │                             │
│   ├───┤                             │
│   │ Z₀│ ← Bottom (stack marker)     │
│   └───┘                             │
└─────────────────────────────────────┘
```

### Stack Operations

| Operation | Notation | Action |
|-----------|----------|--------|
| Push | A/AB | Pop A, push AB (B then A, so A on top) |
| Pop | A/ε | Remove top A |
| No change | A/A | Pop A, push A back |
| Replace | A/B | Pop A, push B |

---

## 5.3 Formal Definition

### PDA Definition

A **PDA** is a 7-tuple P = (Q, Σ, Γ, δ, q₀, Z₀, F) where:

| Component | Meaning |
|-----------|---------|
| Q | Finite set of states |
| Σ | Input alphabet |
| Γ | Stack alphabet |
| δ | Transition function: Q × (Σ ∪ {ε}) × Γ → P(Q × Γ*) |
| q₀ | Start state (q₀ ∈ Q) |
| Z₀ | Initial stack symbol (Z₀ ∈ Γ) |
| F | Set of accepting states (F ⊆ Q) |

### Transition Function

δ(q, a, X) = {(p₁, γ₁), (p₂, γ₂), ...}

**Meaning:** In state q, reading input a, with X on top of stack:
- Move to state pᵢ
- Replace X with γᵢ (push γᵢ after popping X)

**Special case:** δ(q, ε, X) - no input consumed (ε-transition)

### Transition Notation

```
δ(q, a, X) = {(p, YZ)}

Diagram notation:
    a, X/YZ
q ──────────→ p

Meaning:
- In state q
- Read input 'a' (or ε for no input)
- Pop X from stack
- Push YZ (Z first, then Y, so Y is on top)
- Go to state p
```

---

## 5.4 Instantaneous Descriptions (IDs)

### Configuration of PDA

An **Instantaneous Description (ID)** captures the complete configuration:

```
(q, w, γ)

q = current state
w = remaining input
γ = stack contents (leftmost = top)
```

### Move Relation (⊢)

```
(q, aw, Xβ) ⊢ (p, w, αβ)

if δ(q, a, X) contains (p, α)

Multi-step: ⊢* (zero or more moves)
```

### Example: Processing "aabb"

```
PDA for {aⁿbⁿ | n ≥ 1}:
δ(q₀, a, Z₀) = {(q₀, AZ₀)}
δ(q₀, a, A) = {(q₀, AA)}
δ(q₀, b, A) = {(q₁, ε)}
δ(q₁, b, A) = {(q₁, ε)}
δ(q₁, ε, Z₀) = {(q₂, ε)}
F = {q₂}

Trace:
(q₀, aabb, Z₀) ⊢ (q₀, abb, AZ₀)     [push A]
               ⊢ (q₀, bb, AAZ₀)      [push A]
               ⊢ (q₁, b, AZ₀)        [pop A for b]
               ⊢ (q₁, ε, Z₀)         [pop A for b]
               ⊢ (q₂, ε, ε)          [accept]
```

---

## 5.5 Acceptance Modes

### Two Types of Acceptance

**1. Acceptance by Final State**
```
L(P) = {w | (q₀, w, Z₀) ⊢* (q, ε, γ) for some q ∈ F, any γ}
```
Accept if: Input exhausted AND in accepting state

**2. Acceptance by Empty Stack**
```
N(P) = {w | (q₀, w, Z₀) ⊢* (q, ε, ε) for any q}
```
Accept if: Input exhausted AND stack empty

### Equivalence

Both acceptance modes are equivalent in power:

```
┌─────────────────────────────────────────────┐
│  For any PDA P₁ accepting by final state,   │
│  ∃ PDA P₂ accepting by empty stack:         │
│        L(P₁) = N(P₂)                        │
│                                             │
│  And vice versa.                            │
└─────────────────────────────────────────────┘
```

### Converting Final State → Empty Stack

```
Idea: After reaching accepting state, empty the stack

New PDA:
1. Add new start state q'₀ that pushes special marker X₀
2. Add new state q_f 
3. From each accepting state, add ε-transition to q_f
4. From q_f, add ε-transitions to pop everything
```

### Converting Empty Stack → Final State

```
Idea: When stack becomes empty, go to accepting state

New PDA:
1. Add new start state that pushes marker
2. Add new accepting state q_f
3. When original PDA empties stack (marker visible), go to q_f
```

---

## 5.6 PDA Examples

### Example 1: L = {aⁿbⁿ | n ≥ 0}

```
States: q₀ (start), q₁ (reading b's), q₂ (accept)
Stack alphabet: {A, Z₀}
Accept by final state

Transitions:
δ(q₀, ε, Z₀) = {(q₂, Z₀)}        [accept ε]
δ(q₀, a, Z₀) = {(q₀, AZ₀)}       [first a]
δ(q₀, a, A) = {(q₀, AA)}         [more a's]
δ(q₀, b, A) = {(q₁, ε)}          [switch to b's]
δ(q₁, b, A) = {(q₁, ε)}          [more b's]
δ(q₁, ε, Z₀) = {(q₂, Z₀)}        [accept]

Diagram:
        ε,Z₀/Z₀
    ┌─────────────→ (q₂) [accept]
    │               ↑
    │   a,Z₀/AZ₀    │ ε,Z₀/Z₀
    │   a,A/AA      │
    ↓  ┌───────┐    │
→ (q₀)─┘       │────┤ b,A/ε
         b,A/ε │    │
               ↓    │
             (q₁)───┘
              ↺
            b,A/ε
```

### Example 2: L = {ww^R | w ∈ {a,b}*} (Even-length palindromes)

```
Strategy: 
- Push first half onto stack
- Pop and match second half
- Non-deterministic guess for middle

Transitions:
δ(q₀, a, Z₀) = {(q₀, aZ₀)}
δ(q₀, b, Z₀) = {(q₀, bZ₀)}
δ(q₀, a, a) = {(q₀, aa), (q₁, ε)}    [push or start matching]
δ(q₀, a, b) = {(q₀, ab)}
δ(q₀, b, a) = {(q₀, ba)}
δ(q₀, b, b) = {(q₀, bb), (q₁, ε)}    [push or start matching]
δ(q₀, ε, Z₀) = {(q₁, Z₀)}            [ε case]
δ(q₁, a, a) = {(q₁, ε)}
δ(q₁, b, b) = {(q₁, ε)}
δ(q₁, ε, Z₀) = {(q₂, Z₀)}
F = {q₂}
```

### Example 3: L = {aⁿbᵐcⁿ⁺ᵐ | n,m ≥ 0}

```
Strategy:
- Push A for each a
- Push B for each b  
- Pop one symbol for each c

Transitions:
δ(q₀, a, Z₀) = {(q₀, AZ₀)}
δ(q₀, a, A) = {(q₀, AA)}
δ(q₀, b, Z₀) = {(q₁, BZ₀)}
δ(q₀, b, A) = {(q₁, BA)}
δ(q₁, b, B) = {(q₁, BB)}
δ(q₀, c, A) = {(q₂, ε)}
δ(q₁, c, B) = {(q₂, ε)}
δ(q₂, c, A) = {(q₂, ε)}
δ(q₂, c, B) = {(q₂, ε)}
δ(q₂, ε, Z₀) = {(q₃, Z₀)}
δ(q₀, ε, Z₀) = {(q₃, Z₀)}    [empty string]
F = {q₃}
```

---

## 5.7 Deterministic PDA (DPDA)

### Definition

A PDA is **deterministic** if:

1. δ(q, a, X) has at most one element for all q, a, X
2. If δ(q, ε, X) ≠ ∅, then δ(q, a, X) = ∅ for all a ∈ Σ
   (No choice between ε-move and input move)

### DPDA vs NPDA

```
DPDA ⊂ NPDA

Languages recognized by DPDA: DCFL (Deterministic CFL)
Languages recognized by NPDA: CFL

DCFL ⊂ CFL

Example:
{ww^R} requires NPDA (guess the middle)
{aⁿbⁿ} can be done with DPDA
```

### Properties of DCFLs

| Property | DCFL | CFL |
|----------|------|-----|
| Closed under complement | Yes | No |
| Closed under intersection | No | No |
| Closed under union | No | No |
| Parsing complexity | O(n) | O(n³) |
| Unambiguous grammar | Always | Not always |

### Importance of DPDA

- Most programming languages are DCFLs
- Efficient parsers (LL, LR parsers)
- Predictable behavior (no backtracking needed)

---

## 5.8 Equivalence of PDA and CFG

### Fundamental Theorem

```
┌─────────────────────────────────────────────────────────┐
│    Language is CFL  ⟺  Language is accepted by PDA      │
│                                                          │
│    CFG ──→ PDA (can construct)                          │
│    PDA ──→ CFG (can construct)                          │
└─────────────────────────────────────────────────────────┘
```

### CFG → PDA Construction

**Method:** Build PDA that simulates leftmost derivation

**Algorithm:**
1. One state q (plus start/accept states)
2. Start by pushing S (start symbol)
3. For variable A on top: guess and push right side of production
4. For terminal a on top: match with input

```
Given G = (V, Σ, P, S):
Build PDA P = ({q₀, q, qf}, Σ, V ∪ Σ, δ, q₀, Z₀, {qf})

Transitions:
δ(q₀, ε, Z₀) = {(q, SZ₀)}              [push start symbol]
δ(q, ε, A) = {(q, α) | A → α ∈ P}      [expand variable]
δ(q, a, a) = {(q, ε)}                   [match terminal]
δ(q, ε, Z₀) = {(qf, Z₀)}               [accept when done]
```

**Example:** G: S → aSb | ε
```
Transitions:
δ(q₀, ε, Z₀) = {(q, SZ₀)}
δ(q, ε, S) = {(q, aSb), (q, ε)}
δ(q, a, a) = {(q, ε)}
δ(q, b, b) = {(q, ε)}
δ(q, ε, Z₀) = {(qf, Z₀)}
```

### PDA → CFG Construction

**Method:** Variables represent "what happens between pushing and popping"

**Algorithm:**
1. Variables: [q, A, p] means "starting in state q with A on stack, end in state p with A popped"
2. Start symbol: [q₀, Z₀, qf] for some accepting qf

```
For each transition δ(q, a, X) containing (r, Y₁Y₂...Yₖ):
Add productions:
[q, X, s] → a[r, Y₁, r₁][r₁, Y₂, r₂]...[rₖ₋₁, Yₖ, s]
for all possible states r₁, r₂, ..., s

If k = 0 (stack symbol popped):
[q, X, r] → a
```

---

## 5.9 PDA Design Strategies

### Strategy 1: Matching Pairs

For languages like {aⁿbⁿ}:
```
1. Push when seeing opening symbol
2. Pop when seeing closing symbol
3. Accept when stack is in expected state
```

### Strategy 2: Guess and Verify

For languages requiring non-determinism:
```
1. Non-deterministically guess key point (e.g., middle)
2. Change behavior at guessed point
3. Some branch will succeed if string is in language
```

### Strategy 3: Multiple Counters

For tracking multiple quantities:
```
1. Use different stack symbols for different counts
2. Push/pop as needed
3. Check relationships at end
```

### Strategy 4: Delayed Matching

```
1. Store first part on stack
2. Process without stack changes
3. Match with stored values
```

---

## 5.10 Comparison: FA vs PDA

| Aspect | FA | PDA |
|--------|-----|-----|
| Memory | Current state only | State + Stack |
| Languages | Regular | Context-Free |
| Example | a*b* | aⁿbⁿ |
| Determinism | DFA = NFA | DPDA ⊂ NPDA |
| Closure properties | More | Fewer |
| Parsing complexity | O(n) | O(n) DPDA, O(n³) general |

---

## 5.11 Two-Stack PDA

### What if we had two stacks?

A PDA with TWO stacks is equivalent to a **Turing Machine**!

```
One Stack:     Context-Free Languages
Two Stacks:    Recursively Enumerable Languages (all TM-recognizable)

The second stack provides enough power to simulate 
arbitrary Turing Machine computation.
```

---

## 5.12 Practice Problems

### Problem 1: Design PDAs

1. L = {aⁿb²ⁿ | n ≥ 0}
2. L = {w ∈ {a,b}* | #a(w) = #b(w)}
3. L = {aⁱbʲcᵏ | i = j or j = k}

### Problem 2: Trace Execution

Trace the PDA for {aⁿbⁿ} on input "aaabbb"

### Problem 3: Identify Acceptance

For this PDA, which strings are accepted?
```
δ(q₀, a, Z₀) = {(q₀, AZ₀)}
δ(q₀, a, A) = {(q₀, AA)}
δ(q₀, b, A) = {(q₁, ε)}
δ(q₁, b, A) = {(q₁, ε)}
Accept by empty stack
```

### Solutions

**Problem 1.1:** L = {aⁿb²ⁿ}
```
Push two B's for each a, pop one B for each b

δ(q₀, a, Z₀) = {(q₀, BBZ₀)}
δ(q₀, a, B) = {(q₀, BBB)}
δ(q₀, b, B) = {(q₁, ε)}
δ(q₁, b, B) = {(q₁, ε)}
δ(q₁, ε, Z₀) = {(q₂, ε)}
Accept by final state {q₂} or empty stack
```

**Problem 1.2:** Equal a's and b's
```
Use stack to track imbalance:
- When more a's: stack has A's
- When more b's: stack has B's

δ(q₀, a, Z₀) = {(q₀, AZ₀)}
δ(q₀, b, Z₀) = {(q₀, BZ₀)}
δ(q₀, a, A) = {(q₀, AA)}
δ(q₀, b, B) = {(q₀, BB)}
δ(q₀, a, B) = {(q₀, ε)}     [a cancels b]
δ(q₀, b, A) = {(q₀, ε)}     [b cancels a]
δ(q₀, ε, Z₀) = {(qf, Z₀)}
F = {qf}
```

**Problem 2:** Trace for "aaabbb"
```
(q₀, aaabbb, Z₀)
⊢ (q₀, aabbb, AZ₀)
⊢ (q₀, abbb, AAZ₀)
⊢ (q₀, bbb, AAAZ₀)
⊢ (q₁, bb, AAZ₀)
⊢ (q₁, b, AZ₀)
⊢ (q₁, ε, Z₀)
⊢ (q₂, ε, Z₀) [accept]
```

---

## 📌 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  1. PDA = FA + Stack (LIFO memory)                              │
│  2. PDA recognizes Context-Free Languages                       │
│  3. Two acceptance modes: Final state OR Empty stack            │
│  4. CFG ↔ PDA (equivalent in power)                             │
│  5. DPDA < NPDA (deterministic less powerful)                   │
│  6. Transition: δ(state, input, stack_top) → (new_state, push)  │
│  7. ID: (q, remaining_input, stack_contents)                    │
│  8. DPDA used for practical parsing (LL, LR parsers)            │
│  9. Two-stack PDA = Turing Machine power                        │
└─────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Turing Machines](TOA-06-Turing-Machines.md)*
