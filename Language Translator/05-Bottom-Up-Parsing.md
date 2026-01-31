# Chapter 5: Bottom-Up Parsing

## 5.1 Introduction to Bottom-Up Parsing

**Bottom-Up Parsing** constructs the parse tree from the **leaves** (terminals) to the **root** (start symbol). It attempts to find a **rightmost derivation in reverse**.

### 🎯 The Core Problem: Why Do We Need Bottom-Up Parsing?

**Problem with Top-Down Parsing:**
- Top-down parsers (LL parsers) look at the current non-terminal and try to **predict** which production to use
- They require the grammar to be LL(1): no left recursion, must be left-factored
- Many natural programming language constructs **cannot** be expressed in LL(1) grammars
- Example: Left-recursive grammars like `E → E + T` are natural for expressing left-associativity but impossible for LL parsers

**The Solution - Bottom-Up Parsing:**
- Instead of predicting, we **observe** what we've read and **recognize** patterns
- We don't need to decide which production to use until we've seen the **entire right-hand side**
- This "wait and see" approach handles a much larger class of grammars
- Left recursion is **not a problem** (in fact, it's often preferred!)

### The Intuition: Building a Tree Upside Down

Imagine you're doing a jigsaw puzzle:
- **Top-down** = Start with the picture frame (goal) and try to guess which pieces fit inside
- **Bottom-up** = Start with individual pieces, group matching pieces together, keep grouping until you complete the picture

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

The most common form of bottom-up parsing uses two fundamental operations.

### 🎯 The Problem: How Do We Actually Perform Bottom-Up Parsing?

**The Challenge:**
- We want to recognize patterns in the input and group them
- But we're reading **left-to-right** while we need to build the tree **bottom-to-top**
- How do we keep track of what we've seen while looking for patterns?

**The Solution - Stack-Based Approach:**
- Use a **stack** to accumulate symbols we've read
- The stack acts as "memory" of what we've seen so far
- When the top of stack matches a production's right-hand side → we found a pattern!
- Replace (reduce) those symbols with the corresponding left-hand side

### The Two Operations

| Operation | Description | When to Use |
|-----------|-------------|-------------|
| **Shift** | Push the next input symbol onto the stack | When we haven't seen enough yet to recognize a pattern |
| **Reduce** | Replace symbols on top of stack with a non-terminal (applying a production in reverse) | When we recognize a complete pattern (handle) |

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

### Handle: The Key Concept

**🎯 The Problem:** 
We see many substrings that match right-hand sides of productions. Which one should we reduce?

**Example Confusion:**
For grammar `E → E + T | T` and `T → T * F | F` and `F → id`

If stack has: `E + T * F`
- `T * F` matches `T → T * F` (with T and F inside)
- `F` alone matches... nothing that helps
- `E + T` matches `E → E + T`

Which reduction is correct? **Only `T * F` → `T` is the correct choice here!**

**The Solution - Handle:**
A **handle** is a substring that matches the right side of a production AND reducing it is a step in the reverse of a rightmost derivation.

**Formally**: If S ⟹*ᵣₘ αAw ⟹ᵣₘ αβw, then β in position following α is a handle of αβw.

**Intuitive Understanding:**
- The handle is **always at the top of the stack** (never buried inside)
- It's the pattern that was **most recently created** during the derivation
- Reducing the handle is like "undoing" the last step of the derivation
- If you reduce anything OTHER than the handle, you'll get stuck!

**Why Handle is Always at Top:**
```
Rightmost derivation always expands the RIGHTMOST non-terminal.
So the most recent expansion is at the RIGHT end.
Since stack grows left-to-right, the handle is at the TOP.
```

### Example: Shift-Reduce Parsing

**Grammar:**
```
E → E + T | T
T → T * F | F
F → ( E ) | id
```

**Parsing: `id + id * id`**

| Stack | Input | Action | Why This Action? |
|-------|-------|--------|------------------|
| $ | id + id * id $ | Shift | Nothing to reduce yet |
| $ id | + id * id $ | Reduce F → id | `id` matches `F → id` |
| $ F | + id * id $ | Reduce T → F | `F` matches `T → F` |
| $ T | + id * id $ | Reduce E → T | `T` matches `E → T` |
| $ E | + id * id $ | Shift | Can't reduce `E` alone |
| $ E + | id * id $ | Shift | Need something after + |
| $ E + id | * id $ | Reduce F → id | `id` matches `F → id` |
| $ E + F | * id $ | Reduce T → F | `F` matches `T → F` |
| $ E + T | * id $ | **Shift** | ⭐ KEY: Don't reduce yet! See below |
| $ E + T * | id $ | Shift | Need something after * |
| $ E + T * id | $ | Reduce F → id | `id` matches `F → id` |
| $ E + T * F | $ | Reduce T → T * F | `T * F` matches production |
| $ E + T | $ | Reduce E → E + T | `E + T` matches production |
| $ E | $ | **Accept** | Stack has S, input exhausted! |

**⭐ Critical Decision at Step 9:**
```
Stack: $ E + T      Input: * id $

Option A: Reduce E + T → E (using E → E + T)
Option B: Shift * 

We chose SHIFT! Why?
- * has HIGHER precedence than +
- If we reduced now, we'd get: (id + id) * id  ← WRONG!
- By shifting, we get: id + (id * id)  ← CORRECT!

This is how precedence is enforced by the choice of shift vs reduce!
```

---

## 5.3 Conflicts in Shift-Reduce Parsing

### 🎯 The Core Problem: Ambiguity in Action Selection

**The Fundamental Issue:**
At any point during parsing, we have the stack contents and the next input symbol. We need to decide:
1. Should we **shift** (read more input)?
2. Should we **reduce** (recognize a pattern)?
3. If reduce, **which production** to use?

**Why This Is Hard:**
- We don't have unlimited lookahead - we can only see limited input ahead
- Multiple patterns might match the stack top
- Sometimes shifting vs reducing both seem valid

**The Result: CONFLICTS**
When the parser cannot make a unique decision, we have a conflict. This means either:
- The grammar is **ambiguous** (genuinely multiple meanings)
- The parser method is **not powerful enough** for this grammar

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

**Why Prefer Shift?**
- Shifting delays the decision - we gather more information
- In the if-else case, shifting `else` means we associate it with the **nearest** `if`
- This matches programmer expectations and most language specifications
- It's the **standard convention** in parser generators like YACC

### Reduce-Reduce Conflict

The parser cannot decide which production to use for reduction.

**This is more serious than shift-reduce conflicts!**

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

**Why Reduce-Reduce Is Worse:**
- Shift-reduce can be resolved by convention (prefer shift)
- Reduce-reduce means **fundamentally ambiguous situation**
- Often indicates a **design flaw** in the grammar
- Usually requires **rewriting the grammar** to distinguish the cases

**Common Fix:** Add more context or split productions:
```
# Instead of ambiguous:
item → id ( list )

# Split into distinct contexts:
func_call → id ( arg_list )
array_access → id [ index_list ]
```

---

## 5.4 LR Parsing

### 🎯 The Problem: We Need a Systematic Way to Find Handles

**The Challenge:**
- Basic shift-reduce works, but how do we **know** when we have a handle?
- Looking at just the stack top isn't enough
- We need to track **parsing history** (what states we've gone through)

**The Solution - LR Parsing with States:**
- Use a **finite automaton** to track the parsing state
- Each state encodes what we've seen and what we're expecting
- The automaton tells us exactly when to shift and when to reduce
- States are pushed onto the stack alongside symbols

**LR Parsing** is the most powerful shift-reduce parsing technique for deterministic context-free grammars.

### What LR Means

```
L - Left-to-right scan of input
R - Rightmost derivation (constructed in reverse)

Note: Even though we BUILD left-to-right, the derivation
we DISCOVER is the rightmost derivation in reverse!
```

### Why LR Parsing Is So Powerful

| Advantage | Explanation |
|-----------|-------------|
| **Handles more grammars** | LR grammars ⊃ LL grammars - strictly more powerful |
| **Left recursion OK** | `E → E + T` works perfectly (preferred for left-associativity) |
| **Early error detection** | Detects errors as soon as prefix cannot lead to valid program |
| **No backtracking** | Every decision is deterministic (no guessing and undoing) |
| **Efficient** | Linear time O(n) parsing |

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

### 🎯 The Problem: How Do We Build the LR Automaton?

**The Challenge:**
- We need states that track "what productions might we be in the middle of?"
- During parsing, we might be partially through multiple productions simultaneously
- How do we represent this?

**The Solution - LR Items:**
- An **item** represents a **partially recognized production**
- The **dot (•)** marks how far we've progressed
- A **state** is a set of items (all the productions we might be parsing)

### LR(0) Item

An **LR(0) item** is a production with a dot (•) indicating how much has been seen.

For production A → XYZ, the items are:
```
A → •XYZ    (about to see X - at the beginning)
A → X•YZ    (seen X, expecting Y next)
A → XY•Z    (seen XY, expecting Z next)
A → XYZ•    (seen everything, ready to reduce!)
```

### Deep Intuition: What Items Really Mean

**Think of each item as a "hypothesis":**

| Item | Meaning | What We're Doing |
|------|---------|------------------|
| `A → •XYZ` | "I think we might be about to see production A → XYZ" | Looking for X |
| `A → X•YZ` | "We've confirmed X, still building toward A" | Looking for Y |
| `A → XYZ•` | "We've found the complete pattern XYZ!" | Ready to reduce |

**Why dot at end means reduce:**
```
A → XYZ•  means "I've seen X, then Y, then Z on the stack"
          The full right-hand side is recognized!
          We can now replace XYZ with A (reduce).
```

**Why dot before terminal means shift:**
```
A → X•YZ  means "I need to see Y next"
          If Y is a terminal and it's in the input → shift it!
```

### Augmented Grammar

To build the parser, we augment the grammar with a new start symbol:

```
Original:          Augmented:
S → ...            S' → S
                   S → ...
```

**Why Augment?**
- The item `S' → S•` gives us a **unique accepting configuration**
- When we see this item with `$` (end of input), we know parsing is **complete**
- Without it, accepting condition would be more complex

---

## 5.7 Constructing LR(0) Automaton

### 🎯 The Problem: How Do We Compute All Possible Parser States?

**The Challenge:**
- A state represents all productions we might be in the middle of
- When we're expecting a non-terminal, we must also expect all its productions
- How do we systematically compute this?

**The Solution: Two Key Operations:**
1. **CLOSURE** - Expand a state to include all implied items
2. **GOTO** - Compute the next state after seeing a symbol

### Closure Operation: Expanding Expectations

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

**🔑 The Key Insight:**

If we have `A → α•Bβ` (expecting non-terminal B), we must ALSO be ready for whatever B can produce!

```
Example: If we have  E → E + •T
         We're expecting T
         But T → T * F | F
         So we must be ready for T → •T * F  and  T → •F
         And F → ( E ) | id
         So also F → •( E )  and  F → •id
```

**Analogy:** If you're "expecting a vehicle," you must also be ready for "a car" or "a truck" or "a bike" - all specific types of vehicles.

### GOTO Operation: Transitioning Between States

**GOTO(I, X)** where I is a set of items and X is a grammar symbol:

```
GOTO(I, X):
    J = empty set
    for each item A → α•Xβ in I:
        add A → αX•β to J
    return CLOSURE(J)
```

**🔑 What GOTO Does:**

GOTO answers: "If we're in state I and we see/shift symbol X, what state do we go to?"

```
Step 1: Find all items in I that are "expecting" X (dot before X)
Step 2: Move the dot past X (we've now "seen" X)
Step 3: Take closure (expand any new non-terminal expectations)
```

**Example:**
```
State I has: E → E + •T     (expecting T)
             T → •T * F     (expecting T)
             T → •F         (expecting F)

GOTO(I, T) produces:
    From E → E + •T → E → E + T•  (seen T!)
    From T → •T * F → T → T• * F  (seen T, now expecting *)

Then take closure: no new non-terminals expected after • so done.
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
E → E + T•    (reduce: we can reduce to E)
E → E• + T    (shift: we can shift + )
```

This is a **shift-reduce conflict** - LR(0) cannot handle this grammar!

### 🎯 Why LR(0) Is Too Weak

**The Problem:**
LR(0) has **no lookahead** - it doesn't consider what comes next in the input.
- When we see `E → E + T•`, LR(0) says "reduce on ANY input symbol"
- But that's wrong! We should only reduce when appropriate.

**Real Example:**
Input: `id + id + id`

After seeing `id + id`, stack has: `E + T`
- If next input is `+`, we should **reduce** `E + T` → `E` (left associativity)
- If next input is `*`, we should... wait, that's not in this grammar

But for grammars with multiple operators and precedence, we need to look ahead!

---

## 5.9 SLR(1) Parsing

### 🎯 The Problem: LR(0) Reduces Too Eagerly

**The Issue:**
- LR(0) reduce items say "reduce on ALL input symbols"
- This causes conflicts because sometimes we shouldn't reduce yet
- We need to be **smarter** about when to reduce

**The Solution - Use FOLLOW Sets:**
- Only reduce `A → α` when the input symbol can legitimately **follow** A
- If input symbol ∉ FOLLOW(A), then reducing to A would be wrong anyway
- This eliminates many spurious conflicts!

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

### 🎯 The Problem: FOLLOW Sets Are Too Coarse

SLR uses FOLLOW sets, which consider ALL contexts where a non-terminal can appear. But sometimes, in a **specific** context, only a **subset** of FOLLOW is valid.

**Example Grammar where SLR fails:**
```
S → L = R | R
L → * R | id
R → L
```

**What this grammar means:**
- Statements are either assignments (`L = R`) or just expressions (`R`)
- Left-values (L) can be dereferenced pointers (`*R`) or identifiers
- Right-values (R) are just left-values (in this simple grammar)

**Problem State:**
```
State I₂: S → L•= R    (we might be in an assignment, expecting =)
          R → L•       (we might have completed R → L)
```

**SLR's Mistake:**
- FOLLOW(R) = {=, $} (R can be followed by = in `L = R`, or by $ at end)
- SLR says: Reduce `R → L` on BOTH = and $
- But in state I₂, if we see `=`, we should SHIFT, not reduce!
- Why? Because we came to I₂ via the `S → L•= R` path!

**The Root Cause:**
FOLLOW(R) includes `=` because in `S → L = R`, the R is followed by nothing (which means $). But the `=` in FOLLOW comes from a DIFFERENT production context, not this one!

**Conflict:** ACTION[I₂, =] = shift AND reduce → SLR conflict!

**SLR cannot handle all grammars that CLR/LALR can.**

---

## 5.10 CLR(1) Parsing (Canonical LR)

### 🎯 The Problem: We Need Context-Specific Lookahead

**The Issue:**
- SLR uses FOLLOW sets which mix up different contexts
- We need to know EXACTLY which lookahead is valid in EACH state
- The lookahead should be **propagated** from where the item originated

**The Solution - LR(1) Items:**
- Attach a **lookahead symbol** to each item
- This lookahead is the symbol that can follow THIS specific instance
- Lookaheads are computed precisely during closure

**CLR(1)** uses LR(1) items which include lookahead information.

### LR(1) Item

An LR(1) item has the form:
```
[A → α•β, a]
```

Where:
- A → αβ is a production
- a is a **lookahead terminal** (or $)

**🔑 Key Insight: When Does Lookahead Matter?**

The lookahead is only relevant when β is empty (reduction time).

```
[A → α•β, a]  where β ≠ ε:
    Lookahead 'a' is carried but not used yet.
    We're still in the middle of the production.

[A → α•, a]   where β = ε (dot at end):
    NOW the lookahead matters!
    Only reduce if current input = 'a'
```

**Intuition:**
- The lookahead says: "When you complete this production, the next input MUST be 'a'"
- If input is NOT 'a', don't reduce (even though pattern matches)
- This prevents the SLR mistake of reducing in wrong contexts

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

**🔑 Why FIRST(βa) for Lookahead?**

This is the crucial difference from LR(0) closure!

```
If we have item: [A → α•Bβ, a]

When B finishes (reduces), what comes after B in this context?
  → The string βa

So when we add items for B → γ, what's B's lookahead?
  → Whatever can START the string βa
  → That's FIRST(βa)!

If β can derive ε (empty), then 'a' itself might follow B
If β starts with terminal 't', then 't' follows B
```

**Example:**
```
[S → •CC, $]      Item we have

Adding items for C (C → cC | d):
  What follows C? The second C, then $
  FIRST(C$) = FIRST(C) = {c, d}
  
So we add: [C → •cC, c]  and  [C → •cC, d]
           [C → •d, c]   and  [C → •d, d]
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

### 🎯 The Problem: CLR Tables Are Too Big!

**The Issue:**
- CLR(1) is powerful and handles more grammars than SLR(1)
- But CLR(1) tables can be **10x larger** than SLR(1)!
- For real programming languages, this means megabytes of tables
- Memory was precious (especially historically), and big tables are slow to access

**Observation:**
Many CLR(1) states are **almost identical** - they differ only in lookahead!

```
Example from our grammar:
  State I₄: [C → d•, c/d]   ← Can reduce on c or d
  State I₇: [C → d•, $]     ← Can reduce on $ only
  
  Same "core" (C → d•), different lookaheads!
```

**The Solution - Merge Similar States:**
- Combine states that have the same **core** (LR(0) items)
- Union their lookahead sets
- Fewer states = smaller tables!

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

### 🎯 The Trade-off: Power vs Size

LALR merging can introduce **reduce-reduce conflicts** not present in CLR:

**Example:**
```
Grammar:
S → aAd | bBd | aBe | bAe
A → c
B → c
```

**What's happening:**
- After 'a...c', if 'd' follows, it came from `aAd`, so reduce to A
- After 'a...c', if 'e' follows, it came from `aBe`, so reduce to B
- After 'b...c', it's the OPPOSITE!

**CLR States (partial):**
```
State Iₓ: [A → c•, d], [B → c•, e]   (reached via 'a', then 'c')
State Iᵧ: [A → c•, e], [B → c•, d]   (reached via 'b', then 'c')
```
CLR can distinguish these - no conflict!

**After LALR Merging:**
```
Merged: [A → c•, d/e], [B → c•, d/e]
```

**Problem:** 
- On input 'd', should we reduce to A or B? 
- The lookahead says BOTH are valid! 
- → **Reduce-reduce conflict!**

**Why This Happens:**
- Merging combined two states with **opposite** lookahead relationships
- The distinction that CLR maintained (which path we took) is **lost**
- For most practical grammars, this rarely occurs

**Note:** LALR **never** introduces shift-reduce conflicts that weren't in CLR.

**Why not shift-reduce?**
- Shift actions depend on the CORE (what symbol comes next)
- Reduce actions depend on LOOKAHEAD
- Merging only affects lookahead, so only reduce-reduce can be introduced

### Practical LALR(1) Tools

LALR(1) is used by most parser generators:
- **YACC** (Yet Another Compiler Compiler)
- **Bison** (GNU version of YACC)
- **PLY** (Python Lex-Yacc)
- **CUP** (Java)

---

## 5.12 Comparison of All LR Methods

### 🎯 The Big Picture: Why Multiple Methods?

**The Trade-off Triangle:**
```
                    POWER
                      ▲
                     /|\
                    / | \
                   /  |  \
                  /   |   \
           CLR(1)    |    
                /     |     \
               /      |      \
              /       |       \
       LALR(1)       |        Small Tables
            /         |          \
           /          |           \
          /           |            \
    SLR(1)-----------+-------------LR(0)
         
    More Power = More Precision = Bigger Tables (usually)
```

**Each method makes a different trade-off:**

| Method | Philosophy |
|--------|------------|
| **LR(0)** | "Reduce whenever the pattern matches, ignore what comes next" |
| **SLR(1)** | "Reduce only if the next symbol could follow this non-terminal anywhere" |
| **LALR(1)** | "Reduce only if the next symbol could follow in this merged context" |
| **CLR(1)** | "Reduce only if the next symbol could follow in this exact context" |

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

### 🔍 Visual Example: Same State, Different Methods

**Consider a state with these items:**
```
State I: 
    E → E + T•        (completed - can reduce)
    E → E• + T        (expecting + - can shift)
```

**How each method handles ACTION[I, +]:**

```
Method    | Sees                           | ACTION[I, +]
─────────────────────────────────────────────────────────────
LR(0)     | E → E + T• → reduce on ALL     | CONFLICT!
          | E → E• + T → shift on +        | (shift AND reduce)
─────────────────────────────────────────────────────────────
SLR(1)    | E → E + T• → reduce on FOLLOW(E)| 
          | FOLLOW(E) = {+, ), $}          | CONFLICT!
          | E → E• + T → shift on +        | (+ is in FOLLOW)
─────────────────────────────────────────────────────────────
LALR/CLR  | [E → E + T•, )/$ ]             | NO CONFLICT!
          | Reduce only on ) or $, not +   | shift on +
          | [E → E• + T, ...]              | reduce on ), $
