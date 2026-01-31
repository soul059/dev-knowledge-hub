# Chapter 4: Top-Down Parsing

## 4.1 Introduction to Top-Down Parsing

**Top-Down Parsing** constructs the parse tree from the **root** (start symbol) to the **leaves** (terminals), attempting to find a leftmost derivation for the input string.

Problem: We need a method to decide which productions to apply while reading input.

Solution: Top-down parsing starts from the start symbol and expands productions to match the input.

### The Intuition

Imagine you're trying to determine if a sentence is grammatically correct:
1. Start with the sentence structure (S → NP VP)
2. Try to match the structure with the actual words
3. If a mismatch occurs, try another production (backtrack)
4. Success if all words match and structure is complete

### Top-Down Parsing Methods

```
                Top-Down Parsing
                       │
         ┌─────────────┴─────────────┐
         │                           │
    Backtracking                 Predictive
    (Brute Force)               (No Backtracking)
         │                           │
    Recursive                   ┌────┴────┐
    Descent                     │         │
    with backtracking    Recursive    Table-Driven
                        Descent       LL(1) Parser
                        (LL(1))
```

---

## 4.2 Recursive Descent Parsing

**Recursive Descent Parsing** is a top-down method where each non-terminal in the grammar has a corresponding procedure (function) that recognizes that non-terminal.

Problem: We need a simple, direct implementation of a top-down parser.

Solution: Map each non-terminal to a function that consumes input according to grammar rules.

### The Basic Idea

For each grammar rule, create a function:
- For `A → α₁ | α₂ | ... | αₙ`
- Function A() tries each alternative in order
- Uses backtracking if needed

### Example Grammar

```
S → cAd
A → ab | a
```

### Recursive Descent Parser (with backtracking)

```c
// Global variables
char *input;
int pos = 0;

char peek() { return input[pos]; }
char advance() { return input[pos++]; }
void backtrack(int saved_pos) { pos = saved_pos; }

// Parser functions
bool S() {
    int saved = pos;
    
    if (advance() == 'c' && A() && advance() == 'd') {
        return true;
    }
    
    backtrack(saved);
    return false;
}

bool A() {
    int saved = pos;
    
    // Try A → ab first
    if (advance() == 'a' && advance() == 'b') {
        return true;
    }
    
    backtrack(saved);
    
    // Try A → a
    if (advance() == 'a') {
        return true;
    }
    
    backtrack(saved);
    return false;
}

int main() {
    input = "cad";
    if (S() && input[pos] == '\0') {
        printf("Accepted!\n");
    } else {
        printf("Rejected!\n");
    }
}
```

### The Problem with Backtracking

For input `"cad"`:
1. S matches 'c', calls A
2. A tries `ab`, matches 'a', but 'b' ≠ 'd' (fails)
3. A backtracks, tries `a`, matches
4. S checks for 'd', matches
5. **Accepted!**

**Issue**: Backtracking is **inefficient** and can be exponential in worst case.

Problem: Backtracking wastes time by re-reading the same input.

Solution: Predictive parsing removes backtracking by using lookahead and FIRST/FOLLOW sets.

---

## 4.3 Predictive Parsing (No Backtracking)

**Predictive Parsing** eliminates backtracking by looking at the next input symbol(s) to decide which production to use.

### Requirements

1. Grammar must be **left-factored**
2. Grammar must be **free of left recursion**
3. Grammar must be **LL(1)**

### What is LL(1)?

```
L - Left-to-right scanning of input
L - Leftmost derivation
1 - One symbol lookahead
```

An LL(1) grammar allows the parser to make a unique choice of production by examining only **one** input symbol ahead.

---

## 4.4 FIRST and FOLLOW Sets

To build a predictive parser, we need two important sets for each non-terminal.

### FIRST Set

**FIRST(α)** is the set of terminals that can begin strings derived from α.

**Rules for computing FIRST(X):**

1. If X is a terminal: **FIRST(X) = {X}**

2. If X is a non-terminal with production X → ε:
   - Add ε to FIRST(X)

3. If X is a non-terminal with production X → Y₁Y₂...Yₖ:
   - Add FIRST(Y₁) - {ε} to FIRST(X)
   - If ε ∈ FIRST(Y₁), add FIRST(Y₂) - {ε} to FIRST(X)
   - Continue until you find a Yᵢ where ε ∉ FIRST(Yᵢ)
   - If ε ∈ FIRST(Yᵢ) for all i, add ε to FIRST(X)

### FOLLOW Set

**FOLLOW(A)** is the set of terminals that can appear immediately to the right of A in some sentential form.

