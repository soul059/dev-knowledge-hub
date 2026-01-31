# Chapter 5: Bottom-Up Parsing

## 5.1 Introduction to Bottom-Up Parsing

**Bottom-Up Parsing** constructs the parse tree from the **leaves** (terminals) to the **root** (start symbol). It attempts to find a **rightmost derivation in reverse**.

### The Intuition

Think of it as:
1. Start with the input tokens (leaves of parse tree)
2. Find patterns that match right-hand sides of productions
3. **Reduce** them to left-hand sides (non-terminals)
4. Continue until you reach the start symbol

### Key Insight: Rightmost Derivation in Reverse

If the rightmost derivation is:
```
S ⟹ᵣₘ γ₁ ⟹ᵣₘ γ₂ ⟹ᵣₘ ... ⟹ᵣₘ γₙ = w
```

Then bottom-up parsing performs:
```
w = γₙ ⟸ γₙ₋₁ ⟸ ... ⟸ γ₁ ⟸ S
```

### Comparison with Top-Down

```
Top-Down:                      Bottom-Up:
    S                              S
   ╱│╲                            ╱│╲
  A B C                          A B C
  │ │ │                          │ │ │
  a b c   ←── direction ───     a b c
  
  Expand from root              Reduce toward root
```

---

## 5.2 Shift-Reduce Parsing

The most common form of bottom-up parsing uses two fundamental operations:

### The Two Operations

| Operation | Description |
|-----------|-------------|
| **Shift** | Push the next input symbol onto the stack |
| **Reduce** | Replace symbols on top of stack with a non-terminal (applying a production in reverse) |

### Additional Operations

| Operation | Description |
|-----------|-------------|
| **Accept** | Parsing is complete successfully |
| **Error** | Syntax error detected |

### Parser Components

```
┌──────────────────────────────────────────┐
│                 PARSER                    │
│  ┌─────────────┐     ┌───────────────┐   │
│  │    Stack    │     │  Input Buffer │   │
│  │             │     │               │   │
│  │ $ ... α     │     │    a ... $    │   │
│  │     ↑       │     │    ↑          │   │
│  │    top      │     │  current      │   │
│  └─────────────┘     └───────────────┘   │
│                                           │
│         Stack + Input = Right SF          │
└──────────────────────────────────────────┘
```

### Handle

A **handle** is a substring that matches the right side of a production AND reducing it is a step in the reverse of a rightmost derivation.

**Formally**: If S ⟹*ᵣₘ αAw ⟹ᵣₘ αβw, then β in position following α is a handle of αβw.

### Example: Shift-Reduce Parsing

**Grammar:**
```
E → E + T | T
T → T * F | F
F → ( E ) | id
```

**Parsing: `id + id * id`**

| Stack | Input | Action |
|-------|-------|--------|
| $ | id + id * id $ | Shift |
| $ id | + id * id $ | Reduce F → id |
| $ F | + id * id $ | Reduce T → F |
| $ T | + id * id $ | Reduce E → T |
| $ E | + id * id $ | Shift |
| $ E + | id * id $ | Shift |
| $ E + id | * id $ | Reduce F → id |
| $ E + F | * id $ | Reduce T → F |
| $ E + T | * id $ | Shift |
| $ E + T * | id $ | Shift |
| $ E + T * id | $ | Reduce F → id |
| $ E + T * F | $ | Reduce T → T * F |
| $ E + T | $ | Reduce E → E + T |
| $ E | $ | **Accept** |

---

## 5.3 Conflicts in Shift-Reduce Parsing

### Shift-Reduce Conflict

The parser cannot decide whether to **shift** or **reduce**.

**Example Grammar:**
```
stmt → if expr then stmt
     | if expr then stmt else stmt
     | other
```

For: `if E then if E then S else S`

When stack has: `if E then if E then S`
And input has: `else S`

**Options:**
- Reduce: `if E then S` → stmt
- Shift: Read `else` to eventually reduce `if E then stmt else stmt`

**Resolution**: Usually resolved by preferring **shift** (match else with nearest if)

### Reduce-Reduce Conflict

The parser cannot decide which production to use for reduction.

**Example Grammar:**
```
stmt → id ( parameter_list )
expr → id ( expr_list )
parameter_list → parameter_list , id | id
expr_list → expr_list , expr | expr
```

For input: `A(i, j)`

After reading: `A ( id`

**Options:**
- Reduce id to parameter_list → id
- Reduce id to expr → id

**Resolution**: Requires looking at context (often grammar rewriting needed)

---

## 5.4 LR Parsing

**LR Parsing** is the most powerful shift-reduce parsing technique for deterministic context-free grammars.

### What LR Means

```
L - Left-to-right scan of input
R - Rightmost derivation (constructed in reverse)
```

### Why LR Parsing?

1. Handles virtually all programming language constructs
2. Most general non-backtracking shift-reduce method
3. Detects errors as soon as possible
4. Class of LR-parsable grammars is a superset of LL grammars

### Types of LR Parsers

```
                  LR Parsers
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    SLR(1)         LALR(1)        CLR(1)
    Simple LR     Look-Ahead LR   Canonical LR
        │             │             │
    Smallest      Practical      Most Powerful
    Tables        (YACC/Bison)   Largest Tables
```

