# Chapter 3: Regular Expressions and Languages

## 3.1 Introduction to Regular Expressions

**Regular Expressions** (regex) are a powerful notation for describing patterns in strings. They are equivalent to finite automata in expressive power.

### Why Regular Expressions?

| Use Case | Example |
|----------|---------|
| Text search | Finding email addresses in documents |
| Lexical analysis | Tokenizing source code |
| Data validation | Checking input formats |
| Text processing | Find and replace operations |
| Pattern matching | grep, sed, awk utilities |

---

## 3.2 Formal Definition

### Regular Expression over Alphabet Σ

Regular expressions are defined **recursively**:

**Base Cases:**
```
1. ∅ is a regular expression (denotes empty language {})
2. ε is a regular expression (denotes language {ε})
3. For each a ∈ Σ, 'a' is a regular expression (denotes {a})
```

**Recursive Cases:**
If r and s are regular expressions, then:
```
4. (r + s) is a regular expression    [Union]
5. (r · s) is a regular expression    [Concatenation]
6. (r*) is a regular expression       [Kleene Star]
```

**Nothing else is a regular expression.**

### Language Denoted by Regular Expression

| Expression | Language L(r) |
|------------|---------------|
| ∅ | {} (empty set) |
| ε | {ε} |
| a | {a} |
| r + s | L(r) ∪ L(s) |
| r · s | L(r) · L(s) |
| r* | L(r)* = {ε} ∪ L(r) ∪ L(r)² ∪ ... |

---

## 3.3 Operator Precedence and Notation

### Precedence (highest to lowest)

```
1. Kleene Star (*)     - highest
2. Concatenation (·)   
3. Union (+)           - lowest

Example: a + b · c* means a + (b · (c*))
         = a + (bc*)
```

### Notational Conventions

| Full Notation | Shorthand | Meaning |
|---------------|-----------|---------|
| (r · s) | rs | Concatenation |
| (r + s) | r + s or r \| s | Union |
| r* | r* | Zero or more |
| rr* | r⁺ | One or more |
| r + ε | r? | Zero or one (optional) |

### Extended Regular Expression Operators

```
r⁺ = rr* = r · r*     (one or more)
r? = r + ε            (zero or one)
rⁿ = rrr...r (n times)
[a-z] = a+b+c+...+z   (character class)
. = any character     (wildcard)
```

---

## 3.4 Examples of Regular Expressions

### Over Σ = {0, 1}

| Regular Expression | Language Description |
|-------------------|----------------------|
| 0 | {0} |
| 0 + 1 | {0, 1} |
| 01 | {01} |
| 0* | {ε, 0, 00, 000, ...} |
| (0 + 1)* | All strings = Σ* |
| 0*1* | Zero or more 0s followed by zero or more 1s |
| (01)* | {ε, 01, 0101, 010101, ...} |
| 0(0 + 1)*0 | Strings starting and ending with 0 |
| (0 + 1)*01 | Strings ending with 01 |
| 1*(01*01*)* | Strings with even number of 0s |
| (0 + 1)*00(0 + 1)* | Strings containing 00 |
| 0*10*10* | Strings with exactly two 1s |
| (0 + 1)*1(0 + 1)*1(0 + 1)* | Strings with at least two 1s |

### Over Σ = {a, b}

| Regular Expression | Language |
|-------------------|----------|
| a*b* | a's followed by b's |
| (ab)* | Alternating, starting with a, equal count |
| (a + b)*aba(a + b)* | Contains "aba" as substring |
| b*ab*ab* | Exactly two a's |
| (a + b)*(aa + bb)(a + b)* | Contains "aa" or "bb" |
| (a*b*)* | = (a + b)* = Σ* |

---

## 3.5 Algebraic Laws of Regular Expressions

### Basic Identities

```
Union:
r + s = s + r                    (Commutative)
(r + s) + t = r + (s + t)        (Associative)
r + ∅ = r                        (Identity)
r + r = r                        (Idempotent)

Concatenation:
(rs)t = r(st)                    (Associative)
rε = εr = r                      (Identity)
r∅ = ∅r = ∅                      (Annihilator)

Distribution:
r(s + t) = rs + rt               (Left distributive)
(r + s)t = rt + st               (Right distributive)

Kleene Star:
(r*)* = r*
∅* = ε
ε* = ε
r* = ε + r·r*
r* = ε + r*·r
(r + s)* = (r*s*)*
r*s* ≠ s*r* (NOT commutative!)
```