**Rules for computing FOLLOW(A):**

1. If A is the start symbol: **Add $ to FOLLOW(A)** (end marker)

2. If there is a production B → αAβ:
   - Add FIRST(β) - {ε} to FOLLOW(A)

3. If there is a production B → αA, or B → αAβ where ε ∈ FIRST(β):
   - Add FOLLOW(B) to FOLLOW(A)

### Complete Example

**Grammar:**
```
E  → T E'
E' → + T E' | ε
T  → F T'
T' → * F T' | ε
F  → ( E ) | id
```

**Step 1: Compute FIRST sets**

| Non-terminal | FIRST |
|--------------|-------|
| F | { (, id } |
| T' | { *, ε } |
| T | { (, id } |
| E' | { +, ε } |
| E | { (, id } |

**Detailed calculation:**

```
FIRST(F):
- F → ( E ) : Add '(' 
- F → id   : Add 'id'
- FIRST(F) = { (, id }

FIRST(T'):
- T' → * F T' : Add '*'
- T' → ε      : Add 'ε'
- FIRST(T') = { *, ε }

FIRST(T):
- T → F T'
- FIRST(F) = { (, id } (no ε)
- FIRST(T) = { (, id }

FIRST(E'):
- E' → + T E' : Add '+'
- E' → ε      : Add 'ε'
- FIRST(E') = { +, ε }

FIRST(E):
- E → T E'
- FIRST(T) = { (, id } (no ε)
- FIRST(E) = { (, id }
```

**Step 2: Compute FOLLOW sets**

| Non-terminal | FOLLOW |
|--------------|--------|
| E | { ), $ } |
| E' | { ), $ } |
| T | { +, ), $ } |
| T' | { +, ), $ } |
| F | { *, +, ), $ } |

**Detailed calculation:**

```
FOLLOW(E):
- E is start symbol: Add '$'
- F → ( E ) : Add ')'
- FOLLOW(E) = { ), $ }

FOLLOW(E'):
- E → T E' : Add FOLLOW(E) = { ), $ }
- E' → + T E' : Add FOLLOW(E') (already included)
- FOLLOW(E') = { ), $ }

FOLLOW(T):
- E → T E' : FIRST(E') - {ε} = {+}, Add '+'
           : ε ∈ FIRST(E'), Add FOLLOW(E) = { ), $ }
- E' → + T E' : Same as above
- FOLLOW(T) = { +, ), $ }

FOLLOW(T'):
- T → F T' : Add FOLLOW(T) = { +, ), $ }
- FOLLOW(T') = { +, ), $ }

FOLLOW(F):
- T → F T' : FIRST(T') - {ε} = {*}, Add '*'
           : ε ∈ FIRST(T'), Add FOLLOW(T) = { +, ), $ }
- FOLLOW(F) = { *, +, ), $ }
```

---

## 4.5 LL(1) Parsing Table Construction

### Algorithm to Build LL(1) Parsing Table

For each production A → α:

1. For each terminal **a** in FIRST(α):
   - Add A → α to M[A, a]

2. If ε ∈ FIRST(α):
   - For each terminal **b** in FOLLOW(A):
     - Add A → α to M[A, b]
   - If $ ∈ FOLLOW(A):
     - Add A → α to M[A, $]

### LL(1) Parsing Table for Expression Grammar

```
┌──────┬────────────────┬────────────────┬────────────────┬──────────┬──────────┬──────────┐
│      │       id       │       +        │       *        │    (     │    )     │    $     │
├──────┼────────────────┼────────────────┼────────────────┼──────────┼──────────┼──────────┤
│  E   │   E → T E'     │                │                │ E → T E' │          │          │
├──────┼────────────────┼────────────────┼────────────────┼──────────┼──────────┼──────────┤
│  E'  │                │  E' → + T E'   │                │          │ E' → ε   │ E' → ε   │
├──────┼────────────────┼────────────────┼────────────────┼──────────┼──────────┼──────────┤
│  T   │   T → F T'     │                │                │ T → F T' │          │          │
├──────┼────────────────┼────────────────┼────────────────┼──────────┼──────────┼──────────┤
│  T'  │                │   T' → ε       │  T' → * F T'   │          │ T' → ε   │ T' → ε   │
├──────┼────────────────┼────────────────┼────────────────┼──────────┼──────────┼──────────┤
│  F   │   F → id       │                │                │ F → (E)  │          │          │
└──────┴────────────────┴────────────────┴────────────────┴──────────┴──────────┴──────────┘
```

### Checking for LL(1)

A grammar is LL(1) if and only if the parsing table has **at most one production** in each cell.