```

**Key Insight:**
- LR(0) and SLR(1) have conflicts because they're not precise enough
- CLR/LALR track that in THIS context, E is followed by `)` or `$`, not `+`
- The `+` after E appears in a DIFFERENT derivation context

### When to Use Which?

1. **LR(0)**: Only for learning concepts
2. **SLR(1)**: Simple grammars, academic exercises
3. **LALR(1)**: **Production compilers** (best balance)
4. **CLR(1)**: When LALR has conflicts (very rare)

### 🔑 Decision Flowchart

```
Start with your grammar
           │
           ▼
┌───────────────────────────┐
│ Is it LR(0)? (No conflicts │
│ with reduce on all terms)  │
└────────────┬──────────────┘
             │ No (conflicts exist)
             ▼
┌───────────────────────────┐
│ Try SLR(1) - use FOLLOW to │
│ restrict reduce actions    │
└────────────┬──────────────┘
             │ Still conflicts?
             ▼
┌───────────────────────────┐
│ Try LALR(1) - use precise  │
│ merged lookaheads          │
└────────────┬──────────────┘
             │ Still conflicts?
             ▼
┌───────────────────────────┐
│ Try CLR(1) - full LR(1)    │
│ with separate states       │
└────────────┬──────────────┘
             │ Still conflicts?
             ▼