| Parser | Power | Table Size | Usage |
|--------|-------|------------|-------|
| SLR(1) | Least | Small | Academic |
| LALR(1) | Medium | Medium | Practical (YACC) |
| CLR(1) | Most | Large | Rarely used |

---

## 5.5 LR Parser Structure

### Components

```
┌────────────────────────────────────────────────────────────┐
│                        LR PARSER                            │
│                                                             │
│  ┌────────────────────┐          ┌─────────────────────┐   │
│  │       Stack        │          │    Input Buffer     │   │
│  │  s₀ X₁ s₁ X₂ s₂... │          │    a₁ a₂ ... aₙ $   │   │
│  │                    │          │                     │   │
│  │  (state-symbol     │          │    (tokens)         │   │
│  │   pairs)           │          │                     │   │
│  └────────────────────┘          └─────────────────────┘   │
│           │                               │                 │
│           └───────────────┬───────────────┘                 │
│                           ▼                                 │
│                   ┌───────────────┐                         │
│                   │ Parsing Table │                         │
│                   │ ┌─────┬─────┐ │                         │
│                   │ │ACTION│GOTO │ │                         │
│                   │ └─────┴─────┘ │                         │
│                   └───────────────┘                         │
│                           │                                 │
│                           ▼                                 │
│                       Output                                │
└────────────────────────────────────────────────────────────┘
```

### Parsing Table

The table has two parts:

**ACTION[s, a]**: What to do in state s with lookahead a
- **shift s'**: Push a and goto state s'
- **reduce A → β**: Pop |β| symbols, push A, consult GOTO
- **accept**: Parsing complete
- **error**: Syntax error

**GOTO[s, A]**: Next state after reducing to non-terminal A

### LR Parsing Algorithm

```
Initialize: push state 0 onto stack

loop:
    let s = state on top of stack
    let a = current input symbol
    
    case ACTION[s, a] of:
        shift s':
            push a
            push s'
            advance input
            
        reduce A → β:
            pop 2|β| symbols (|β| state-symbol pairs)
            let s' = state now on top of stack
            push A
            push GOTO[s', A]
            output production A → β
            
        accept:
            return SUCCESS
            
        error:
            call error recovery
```

---

## 5.6 LR(0) Items and States

### LR(0) Item

An **LR(0) item** is a production with a dot (•) indicating how much has been seen.

For production A → XYZ, the items are:
```
A → •XYZ    (about to see X)
A → X•YZ    (seen X, about to see Y)
A → XY•Z    (seen XY, about to see Z)
A → XYZ•    (seen everything, ready to reduce)
```

### Intuition

- **Dot position** shows how much of the right side has been recognized
- Items with dot at the **end** → reduction is possible
- Items with dot before a **terminal** → shift is possible

### Augmented Grammar

To build the parser, we augment the grammar with a new start symbol:

```
Original:          Augmented:
S → ...            S' → S
                   S → ...
```

This ensures a unique accepting state.

---

## 5.7 Constructing LR(0) Automaton

### Closure Operation

**CLOSURE(I)** where I is a set of items:

```
CLOSURE(I):
    repeat
        for each item A → α•Bβ in I:
            for each production B → γ:
                add B → •γ to I
    until no new items added
    return I
```

### GOTO Operation

**GOTO(I, X)** where I is a set of items and X is a grammar symbol:

```
GOTO(I, X):
    J = empty set
    for each item A → α•Xβ in I:
        add A → αX•β to J
    return CLOSURE(J)
```

### Building the Canonical Collection

```
items(G'):
    C = { CLOSURE({S' → •S}) }
    repeat
        for each set I in C:
            for each grammar symbol X:
                if GOTO(I, X) is not empty and not in C:
                    add GOTO(I, X) to C
    until no new sets added
    return C
```

### Example: Building LR(0) States

**Grammar:**
```
S' → S
S → CC
C → cC | d
```