**If multiple entries exist in a cell → Grammar is NOT LL(1)**

---

## 4.6 Table-Driven LL(1) Parser

### Components

1. **Input buffer**: Holds the input string with $ at the end
2. **Stack**: Holds grammar symbols with $ at the bottom
3. **Parsing table**: M[A, a] gives production to use
4. **Output**: Sequence of productions used

### LL(1) Parsing Algorithm

```
push $ onto stack
push start symbol onto stack
point ip to first symbol of input

repeat:
    let X = top of stack, a = current input symbol
    
    if X is a terminal:
        if X == a:
            pop X from stack
            advance ip to next input symbol
        else:
            error()
    
    else if X is a non-terminal:
        if M[X, a] == X → Y₁Y₂...Yₖ:
            pop X from stack
            push Yₖ, Yₖ₋₁, ..., Y₁ onto stack (Y₁ on top)
            output the production X → Y₁Y₂...Yₖ
        else:
            error()
    
until X == $  // Both stack and input are empty
```

### Parsing Example

**Input**: `id + id * id $`

**Trace:**

| Stack | Input | Action |
|-------|-------|--------|
| $ E | id + id * id $ | E → T E' |
| $ E' T | id + id * id $ | T → F T' |
| $ E' T' F | id + id * id $ | F → id |
| $ E' T' id | id + id * id $ | Match id |
| $ E' T' | + id * id $ | T' → ε |
| $ E' | + id * id $ | E' → + T E' |
| $ E' T + | + id * id $ | Match + |
| $ E' T | id * id $ | T → F T' |
| $ E' T' F | id * id $ | F → id |
| $ E' T' id | id * id $ | Match id |
| $ E' T' | * id $ | T' → * F T' |
| $ E' T' F * | * id $ | Match * |
| $ E' T' F | id $ | F → id |
| $ E' T' id | id $ | Match id |
| $ E' T' | $ | T' → ε |
| $ E' | $ | E' → ε |
| $ | $ | **Accept!** |

---

## 4.7 Recursive Descent Parser for LL(1) Grammar

Each non-terminal becomes a function that:
- Looks at current input (lookahead)
- Chooses the appropriate production
- Calls functions for right-hand side symbols

### Predictive Parser for Expression Grammar

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

char lookahead;

void error(char *msg) {
    printf("Syntax Error: %s\n", msg);
    exit(1);
}

void match(char expected) {
    if (lookahead == expected) {
        lookahead = getchar();
    } else {
        char msg[50];
        sprintf(msg, "Expected '%c', found '%c'", expected, lookahead);
        error(msg);
    }
}

// Forward declarations
void E();
void E_prime();
void T();
void T_prime();
void F();

// E → T E'
void E() {
    T();
    E_prime();
}

// E' → + T E' | ε
void E_prime() {
    if (lookahead == '+') {
        match('+');
        T();
        E_prime();
    }
    // else: E' → ε (do nothing)
}

// T → F T'
void T() {
    F();
    T_prime();
}

// T' → * F T' | ε
void T_prime() {
    if (lookahead == '*') {
        match('*');
        F();
        T_prime();
    }
    // else: T' → ε (do nothing)
}

// F → ( E ) | id
void F() {
    if (lookahead == '(') {
        match('(');
        E();
        match(')');
    } else if (isalpha(lookahead)) {
        match(lookahead);  // Match identifier
    } else {
        error("Expected '(' or identifier");
    }
}

int main() {
    printf("Enter expression: ");
    lookahead = getchar();
    
    E();
    
    if (lookahead == '\n' || lookahead == EOF) {
        printf("Parsing successful!\n");
    } else {
        error("Unexpected characters after expression");
    }
    
    return 0;
}
```

### How the Parser Works

For input `a + b * c`:

```
1. E() called
   └── T() called
       └── F() called: matches 'a'
       └── T_prime(): lookahead='+', returns (ε)
   └── E_prime(): lookahead='+', matches
       └── T() called
           └── F() called: matches 'b'
           └── T_prime(): lookahead='*', matches
               └── F() called: matches 'c'
               └── T_prime(): lookahead='\n', returns
           (returns)
       └── E_prime(): lookahead='\n', returns
   (returns)
2. Success!
```

---

## 4.8 Non-LL(1) Grammars

Some grammars cannot be parsed with LL(1) parsing.

### Common Issues

#### 1. Left Recursion
```
A → Aα | β
```
**Problem**: FIRST(Aα) and FIRST(β) overlap

**Solution**: Eliminate left recursion

#### 2. Common Prefixes
```
A → αβ₁ | αβ₂
```
**Problem**: Can't distinguish between alternatives

**Solution**: Left factoring

#### 3. FIRST-FOLLOW Conflict
```
A → α | ε