┌───────────────────────────┐
│ Grammar is NOT LR(1)!      │
│ Rewrite the grammar or use │
│ conflict resolution rules  │
└───────────────────────────┘
```

### Real-World Usage

**Why LALR(1) Won:**
- Almost as powerful as CLR(1) in practice
- Same small table size as SLR(1)/LR(0)
- Handles 99%+ of real programming language grammars
- That's why YACC/Bison use LALR(1)!

---

## 5.13 Operator Precedence Parsing

### 🎯 The Problem: LR Parsing Seems Overkill for Expressions

**The Observation:**
- Expression grammars have a special structure: just operators and operands
- Precedence and associativity completely determine the parse
- Do we really need full LR machinery for this?

**The Solution - Operator Precedence Parsing:**
- A simpler, specialized bottom-up technique
- Only compares adjacent terminal symbols
- Decides shift/reduce based on precedence relations
- Much simpler tables (just terminal vs terminal)

**Operator Precedence Parsing** is a simple bottom-up technique for expressions with operators.

### Precedence Relations: The Core Idea

Between adjacent terminals a and b, we define:
- **a < b**: a "yields precedence" to b (shift b - it has higher precedence)
- **a = b**: a has same precedence as b (part of same construct, like `( = )`)
- **a > b**: a "takes precedence" over b (reduce - a's operator should be applied first)

**Intuition:**
```
+ < *   means: when we have + and see *, shift * (because * binds tighter)
* > +   means: when we have * and see +, reduce (apply * before +)
( < +   means: after (, any operator should be shifted (we're inside parens)
+ > )   means: before ), reduce (clear out pending operators)
```

### Precedence Table for Arithmetic

**How to Read This Table:**
- Row = what's on the stack (the left symbol)
- Column = what's in the input (the right symbol)  
- Entry = relationship between them

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

**Reading Examples:**
- `+ < *`: If + is on stack and * is input → shift (multiply has higher precedence)
- `* > +`: If * is on stack and + is input → reduce (do multiplication first)
- `( = )`: Parentheses are equal → they match (special handling)

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

### 🎯 The Problem: Building LR Tables By Hand Is Tedious and Error-Prone

**The Challenge:**
- Computing LR states manually takes hours (for real grammars, days)
- Easy to make mistakes in FIRST, FOLLOW, closure, GOTO calculations
- Any grammar change requires recomputing everything
- This is mechanical work - can a computer do it?

**The Solution - Parser Generators:**
- Write the grammar in a special notation
- The tool automatically:
  - Computes all LR states
  - Builds the parsing table
  - Generates the parser code
  - Reports any conflicts
- You focus on the grammar; the tool handles the mechanics

**YACC** (Yet Another Compiler Compiler) generates LALR(1) parsers from grammar specifications.

### Why YACC Uses LALR(1)

- **SLR(1)** would have too many conflicts for real grammars
- **CLR(1)** would generate tables too large
- **LALR(1)** is the sweet spot: handles most grammars, reasonable table size

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

**When YACC encounters conflicts, it uses these defaults:**

1. **Shift-reduce**: YACC prefers **shift**
   - *Why?* Shifting delays the decision, often resolves ambiguity
   - Example: Dangling else - shift associates else with nearest if

2. **Reduce-reduce**: YACC uses **first production** listed
   - *Why?* Arbitrary choice, but at least it's consistent
   - Better to rewrite grammar to avoid these!

3. **%left, %right, %nonassoc**: Specify associativity
   - `%left '+'` means + is left-associative: `a+b+c = (a+b)+c`
   - `%right '^'` means ^ is right-associative: `a^b^c = a^(b^c)`
   - `%nonassoc '<'` means no chaining: `a<b<c` is an error

4. **%prec**: Override default precedence
   - Used for unary minus: `-x` should bind tighter than binary minus
   - Example: `'-' expr %prec UMINUS` uses UMINUS precedence

**🔑 Precedence Declaration Order:**
```yacc
%left '+' '-'       // Lowest precedence (declared first)
%left '*' '/'       // Higher precedence
%right UMINUS       // Highest precedence (declared last)
```
Later declarations = higher precedence!

---

## 5.15 Error Recovery in LR Parsing

### 🎯 The Problem: Syntax Errors Shouldn't Crash the Parser

**The Challenge:**
- Real programs often have multiple syntax errors
- We want to report ALL errors, not just the first one
- Stopping at first error wastes programmer time
- But after an error, the parser state is corrupted - how do we continue?

**The Solution - Error Recovery:**
- Detect the error (parser has no valid action)
- Skip some input or pop some stack (get to a "safe" state)
- Continue parsing to find more errors
- May report spurious errors, but better than stopping

### Panic Mode Recovery

**The Idea:** Find a "synchronization point" and continue from there.

1. Pop states until finding one with GOTO on error symbol
2. Shift a special "error" token
3. Discard input until a "synchronizing" token is found (like `;` or `}`)
4. Continue parsing

**Why This Works:**
- Synchronizing tokens (`;`, `}`, `end`) mark statement/block boundaries
- After finding one, we're likely at a "clean" point in the grammar
- Parser can resume with fresh input

### YACC Error Recovery

```yacc
stmt : error ';'    { yyerror("bad statement"); }
     | /* other productions */
     ;