### Important Equivalences

```
1. r*r* = r*
2. (r*)* = r*
3. r*rr* = rr* = r*r = r⁺
4. (r + s)* = (r* + s*)* = (r*s*)* 
5. ε + rr* = r*
6. (ε + r)* = r*
7. r(sr)* = (rs)*r
```

---

## 3.6 Regular Expression → Finite Automaton

### Theorem (Kleene's Theorem - Part 1)

Every regular expression has an equivalent NFA (and hence DFA).

### Thompson's Construction

Build NFA recursively following RE structure:

**Base Cases:**

```
For ε:                    For symbol a:
→ ◎                       → ○ ──a──→ ◎

For ∅:
→ ○        ◎ (no path between them)
```

**Union: r + s**
```
                    ┌──ε──→ (NFA for r) ──ε──┐
                    │                         │
→ ○ ────────────────┤                         ├──→ ◎
                    │                         │
                    └──ε──→ (NFA for s) ──ε──┘
```

**Concatenation: rs**
```
→ (NFA for r) ──ε──→ (NFA for s) →
  [start]     [r's final → s's start]  [final]
```

**Kleene Star: r***
```
        ┌─────────── ε ───────────┐
        │                         │
        ↓     ┌─────ε─────┐       │
→ ○ ──ε──→ (NFA for r) ───┴──ε──→ ◎
              ↑           │
              └─────ε─────┘
```

### Example: Convert (a+b)*abb to NFA

```
Step 1: NFA for 'a'
→ ○ ──a──→ ◎

Step 2: NFA for 'b'
→ ○ ──b──→ ◎

Step 3: NFA for (a + b)
         ┌──ε──→ ○ ──a──→ ○ ──ε──┐
→ ○ ─────┤                        ├──→ ◎
         └──ε──→ ○ ──b──→ ○ ──ε──┘

Step 4: NFA for (a + b)*
Add ε-loops for repetition

Step 5: Concatenate with 'a', 'b', 'b'
Connect (a+b)* to literal sequence "abb"
```

---

## 3.7 Finite Automaton → Regular Expression

### Theorem (Kleene's Theorem - Part 2)

Every DFA/NFA has an equivalent regular expression.

### State Elimination Method

**Algorithm:**
1. Add unique start and accept states if needed
2. Eliminate states one by one
3. Replace transitions with regular expressions
4. Final edge label is the answer

**Eliminating a State:**

When removing state q:
```
Before:                    After:
p ──R──→ q ──S──→ r       p ──R·T*·S──→ r
           ↺
           T (self-loop)

For all pairs (p, r) with path through q:
New label = R · T* · S
```

### Example: DFA to Regular Expression

```
DFA for strings ending with 'ab':

→ (q₀) ──a──→ (q₁) ──b──→ ((q₂))
    ↺b         ↺a,↺(b→q₀)   a→q₁, b→q₀

Step 1: Add unique start/end
Step 2: Express as equation system
Step 3: Solve using Arden's theorem

Arden's Theorem:
If X = AX + B (where ε ∉ L(A))
Then X = A*B
```

### Algebraic Method (State Equations)

Write equations for each state:
```
q₀ = ε + q₀b + q₂b
q₁ = q₀a + q₁a
q₂ = q₁b

Solve bottom-up:
From q₂: We need expressions reaching q₂
From q₁: q₁ = q₀a + q₁a = q₀a(a)*  = q₀aa*
From q₂: q₂ = q₁b = q₀aa*b
From q₀: q₀ = ε + q₀b + q₀aa*bb (continue solving)

Final: (b + aa*bb)* aa*b
      = (b* aa*bb)* b* aa*b
      = b*(ab*ab)*ab
```

---

## 3.8 Equivalence of RE and FA

### Kleene's Theorem (Complete)

```
┌─────────────────────────────────────────────────────────┐
│    Regular Expressions ≡ DFA ≡ NFA ≡ ε-NFA              │
│                                                          │
│    All describe exactly the REGULAR LANGUAGES            │
│                                                          │
│    RE ──Thompson's──→ NFA ──Subset──→ DFA               │
│     ↑                                  │                 │
│     └────── State Elimination ─────────┘                 │
└─────────────────────────────────────────────────────────┘
```

