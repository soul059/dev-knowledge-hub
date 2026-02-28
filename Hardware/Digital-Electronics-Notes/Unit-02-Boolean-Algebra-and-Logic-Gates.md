# Unit 2: Boolean Algebra and Logic Gates

---

## Table of Contents
1. [Introduction to Boolean Algebra](#1-introduction-to-boolean-algebra)
2. [Basic Logic Gates](#2-basic-logic-gates)
3. [Universal Gates](#3-universal-gates)
4. [Boolean Algebra Laws and Theorems](#4-boolean-algebra-laws-and-theorems)
5. [De Morgan's Theorems](#5-de-morgans-theorems)
6. [Canonical Forms](#6-canonical-forms)
7. [Logic Gate Implementations](#7-logic-gate-implementations)
8. [Boolean Expression Simplification](#8-boolean-expression-simplification)
9. [Summary Tables](#9-summary-tables)
10. [Quick Revision Questions](#10-quick-revision-questions)

---

## 1. Introduction to Boolean Algebra

### What is Boolean Algebra?

**Boolean Algebra** is a mathematical system for analyzing and simplifying digital logic circuits. Developed by George Boole in 1854, it operates on binary variables (0 and 1) using logical operations.

```
┌─────────────────────────────────────────────────────────────────┐
│                    BOOLEAN ALGEBRA BASICS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Variables:    A, B, C, X, Y, Z (can only be 0 or 1)          │
│                                                                 │
│   Operations:   AND (·), OR (+), NOT (')                       │
│                                                                 │
│   Constants:    0 (FALSE), 1 (TRUE)                            │
│                                                                 │
│   Logic Levels: HIGH (1), LOW (0)                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Boolean Variables and Values

| Logic Level | Boolean Value | Voltage (TTL) | Voltage (CMOS 5V) |
|-------------|---------------|---------------|-------------------|
| LOW | 0 (FALSE) | 0V - 0.8V | 0V - 1.5V |
| HIGH | 1 (TRUE) | 2V - 5V | 3.5V - 5V |

### Notation Conventions

```
Notation for Operations:
────────────────────────────────────────────────────────
AND:    A·B  =  AB  =  A∧B  =  A AND B
OR:     A+B  =  A∨B  =  A OR B
NOT:    A'   =  Ā   =  ¬A   =  NOT A  =  ~A

Precedence (highest to lowest):
1. Parentheses ()
2. NOT (')
3. AND (·)
4. OR (+)

Example: A + B·C' means A + (B · (C'))
```

---

## 2. Basic Logic Gates

### 2.1 AND Gate

**Function**: Output is HIGH only when ALL inputs are HIGH

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬───────┐
    A ───┐                       │  A  │  B  │ Y=A·B │
         │ D─── Y                ├─────┼─────┼───────┤
    B ───┘                       │  0  │  0  │   0   │
                                 │  0  │  1  │   0   │
Boolean: Y = A · B = AB          │  1  │  0  │   0   │
                                 │  1  │  1  │   1   │
                                 └─────┴─────┴───────┘

Timing Diagram:
    A   ─────┐     ┌─────────┐     ┌─────
             └─────┘         └─────┘
    B   ───────────┐   ┌─────────────────
                   └───┘
  Y=AB  ───────────┐   ┌─────┐     ┌─────
                   └───┘     └─────┘
        
        Output HIGH only when BOTH inputs HIGH
```

**Analogy**: Series circuit - both switches must be closed

```
    +V ────/\────/\──── Light
          SW1   SW2    (ON only if both closed)
```

### 2.2 OR Gate

**Function**: Output is HIGH when ANY input is HIGH

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬───────┐
    A ───┐                       │  A  │  B  │ Y=A+B │
         │)─── Y                 ├─────┼─────┼───────┤
    B ───┘                       │  0  │  0  │   0   │
                                 │  0  │  1  │   1   │
Boolean: Y = A + B               │  1  │  0  │   1   │
                                 │  1  │  1  │   1   │
                                 └─────┴─────┴───────┘

Timing Diagram:
    A   ─────┐     ┌─────────┐     ┌─────
             └─────┘         └─────┘
    B   ───────────┐   ┌─────────────────
                   └───┘
  Y=A+B ─────┐         ┌─────────────────
             └─────────┘
        
        Output HIGH when ANY input is HIGH
```

**Analogy**: Parallel circuit - either switch can light the bulb

```
         ┌────/\────┐
    +V ──┤   SW1    ├──── Light
         └────/\────┘     (ON if ANY closed)
              SW2
```

### 2.3 NOT Gate (Inverter)

**Function**: Output is the complement of input

```
Symbol:                          Truth Table:
                                 ┌─────┬───────┐
    A ───|>o─── Y                │  A  │  Y=A' │
                                 ├─────┼───────┤
Boolean: Y = A' = Ā              │  0  │   1   │
                                 │  1  │   0   │
                                 └─────┴───────┘

Timing Diagram:
    A   ─────┐     ┌─────────┐     ┌─────
             └─────┘         └─────┘
   Y=A' ─────┘     └─────────┘     └─────
             ▲           ▲           ▲
             │           │           │
          Inverted   Inverted   Inverted
```

### 2.4 NAND Gate (NOT-AND)

**Function**: Complement of AND - Output LOW only when ALL inputs HIGH

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬────────┐
    A ───┐                       │  A  │  B  │Y=(AB)' │
         │ D──o── Y              ├─────┼─────┼────────┤
    B ───┘                       │  0  │  0  │   1    │
                                 │  0  │  1  │   1    │
Boolean: Y = (A·B)' = (AB)'      │  1  │  0  │   1    │
                                 │  1  │  1  │   0    │
                                 └─────┴─────┴────────┘

Note: NAND is a UNIVERSAL GATE
```

### 2.5 NOR Gate (NOT-OR)

**Function**: Complement of OR - Output HIGH only when ALL inputs LOW

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬─────────┐
    A ───┐                       │  A  │  B  │Y=(A+B)'│
         │)──o── Y               ├─────┼─────┼─────────┤
    B ───┘                       │  0  │  0  │    1   │
                                 │  0  │  1  │    0   │
Boolean: Y = (A+B)'              │  1  │  0  │    0   │
                                 │  1  │  1  │    0   │
                                 └─────┴─────┴─────────┘

Note: NOR is a UNIVERSAL GATE
```

### 2.6 XOR Gate (Exclusive-OR)

**Function**: Output HIGH when inputs are DIFFERENT

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬────────┐
    A ───┐                       │  A  │  B  │ Y=A⊕B  │
         │))─── Y                ├─────┼─────┼────────┤
    B ───┘                       │  0  │  0  │   0    │
                                 │  0  │  1  │   1    │
Boolean: Y = A ⊕ B               │  1  │  0  │   1    │
         Y = A'B + AB'           │  1  │  1  │   0    │
         Y = (A+B)(A'+B')        └─────┴─────┴────────┘

Properties:
  - Odd function: Output 1 when odd number of 1s in input
  - A ⊕ 0 = A
  - A ⊕ 1 = A'
  - A ⊕ A = 0
  - A ⊕ A' = 1
```

### 2.7 XNOR Gate (Exclusive-NOR)

**Function**: Output HIGH when inputs are SAME (Equivalence gate)

```
Symbol:                          Truth Table:
                                 ┌─────┬─────┬────────┐
    A ───┐                       │  A  │  B  │Y=(A⊕B)'│
         │))──o── Y              ├─────┼─────┼────────┤
    B ───┘                       │  0  │  0  │   1    │
                                 │  0  │  1  │   0    │
Boolean: Y = (A ⊕ B)'            │  1  │  0  │   0    │
         Y = A'B' + AB           │  1  │  1  │   1    │
         Y = A ⊙ B               └─────┴─────┴────────┘

Properties:
  - Even function: Output 1 when even number of 1s
  - A ⊙ B = (A ⊕ B)'
```

### Gate Symbol Summary

```
┌──────────────────────────────────────────────────────────────────────┐
│                      LOGIC GATE SYMBOLS                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   AND          OR           NOT          BUFFER                      │
│     ┌─┐         /\            │>o          │>                        │
│   ──┤ D──     ──(  )──      ──┼──        ──┼──                       │
│   ──┤ │       ──(  )──                                               │
│     └─┘         \/                                                   │
│                                                                      │
│   NAND         NOR          XOR          XNOR                        │
│     ┌─┐o        /\o          /\\           /\\o                      │
│   ──┤ D──     ──(  )──     ──(  ))──     ──(  ))──                   │
│   ──┤ │       ──(  )──     ──(  ))──     ──(  ))──                   │
│     └─┘         \/           \//           \//                       │
│                                                                      │
│   ○ (bubble) at output indicates inversion                          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. Universal Gates

### What Makes a Gate "Universal"?

A **universal gate** can implement ANY Boolean function by itself. NAND and NOR are universal gates.

### 3.1 Implementing Gates using NAND

```
┌────────────────────────────────────────────────────────────────────┐
│                    NAND AS UNIVERSAL GATE                          │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  NOT using NAND:                                                   │
│                                                                    │
│      A ───┬───┐                                                    │
│           │   │ D──o── A'                                          │
│           └───┘                                                    │
│                                                                    │
│  AND using NAND (2 gates):                                         │
│                                                                    │
│      A ───┐                                                        │
│           │ D──o───┬───┐                                           │
│      B ───┘        │   │ D──o── AB                                 │
│                    └───┘                                           │
│                                                                    │
│  OR using NAND (3 gates):                                          │
│                                                                    │
│      A ───┬───┐                                                    │
│           │   │ D──o──┐                                            │
│           └───┘       │ D──o── A+B                                 │
│      B ───┬───┐       │                                            │
│           │   │ D──o──┘                                            │
│           └───┘                                                    │
│                                                                    │
│  XOR using NAND (4 gates):                                         │
│                                                                    │
│      A ─────────┬────┐                                             │
│                 │    │ D──o──┐                                     │
│      B ────┬────┘            │ D──o── A⊕B                          │
│            │    ┌────────────┘                                     │
│      A ────┤    │                                                  │
│            │ D──o──┐                                               │
│      B ────┘       │                                               │
│                    │ D──o──┐                                       │
│      B ────────────┘       │                                       │
│                            │ D──o────────────►                     │
│                            │                                       │
│      A ────────────────────┘                                       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 3.2 Implementing Gates using NOR

```
┌────────────────────────────────────────────────────────────────────┐
│                     NOR AS UNIVERSAL GATE                          │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  NOT using NOR:                                                    │
│                                                                    │
│      A ───┬───┐                                                    │
│           │   │)──o── A'                                           │
│           └───┘                                                    │
│                                                                    │
│  OR using NOR (2 gates):                                           │
│                                                                    │
│      A ───┐                                                        │
│           │)──o───┬───┐                                            │
│      B ───┘       │   │)──o── A+B                                  │
│                   └───┘                                            │
│                                                                    │
│  AND using NOR (3 gates):                                          │
│                                                                    │
│      A ───┬───┐                                                    │
│           │   │)──o──┐                                             │
│           └───┘      │)──o── AB                                    │
│      B ───┬───┐      │                                             │
│           │   │)──o──┘                                             │
│           └───┘                                                    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Universal Gate Comparison

| Function | NAND Gates Required | NOR Gates Required |
|----------|--------------------|--------------------|
| NOT | 1 | 1 |
| AND | 2 | 3 |
| OR | 3 | 2 |
| XOR | 4 | 5 |
| XNOR | 5 | 4 |

---

## 4. Boolean Algebra Laws and Theorems

### Basic Postulates

```
┌─────────────────────────────────────────────────────────────────┐
│                    HUNTINGTON'S POSTULATES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  P1. Closure:         a + b ∈ B,  a · b ∈ B                    │
│                                                                 │
│  P2. Identity:        a + 0 = a,   a · 1 = a                   │
│                                                                 │
│  P3. Commutative:     a + b = b + a,  a · b = b · a            │
│                                                                 │
│  P4. Distributive:    a·(b+c) = ab + ac                        │
│                       a+(b·c) = (a+b)·(a+c)  ← Unique to Boolean│
│                                                                 │
│  P5. Complement:      a + a' = 1,  a · a' = 0                  │
│                                                                 │
│  P6. Two elements:    At least 0 and 1, where 0 ≠ 1            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Boolean Algebra Laws

```
┌───────────────────────────────────────────────────────────────────────┐
│                        BOOLEAN ALGEBRA LAWS                           │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  IDENTITY LAW:                                                        │
│      A + 0 = A                 A · 1 = A                              │
│      A + 1 = 1                 A · 0 = 0                              │
│                                                                       │
│  IDEMPOTENT LAW:                                                      │
│      A + A = A                 A · A = A                              │
│                                                                       │
│  COMPLEMENT LAW:                                                      │
│      A + A' = 1                A · A' = 0                             │
│      (A')' = A  (Double Negation / Involution)                        │
│                                                                       │
│  COMMUTATIVE LAW:                                                     │
│      A + B = B + A             A · B = B · A                          │
│                                                                       │
│  ASSOCIATIVE LAW:                                                     │
│      (A + B) + C = A + (B + C)                                        │
│      (A · B) · C = A · (B · C)                                        │
│                                                                       │
│  DISTRIBUTIVE LAW:                                                    │
│      A · (B + C) = A·B + A·C                                          │
│      A + (B · C) = (A+B) · (A+C)  ← Special in Boolean!               │
│                                                                       │
│  ABSORPTION LAW:                                                      │
│      A + A·B = A               A · (A + B) = A                        │
│      A + A'·B = A + B          A · (A' + B) = A·B                     │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

### Important Theorems

```
┌───────────────────────────────────────────────────────────────────────┐
│                       IMPORTANT THEOREMS                              │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  CONSENSUS THEOREM:                                                   │
│      AB + A'C + BC = AB + A'C                                         │
│      (A+B)(A'+C)(B+C) = (A+B)(A'+C)                                   │
│                                                                       │
│  Proof:                                                               │
│      AB + A'C + BC                                                    │
│    = AB + A'C + BC(A + A')           [A + A' = 1]                     │
│    = AB + A'C + ABC + A'BC                                            │
│    = AB(1 + C) + A'C(1 + B)          [1 + X = 1]                      │
│    = AB + A'C                         ✓                               │
│                                                                       │
│  TRANSPOSITION THEOREM:                                               │
│      AB + A'C = (A + C)(A' + B)                                       │
│                                                                       │
│  SIMPLIFICATION THEOREMS:                                             │
│      A + AB = A                       (Absorption)                    │
│      A(A + B) = A                     (Absorption)                    │
│      (A + B)(A + B') = A              (Combining)                     │
│      A'B + AB = B                     (Combining)                     │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

### Proof Examples

**Proof using Boolean Laws**:

```
Prove: A + A'B = A + B

Proof:
  A + A'B
= (A + A')(A + B)          [Distributive: X + YZ = (X+Y)(X+Z)]
= 1 · (A + B)              [Complement: A + A' = 1]
= A + B                    [Identity: 1 · X = X]  ✓
```

**Proof using Perfect Induction (Truth Table)**:

```
Prove: AB + AB' = A

┌─────┬─────┬──────┬───────┬──────────────┬─────┐
│  A  │  B  │  AB  │  AB'  │  AB + AB'    │  A  │
├─────┼─────┼──────┼───────┼──────────────┼─────┤
│  0  │  0  │   0  │   0   │      0       │  0  │
│  0  │  1  │   0  │   0   │      0       │  0  │
│  1  │  0  │   0  │   1   │      1       │  1  │
│  1  │  1  │   1  │   0   │      1       │  1  │
└─────┴─────┴──────┴───────┴──────────────┴─────┘

AB + AB' = A  (All rows match) ✓
```

---

## 5. De Morgan's Theorems

### The Two Theorems

```
┌─────────────────────────────────────────────────────────────────┐
│                    DE MORGAN'S THEOREMS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  THEOREM 1: (A · B)' = A' + B'                                 │
│             "NOT of AND equals OR of NOTs"                      │
│             "Break the bar, change the sign"                    │
│                                                                 │
│  THEOREM 2: (A + B)' = A' · B'                                 │
│             "NOT of OR equals AND of NOTs"                      │
│             "Break the bar, change the sign"                    │
│                                                                 │
│  Generalized (n variables):                                     │
│  (A · B · C · ... · N)' = A' + B' + C' + ... + N'              │
│  (A + B + C + ... + N)' = A' · B' · C' · ... · N'              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Proof of De Morgan's Theorem 1

**Using Truth Table (Perfect Induction)**:

```
Prove: (A · B)' = A' + B'

┌─────┬─────┬──────┬────────┬──────┬──────┬─────────┐
│  A  │  B  │  AB  │ (AB)'  │  A'  │  B'  │ A' + B' │
├─────┼─────┼──────┼────────┼──────┼──────┼─────────┤
│  0  │  0  │   0  │   1    │   1  │   1  │    1    │
│  0  │  1  │   0  │   1    │   1  │   0  │    1    │
│  1  │  0  │   0  │   1    │   0  │   1  │    1    │
│  1  │  1  │   1  │   0    │   0  │   0  │    0    │
└─────┴─────┴──────┴────────┴──────┴──────┴─────────┘

(AB)' = A' + B'  (Columns match) ✓
```

### Visual Representation

```
De Morgan's Theorem 1: (AB)' = A' + B'

    NAND Gate                     OR gate with inverted inputs
    
    A ───┐                        A ───|>o───┐
         │ D──o── (AB)'                      │)─── A' + B'
    B ───┘                        B ───|>o───┘

    Both circuits are EQUIVALENT!


De Morgan's Theorem 2: (A+B)' = A'B'

    NOR Gate                      AND gate with inverted inputs
    
    A ───┐                        A ───|>o───┐
         │)──o── (A+B)'                      │ D─── A'B'
    B ───┘                        B ───|>o───┘

    Both circuits are EQUIVALENT!
```

### Applications of De Morgan's Theorems

**1. Converting NAND/NOR expressions**:

```
Express NAND using OR:
    (AB)' = A' + B'
    
Express NOR using AND:
    (A+B)' = A'B'
```

**2. Complementing expressions**:

```
Find complement of F = AB + CD

F' = (AB + CD)'
   = (AB)' · (CD)'        [De Morgan on +]
   = (A' + B') · (C' + D') [De Morgan on ·]
   = A'C' + A'D' + B'C' + B'D'  [Distribute]
```

**3. Converting SOP to POS and vice versa**:

```
Convert F = AB + CD to POS:

F = AB + CD
F' = (A' + B')(C' + D')    [Using De Morgan]

To get F in POS, take complement of F':
F = (F')' 
  = ((A' + B')(C' + D'))'
  = (A' + B')' + (C' + D')'  [De Morgan]
  = AB + CD                   [Back to SOP!]

Alternative approach for POS:
F = AB + CD
  = (A + C)(A + D)(B + C)(B + D)  [Use X + YZ = (X+Y)(X+Z)]
```

### De Morgan's Bubble Pushing Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                    BUBBLE PUSHING RULES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Rule 1: Push bubbles through gate, change gate type           │
│                                                                 │
│       A ──o──┐                    A ───┐                        │
│              │ D──── = ────           │)──o──                   │
│       B ──o──┘                    B ───┘                        │
│      (Bubbles on AND inputs = NOR)                              │
│                                                                 │
│       A ──o──┐                    A ───┐                        │
│              │)──── = ────            │ D──o──                  │
│       B ──o──┘                    B ───┘                        │
│      (Bubbles on OR inputs = NAND)                              │
│                                                                 │
│  Rule 2: Add/remove bubbles in pairs                           │
│                                                                 │
│      ──o──o── = ────  (Double inversion cancels)               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Canonical Forms

### What are Canonical Forms?

Canonical forms are **standard ways** to express Boolean functions. Each function has a unique representation in each canonical form.

```
┌─────────────────────────────────────────────────────────────────┐
│                      CANONICAL FORMS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SOP (Sum of Products) / Disjunctive Normal Form               │
│      - Sum (OR) of product (AND) terms                         │
│      - Each term is a MINTERM                                   │
│      - Example: F = ABC + A'BC + AB'C                          │
│                                                                 │
│  POS (Product of Sums) / Conjunctive Normal Form               │
│      - Product (AND) of sum (OR) terms                         │
│      - Each term is a MAXTERM                                   │
│      - Example: F = (A+B+C)(A'+B+C)(A+B'+C)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Minterms and Maxterms

**Minterm (m)**: Product term containing ALL variables (complemented or not)  
**Maxterm (M)**: Sum term containing ALL variables (complemented or not)

```
For 3 variables A, B, C:

┌─────┬─────┬─────┬─────────┬──────────┬───────────┬──────────┐
│  A  │  B  │  C  │ Minterm │ Symbol   │  Maxterm  │  Symbol  │
├─────┼─────┼─────┼─────────┼──────────┼───────────┼──────────┤
│  0  │  0  │  0  │  A'B'C' │   m₀     │  A+B+C    │   M₀     │
│  0  │  0  │  1  │  A'B'C  │   m₁     │  A+B+C'   │   M₁     │
│  0  │  1  │  0  │  A'BC'  │   m₂     │  A+B'+C   │   M₂     │
│  0  │  1  │  1  │  A'BC   │   m₃     │  A+B'+C'  │   M₃     │
│  1  │  0  │  0  │  AB'C'  │   m₄     │  A'+B+C   │   M₄     │
│  1  │  0  │  1  │  AB'C   │   m₅     │  A'+B+C'  │   M₅     │
│  1  │  1  │  0  │  ABC'   │   m₆     │  A'+B'+C  │   M₆     │
│  1  │  1  │  1  │  ABC    │   m₇     │  A'+B'+C' │   M₇     │
└─────┴─────┴─────┴─────────┴──────────┴───────────┴──────────┘

Key Relationships:
  - mᵢ = Mᵢ'  (Minterm i is complement of Maxterm i)
  - Minterm = 1 for only ONE input combination
  - Maxterm = 0 for only ONE input combination
```

### Deriving Canonical SOP

**From Truth Table**: OR together minterms where output is 1

```
Example Truth Table:
┌─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  F  │
├─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  0  │
│  0  │  0  │  1  │  1  │ ← m₁ = A'B'C
│  0  │  1  │  0  │  0  │
│  0  │  1  │  1  │  1  │ ← m₃ = A'BC
│  1  │  0  │  0  │  0  │
│  1  │  0  │  1  │  1  │ ← m₅ = AB'C
│  1  │  1  │  0  │  0  │
│  1  │  1  │  1  │  1  │ ← m₇ = ABC
└─────┴─────┴─────┴─────┘

Canonical SOP:
F = Σm(1, 3, 5, 7)
F = A'B'C + A'BC + AB'C + ABC
F = m₁ + m₃ + m₅ + m₇
```

### Deriving Canonical POS

**From Truth Table**: AND together maxterms where output is 0

```
Same Truth Table:
┌─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  F  │
├─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  0  │ ← M₀ = A+B+C
│  0  │  0  │  1  │  1  │
│  0  │  1  │  0  │  0  │ ← M₂ = A+B'+C
│  0  │  1  │  1  │  1  │
│  1  │  0  │  0  │  0  │ ← M₄ = A'+B+C
│  1  │  0  │  1  │  1  │
│  1  │  1  │  0  │  0  │ ← M₆ = A'+B'+C
│  1  │  1  │  1  │  1  │
└─────┴─────┴─────┴─────┘

Canonical POS:
F = ΠM(0, 2, 4, 6)
F = (A+B+C)(A+B'+C)(A'+B+C)(A'+B'+C)
F = M₀ · M₂ · M₄ · M₆
```

### Converting Between SOP and POS

```
┌─────────────────────────────────────────────────────────────────┐
│              SOP ↔ POS CONVERSION                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Rule: The minterms NOT in SOP become maxterms in POS          │
│                                                                 │
│  Example (3 variables, 8 possible terms):                       │
│                                                                 │
│  F = Σm(1, 3, 5, 7)  ←─────────────────────────────────┐       │
│                                                         │       │
│  Missing minterms: 0, 2, 4, 6                          │       │
│                                                         │       │
│  F = ΠM(0, 2, 4, 6)  ←─── These become maxterms ───────┘       │
│                                                                 │
│  General: If F = Σm(a, b, c, ...)                              │
│           Then F = ΠM(remaining indices)                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Standard vs Canonical Forms

```
Canonical Form (All variables in each term):
  F = A'B'C + A'BC + AB'C + ABC

Standard Form (May have missing variables):
  F = A'C + BC + AC

Converting Standard to Canonical:
  Expand missing variables using X = X(Y + Y')
  
  Example: A'C → A'C(B + B') = A'BC + A'B'C
```

---

## 7. Logic Gate Implementations

### AND-OR Implementation (SOP)

```
Implement F = AB + CD + EF

         A ───┐
              │ D─────┐
         B ───┘       │
                      │
         C ───┐       │
              │ D─────┼────┐
         D ───┘       │    │)─── F
                      │    │
         E ───┐       │    │
              │ D─────┘────┘
         F ───┘

Layer 1: AND gates (products)
Layer 2: OR gate (sum)

This is called a TWO-LEVEL implementation
```

### OR-AND Implementation (POS)

```
Implement F = (A+B)(C+D)(E+F)

         A ───┐
              │)─────┐
         B ───┘      │
                     │
         C ───┐      │
              │)─────┼────┐
         D ───┘      │    │ D─── F
                     │    │
         E ───┐      │    │
              │)─────┘────┘
         F ───┘

Layer 1: OR gates (sums)
Layer 2: AND gate (product)
```

### NAND-NAND Implementation (SOP)

Any SOP expression can be implemented using only NAND gates:

```
F = AB + CD

Step 1: Double complement
F = ((AB + CD)')'

Step 2: Apply De Morgan inside
F = ((AB)'(CD)')'

Implementation:

         A ───┐
              │ D──o──┐
         B ───┘       │ D──o─── F
                      │
         C ───┐       │
              │ D──o──┘
         D ───┘

All NAND gates! (3 gates total)
```

### NOR-NOR Implementation (POS)

Any POS expression can be implemented using only NOR gates:

```
F = (A+B)(C+D)

Step 1: Double complement
F = (((A+B)(C+D))')'

Step 2: Apply De Morgan inside
F = ((A+B)' + (C+D)')'

Implementation:

         A ───┐
              │)──o──┐
         B ───┘      │)──o─── F
                     │
         C ───┐      │
              │)──o──┘
         D ───┘

All NOR gates! (3 gates total)
```

### Multi-Level Implementation

For complex expressions, multi-level circuits may use fewer gates:

```
Example: F = ABC + ABD + AEF + AEG

Standard two-level: 4 AND gates (3-input) + 1 OR gate (4-input)
                    = 5 gates, 16 gate inputs

Factored form: F = A(BC + BD + EF + EG)
             = A(B(C+D) + E(F+G))

Multi-level implementation:

         C ───┐
              │)───┐
         D ───┘    │
                   │ D───┐
         B ────────┘     │
                         │)───┐
         F ───┐          │    │ D─── F
              │)───┐     │    │
         G ───┘    │     │    │
                   │ D───┘────┘
         E ────────┘
         
         A ────────────────────┘

Multi-level: 4 OR/AND gates, fewer gate inputs
Trade-off: More levels = more propagation delay
```

---

## 8. Boolean Expression Simplification

### Algebraic Simplification

**Goal**: Reduce number of literals and terms

```
Example 1: Simplify F = A'B + AB' + AB

F = A'B + AB' + AB
  = A'B + A(B' + B)      [Factor A]
  = A'B + A(1)           [B' + B = 1]
  = A'B + A              [Identity]
  = A + B                [Absorption: X + X'Y = X + Y]

Example 2: Simplify F = A'BC + AB'C + ABC' + ABC

F = A'BC + AB'C + ABC' + ABC
  = A'BC + ABC + AB'C + ABC' + ABC    [Add ABC - already present]
  = BC(A' + A) + AC(B' + B) + AB(C' + C)
  = BC(1) + AC(1) + AB(1)
  = AB + BC + AC

Example 3: Simplify F = (A + B)(A + C)(B + C)

Using Consensus Theorem: XY + X'Z + YZ = XY + X'Z

Let's expand:
F = (A + B)(A + C)(B + C)
  = (A + BC)(B + C)                [Distribute first two]
  = AB + AC + BC + BC              [Distribute]
  = AB + AC + BC                   [Idempotent]

Actually by consensus: AB + A'C + BC = AB + A'C
So: AB + AC + BC = AB + AC + BC... need different approach

Using dual form: (X+Y)(X'+Z)(Y+Z) = (X+Y)(X'+Z)
F = (A+B)(A+C)(B+C) = (A+B)(A+C)
  = A + BC                          [Distribute: X + YZ]
```

### Simplification Techniques Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                  SIMPLIFICATION TECHNIQUES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Look for common terms to factor                            │
│     ABC + ABD = AB(C + D)                                      │
│                                                                 │
│  2. Apply absorption                                           │
│     A + AB = A                                                 │
│     A + A'B = A + B                                            │
│                                                                 │
│  3. Combine terms differing by one variable                    │
│     ABC + ABC' = AB(C + C') = AB                               │
│                                                                 │
│  4. Apply consensus theorem                                    │
│     AB + A'C + BC = AB + A'C                                   │
│                                                                 │
│  5. Use De Morgan for POS ↔ SOP conversion                     │
│                                                                 │
│  6. Add redundant terms to enable simplification               │
│     (Advanced technique)                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary Tables

### Logic Gate Summary

| Gate | Symbol | Boolean | Output = 1 when | IC Number (7400 series) |
|------|--------|---------|-----------------|------------------------|
| AND | D | Y = A·B | All inputs = 1 | 7408 |
| OR | )| Y = A+B | Any input = 1 | 7432 |
| NOT | >o | Y = A' | Input = 0 | 7404 |
| NAND | D○ | Y = (AB)' | Any input = 0 | 7400 |
| NOR | )○ | Y = (A+B)' | All inputs = 0 | 7402 |
| XOR | )) | Y = A⊕B | Odd # of 1s | 7486 |
| XNOR | ))○ | Y = (A⊕B)' | Even # of 1s | 74266 |

### Boolean Laws Quick Reference

| Law | AND Form | OR Form |
|-----|----------|---------|
| Identity | A · 1 = A | A + 0 = A |
| Null | A · 0 = 0 | A + 1 = 1 |
| Idempotent | A · A = A | A + A = A |
| Complement | A · A' = 0 | A + A' = 1 |
| Involution | (A')' = A | (A')' = A |
| Commutative | AB = BA | A+B = B+A |
| Associative | (AB)C = A(BC) | (A+B)+C = A+(B+C) |
| Distributive | A(B+C) = AB+AC | A+BC = (A+B)(A+C) |
| Absorption | A(A+B) = A | A+AB = A |
| De Morgan | (AB)' = A'+B' | (A+B)' = A'B' |

### Canonical Form Summary

| Aspect | SOP (Sum of Products) | POS (Product of Sums) |
|--------|----------------------|----------------------|
| Structure | OR of ANDs | AND of ORs |
| Terms | Minterms (m) | Maxterms (M) |
| Notation | Σm(indices) | ΠM(indices) |
| Output = 1 | Minterm = 1 | All maxterms = 1 |
| Output = 0 | All minterms = 0 | Maxterm = 0 |
| Implementation | AND-OR or NAND-NAND | OR-AND or NOR-NOR |

### Universal Gate Equivalents

| Function | Using NAND | Using NOR |
|----------|-----------|----------|
| NOT A | A NAND A | A NOR A |
| A AND B | (A NAND B) NAND (A NAND B) | ((A NOR A) NOR (B NOR B)) NOR ((A NOR A) NOR (B NOR B)) |
| A OR B | (A NAND A) NAND (B NAND B) | (A NOR B) NOR (A NOR B) |

---

## 10. Quick Revision Questions

### Question 1
Simplify using Boolean algebra: F = A'B'C + A'BC + AB'C + ABC

<details>
<summary>Click for Answer</summary>

```
F = A'B'C + A'BC + AB'C + ABC
  = A'C(B' + B) + AC(B' + B)      [Factor]
  = A'C(1) + AC(1)                 [B' + B = 1]
  = A'C + AC                       [Identity]
  = C(A' + A)                      [Factor]
  = C(1)                           [A' + A = 1]
  = C

Answer: F = C
```
</details>

### Question 2
Implement F = AB + CD using only NAND gates. How many gates are needed?

<details>
<summary>Click for Answer</summary>

```
F = AB + CD
  = ((AB + CD)')'                  [Double complement]
  = ((AB)' · (CD)')'               [De Morgan]

Implementation:
  - Gate 1: A NAND B = (AB)'
  - Gate 2: C NAND D = (CD)'
  - Gate 3: (AB)' NAND (CD)' = ((AB)'(CD)')' = AB + CD

Answer: 3 NAND gates
```
</details>

### Question 3
Express F(A,B,C) = Σm(1,2,5,7) in canonical POS form.

<details>
<summary>Click for Answer</summary>

```
For 3 variables, possible minterms: 0,1,2,3,4,5,6,7

Given: F = Σm(1,2,5,7)
Missing minterms: 0,3,4,6

These become maxterms in POS:
F = ΠM(0,3,4,6)
F = M₀ · M₃ · M₄ · M₆
F = (A+B+C)(A+B'+C')(A'+B+C)(A'+B'+C')

Answer: F = (A+B+C)(A+B'+C')(A'+B+C)(A'+B'+C')
```
</details>

### Question 4
Apply De Morgan's theorem to find the complement of F = AB' + C'D.

<details>
<summary>Click for Answer</summary>

```
F = AB' + C'D
F' = (AB' + C'D)'
   = (AB')' · (C'D)'              [De Morgan on OR]
   = (A' + B) · (C + D')          [De Morgan on AND]

Answer: F' = (A' + B)(C + D')
Or expanded: F' = A'C + A'D' + BC + BD'
```
</details>

### Question 5
Prove using truth table: A ⊕ B = AB' + A'B

<details>
<summary>Click for Answer</summary>

```
┌─────┬─────┬──────┬──────┬──────────┬───────┐
│  A  │  B  │  AB' │  A'B │ AB'+A'B  │ A⊕B   │
├─────┼─────┼──────┼──────┼──────────┼───────┤
│  0  │  0  │   0  │   0  │    0     │   0   │
│  0  │  1  │   0  │   1  │    1     │   1   │
│  1  │  0  │   1  │   0  │    1     │   1   │
│  1  │  1  │   0  │   0  │    0     │   0   │
└─────┴─────┴──────┴──────┴──────────┴───────┘

The columns match, therefore: A ⊕ B = AB' + A'B  ✓
```
</details>

### Question 6
What is the minimum number of 2-input NAND gates required to implement a 3-input AND gate?

<details>
<summary>Click for Answer</summary>

```
3-input AND: F = ABC

Using 2-input NAND gates:

Step 1: AB using NAND
  - Gate 1: A NAND B = (AB)'
  - Gate 2: (AB)' NAND (AB)' = ((AB)')' = AB

Step 2: (AB)C using NAND
  - Gate 3: AB NAND C = (ABC)'
  - Gate 4: (ABC)' NAND (ABC)' = ABC

Answer: 4 NAND gates

Alternative (3 gates):
  - Gate 1: A NAND B = (AB)'
  - Gate 2: (AB)' NAND C = ((AB)'C)' = (AB)' + C' = A' + B' + C'
  ...this doesn't work directly

Actually optimal:
  - Gate 1: A NAND B = (AB)'
  - Gate 2: (AB)' NAND (AB)' = AB
  - Gate 3: AB NAND C = (ABC)'
  - Gate 4: (ABC)' NAND (ABC)' = ABC

Answer: 4 NAND gates minimum
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 1: Number Systems](Unit-01-Number-Systems-and-Codes.md) | Return to Index | [Unit 3: Minimization Techniques](Unit-03-Minimization-Techniques.md) |

---

*End of Unit 2*