```

The `error` token matches any sequence of erroneous input.

---

## 5.16 Summary

### 🎯 Key Concepts and Why They Matter:

| Concept | What It Is | Why It Matters |
|---------|------------|----------------|
| **Bottom-Up Parsing** | Build tree from leaves to root | Handles more grammars than top-down, including left recursion |
| **Shift-Reduce** | Two operations: shift input, reduce patterns | Simple mechanism that powers all LR parsing |
| **Handle** | The correct substring to reduce | Finding the handle is THE central problem |
| **LR States** | Track parsing progress | Enable deterministic handle recognition |
| **LR Items** | Productions with dot marker | Represent "what we might be in the middle of" |
| **Lookahead** | Peeking at next input | Resolves conflicts about when to reduce |

### The Evolution of Solutions:

```
Problem: "How do we know when to reduce?"
    │
    ▼
LR(0): "Reduce whenever pattern matches"
    │ Too many conflicts!
    ▼
SLR(1): "Use FOLLOW sets to restrict reduce"
    │ FOLLOW is sometimes too broad
    ▼
LALR(1): "Track lookahead through states, then merge"
    │ Best practical balance!
    ▼
CLR(1): "Track exact lookahead, never merge"
    │ Most powerful but huge tables
```

### Core Takeaways:

1. **Bottom-Up Parsing** reduces input to start symbol (reverse rightmost derivation)

2. **Shift-Reduce** operations build the parse tree from leaves to root

3. **Handle** is the substring to reduce next - it's always at stack top

4. **LR Parsing** uses:
   - Stack with states and symbols
   - Parsing table (ACTION + GOTO)
   - Shift, reduce, accept, error actions

5. **LR Variants:**
   - **SLR(1)**: Uses FOLLOW sets, simple but may have spurious conflicts
   - **CLR(1)**: Uses precise lookahead in items, most powerful but huge tables
   - **LALR(1)**: Merges CLR states, best practical balance → **industry standard**

6. **Parser Generators** (YACC/Bison) automate LALR(1) parser construction

### 💡 Remember This:

> "**SLR looks at FOLLOW, CLR looks at context, LALR looks at merged context.**"
> 
> "**The difference is HOW PRECISELY we track when reduction is valid.**"

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
