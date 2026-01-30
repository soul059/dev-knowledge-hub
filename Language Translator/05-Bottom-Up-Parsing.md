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

## 5.8 SLR(1) Parsing

**SLR(1)** (Simple LR) uses LR(0) items plus FOLLOW sets to resolve conflicts.

### SLR Parsing Table Construction

**For each state I in the canonical collection:**

1. If A → α•aβ is in I and GOTO(I, a) = Iⱼ:
   - Set ACTION[I, a] = "shift j"

2. If A → α• is in I (and A ≠ S'):
   - For all a in FOLLOW(A):
     - Set ACTION[I, a] = "reduce A → α"

3. If S' → S• is in I:
   - Set ACTION[I, $] = "accept"

4. If GOTO(I, A) = Iⱼ for non-terminal A:
   - Set GOTO[I, A] = j

### SLR(1) Table Example

For the grammar S' → S, S → CC, C → cC | d:

**FOLLOW sets:**
- FOLLOW(S) = {$}
- FOLLOW(C) = {c, d, $}

**ACTION and GOTO Table:**

```
┌───────┬─────────────────────────┬───────────┐
│ State │        ACTION           │   GOTO    │
│       ├────────┬────────┬───────┼─────┬─────┤
│       │   c    │   d    │   $   │  S  │  C  │
├───────┼────────┼────────┼───────┼─────┼─────┤
│   0   │   s3   │   s4   │       │  1  │  2  │
│   1   │        │        │  acc  │     │     │
│   2   │   s3   │   s4   │       │     │  5  │
│   3   │   s3   │   s4   │       │     │  6  │
│   4   │   r3   │   r3   │  r3   │     │     │
│   5   │        │        │  r1   │     │     │
│   6   │   r2   │   r2   │  r2   │     │     │
└───────┴────────┴────────┴───────┴─────┴─────┘

Productions: (1) S → CC  (2) C → cC  (3) C → d
```

### Limitations of SLR(1)

SLR uses FOLLOW sets, which may be too imprecise:

```
Grammar:
S → L = R | R
L → * R | id
R → L

For state with: R → L•

SLR says: Reduce on all of FOLLOW(R) = {=, $}

But when we have: id = ...
- After reducing id to L, we're in state with L = R•L
- The = is coming next, but we shouldn't reduce R → L here
- We should shift the =
```

**SLR cannot handle all grammars that CLR/LALR can.**

---

## 5.9 CLR(1) Parsing (Canonical LR)

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

### CLR(1) Example

**Grammar:**
```
S' → S
S → CC
C → cC | d
```

**State I₀:**
```
[S' → •S, $]
[S  → •CC, $]
[C  → •cC, c/d]   (FIRST(C$) = {c, d})
[C  → •d, c/d]
```

**State I₁:** GOTO(I₀, S)
```
[S' → S•, $]
```

**State I₂:** GOTO(I₀, C)
```
[S → C•C, $]
[C → •cC, $]     (FIRST($) = {$})
[C → •d, $]
```

Notice: Different lookaheads create different states!

### CLR(1) Table Construction

Same as SLR, but reduce action uses lookahead from item:

For item [A → α•, a]:
- Set ACTION[I, a] = "reduce A → α"
- Only for the specific lookahead a, not all of FOLLOW(A)

### CLR Power vs Table Size

**CLR(1)** can parse more grammars than SLR(1), but tables can be **very large** because states are split based on lookahead.

---

## 5.10 LALR(1) Parsing

**LALR(1)** (Look-Ahead LR) combines the power of CLR(1) with the table size of SLR(1).

### The Key Insight

Many CLR(1) states differ only in lookahead symbols but have the same **core** (LR(0) part).

### Merging States

States with the same core are merged:
- Combine their lookahead sets
- Results in fewer states

**Example:**
```
CLR State I₄: [C → d•, c/d]
CLR State I₇: [C → d•, $]

LALR merges to: [C → d•, c/d/$]
```

### LALR Construction (from CLR)

1. Build CLR(1) canonical collection
2. Find states with same cores
3. Merge such states (union lookaheads)
4. Build parsing table from merged states

### LALR vs SLR vs CLR

```
Grammar Class Hierarchy:

    SLR(1) ⊂ LALR(1) ⊂ LR(1)
```

| Aspect | SLR(1) | LALR(1) | CLR(1) |
|--------|--------|---------|--------|
| Table Size | Small | Medium | Large |
| Power | Least | More | Most |
| Conflicts | Most | Some | Fewest |
| Tools | - | YACC, Bison | - |

### LALR May Introduce Conflicts

Merging states can introduce **reduce-reduce conflicts** that weren't in CLR.

However, LALR never introduces **shift-reduce conflicts** that weren't in CLR.

---

## 5.11 Operator Precedence Parsing

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

## 5.12 YACC/Bison - Parser Generators

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

## 5.13 Error Recovery in LR Parsing

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

## 5.14 Summary

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