---

## 3.9 Properties of Regular Languages

### Closure Properties

Regular languages are **closed under**:

| Operation | If L₁, L₂ regular | Result |
|-----------|-------------------|--------|
| Union | L₁ ∪ L₂ | Regular |
| Intersection | L₁ ∩ L₂ | Regular |
| Complement | L̄₁ = Σ* - L₁ | Regular |
| Concatenation | L₁ · L₂ | Regular |
| Kleene Star | L₁* | Regular |
| Difference | L₁ - L₂ | Regular |
| Reversal | L₁ᴿ | Regular |
| Homomorphism | h(L₁) | Regular |
| Inverse Homomorphism | h⁻¹(L₁) | Regular |

### Proofs of Closure

**Union:** If L₁ = L(r₁) and L₂ = L(r₂), then L₁ ∪ L₂ = L(r₁ + r₂)

**Complement:** 
1. Build DFA for L
2. Swap accepting and non-accepting states
3. Result: DFA for L̄

**Intersection:** L₁ ∩ L₂ = (L̄₁ ∪ L̄₂)̄  (De Morgan's Law)
Or: Product construction with F = F₁ × F₂

**Reversal:** If L = L(r), then Lᴿ = L(rᴿ) where:
```
∅ᴿ = ∅
εᴿ = ε
aᴿ = a
(r + s)ᴿ = rᴿ + sᴿ
(rs)ᴿ = sᴿrᴿ
(r*)ᴿ = (rᴿ)*
```

---

## 3.10 Decision Properties of Regular Languages

### Decidable Problems for Regular Languages

| Problem | Question | Decidable? | Method |
|---------|----------|------------|--------|
| Membership | Is w ∈ L? | Yes | Simulate DFA |
| Emptiness | Is L = ∅? | Yes | Check if any accepting state reachable |
| Finiteness | Is L finite? | Yes | Check for cycles to accepting states |
| Equivalence | Is L₁ = L₂? | Yes | Check L₁ ⊕ L₂ = ∅ |
| Subset | Is L₁ ⊆ L₂? | Yes | Check L₁ ∩ L̄₂ = ∅ |

### Algorithms

**Emptiness Test:**
```
1. Build DFA
2. Do BFS/DFS from start state
3. If any accepting state reachable → NOT empty
```

**Finiteness Test:**
```
1. Remove unreachable states
2. Remove states that don't reach accepting state
3. If remaining graph has cycle → INFINITE
```

**Equivalence Test:**
```
1. Build DFA for L₁ ⊕ L₂ (symmetric difference)
   L₁ ⊕ L₂ = (L₁ - L₂) ∪ (L₂ - L₁)
2. Check if result is empty
3. If empty → L₁ = L₂
```

---

## 3.11 Pumping Lemma for Regular Languages

### The Pumping Lemma

**Statement:** If L is a regular language, then there exists a constant p (pumping length) such that:

For every string w ∈ L with |w| ≥ p:
```
w can be split as w = xyz where:
1. |y| > 0         (y is not empty)
2. |xy| ≤ p        (first two parts not too long)
3. xyⁱz ∈ L        for all i ≥ 0 (y can be "pumped")
```

### Visual Understanding

```
If DFA has p states and accepts string of length ≥ p:

State sequence: q₀ → q₁ → ... → q_p
                     ↑           ↑
                     └───y loop──┘

By pigeonhole principle, some state repeats.
That creates a "pumpable" loop.

Original:    x    y    z
          ──────○────────
               ↺

Pumped (i=0): x    z
          ──────────

Pumped (i=2): x    yy   z
          ──────○○────────
               ↺↺
```

### Using Pumping Lemma to Prove Non-Regularity

**Template:**
1. Assume L is regular
2. Let p be the pumping length
3. Choose a "bad" string w ∈ L with |w| ≥ p
4. Consider all possible divisions w = xyz satisfying conditions 1 & 2
5. Show that for some i, xyⁱz ∉ L
6. Contradiction → L is not regular

### Example: Prove L = {aⁿbⁿ | n ≥ 0} is not regular

```
Proof:
1. Assume L is regular with pumping length p
2. Choose w = aᵖbᵖ ∈ L (since p a's followed by p b's)
   |w| = 2p ≥ p ✓
3. Any division xyz with |xy| ≤ p means xy is all a's
   So y = aᵏ for some k > 0

4. Pump down (i = 0):
   xy⁰z = xz = aᵖ⁻ᵏbᵖ
   
   Since k > 0, we have p - k < p
   So aᵖ⁻ᵏbᵖ ∉ L (unequal a's and b's)

5. Contradiction! Therefore L is not regular.
```

### More Examples of Non-Regular Languages

| Language | Pumping Strategy |
|----------|------------------|
| {aⁿbⁿ} | Choose aᵖbᵖ, pump y (all a's) |
| {ww \| w ∈ Σ*} | Choose aᵖbaᵖb |
| {aⁿ² \| n ≥ 0} | Choose aᵖ², show gaps too large |
| {aⁿbᵐ \| n > m} | Choose aᵖ⁺¹bᵖ |
| {aᵖ \| p is prime} | Choose aᵖ where p is prime |

---

## 3.12 Myhill-Nerode Theorem

### Distinguishable Strings

Two strings x and y are **L-distinguishable** if:
```
∃z: exactly one of xz, yz is in L
```

### Theorem Statement

L is regular if and only if the equivalence relation ≡_L has **finite index** (finite number of equivalence classes).

```
x ≡_L y  ⟺  ∀z ∈ Σ*: (xz ∈ L ⟷ yz ∈ L)
```

### Using Myhill-Nerode to Prove Non-Regularity

**For L = {aⁿbⁿ}:**
```
Consider: a, aa, aaa, aaaa, ...

Are aⁱ and aʲ (i ≠ j) distinguishable?
- aⁱbⁱ ∈ L
- aʲbⁱ ∉ L (if j ≠ i)

So aⁱ and aʲ are distinguishable for all i ≠ j.
This means infinitely many equivalence classes.
Therefore L is not regular.
```

### Advantage over Pumping Lemma

Myhill-Nerode is both necessary AND sufficient for regularity:
- Pumping Lemma: L regular → satisfies lemma (one direction)
- Myhill-Nerode: L regular ⟺ finite equivalence classes (both directions)

---

## 3.13 Practice Problems

### Problem 1: Write Regular Expressions

1. All strings over {a, b} of length exactly 3
2. All strings over {0, 1} with no consecutive 1s
3. All strings over {a, b} with an odd number of a's
4. All strings over {0, 1} that represent binary numbers divisible by 2
5. All valid C identifiers (start with letter/underscore, then letters/digits/underscores)

### Problem 2: Prove Non-Regularity

Prove these languages are not regular:
1. L = {aⁿbⁿcⁿ | n ≥ 0}
2. L = {ww | w ∈ {a,b}*}
3. L = {0ⁿ | n is a perfect square}

### Solutions

**Problem 1:**
1. (a+b)(a+b)(a+b)
2. (0 + 10)*(1 + ε)  or  (1 + ε)(0 + 01)*
3. b*a(b*ab*ab*)* or b*(ab*ab*)*ab*
4. (0+1)*0 (binary numbers ending in 0)
5. (a+...+z+A+...+Z+_)(a+...+z+A+...+Z+0+...+9+_)*

**Problem 2.1:** L = {aⁿbⁿcⁿ}
```
Assume regular with pumping length p.
Choose w = aᵖbᵖcᵖ.
Any y in first p symbols is only a's.
Pumping changes number of a's but not b's or c's.
Contradiction.
```

---

## 📌 Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│  1. RE operators: Union (+), Concatenation (·), Star (*)       │
│  2. Precedence: * > · > +                                      │
│  3. Kleene's Theorem: RE ≡ DFA ≡ NFA (all equivalent)          │
│  4. Regular languages closed under ∪, ∩, complement, *, etc.   │
│  5. Pumping Lemma: Tool to prove languages are NOT regular     │
│  6. Myhill-Nerode: L regular ⟺ finite equivalence classes      │
│  7. Key non-regular: aⁿbⁿ, ww, aⁿ² (need unbounded memory)     │
└────────────────────────────────────────────────────────────────┘
```

---

*Next Chapter: [Context-Free Grammars](TOA-04-Context-Free-Grammars.md)*