**State I₀:** CLOSURE({S' → •S})
```
S' → •S
S  → •CC
C  → •cC
C  → •d
```

**State I₁:** GOTO(I₀, S)
```
S' → S•
```

**State I₂:** GOTO(I₀, C)
```
S → C•C
C → •cC
C → •d
```

**State I₃:** GOTO(I₀, c)
```
C → c•C
C → •cC
C → •d
```

**State I₄:** GOTO(I₀, d)
```
C → d•
```

**State I₅:** GOTO(I₂, C)
```
S → CC•
```

**Continue for GOTO(I₂, c), GOTO(I₂, d), etc.**

### LR(0) Automaton Diagram

```
                      S
            ┌──────────────────┐
            │                  ▼
           I₀ ────────────── I₁ (accept)
            │        
            │ C      
            ▼        
           I₂ ─────C──────▶ I₅
            │                 
         c  │  d              
            ▼                 
           I₃ ◄───c─────┐     
            │           │     
         C  │  d        │     
            ▼           │     
           I₆         I₄ (reduce C→d)
```

---

## 5.8 LR(0) Parsing Table Construction

### Complete LR(0) States for Grammar

**Grammar:**
```
(0) S' → S
(1) S  → CC
(2) C  → cC
(3) C  → d
```

**All States with Items:**

| State | Items |
|-------|-------|
| I₀ | S' → •S, S → •CC, C → •cC, C → •d |
| I₁ | S' → S• |
| I₂ | S → C•C, C → •cC, C → •d |
| I₃ | C → c•C, C → •cC, C → •d |
| I₄ | C → d• |
| I₅ | S → CC• |
| I₆ | C → cC• |

### LR(0) Table Construction Rules

**For each state I:**

1. **Shift**: If A → α•aβ is in I and GOTO(I, a) = Iⱼ (a is terminal):
   - Set ACTION[I, a] = "shift j" (sj)

2. **Reduce**: If A → α• is in I (dot at end, A ≠ S'):
   - Set ACTION[I, a] = "reduce A → α" for **ALL terminals** (including $)

3. **Accept**: If S' → S• is in I:
   - Set ACTION[I, $] = "accept" (acc)

4. **GOTO**: If GOTO(I, A) = Iⱼ for non-terminal A:
   - Set GOTO[I, A] = j

### LR(0) Parsing Table

```
┌───────┬───────────────────────────┬───────────┐
│ State │          ACTION           │   GOTO    │
│       ├─────────┬─────────┬───────┼─────┬─────┤
│       │    c    │    d    │   $   │  S  │  C  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   0   │   s3    │   s4    │       │  1  │  2  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   1   │         │         │  acc  │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   2   │   s3    │   s4    │       │     │  5  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   3   │   s3    │   s4    │       │     │  6  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   4   │   r3    │   r3    │  r3   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   5   │   r1    │   r1    │  r1   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   6   │   r2    │   r2    │  r2   │     │     │
└───────┴─────────┴─────────┴───────┴─────┴─────┘

Legend: s3 = shift to state 3, r2 = reduce by production 2
Productions: (1) S → CC  (2) C → cC  (3) C → d
```

### LR(0) String Parsing Example

**Parse the string: `cdd`**

| Step | Stack | Input | Action | Explanation |
|------|-------|-------|--------|-------------|
| 1 | 0 | cdd$ | s3 | ACTION[0,c]=s3, shift c, goto state 3 |
| 2 | 0 c 3 | dd$ | s4 | ACTION[3,d]=s4, shift d, goto state 4 |
| 3 | 0 c 3 d 4 | d$ | r3 | ACTION[4,d]=r3, reduce C→d |
| 4 | 0 c 3 C 6 | d$ | r2 | Pop d,4; GOTO[3,C]=6, reduce C→cC |
| 5 | 0 C 2 | d$ | s4 | Pop c,3,C,6; GOTO[0,C]=2, shift d |
| 6 | 0 C 2 d 4 | $ | r3 | ACTION[4,$]=r3, reduce C→d |
| 7 | 0 C 2 C 5 | $ | r1 | Pop d,4; GOTO[2,C]=5, reduce S→CC |
| 8 | 0 S 1 | $ | acc | Pop C,2,C,5; GOTO[0,S]=1, ACCEPT! |

### Detailed Step-by-Step Trace

**Step 1-2: Shifting**
```
Stack: 0           Input: cdd$    → Shift c
Stack: 0 c 3       Input: dd$     → Shift d
Stack: 0 c 3 d 4   Input: d$
```

**Step 3-4: First Reduction Sequence**
```
Stack: 0 c 3 d 4   Input: d$
- ACTION[4, d] = r3 (reduce C → d)
- Pop 2 symbols (d, 4), now top = state 3
- GOTO[3, C] = 6
- Push C and 6

Stack: 0 c 3 C 6   Input: d$
- ACTION[6, d] = r2 (reduce C → cC)
- Pop 4 symbols (c, 3, C, 6), now top = state 0
- GOTO[0, C] = 2
- Push C and 2

Stack: 0 C 2       Input: d$
```

**Step 5-6: Second Reduction Sequence**
```
Stack: 0 C 2       Input: d$
- ACTION[2, d] = s4 (shift d)
Stack: 0 C 2 d 4   Input: $

- ACTION[4, $] = r3 (reduce C → d)
- Pop 2 symbols (d, 4), now top = state 2
- GOTO[2, C] = 5
- Push C and 5

Stack: 0 C 2 C 5   Input: $
```

**Step 7-8: Final Reduction and Accept**
```
Stack: 0 C 2 C 5   Input: $
- ACTION[5, $] = r1 (reduce S → CC)
- Pop 4 symbols (C, 2, C, 5), now top = state 0
- GOTO[0, S] = 1
- Push S and 1

Stack: 0 S 1       Input: $
- ACTION[1, $] = acc → ACCEPT!
```

### LR(0) Conflicts

**LR(0) Grammar** has **no conflicts** if:
- No state has both shift and reduce items
- No state has multiple reduce items

**Example of LR(0) Conflict:**

Grammar: E → E + T | T, T → id

State may contain:
```
E → E + T•    (reduce)
E → E• + T    (shift on +)
```

This is a **shift-reduce conflict** - LR(0) cannot handle this grammar!

---

## 5.9 SLR(1) Parsing

**SLR(1)** (Simple LR) uses LR(0) items plus FOLLOW sets to resolve conflicts.

### Key Difference from LR(0)

| LR(0) | SLR(1) |
|-------|--------|
| Reduce on ALL terminals | Reduce only on FOLLOW(A) |
| More conflicts | Fewer conflicts |
| Weaker | Stronger |

### SLR(1) Table Construction Rules

**For each state I in the canonical collection:**

1. **Shift**: If A → α•aβ is in I and GOTO(I, a) = Iⱼ:
   - Set ACTION[I, a] = "shift j"

2. **Reduce**: If A → α• is in I (and A ≠ S'):
   - For all a in **FOLLOW(A)** only:
     - Set ACTION[I, a] = "reduce A → α"

3. **Accept**: If S' → S• is in I:
   - Set ACTION[I, $] = "accept"

4. **GOTO**: If GOTO(I, A) = Iⱼ for non-terminal A:
   - Set GOTO[I, A] = j

### Complete SLR(1) Example

**Grammar:**
```
(0) S' → S
(1) S  → AA
(2) A  → aA
(3) A  → b
```

**Step 1: Compute FIRST and FOLLOW Sets**

| Non-terminal | FIRST | FOLLOW |
|--------------|-------|--------|
| S | {a, b} | {$} |
| A | {a, b} | {a, b, $} |

**Step 2: Construct LR(0) States**

| State | Items |
|-------|-------|
| I₀ | S' → •S, S → •AA, A → •aA, A → •b |
| I₁ | S' → S• |
| I₂ | S → A•A, A → •aA, A → •b |
| I₃ | A → a•A, A → •aA, A → •b |
| I₄ | A → b• |
| I₅ | S → AA• |
| I₆ | A → aA• |

**Step 3: Construct SLR(1) Parsing Table**

```
┌───────┬───────────────────────────┬───────────┐
│ State │          ACTION           │   GOTO    │
│       ├─────────┬─────────┬───────┼─────┬─────┤
│       │    a    │    b    │   $   │  S  │  A  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   0   │   s3    │   s4    │       │  1  │  2  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   1   │         │         │  acc  │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   2   │   s3    │   s4    │       │     │  5  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   3   │   s3    │   s4    │       │     │  6  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   4   │   r3    │   r3    │  r3   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   5   │         │         │  r1   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   6   │   r2    │   r2    │  r2   │     │     │
└───────┴─────────┴─────────┴───────┴─────┴─────┘

Productions: (1) S → AA  (2) A → aA  (3) A → b
```

**Table Construction Explanation:**

- **State 4 (A → b•)**: Reduce by A → b on FOLLOW(A) = {a, b, $}
- **State 5 (S → AA•)**: Reduce by S → AA on FOLLOW(S) = {$}
- **State 6 (A → aA•)**: Reduce by A → aA on FOLLOW(A) = {a, b, $}

### SLR(1) String Parsing Example

**Parse the string: `aabb`**

| Step | Stack | Input | Action | Explanation |
|------|-------|-------|--------|-------------|
| 1 | 0 | aabb$ | s3 | ACTION[0,a]=s3 |
| 2 | 0 a 3 | abb$ | s3 | ACTION[3,a]=s3 |
| 3 | 0 a 3 a 3 | bb$ | s4 | ACTION[3,b]=s4 |
| 4 | 0 a 3 a 3 b 4 | b$ | r3 | ACTION[4,b]=r3, reduce A→b |
| 5 | 0 a 3 a 3 A 6 | b$ | r2 | GOTO[3,A]=6, reduce A→aA |
| 6 | 0 a 3 A 6 | b$ | r2 | GOTO[3,A]=6, reduce A→aA |
| 7 | 0 A 2 | b$ | s4 | GOTO[0,A]=2, shift b |
| 8 | 0 A 2 b 4 | $ | r3 | ACTION[4,$]=r3, reduce A→b |
| 9 | 0 A 2 A 5 | $ | r1 | GOTO[2,A]=5, reduce S→AA |
| 10 | 0 S 1 | $ | acc | GOTO[0,S]=1, ACCEPT! |

### Detailed Step-by-Step Trace

**Shifting Phase:**
```
Stack: 0              Input: aabb$   → Shift a, goto 3
Stack: 0 a 3          Input: abb$    → Shift a, goto 3
Stack: 0 a 3 a 3      Input: bb$     → Shift b, goto 4
Stack: 0 a 3 a 3 b 4  Input: b$
```

**First Reduction Sequence:**
```
Stack: 0 a 3 a 3 b 4  Input: b$
- ACTION[4, b] = r3 (reduce A → b)
- Pop 2 symbols (b, 4), top = state 3
- GOTO[3, A] = 6, push A and 6

Stack: 0 a 3 a 3 A 6  Input: b$
- ACTION[6, b] = r2 (reduce A → aA)
- Pop 4 symbols (a, 3, A, 6), top = state 3
- GOTO[3, A] = 6, push A and 6

Stack: 0 a 3 A 6      Input: b$
- ACTION[6, b] = r2 (reduce A → aA)
- Pop 4 symbols (a, 3, A, 6), top = state 0
- GOTO[0, A] = 2, push A and 2

Stack: 0 A 2          Input: b$
```

**Second A and Final Accept:**
```
Stack: 0 A 2          Input: b$
- ACTION[2, b] = s4, shift b, goto 4

Stack: 0 A 2 b 4      Input: $
- ACTION[4, $] = r3 (reduce A → b)
- Pop 2 symbols (b, 4), top = state 2
- GOTO[2, A] = 5, push A and 5

Stack: 0 A 2 A 5      Input: $
- ACTION[5, $] = r1 (reduce S → AA)
- Pop 4 symbols (A, 2, A, 5), top = state 0
- GOTO[0, S] = 1, push S and 1

Stack: 0 S 1          Input: $
- ACTION[1, $] = acc → ACCEPT!
```

### Limitations of SLR(1)

SLR uses FOLLOW sets, which may be too imprecise:

**Example Grammar where SLR fails:**
```
S → L = R | R
L → * R | id
R → L
```

**Problem State:**
```
State I₂: S → L•= R
          R → L•
```

- FOLLOW(R) = {=, $}
- SLR says: Reduce R → L on both = and $
- But on input `id = ...`, we should SHIFT =, not reduce!

**Conflict:** ACTION[I₂, =] = shift AND reduce → SLR conflict!

**SLR cannot handle all grammars that CLR/LALR can.**

---

## 5.10 CLR(1) Parsing (Canonical LR)

**CLR(1)** uses LR(1) items which include lookahead information.

### LR(1) Item

An LR(1) item has the form:
```
[A → α•β, a]
```

Where:
- A → αβ is a production
- a is a lookahead terminal (or $)

The lookahead is only relevant when β is empty (reduction time).

### Closure for LR(1) Items

```
CLOSURE(I):
    repeat
        for each item [A → α•Bβ, a] in I:
            for each production B → γ:
                for each terminal b in FIRST(βa):
                    add [B → •γ, b] to I
    until no new items added
    return I
```

### GOTO for LR(1) Items

Same as LR(0), but carries lookahead:
```
GOTO(I, X):
    J = empty set
    for each item [A → α•Xβ, a] in I:
        add [A → αX•β, a] to J
    return CLOSURE(J)
```

### Complete CLR(1) Example

**Grammar:**
```
(0) S' → S
(1) S  → CC
(2) C  → cC
(3) C  → d
```

**Step 1: Compute FIRST Sets**

| Non-terminal | FIRST |
|--------------|-------|
| S | {c, d} |
| C | {c, d} |

**Step 2: Construct All LR(1) States**

**State I₀:** CLOSURE({[S' → •S, $]})
```
[S' → •S, $]
[S  → •CC, $]
[C  → •cC, c/d]    ← FIRST(C$) = {c, d}
[C  → •d, c/d]
```

**State I₁:** GOTO(I₀, S)
```
[S' → S•, $]
```

**State I₂:** GOTO(I₀, C)
```
[S → C•C, $]
[C → •cC, $]       ← FIRST($) = {$}
[C → •d, $]
```

**State I₃:** GOTO(I₀, c)
```
[C → c•C, c/d]
[C → •cC, c/d]
[C → •d, c/d]
```

**State I₄:** GOTO(I₀, d)
```
[C → d•, c/d]      ← Lookahead: c, d
```

**State I₅:** GOTO(I₂, C)
```
[S → CC•, $]
```

**State I₆:** GOTO(I₂, c)
```
[C → c•C, $]
[C → •cC, $]
[C → •d, $]
```

**State I₇:** GOTO(I₂, d)
```
[C → d•, $]        ← Lookahead: $ only
```

**State I₈:** GOTO(I₃, C)
```
[C → cC•, c/d]
```

**State I₉:** GOTO(I₆, C)
```
[C → cC•, $]
```

**Note:** I₃ and I₆ have same core but different lookaheads!
**Note:** I₄ and I₇ have same core but different lookaheads!
**Note:** I₈ and I₉ have same core but different lookaheads!

**Step 3: CLR(1) Parsing Table Construction Rules**

For item [A → α•, a]:
- Set ACTION[I, a] = "reduce A → α"
- **Only for the specific lookahead a**, not all of FOLLOW(A)

**CLR(1) Parsing Table:**

```
┌───────┬───────────────────────────┬───────────┐
│ State │          ACTION           │   GOTO    │
│       ├─────────┬─────────┬───────┼─────┬─────┤
│       │    c    │    d    │   $   │  S  │  C  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   0   │   s3    │   s4    │       │  1  │  2  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   1   │         │         │  acc  │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   2   │   s6    │   s7    │       │     │  5  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   3   │   s3    │   s4    │       │     │  8  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   4   │   r3    │   r3    │       │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   5   │         │         │  r1   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   6   │   s6    │   s7    │       │     │  9  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   7   │         │         │  r3   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   8   │   r2    │   r2    │       │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   9   │         │         │  r2   │     │     │
└───────┴─────────┴─────────┴───────┴─────┴─────┘

Productions: (1) S → CC  (2) C → cC  (3) C → d
```

**Key Observations:**
- State 4: Reduce C→d on lookahead {c, d} only
- State 7: Reduce C→d on lookahead {$} only
- State 8: Reduce C→cC on lookahead {c, d} only
- State 9: Reduce C→cC on lookahead {$} only

### CLR(1) String Parsing Example

**Parse the string: `cdd`**

| Step | Stack | Input | Action | Explanation |
|------|-------|-------|--------|-------------|
| 1 | 0 | cdd$ | s3 | ACTION[0,c]=s3 |
| 2 | 0 c 3 | dd$ | s4 | ACTION[3,d]=s4 |
| 3 | 0 c 3 d 4 | d$ | r3 | ACTION[4,d]=r3, reduce C→d |
| 4 | 0 c 3 C 8 | d$ | r2 | GOTO[3,C]=8, ACTION[8,d]=r2 |
| 5 | 0 C 2 | d$ | s7 | GOTO[0,C]=2, ACTION[2,d]=s7 |
| 6 | 0 C 2 d 7 | $ | r3 | ACTION[7,$]=r3, reduce C→d |
| 7 | 0 C 2 C 5 | $ | r1 | GOTO[2,C]=5, ACTION[5,$]=r1 |
| 8 | 0 S 1 | $ | acc | GOTO[0,S]=1, ACCEPT! |

### Detailed Step-by-Step Trace

**Shifting and First Reductions:**
```
Stack: 0            Input: cdd$
- ACTION[0, c] = s3 → Shift c, goto 3

Stack: 0 c 3        Input: dd$
- ACTION[3, d] = s4 → Shift d, goto 4

Stack: 0 c 3 d 4    Input: d$
- ACTION[4, d] = r3 (reduce C → d, lookahead is d)
- Pop 2 symbols (d, 4), top = state 3
- GOTO[3, C] = 8, push C and 8

Stack: 0 c 3 C 8    Input: d$
- ACTION[8, d] = r2 (reduce C → cC, lookahead is d)
- Pop 4 symbols (c, 3, C, 8), top = state 0
- GOTO[0, C] = 2, push C and 2
```

**Second C and Accept:**
```
Stack: 0 C 2        Input: d$
- ACTION[2, d] = s7 → Shift d, goto 7

Stack: 0 C 2 d 7    Input: $
- ACTION[7, $] = r3 (reduce C → d, lookahead is $)
- Pop 2 symbols (d, 7), top = state 2
- GOTO[2, C] = 5, push C and 5

Stack: 0 C 2 C 5    Input: $
- ACTION[5, $] = r1 (reduce S → CC)
- Pop 4 symbols (C, 2, C, 5), top = state 0
- GOTO[0, S] = 1, push S and 1

Stack: 0 S 1        Input: $
- ACTION[1, $] = acc → ACCEPT!
```

### CLR(1) Power vs Table Size

**Advantages:**
- Most powerful LR parsing method
- Can handle more grammars than SLR or LALR
- Precise lookahead prevents spurious conflicts

**Disadvantages:**
- **Very large parsing tables** (states split by lookahead)
- For this small grammar: 10 CLR states vs 7 SLR states
- Real grammars can have 10x more CLR states than LALR

---

## 5.11 LALR(1) Parsing

**LALR(1)** (Look-Ahead LR) combines the power of CLR(1) with the table size of SLR(1).

### The Key Insight

Many CLR(1) states differ only in lookahead symbols but have the same **core** (LR(0) part).

### What is a Core?

The **core** of an LR(1) item set is the set of LR(0) items (ignoring lookaheads).

**Example:**
```
CLR State I₄: [C → d•, c/d]    Core: {C → d•}
CLR State I₇: [C → d•, $]      Core: {C → d•}

Same core! → Can be merged
```

### LALR(1) Construction Method

**Method 1: From CLR (Merging)**
1. Build CLR(1) canonical collection
2. Find states with identical cores
3. Merge such states (union their lookaheads)
4. Build parsing table from merged states

**Method 2: Direct Construction (Efficient)**
- Build LR(0) automaton
- Propagate lookaheads through the automaton
- More efficient for large grammars

### Complete LALR(1) Example

**Grammar:**
```
(0) S' → S
(1) S  → CC
(2) C  → cC
(3) C  → d
```

**Step 1: Identify CLR States with Same Cores**

| CLR States | Core | Merged LALR State |
|------------|------|-------------------|
| I₀ | S'→•S, S→•CC, C→•cC, C→•d | I₀ |
| I₁ | S'→S• | I₁ |
| I₂ | S→C•C, C→•cC, C→•d | I₂ |
| I₃, I₆ | C→c•C, C→•cC, C→•d | I₃₆ |
| I₄, I₇ | C→d• | I₄₇ |
| I₅ | S→CC• | I₅ |
| I₈, I₉ | C→cC• | I₈₉ |

**Step 2: Merge Lookaheads**

| LALR State | Items with Merged Lookaheads |
|------------|------------------------------|
| I₀ | [S'→•S,$], [S→•CC,$], [C→•cC,c/d], [C→•d,c/d] |
| I₁ | [S'→S•,$] |
| I₂ | [S→C•C,$], [C→•cC,$], [C→•d,$] |
| I₃₆ | [C→c•C,c/d/$], [C→•cC,c/d/$], [C→•d,c/d/$] |
| I₄₇ | [C→d•, c/d/$] ← Merged lookaheads |
| I₅ | [S→CC•,$] |
| I₈₉ | [C→cC•, c/d/$] ← Merged lookaheads |

**Step 3: LALR(1) Parsing Table**

```
┌───────┬───────────────────────────┬───────────┐
│ State │          ACTION           │   GOTO    │
│       ├─────────┬─────────┬───────┼─────┬─────┤
│       │    c    │    d    │   $   │  S  │  C  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   0   │   s3    │   s4    │       │  1  │  2  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   1   │         │         │  acc  │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   2   │   s3    │   s4    │       │     │  5  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   3   │   s3    │   s4    │       │     │  6  │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   4   │   r3    │   r3    │  r3   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   5   │         │         │  r1   │     │     │
├───────┼─────────┼─────────┼───────┼─────┼─────┤
│   6   │   r2    │   r2    │  r2   │     │     │
└───────┴─────────┴─────────┴───────┴─────┴─────┘

Productions: (1) S → CC  (2) C → cC  (3) C → d
```

**Note:** LALR table has **7 states** vs CLR's **10 states**!

### LALR(1) String Parsing Example

**Parse the string: `cdd`**

| Step | Stack | Input | Action | Explanation |
|------|-------|-------|--------|-------------|
| 1 | 0 | cdd$ | s3 | ACTION[0,c]=s3 |
| 2 | 0 c 3 | dd$ | s4 | ACTION[3,d]=s4 |
| 3 | 0 c 3 d 4 | d$ | r3 | ACTION[4,d]=r3, reduce C→d |
| 4 | 0 c 3 C 6 | d$ | r2 | GOTO[3,C]=6, ACTION[6,d]=r2, reduce C→cC |
| 5 | 0 C 2 | d$ | s4 | GOTO[0,C]=2, ACTION[2,d]=s4 |
| 6 | 0 C 2 d 4 | $ | r3 | ACTION[4,$]=r3, reduce C→d |
| 7 | 0 C 2 C 5 | $ | r1 | GOTO[2,C]=5, ACTION[5,$]=r1, reduce S→CC |
| 8 | 0 S 1 | $ | acc | GOTO[0,S]=1, ACCEPT! |

### Detailed Step-by-Step Trace

**Phase 1: Building First C**
```
Stack: 0              Input: cdd$
- ACTION[0, c] = s3 → Shift c, push 3

Stack: 0 c 3          Input: dd$
- ACTION[3, d] = s4 → Shift d, push 4

Stack: 0 c 3 d 4      Input: d$
- ACTION[4, d] = r3 → Reduce C → d
- Pop (d, 4), top = 3
- GOTO[3, C] = 6 → Push C, 6

Stack: 0 c 3 C 6      Input: d$
- ACTION[6, d] = r2 → Reduce C → cC
- Pop (c, 3, C, 6), top = 0
- GOTO[0, C] = 2 → Push C, 2

Stack: 0 C 2          Input: d$
```

**Phase 2: Building Second C and Accept**
```
Stack: 0 C 2          Input: d$
- ACTION[2, d] = s4 → Shift d, push 4

Stack: 0 C 2 d 4      Input: $
- ACTION[4, $] = r3 → Reduce C → d
- Pop (d, 4), top = 2
- GOTO[2, C] = 5 → Push C, 5

Stack: 0 C 2 C 5      Input: $
- ACTION[5, $] = r1 → Reduce S → CC
- Pop (C, 2, C, 5), top = 0
- GOTO[0, S] = 1 → Push S, 1

Stack: 0 S 1          Input: $
- ACTION[1, $] = acc → SUCCESS!
```

### LALR vs SLR vs CLR Comparison

```
Grammar Class Hierarchy:

    SLR(1) ⊂ LALR(1) ⊂ CLR(1)
    
    CLR(1) is the largest class (most powerful)
    SLR(1) is the smallest class (least powerful)
```

| Aspect | SLR(1) | LALR(1) | CLR(1) |
|--------|--------|---------|--------|
| States | Same as LR(0) | Same as LR(0) | More states |
| Lookahead | FOLLOW sets | Precise (merged) | Precise |
| Table Size | Smallest | Small | Largest |
| Power | Weakest | Strong | Strongest |
| Conflicts | Most | Fewer | Fewest |
| Usage | Academic | **Practical (YACC)** | Rarely used |

### When LALR Merging Causes Problems

LALR merging can introduce **reduce-reduce conflicts** not present in CLR:

**Example:**
```
Grammar:
S → aAd | bBd | aBe | bAe
A → c
B → c
```

**CLR States (partial):**
```
State Iₓ: [A → c•, d], [B → c•, e]   (from aAd, aBe paths)
State Iᵧ: [A → c•, e], [B → c•, d]   (from bAe, bBd paths)
```

**After LALR Merging:**
```
Merged: [A → c•, d/e], [B → c•, d/e]
```

**Problem:** On input 'd', should we reduce to A or B? → **Reduce-reduce conflict!**

**Note:** LALR **never** introduces shift-reduce conflicts that weren't in CLR.

### Practical LALR(1) Tools

LALR(1) is used by most parser generators:
- **YACC** (Yet Another Compiler Compiler)
- **Bison** (GNU version of YACC)
- **PLY** (Python Lex-Yacc)
- **CUP** (Java)

---

## 5.12 Comparison of All LR Methods

### Summary Table

| Feature | LR(0) | SLR(1) | LALR(1) | CLR(1) |
|---------|-------|--------|---------|--------|
| Lookahead | None | FOLLOW | Precise (merged) | Precise |
| # of States | Smallest | Same as LR(0) | Same as LR(0) | Largest |
| Table Size | Small | Small | Small | Very Large |
| Power | Weakest | Weak | Strong | Strongest |
| Reduce Rule | All terminals | FOLLOW(A) | Specific lookahead | Specific lookahead |
| Practical Use | Educational | Educational | **Industry Standard** | Rarely |

### Conflict Resolution Comparison

**Given state with item: A → α•**

| Method | Reduce ACTION on |
|--------|------------------|
| LR(0) | All terminals |
| SLR(1) | All of FOLLOW(A) |
| LALR(1) | Precise lookaheads (merged from CLR) |
| CLR(1) | Exact lookahead from LR(1) item |

### When to Use Which?

1. **LR(0)**: Only for learning concepts
2. **SLR(1)**: Simple grammars, academic exercises
3. **LALR(1)**: **Production compilers** (best balance)
4. **CLR(1)**: When LALR has conflicts (very rare)

---

## 5.13 Operator Precedence Parsing

**Operator Precedence Parsing** is a simple bottom-up technique for expressions with operators.

### Precedence Relations

Between adjacent terminals a and b:
- **a < b**: a yields precedence to b (b has higher precedence)
- **a = b**: a has same precedence as b
- **a > b**: a takes precedence over b

### Precedence Table for Arithmetic

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│     │  +  │  *  │  id │  (  │  )  │  $  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  +  │  >  │  <  │  <  │  <  │  >  │  >  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  *  │  >  │  >  │  <  │  <  │  >  │  >  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  id │  >  │  >  │     │     │  >  │  >  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  (  │  <  │  <  │  <  │  <  │  =  │     │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  )  │  >  │  >  │     │     │  >  │  >  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  $  │  <  │  <  │  <  │  <  │     │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

### Parsing Algorithm

```
push $ onto stack

while not (stack = $ and input = $):
    let a = topmost terminal on stack
    let b = current input symbol
    
    if a < b or a = b:
        push b
        advance input
    
    else if a > b:
        repeat:
            pop terminal from stack
        until top terminal < popped terminal
        (reduce the "handle")
    
    else:
        error
```

### Limitations

1. Only handles expressions, not full languages
2. Hard to specify all semantic actions
3. Difficult error recovery

---

## 5.14 YACC/Bison - Parser Generators

**YACC** (Yet Another Compiler Compiler) generates LALR(1) parsers from grammar specifications.

### YACC Program Structure

```yacc
%{
    /* C declarations: includes, globals */
    #include <stdio.h>
    int yylex();
    void yyerror(char *s);
%}

/* YACC declarations: tokens, precedence */
%token ID NUMBER
%left '+' '-'
%left '*' '/'
%right UMINUS

%%
    /* Grammar rules */
    
expr : expr '+' expr    { $$ = $1 + $3; }
     | expr '-' expr    { $$ = $1 - $3; }
     | expr '*' expr    { $$ = $1 * $3; }
     | expr '/' expr    { $$ = $1 / $3; }
     | '(' expr ')'     { $$ = $2; }
     | '-' expr %prec UMINUS  { $$ = -$2; }
     | NUMBER           { $$ = $1; }
     ;

%%
    /* User C code */
    
int main() {
    yyparse();
    return 0;
}

void yyerror(char *s) {
    fprintf(stderr, "Error: %s\n", s);
}
```

### YACC Workflow

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   grammar.y  │ YACC │   y.tab.c    │  CC  │    parser    │
│   (YACC spec)│─────▶│   (C code)   │─────▶│ (executable) │
└──────────────┘      └──────────────┘      └──────────────┘
```

### Semantic Values

- `$$` - Value of left-hand side
- `$1, $2, ...` - Values of right-hand side symbols
- `%type <type>` - Specify types for non-terminals

### Conflict Resolution in YACC

1. **Shift-reduce**: YACC prefers **shift**
2. **Reduce-reduce**: YACC uses **first production** listed
3. **%left, %right, %nonassoc**: Specify associativity
4. **%prec**: Override default precedence

---

## 5.15 Error Recovery in LR Parsing

### Panic Mode

1. Pop states until finding one with GOTO on error symbol
2. Shift a special "error" token
3. Discard input until a "synchronizing" token is found
4. Continue parsing

### YACC Error Recovery

```yacc
stmt : error ';'    { yyerror("bad statement"); }
     | /* other productions */
     ;
```

The `error` token matches any sequence of erroneous input.

---

## 5.16 Summary

### Key Concepts:

1. **Bottom-Up Parsing** reduces input to start symbol (reverse rightmost derivation)

2. **Shift-Reduce** operations build the parse tree from leaves to root

3. **Handle** is the substring to reduce next

4. **LR Parsing** uses:
   - Stack with states and symbols
   - Parsing table (ACTION + GOTO)
   - Shift, reduce, accept, error actions

5. **LR Variants:**
   - **SLR(1)**: Uses FOLLOW sets, simple but weak
   - **CLR(1)**: Uses lookahead in items, powerful but large tables
   - **LALR(1)**: Merges CLR states, practical balance

6. **Parser Generators** (YACC/Bison) automate LALR(1) parser construction

---

## 🔍 Practice Questions

1. Trace the shift-reduce parsing of `(a + b) * c` with appropriate grammar.

2. Construct LR(0) items and automaton for:
   ```
   S → aABe
   A → Abc | b
   B → d
   ```

3. Build SLR(1) parsing table for:
   ```
   S → AA
   A → aA | b
   ```

4. Explain the difference between SLR, LALR, and CLR with examples.

5. Write a YACC program for a simple calculator supporting +, -, *, /.

---

*Next Chapter: [Semantic Analysis](06-Semantic-Analysis.md)*
