# Chapter 4: Context-Free Grammars

## 4.1 Introduction

**Context-Free Grammars (CFGs)** are more powerful than regular expressions. They can describe nested structures like matching parentheses, which regular languages cannot express.

### Why CFGs?

| Application | Example |
|-------------|---------|
| Programming languages | Nested blocks, expressions |
| Natural language | Sentence structure |
| Markup languages | HTML, XML (nested tags) |
| Document formats | LaTeX, JSON |
| Compilers | Syntax specification |

---

## 4.2 Formal Definition

### Context-Free Grammar

A **CFG** is a 4-tuple G = (V, Σ, P, S) where:

| Component | Meaning |
|-----------|---------|
| V | Finite set of **variables** (non-terminals) |
| Σ | Finite set of **terminals** (alphabet), V ∩ Σ = ∅ |
| P | Finite set of **productions** (rules) |
| S | **Start symbol** (S ∈ V) |

### Production Rules

Productions have the form:
```
A → α

where:
- A ∈ V (a single variable on the left)
- α ∈ (V ∪ Σ)* (string of variables and terminals on the right)
```

The key feature: **Left side has exactly one variable** (hence "context-free" - replacement doesn't depend on surrounding context).

### Example: Grammar for aⁿbⁿ

```
G = ({S}, {a, b}, P, S)

Productions P:
S → aSb
S → ε

This generates: {ε, ab, aabb, aaabbb, ...} = {aⁿbⁿ | n ≥ 0}
```

### Notational Conventions

```
Variables: A, B, C, S (uppercase letters)
Terminals: a, b, c, 0, 1 (lowercase letters, digits)
Strings of terminals: u, v, w, x, y, z
Strings of symbols: α, β, γ (Greek letters)
Start symbol: Usually S
```

---

## 4.3 Derivations

### What is a Derivation?

A **derivation** is a sequence of rule applications that generates a string from the start symbol.

**Notation:**
```
⇒   : One step derivation
⇒*  : Zero or more steps
⇒⁺  : One or more steps
```

### Example Derivation

```
Grammar:
S → aSb | ε

Derivation of "aabb":
S ⇒ aSb      (using S → aSb)
  ⇒ aaSbb    (using S → aSb)
  ⇒ aabb     (using S → ε)

We write: S ⇒* aabb
```

### Leftmost and Rightmost Derivations

**Leftmost derivation (⇒_lm):** Always replace the leftmost variable.

**Rightmost derivation (⇒_rm):** Always replace the rightmost variable.

```
Grammar:
E → E + E | E * E | (E) | id

Leftmost derivation of "id + id * id":
E ⇒_lm E + E
  ⇒_lm id + E
  ⇒_lm id + E * E
  ⇒_lm id + id * E
  ⇒_lm id + id * id

Rightmost derivation:
E ⇒_rm E + E
  ⇒_rm E + E * E
  ⇒_rm E + E * id
  ⇒_rm E + id * id
  ⇒_rm id + id * id
```

---

## 4.4 Parse Trees

### What is a Parse Tree?

A **parse tree** (derivation tree) visually represents a derivation.

**Structure:**
- Root: Start symbol
- Internal nodes: Variables
- Leaves: Terminals (or ε)
- Children: Right side of applied production

### Example: Parse Tree for aabb

```
Grammar: S → aSb | ε

           S
          /|\
         / | \
        a  S  b
          /|\
         / | \
        a  S  b
           |
           ε

Reading leaves left-to-right: a a ε b b = aabb
```

### Parse Tree for id + id * id

```
Grammar: E → E + E | E * E | id

Tree 1 (+ at root):           Tree 2 (* at root):
        E                            E
       /|\                          /|\
      E + E                        E * E
      |  /|\                      /|\   |
     id E * E                    E + E  id
        |   |                    |   |
       id  id                   id  id

Different trees → Different interpretations!
Tree 1: (id + (id * id)) - multiplication first
Tree 2: ((id + id) * id) - addition first
```

### Yield of a Parse Tree

The **yield** is the string obtained by reading leaves left-to-right.

---

## 4.5 Ambiguity

### Ambiguous Grammar

A grammar is **ambiguous** if there exists at least one string with two or more different:
- Parse trees, OR
- Leftmost derivations, OR
- Rightmost derivations

### Example: Ambiguous Expression Grammar

```
E → E + E | E * E | (E) | id

String: id + id * id

Two leftmost derivations:
1. E ⇒ E + E ⇒ id + E ⇒ id + E * E ⇒ id + id * E ⇒ id + id * id
2. E ⇒ E * E ⇒ E + E * E ⇒ id + E * E ⇒ id + id * E ⇒ id + id * id

Two different parse trees exist → Grammar is AMBIGUOUS
```

### Problems with Ambiguity

1. **In compilers:** Different interpretations mean different results
2. **In parsing:** Which parse tree to choose?
3. **In semantics:** Undefined meaning

### Removing Ambiguity

**Strategy 1: Enforce precedence using grammar levels**

```
Original (ambiguous):
E → E + E | E * E | (E) | id

Unambiguous (enforcing * before +):
E → E + T | T
T → T * F | F
F → (E) | id

Now id + id * id has only one parse tree:
        E
       /|\
      E + T
      |  /|\
      T T * F
      |  |   |
      F  F  id
      |  |
     id id

This correctly groups as: id + (id * id)
```

**Strategy 2: Enforce associativity**

```
For left associativity (a - b - c = (a-b) - c):
E → E - T | T        (Left recursive → left associative)

For right associativity (a ^ b ^ c = a ^ (b^c)):
E → T ^ E | T        (Right recursive → right associative)
```

### Inherently Ambiguous Languages

Some CFLs have **no unambiguous grammar** - called **inherently ambiguous**.

Example: L = {aⁿbⁿcᵐdᵐ | n,m ≥ 1} ∪ {aⁿbᵐcᵐdⁿ | n,m ≥ 1}

---

## 4.6 Language of a Grammar

### Definition

The language of grammar G is:
```
L(G) = {w ∈ Σ* | S ⇒* w}
```

All strings of terminals derivable from the start symbol.

### Context-Free Language (CFL)

A language L is **context-free** if there exists a CFG G such that L = L(G).

---

## 4.7 Normal Forms

### Why Normal Forms?

Normal forms simplify:
- Parsing algorithms
- Proofs about CFLs
- Grammar analysis

### Chomsky Normal Form (CNF)

Every production has the form:
```
A → BC   (two variables)
A → a    (single terminal)
S → ε    (only if ε ∈ L, and S never on right side)
```

### Converting to CNF

**Step 1: Eliminate ε-productions** (A → ε)

```
1. Find all nullable variables (can derive ε)
2. For each production A → α with nullable symbols,
   add versions with those symbols removed
3. Remove all ε-productions (except S → ε if needed)

Example:
S → AB
A → a | ε
B → b | ε

Nullable: A, B, S
After elimination:
S → AB | A | B | ε
A → a
B → b
```

**Step 2: Eliminate unit productions** (A → B)

```
1. Find all unit pairs (A, B) where A ⇒* B
2. For each unit pair (A, B) and non-unit production B → α,
   add A → α
3. Remove all unit productions

Example:
S → A | Bb
A → B | a
B → b

Unit pairs: (S,A), (S,B), (A,B), (S,S), (A,A), (B,B)
After elimination:
S → a | b | Bb
A → b | a
B → b
```

**Step 3: Convert to proper form**

```
1. Replace terminals in long productions:
   A → bCD  becomes  A → B'CD, B' → b

2. Break long productions:
   A → BCD  becomes  A → BD', D' → CD
```

### Example: Full CNF Conversion

```
Original:
S → aSb | ε

Step 1 (ε-elimination):
S → aSb | ab    (S is nullable, so add ab)

Step 2 (no unit productions)

Step 3 (CNF form):
S → AC | AB
A → a
B → b
C → SB

Verification: generates same language {aⁿbⁿ | n ≥ 1}
(ε case needs S → ε and S doesn't appear on right side)
```

### Greibach Normal Form (GNF)

Every production has the form:
```
A → aα   where a ∈ Σ, α ∈ V*

(Terminal first, followed by zero or more variables)
```

**Benefits:**
- Each step consumes exactly one terminal
- Useful for PDA construction
- No left recursion

---

## 4.8 Simplification of CFG

### Useless Symbols

A symbol X is **useful** if it appears in some derivation S ⇒* αXβ ⇒* w

A symbol is **useless** if it's not useful. Remove them!

### Removing Useless Symbols

**Step 1: Find generating symbols** (can derive terminal string)
```
1. All terminals generate (trivially)
2. If A → α and all symbols in α generate, then A generates
3. Repeat until no change
```

**Step 2: Find reachable symbols** (reachable from S)
```
1. S is reachable
2. If A is reachable and A → α, all symbols in α are reachable
3. Repeat until no change
```

**Step 3:** Keep only symbols that are both generating AND reachable.

### Example: Remove Useless Symbols

```
S → AB | a
A → a
B → b | C
C → C

Step 1 - Generating:
a, b generate (terminals)
A generates (A → a)
B generates (B → b)
S generates (S → a)
C does NOT generate (only C → C, infinite loop)

Remove C:
S → AB | a
A → a
B → b

Step 2 - Reachable from S:
S reachable
A, B reachable (from S → AB)
a reachable (from A → a, S → a)
b reachable (from B → b)

All remaining symbols reachable.
```

---

## 4.9 Closure Properties of CFLs

### CFLs are Closed Under

| Operation | Result |
|-----------|--------|
| Union | CFL |
| Concatenation | CFL |
| Kleene Star | CFL |
| Reversal | CFL |
| Homomorphism | CFL |
| Inverse Homomorphism | CFL |
| Intersection with Regular | CFL |

### CFLs are NOT Closed Under

| Operation | Result |
|-----------|--------|
| Intersection | NOT CFL |
| Complement | NOT CFL |
| Difference | NOT CFL |

### Proofs

**Union:** If L₁ = L(G₁) and L₂ = L(G₂):
```
New grammar G:
- Rename variables so they're disjoint
- Add new start symbol S
- Add: S → S₁ | S₂
- L(G) = L₁ ∪ L₂
```

**Concatenation:**
```
S → S₁S₂
```

**Kleene Star:**
```
S → SS₁ | ε
```

**Not closed under intersection:**
```
L₁ = {aⁿbⁿcᵐ | n,m ≥ 0}  - CFL
L₂ = {aᵐbⁿcⁿ | n,m ≥ 0}  - CFL
L₁ ∩ L₂ = {aⁿbⁿcⁿ | n ≥ 0}  - NOT CFL!
```

---

## 4.10 Pumping Lemma for CFLs

### Statement

If L is a context-free language, there exists a pumping length p such that:

For every string w ∈ L with |w| ≥ p:
```
w can be split as w = uvxyz where:
1. |vy| > 0         (v and y not both empty)
2. |vxy| ≤ p        (middle portion bounded)
3. uvⁱxyⁱz ∈ L      for all i ≥ 0
```

### Visual Understanding

```
Parse tree for long string must have repeated variable:

          S                    S
         /|\                  /|\
        / | \                / | \
       /  |  \              /  |  \
      A   ...  ...         A   ...  ...
     /|\                  /|\
    / | \                / | \
   u  A  y    →→→       u  A  y
     /|\                  /A\
    v x z                v A y
                          |||
                         v x z

Repeating variable A allows "pumping" v and y together.
```

### Using Pumping Lemma for CFLs

**Prove L = {aⁿbⁿcⁿ | n ≥ 0} is not context-free:**

```
1. Assume L is CFL with pumping length p
2. Choose w = aᵖbᵖcᵖ, |w| = 3p ≥ p
3. Let w = uvxyz with |vy| > 0, |vxy| ≤ p

4. Since |vxy| ≤ p, vxy cannot contain all three symbols
   - Case 1: vxy contains only a's and b's
   - Case 2: vxy contains only b's and c's
   
5. Pump up (i = 2): uv²xy²z
   - Some symbols increase, others don't
   - Counts become unequal
   
6. Contradiction! L is not context-free.
```

### More Non-Context-Free Languages

| Language | Why not CFL |
|----------|-------------|
| {aⁿbⁿcⁿ} | Need to match three counts |
| {ww \| w ∈ Σ*} | Need to compare two halves |
| {aⁿbⁿcⁿdⁿ} | Four matching counts |
| {a^(2ⁿ)} | Exponential growth |

---

## 4.11 Decision Properties of CFLs

### Decidable Problems

| Problem | Decidable? | Method |
|---------|------------|--------|
| Membership | Yes | CYK algorithm, Earley parser |
| Emptiness | Yes | Check if S generates anything |
| Finiteness | Yes | Check for cycles that reach terminals |

### Undecidable Problems

| Problem | Decidable? |
|---------|------------|
| Equivalence (L₁ = L₂?) | No |
| Ambiguity | No |
| Is L(G) = Σ*? | No |
| Is L₁ ∩ L₂ = ∅? | No |
| Is L regular? | No |
| Is L₁ ⊆ L₂? | No |

### CYK Algorithm (Membership Testing)

**Cocke-Younger-Kasami** algorithm tests if w ∈ L(G) in O(n³) time.

**Requirements:** Grammar must be in CNF.

**Algorithm:**
```
For string w = a₁a₂...aₙ:

1. Build table X[i,j] = {variables that derive aᵢ...aⱼ}

2. Fill diagonal (single characters):
   X[i,i] = {A | A → aᵢ ∈ P}

3. Fill remaining by increasing span:
   X[i,j] = {A | A → BC ∈ P, B ∈ X[i,k], C ∈ X[k+1,j]}
            for some k with i ≤ k < j

4. Accept if S ∈ X[1,n]
```

---

## 4.12 Comparison: Regular vs Context-Free

| Feature | Regular | Context-Free |
|---------|---------|--------------|
| Memory needed | Finite (current state) | Unbounded stack |
| Recognized by | Finite Automata | Pushdown Automata |
| Described by | Regular expressions | CFG |
| Closure under ∩ | Yes | No |
| Closure under complement | Yes | No |
| Example | aⁿ | aⁿbⁿ |
| Non-example | aⁿbⁿ | aⁿbⁿcⁿ |
| Parsing complexity | O(n) | O(n³) general, O(n) for specific |

---

## 4.13 Practice Problems

### Problem 1: Write CFGs

1. L = {aⁿbᵐ | n ≥ m}
2. L = {w ∈ {a,b}* | w has equal a's and b's}
3. L = Balanced parentheses
4. L = {aⁿbᵐcᵏ | n = m or m = k}

### Problem 2: Prove Non-CFL

1. L = {aⁿbⁿcⁿ}
2. L = {ww | w ∈ {a,b}*}

### Solutions

**Problem 1.1:** L = {aⁿbᵐ | n ≥ m}
```
S → aS | aSb | ε
```

**Problem 1.2:** Equal a's and b's
```
S → ε | aSbS | bSaS
Or:
S → ε | aB | bA
A → a | aS | bAA
B → b | bS | aBB
```

**Problem 1.3:** Balanced parentheses
```
S → (S) | SS | ε
```

**Problem 1.4:** n = m or m = k
```
S → S₁ | S₂
S₁ → S₁c | A       (A generates aⁿbⁿ)
A → aAb | ε
S₂ → aS₂ | B       (B generates bⁿcⁿ)  
B → bBc | ε
```

---

## 📌 Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│  1. CFG = (V, Σ, P, S) - rules have single variable on left    │
│  2. Derivation: Apply rules to generate strings                 │
│  3. Parse tree: Visual representation of derivation             │
│  4. Ambiguous: Multiple parse trees for same string             │
│  5. CNF: A → BC or A → a (for CYK algorithm)                    │
│  6. GNF: A → aα (terminal first)                                │
│  7. CFLs closed under ∪, ·, * but NOT ∩ or complement           │
│  8. Pumping Lemma: uvxyz, pump v and y together                 │
│  9. CFL > Regular: Can describe nested structures               │
└────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Pushdown Automata](TOA-05-Pushdown-Automata.md)*