Where FIRST(α) ∩ FOLLOW(A) ≠ ∅
```
**Problem**: For symbols in the intersection, can't decide between α and ε

**Solution**: May need grammar rewriting or different parsing method

### Example of Non-LL(1) Grammar

```
S → i E t S S' | a
S' → e S | ε
E → b
```

For input starting with `i`:
- S → i E t S S'
- When S' is on stack and `e` is input:
  - S' → e S (FIRST contains e)
  - S' → ε (FOLLOW(S') contains e because of S → i E t S S')
  - **Conflict!**

---

## 4.9 Error Recovery in Predictive Parsing

### Types of Errors

1. **Terminal on stack doesn't match input**
2. **M[A, a] is empty** (no production to use)
3. **Input ends before stack is empty**

### Panic Mode Recovery

Skip input symbols until a **synchronizing token** is found.

**Synchronizing tokens** for A:
- Tokens in FOLLOW(A)
- Tokens that begin statements (`;`, `}`)
- Keywords that start new constructs

### Error Recovery Algorithm

```
When M[A, a] is empty:
    if a is in FOLLOW(A):
        pop A from stack (assume A derives ε)
    else:
        skip input symbol a
        (try again with next symbol)

When terminal on stack doesn't match input:
    pop the terminal from stack
    (continue with next stack symbol)
```

### Phrase-Level Recovery

Insert missing symbols or replace erroneous symbols locally.

```
For M[A, a] empty where a should be some expected token:
    if a is 'likely' a typo for expected token:
        treat a as expected token
        issue warning
```

---

## 4.10 Constructing Parse Trees

As we parse, we can build a parse tree:

### Tree Building in Recursive Descent

```c
typedef struct Node {
    char *label;
    struct Node **children;
    int num_children;
} Node;

Node* create_node(char *label) {
    Node *n = malloc(sizeof(Node));
    n->label = label;
    n->children = NULL;
    n->num_children = 0;
    return n;
}

void add_child(Node *parent, Node *child) {
    parent->num_children++;
    parent->children = realloc(parent->children, 
                               parent->num_children * sizeof(Node*));
    parent->children[parent->num_children - 1] = child;
}

// Modified E function to build tree
Node* E() {
    Node *node = create_node("E");
    add_child(node, T());
    add_child(node, E_prime());
    return node;
}
```

---

## 4.11 LL(k) Parsing

**LL(k) parsing** uses **k** symbols of lookahead.

### When is LL(k) Needed?

When 1 symbol isn't enough to distinguish alternatives:

```
S → aAb | aBb
A → cd
B → ce
```

For input `aceḃ`:
- After seeing 'a', can't choose between A and B
- Need to see 'c' and then 'd' or 'e' (2 symbols after 'a')
- This is LL(3)

### Trade-offs

| Aspect | LL(1) | LL(k) |
|--------|-------|-------|
| Table size | O(n × t) | O(n × t^k) |
| Grammar restrictions | Strict | More lenient |
| Parsing speed | Fast | Slower |
| Implementation | Simple | Complex |

---

## 4.12 Summary

### Key Concepts:

1. **Top-Down Parsing** builds parse tree from root to leaves

2. **Recursive Descent** implements one function per non-terminal
   - May require backtracking without proper grammar

3. **FIRST(α)**: Terminals that can start strings derived from α

4. **FOLLOW(A)**: Terminals that can appear after A

5. **LL(1) Condition**: For each production, parser can decide using 1 lookahead
   - FIRST sets of alternatives must be disjoint
   - If ε ∈ FIRST(α), then FIRST(α) ∩ FOLLOW(A) = ∅

6. **LL(1) Parsing Table**: M[A, a] tells which production to use

7. **Prerequisites for LL(1)**:
   - No left recursion
   - Left factored grammar
   - No FIRST-FOLLOW conflicts

---

## 🔍 Practice Questions

1. Compute FIRST and FOLLOW sets for:
   ```
   S → ABc
   A → a | ε
   B → b | ε
   ```

2. Construct LL(1) parsing table for:
   ```
   S → aBa
   B → bB | ε
   ```

3. Write a recursive descent parser for balanced parentheses: `S → (S)S | ε`

4. Determine if the following grammar is LL(1):
   ```
   S → Ab | Bc
   A → a | ε
   B → a | ε
   ```

5. Trace the LL(1) parser for input `(a+b)*c` using the expression grammar.

---

*Next Chapter: [Bottom-Up Parsing](05-Bottom-Up-Parsing.md)*
