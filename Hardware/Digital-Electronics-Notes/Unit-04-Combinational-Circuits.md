# Unit 4: Combinational Circuits

---

## Table of Contents
1. [Introduction to Combinational Circuits](#1-introduction-to-combinational-circuits)
2. [Adders](#2-adders)
3. [Subtractors](#3-subtractors)
4. [Multiplexers](#4-multiplexers)
5. [Demultiplexers](#5-demultiplexers)
6. [Encoders](#6-encoders)
7. [Decoders](#7-decoders)
8. [Code Converters](#8-code-converters)
9. [Comparators](#9-comparators)
10. [Parity Generators and Checkers](#10-parity-generators-and-checkers)
11. [ALU Design](#11-alu-design)
12. [Summary Tables](#12-summary-tables)
13. [Quick Revision Questions](#13-quick-revision-questions)

---

## 1. Introduction to Combinational Circuits

### What is a Combinational Circuit?

```
┌─────────────────────────────────────────────────────────────────┐
│                 COMBINATIONAL CIRCUIT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Definition: A circuit whose outputs depend ONLY on the        │
│              current inputs (no memory/feedback)                │
│                                                                 │
│        ┌─────────────────────────────┐                         │
│        │                             │                         │
│  x₁ ───┤    COMBINATIONAL           ├─── z₁                   │
│  x₂ ───┤       LOGIC                ├─── z₂                   │
│  x₃ ───┤                            ├─── z₃                   │
│   .    │     (No Memory)            │    .                    │
│   .    │                            │    .                    │
│  xₙ ───┤                            ├─── zₘ                   │
│        │                             │                         │
│        └─────────────────────────────┘                         │
│                                                                 │
│  Characteristics:                                               │
│    • No feedback loops                                         │
│    • No clock required                                         │
│    • Output changes immediately with input                     │
│    • Can be described by truth table                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Combinational vs Sequential

| Aspect | Combinational | Sequential |
|--------|--------------|------------|
| Memory | No | Yes |
| Clock | Not required | Usually required |
| Output depends on | Current inputs only | Current inputs + past states |
| Feedback | No | Yes |
| Examples | Adders, MUX | Flip-flops, Counters |

### Design Procedure

```
┌─────────────────────────────────────────────────────────────────┐
│           COMBINATIONAL CIRCUIT DESIGN STEPS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SPECIFICATION                                               │
│     - Define the problem clearly                               │
│     - Identify inputs and outputs                              │
│                                                                 │
│  2. TRUTH TABLE                                                 │
│     - List all input combinations                              │
│     - Determine output for each combination                    │
│                                                                 │
│  3. SIMPLIFICATION                                              │
│     - Use K-maps or Boolean algebra                            │
│     - Minimize expressions                                     │
│                                                                 │
│  4. IMPLEMENTATION                                              │
│     - Draw logic circuit                                       │
│     - Choose gate types based on availability                  │
│                                                                 │
│  5. VERIFICATION                                                │
│     - Verify with truth table                                  │
│     - Check timing requirements                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Adders

### 2.1 Half Adder

**Function**: Adds two 1-bit numbers

```
HALF ADDER TRUTH TABLE:
┌─────┬─────┬───────┬───────┐
│  A  │  B  │ Sum   │ Carry │
├─────┼─────┼───────┼───────┤
│  0  │  0  │   0   │   0   │
│  0  │  1  │   1   │   0   │
│  1  │  0  │   1   │   0   │
│  1  │  1  │   0   │   1   │
└─────┴─────┴───────┴───────┘

Boolean Expressions:
  Sum = A ⊕ B = A'B + AB'
  Carry = A · B

HALF ADDER CIRCUIT:
                      ┌─────┐
         A ───────────┤     │
                      │ XOR ├─────── Sum
         B ─────┬─────┤     │
                │     └─────┘
                │     ┌─────┐
                ├─────┤     │
         A ─────┤     │ AND ├─────── Carry
                      │     │
                      └─────┘

BLOCK DIAGRAM:
              ┌─────────────┐
         A ───┤             ├─── Sum
              │     HA      │
         B ───┤             ├─── Cout
              └─────────────┘
```

### 2.2 Full Adder

**Function**: Adds three 1-bit numbers (A, B, and Carry-in)

```
FULL ADDER TRUTH TABLE:
┌─────┬─────┬─────┬───────┬───────┐
│  A  │  B  │ Cᵢₙ │  Sum  │ Cₒᵤₜ  │
├─────┼─────┼─────┼───────┼───────┤
│  0  │  0  │  0  │   0   │   0   │
│  0  │  0  │  1  │   1   │   0   │
│  0  │  1  │  0  │   1   │   0   │
│  0  │  1  │  1  │   0   │   1   │
│  1  │  0  │  0  │   1   │   0   │
│  1  │  0  │  1  │   0   │   1   │
│  1  │  1  │  0  │   0   │   1   │
│  1  │  1  │  1  │   1   │   1   │
└─────┴─────┴─────┴───────┴───────┘

Boolean Expressions:
  Sum = A ⊕ B ⊕ Cᵢₙ
  Cₒᵤₜ = AB + BCᵢₙ + ACᵢₙ = AB + (A ⊕ B)Cᵢₙ

K-MAP FOR SUM:                    K-MAP FOR CARRY:
        Cᵢₙ                              Cᵢₙ
      0     1                          0     1
    ┌─────┬─────┐                    ┌─────┬─────┐
 00 │  0  │  1  │                 00 │  0  │  0  │
    ├─────┼─────┤                    ├─────┼─────┤
 01 │  1  │  0  │                 01 │  0  │  1  │
AB  ├─────┼─────┤              AB    ├─────┼─────┤
 11 │  0  │  1  │                 11 │  1  │  1  │
    ├─────┼─────┤                    ├─────┼─────┤
 10 │  1  │  0  │                 10 │  0  │  1  │
    └─────┴─────┘                    └─────┴─────┘
    
Sum = XOR pattern (checkerboard)
Carry = AB + BCᵢₙ + ACᵢₙ

FULL ADDER USING TWO HALF ADDERS:

         A ─────┬──────────────────────────────────────┐
                │     ┌─────────────┐                  │
         B ─────┼─────┤             ├───┬──────────────┼─────┐
                │     │     HA₁     │   │              │     │
                ├─────┤             ├───┼──────────┐   │     │
                │     └─────────────┘   │          │   │     │
                │           S₁          C₁         │   │     │
                │            │          │          │   │     │
                │     ┌──────┴────┐     │          │   │     │
        Cᵢₙ ────┼─────┤           │     │   ┌──────┴───┴──┐  │
                │     │    HA₂    ├─────┼───┤             │  │
                │     │           │     │   │     OR      ├──┴── Cₒᵤₜ
                │     └─────┬─────┘     │   │             │
                │           │           └───┤             │
                │          Sum              └─────────────┘
                │           │
                └───────────┴─────────────────────────────── Sum

BLOCK DIAGRAM:
              ┌─────────────┐
         A ───┤             ├─── Sum
         B ───┤     FA      │
        Cᵢₙ───┤             ├─── Cₒᵤₜ
              └─────────────┘
```

### 2.3 Ripple Carry Adder (RCA)

**Function**: Adds two n-bit numbers by cascading full adders

```
4-BIT RIPPLE CARRY ADDER:

    A₃ B₃     A₂ B₂     A₁ B₁     A₀ B₀
     │  │      │  │      │  │      │  │
     ▼  ▼      ▼  ▼      ▼  ▼      ▼  ▼
   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
   │      │  │      │  │      │  │      │
   │  FA  │◄─┤  FA  │◄─┤  FA  │◄─┤  FA  │◄── C₀ (usually 0)
   │      │  │      │  │      │  │      │
   └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
      │         │         │         │
     C₄        S₃        S₂        S₁        S₀
   (Cₒᵤₜ)

Timing Analysis:
  - Each FA has delay of 2 gate levels
  - Carry must propagate through all stages
  - Total delay = 2n gate delays (for n-bit adder)
  - For 4-bit: 8 gate delays

Example: Add 1011 + 0110 (11 + 6 = 17)
  
  Position:   3    2    1    0
        A:    1    0    1    1
        B:    0    1    1    0
       Cᵢₙ:   0←   1←   1←   0
      Sum:    0    0    0    1
      Cₒᵤₜ:   1    0    1    1
      
  Result: 10001₂ = 17₁₀ ✓
```

### 2.4 Carry Look-Ahead Adder (CLA)

**Concept**: Calculate carries in parallel to reduce delay

```
CARRY GENERATION AND PROPAGATION:

For each bit position i:
  Gᵢ = AᵢBᵢ         (Generate: produces carry regardless of Cᵢₙ)
  Pᵢ = Aᵢ ⊕ Bᵢ      (Propagate: passes carry from Cᵢₙ to Cₒᵤₜ)

Carry equations:
  C₁ = G₀ + P₀C₀
  C₂ = G₁ + P₁C₁ = G₁ + P₁G₀ + P₁P₀C₀
  C₃ = G₂ + P₂C₂ = G₂ + P₂G₁ + P₂P₁G₀ + P₂P₁P₀C₀
  C₄ = G₃ + P₃C₃ = G₃ + P₃G₂ + P₃P₂G₁ + P₃P₂P₁G₀ + P₃P₂P₁P₀C₀

4-BIT CARRY LOOK-AHEAD GENERATOR:

        A₀B₀  A₁B₁  A₂B₂  A₃B₃
          │     │     │     │
          ▼     ▼     ▼     ▼
       ┌────┐┌────┐┌────┐┌────┐
       │ PG ││ PG ││ PG ││ PG │   PG = Generate P and G
       └─┬──┘└─┬──┘└─┬──┘└─┬──┘
         │     │     │     │
    P₀G₀ │P₁G₁ │P₂G₂ │P₃G₃ │
         │     │     │     │
         ▼     ▼     ▼     ▼
    ┌──────────────────────────────────┐
    │       CARRY LOOK-AHEAD           │
    │           LOGIC                  │
    │                                  │
    │  C₁ = G₀ + P₀C₀                 │◄── C₀
    │  C₂ = G₁ + P₁G₀ + P₁P₀C₀        │
    │  C₃ = G₂ + P₂G₁ + P₂P₁G₀ +      │
    │       P₂P₁P₀C₀                  │
    │  C₄ = G₃ + P₃G₂ + P₃P₂G₁ +      │
    │       P₃P₂P₁G₀ + P₃P₂P₁P₀C₀     │
    └──────────────────────────────────┘
         │     │     │     │
         C₁    C₂    C₃    C₄

Timing Comparison:
┌─────────────────────┬─────────────┬─────────────┐
│                     │   Ripple    │     CLA     │
├─────────────────────┼─────────────┼─────────────┤
│ 4-bit adder delay   │  8 gates    │  4 gates    │
│ 16-bit adder delay  │  32 gates   │  8 gates    │
│ Complexity          │  Simple     │  Complex    │
│ Area                │  Less       │  More       │
└─────────────────────┴─────────────┴─────────────┘
```

---

## 3. Subtractors

### 3.1 Half Subtractor

**Function**: Subtracts two 1-bit numbers

```
HALF SUBTRACTOR TRUTH TABLE:
┌─────┬─────┬──────────┬────────┐
│  A  │  B  │ Diff (D) │ Borrow │
├─────┼─────┼──────────┼────────┤
│  0  │  0  │    0     │   0    │
│  0  │  1  │    1     │   1    │  (0-1 needs borrow)
│  1  │  0  │    1     │   0    │
│  1  │  1  │    0     │   0    │
└─────┴─────┴──────────┴────────┘

Boolean Expressions:
  Diff = A ⊕ B
  Borrow = A'B

HALF SUBTRACTOR CIRCUIT:
                      ┌─────┐
         A ───────────┤     │
                      │ XOR ├─────── Diff
         B ─────┬─────┤     │
                │     └─────┘
                │     ┌─────┐
         A ──|>o┼─────┤     │
                │     │ AND ├─────── Borrow
                └─────┤     │
                      └─────┘
```

### 3.2 Full Subtractor

**Function**: Subtracts with borrow-in

```
FULL SUBTRACTOR TRUTH TABLE:
┌─────┬─────┬─────┬──────────┬─────────┐
│  A  │  B  │ Bᵢₙ │ Diff (D) │ Bₒᵤₜ   │
├─────┼─────┼─────┼──────────┼─────────┤
│  0  │  0  │  0  │    0     │    0    │
│  0  │  0  │  1  │    1     │    1    │
│  0  │  1  │  0  │    1     │    1    │
│  0  │  1  │  1  │    0     │    1    │
│  1  │  0  │  0  │    1     │    0    │
│  1  │  0  │  1  │    0     │    0    │
│  1  │  1  │  0  │    0     │    0    │
│  1  │  1  │  1  │    1     │    1    │
└─────┴─────┴─────┴──────────┴─────────┘

Boolean Expressions:
  Diff = A ⊕ B ⊕ Bᵢₙ
  Bₒᵤₜ = A'B + A'Bᵢₙ + BBᵢₙ = A'(B + Bᵢₙ) + BBᵢₙ
       = A'B + (A ⊕ B)'Bᵢₙ
```

### 3.3 Adder-Subtractor

**Concept**: Use 2's complement for subtraction: A - B = A + (-B) = A + B' + 1

```
4-BIT ADDER-SUBTRACTOR:

                    M (Mode: 0=Add, 1=Subtract)
                    │
         ┌──────────┴──────────┐
         │                     │
    A₃ B₃│    A₂ B₂│    A₁ B₁│    A₀ B₀│
     │  ││     │  ││     │  ││     │  ││
     │  ▼│     │  ▼│     │  ▼│     │  ▼│
     │ ┌─┴─┐   │ ┌─┴─┐   │ ┌─┴─┐   │ ┌─┴─┐
     │ │XOR│   │ │XOR│   │ │XOR│   │ │XOR│
     │ └─┬─┘   │ └─┬─┘   │ └─┬─┘   │ └─┬─┘
     │   │     │   │     │   │     │   │
     ▼   ▼     ▼   ▼     ▼   ▼     ▼   ▼
   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
   │      │  │      │  │      │  │      │
   │  FA  │◄─┤  FA  │◄─┤  FA  │◄─┤  FA  │◄── M
   │      │  │      │  │      │  │      │
   └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
      │         │         │         │
     Cₒᵤₜ      S₃        S₂        S₁        S₀

When M = 0 (Addition):
  - XOR gates pass B unchanged (B ⊕ 0 = B)
  - Cᵢₙ = 0
  - Result = A + B

When M = 1 (Subtraction):
  - XOR gates invert B (B ⊕ 1 = B')
  - Cᵢₙ = 1 (adds the +1 for 2's complement)
  - Result = A + B' + 1 = A - B
```

---

## 4. Multiplexers

### What is a Multiplexer?

**Multiplexer (MUX)**: Selects one of many inputs and routes it to output

```
┌─────────────────────────────────────────────────────────────────┐
│                    MULTIPLEXER CONCEPT                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   "Data Selector" or "MUX"                                     │
│                                                                 │
│   2ⁿ-to-1 MUX:                                                 │
│     - 2ⁿ data inputs (I₀, I₁, ... I₂ₙ₋₁)                       │
│     - n select lines (S₀, S₁, ... Sₙ₋₁)                        │
│     - 1 output (Y)                                             │
│                                                                 │
│   Y = Σ(Iᵢ · mᵢ)  where mᵢ is minterm for select lines        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2-to-1 Multiplexer

```
TRUTH TABLE:
┌─────┬─────┬─────┬───────┐
│  S  │ I₀  │ I₁  │   Y   │
├─────┼─────┼─────┼───────┤
│  0  │  X  │  -  │  I₀   │
│  1  │  -  │  X  │  I₁   │
└─────┴─────┴─────┴───────┘

Y = S'I₀ + SI₁

CIRCUIT:
        I₀ ────────┐
                   │ D────┐
        S  ────|>o─┤      │
                   └──────┼────┐
                          │    │)─── Y
        I₁ ────────┐      │    │
                   │ D────┼────┘
        S  ────────┤      │
                   └──────┘

BLOCK SYMBOL:
              ┌─────────────┐
         I₀ ──┤0            │
              │    2:1      ├── Y
         I₁ ──┤1   MUX      │
              │             │
              └──────┬──────┘
                     │
                     S
```

### 4-to-1 Multiplexer

```
TRUTH TABLE:
┌─────┬─────┬───────────┐
│ S₁  │ S₀  │     Y     │
├─────┼─────┼───────────┤
│  0  │  0  │    I₀     │
│  0  │  1  │    I₁     │
│  1  │  0  │    I₂     │
│  1  │  1  │    I₃     │
└─────┴─────┴───────────┘

Y = S₁'S₀'I₀ + S₁'S₀I₁ + S₁S₀'I₂ + S₁S₀I₃

CIRCUIT:
        I₀ ─────────────┐
                        │ D────┐
        S₁' ────┐       │      │
                │ D─────┤      │
        S₀' ────┘       └──────┼─────┐
                               │     │
        I₁ ─────────────┐      │     │
                        │ D────┤     │
        S₁' ────┐       │      │     │
                │ D─────┤      │     │
        S₀  ────┘       └──────┼─────┼────┐
                               │     │    │)── Y
        I₂ ─────────────┐      │     │    │
                        │ D────┤     │    │
        S₁  ────┐       │      │     │    │
                │ D─────┤      │     │    │
        S₀' ────┘       └──────┼─────┼────┘
                               │     │
        I₃ ─────────────┐      │     │
                        │ D────┤     │
        S₁  ────┐       │      │     │
                │ D─────┤      │     │
        S₀  ────┘       └──────┘     │
                                     │
                                     │

BLOCK SYMBOL:
              ┌─────────────┐
         I₀ ──┤0            │
         I₁ ──┤1   4:1      │
         I₂ ──┤2   MUX      ├── Y
         I₃ ──┤3            │
              └─────┬───────┘
                    │
                 S₁ S₀
```

### Implementing Boolean Functions with MUX

**Method 1**: Using 2ⁿ-to-1 MUX for n-variable function

```
Example: F(A,B,C) = Σm(1,2,6,7)

Use 8-to-1 MUX with select lines A, B, C

Connect inputs based on truth table:
  m₀ = 0  →  I₀ = 0
  m₁ = 1  →  I₁ = 1
  m₂ = 1  →  I₂ = 1
  m₃ = 0  →  I₃ = 0
  m₄ = 0  →  I₄ = 0
  m₅ = 0  →  I₅ = 0
  m₆ = 1  →  I₆ = 1
  m₇ = 1  →  I₇ = 1

              ┌─────────────┐
       0  ────┤I₀           │
       1  ────┤I₁           │
       1  ────┤I₂   8:1     │
       0  ────┤I₃   MUX     ├── F
       0  ────┤I₄           │
       0  ────┤I₅           │
       1  ────┤I₆           │
       1  ────┤I₇           │
              └─────┬───────┘
                    │
                 A  B  C
```

**Method 2**: Using 2ⁿ⁻¹-to-1 MUX for n-variable function

```
Example: F(A,B,C) = Σm(1,2,6,7) using 4-to-1 MUX

Use A, B as select lines, express I in terms of C:

┌────┬────┬───────────────────────────────┐
│ A  │ B  │ F for C=0,1                   │
├────┼────┼───────────────────────────────┤
│ 0  │ 0  │ F(0,0,0)=0, F(0,0,1)=1 → I₀=C │
│ 0  │ 1  │ F(0,1,0)=1, F(0,1,1)=0 → I₁=C'│
│ 1  │ 0  │ F(1,0,0)=0, F(1,0,1)=0 → I₂=0 │
│ 1  │ 1  │ F(1,1,0)=1, F(1,1,1)=1 → I₃=1 │
└────┴────┴───────────────────────────────┘

              ┌─────────────┐
       C  ────┤I₀           │
       C' ────┤I₁   4:1     │
       0  ────┤I₂   MUX     ├── F
       1  ────┤I₃           │
              └─────┬───────┘
                    │
                 A   B
```

---

## 5. Demultiplexers

### What is a Demultiplexer?

**Demultiplexer (DEMUX)**: Routes single input to one of many outputs

```
┌─────────────────────────────────────────────────────────────────┐
│                  DEMULTIPLEXER CONCEPT                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   "Data Distributor" or "DEMUX"                                │
│                                                                 │
│   1-to-2ⁿ DEMUX:                                               │
│     - 1 data input (D)                                         │
│     - n select lines (S₀, S₁, ... Sₙ₋₁)                        │
│     - 2ⁿ outputs (Y₀, Y₁, ... Y₂ₙ₋₁)                           │
│                                                                 │
│   Selected output = D, all others = 0                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1-to-4 Demultiplexer

```
TRUTH TABLE:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ S₁  │ S₀  │ Y₀  │ Y₁  │ Y₂  │ Y₃  │
├─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  D  │  0  │  0  │  0  │
│  0  │  1  │  0  │  D  │  0  │  0  │
│  1  │  0  │  0  │  0  │  D  │  0  │
│  1  │  1  │  0  │  0  │  0  │  D  │
└─────┴─────┴─────┴─────┴─────┴─────┘

Boolean Expressions:
  Y₀ = D · S₁' · S₀'
  Y₁ = D · S₁' · S₀
  Y₂ = D · S₁  · S₀'
  Y₃ = D · S₁  · S₀

BLOCK SYMBOL:
              ┌─────────────┐
              │             ├── Y₀
         D ───┤    1:4      ├── Y₁
              │   DEMUX     ├── Y₂
              │             ├── Y₃
              └─────┬───────┘
                    │
                 S₁ S₀
```

### Relationship Between DEMUX and Decoder

A DEMUX is essentially a decoder with an enable input:
- **Decoder**: Outputs are minterms of select inputs
- **DEMUX**: Same as decoder, but outputs are ANDed with data input D

---

## 6. Encoders

### What is an Encoder?

**Encoder**: Converts 2ⁿ inputs to n-bit binary code

```
┌─────────────────────────────────────────────────────────────────┐
│                      ENCODER CONCEPT                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Opposite of Decoder                                          │
│                                                                 │
│   Assumption: Only ONE input is active at a time               │
│                                                                 │
│   4-to-2 Encoder: 4 inputs → 2-bit output                      │
│   8-to-3 Encoder: 8 inputs → 3-bit output                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4-to-2 Encoder

```
TRUTH TABLE (only one input active):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ I₃  │ I₂  │ I₁  │ I₀  │ Y₁  │ Y₀  │
├─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  1  │  0  │  0  │
│  0  │  0  │  1  │  0  │  0  │  1  │
│  0  │  1  │  0  │  0  │  1  │  0  │
│  1  │  0  │  0  │  0  │  1  │  1  │
└─────┴─────┴─────┴─────┴─────┴─────┘

Boolean Expressions:
  Y₀ = I₁ + I₃
  Y₁ = I₂ + I₃

CIRCUIT:
        I₁ ─────────┐
                    │)─────── Y₀
        I₃ ─────┬───┘
                │
        I₂ ─────┼───┐
                │   │)─────── Y₁
                └───┘
```

### 8-to-3 Priority Encoder

**Priority Encoder**: When multiple inputs active, highest priority input is encoded

```
PRIORITY ENCODER (I₇ = highest priority):
┌──────────────────────────────┬───────────────────┐
│          Inputs              │     Outputs       │
├────┬────┬────┬────┬────┬────┬────┬────┬────┬────┤
│ I₇ │ I₆ │ I₅ │ I₄ │ I₃ │ I₂ │ I₁ │ I₀ │Y₂Y₁Y₀│ V │
├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
│ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ XXX │ 0 │
│ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ 1  │ 000 │ 1 │
│ 0  │ 0  │ 0  │ 0  │ 0  │ 0  │ 1  │ X  │ 001 │ 1 │
│ 0  │ 0  │ 0  │ 0  │ 0  │ 1  │ X  │ X  │ 010 │ 1 │
│ 0  │ 0  │ 0  │ 0  │ 1  │ X  │ X  │ X  │ 011 │ 1 │
│ 0  │ 0  │ 0  │ 1  │ X  │ X  │ X  │ X  │ 100 │ 1 │
│ 0  │ 0  │ 1  │ X  │ X  │ X  │ X  │ X  │ 101 │ 1 │
│ 0  │ 1  │ X  │ X  │ X  │ X  │ X  │ X  │ 110 │ 1 │
│ 1  │ X  │ X  │ X  │ X  │ X  │ X  │ X  │ 111 │ 1 │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘

V = Valid output (at least one input active)
X = Don't care

BLOCK SYMBOL:
              ┌─────────────┐
         I₀ ──┤             │
         I₁ ──┤    8-to-3   ├── Y₀
         I₂ ──┤   PRIORITY  ├── Y₁
         I₃ ──┤   ENCODER   ├── Y₂
         I₄ ──┤             │
         I₅ ──┤             ├── V (Valid)
         I₆ ──┤             │
         I₇ ──┤             │
              └─────────────┘
```

---

## 7. Decoders

### What is a Decoder?

**Decoder**: Converts n-bit input to 2ⁿ output lines (one-hot encoding)

```
┌─────────────────────────────────────────────────────────────────┐
│                      DECODER CONCEPT                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   n inputs → 2ⁿ outputs                                        │
│   Exactly ONE output is active for each input combination      │
│   Outputs are minterms of inputs                               │
│                                                                 │
│   Common types: 2-to-4, 3-to-8, 4-to-16                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2-to-4 Decoder

```
TRUTH TABLE:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│  E  │ A₁  │ A₀  │ Y₀  │ Y₁  │ Y₂  │ Y₃  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  X  │  X  │  0  │  0  │  0  │  0  │  (disabled)
│  1  │  0  │  0  │  1  │  0  │  0  │  0  │
│  1  │  0  │  1  │  0  │  1  │  0  │  0  │
│  1  │  1  │  0  │  0  │  0  │  1  │  0  │
│  1  │  1  │  1  │  0  │  0  │  0  │  1  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Boolean Expressions:
  Y₀ = E · A₁' · A₀'
  Y₁ = E · A₁' · A₀
  Y₂ = E · A₁  · A₀'
  Y₃ = E · A₁  · A₀

CIRCUIT:
        A₀ ──────┬────────────────────────────────
                 │            │            │
        A₁ ──────┼────────────┼────────────┼──────
                 │     │      │     │      │     │
                NOT   NOT    NOT   NOT
                 │     │      │     │      │     │
                 ├──┬──┤      ├──┬──┤      ├──┬──┤
        E ───────│  │  │──────│  │  │──────│  │  │
                AND│ AND│    AND│ AND│    AND│ AND│
                 │     │      │     │      │     │
                 Y₀    Y₁     Y₂    Y₃

BLOCK SYMBOL:
              ┌─────────────┐
              │             ├── Y₀
         A₀ ──┤    2:4      ├── Y₁
         A₁ ──┤  DECODER    ├── Y₂
          E ──┤             ├── Y₃
              └─────────────┘
```

### 3-to-8 Decoder

```
3-TO-8 DECODER USING 2-TO-4 DECODERS:

                              ┌─────────────┐
                              │             ├── Y₀ (A₂'A₁'A₀')
                      A₀ ─────┤    2:4      ├── Y₁ (A₂'A₁'A₀)
                      A₁ ─────┤  DECODER    ├── Y₂ (A₂'A₁A₀')
              A₂' ────────────┤     E       ├── Y₃ (A₂'A₁A₀)
                              └─────────────┘
                              
                              ┌─────────────┐
                              │             ├── Y₄ (A₂A₁'A₀')
                      A₀ ─────┤    2:4      ├── Y₅ (A₂A₁'A₀)
                      A₁ ─────┤  DECODER    ├── Y₆ (A₂A₁A₀')
              A₂  ────────────┤     E       ├── Y₇ (A₂A₁A₀)
                              └─────────────┘
```

### Implementing Functions with Decoders

**Method**: Decoder outputs are minterms, OR together required minterms

```
Example: F(A,B,C) = Σm(1,2,6,7) using 3-to-8 decoder

              ┌─────────────┐
              │             ├── Y₀
         A ───┤             ├── Y₁ ──┐
         B ───┤    3:8      ├── Y₂ ──┼──┐
         C ───┤  DECODER    ├── Y₃   │  │
         1 ───┤     E       ├── Y₄   │  │
              │             ├── Y₅   │  │
              │             ├── Y₆ ──┼──┼──┐
              │             ├── Y₇ ──┼──┼──┼──┐
              └─────────────┘        │  │  │  │
                                     │  │  │  │
                                     └──┴──┴──┴──)── F
                                         OR gate
                                         
F = Y₁ + Y₂ + Y₆ + Y₇
```

---

## 8. Code Converters

### BCD to Excess-3 Converter

```
DESIGN PROCEDURE:

Step 1: Truth Table
┌──────────────────┬──────────────────┐
│       BCD        │     Excess-3     │
├────┬────┬────┬───┼────┬────┬────┬───┤
│ A  │ B  │ C  │ D │ W  │ X  │ Y  │ Z │
├────┼────┼────┼───┼────┼────┼────┼───┤
│ 0  │ 0  │ 0  │ 0 │ 0  │ 0  │ 1  │ 1 │  0→3
│ 0  │ 0  │ 0  │ 1 │ 0  │ 1  │ 0  │ 0 │  1→4
│ 0  │ 0  │ 1  │ 0 │ 0  │ 1  │ 0  │ 1 │  2→5
│ 0  │ 0  │ 1  │ 1 │ 0  │ 1  │ 1  │ 0 │  3→6
│ 0  │ 1  │ 0  │ 0 │ 0  │ 1  │ 1  │ 1 │  4→7
│ 0  │ 1  │ 0  │ 1 │ 1  │ 0  │ 0  │ 0 │  5→8
│ 0  │ 1  │ 1  │ 0 │ 1  │ 0  │ 0  │ 1 │  6→9
│ 0  │ 1  │ 1  │ 1 │ 1  │ 0  │ 1  │ 0 │  7→10
│ 1  │ 0  │ 0  │ 0 │ 1  │ 0  │ 1  │ 1 │  8→11
│ 1  │ 0  │ 0  │ 1 │ 1  │ 1  │ 0  │ 0 │  9→12
└────┴────┴────┴───┴────┴────┴────┴───┘
(10-15 are don't cares)

Step 2: K-maps for each output (W, X, Y, Z)

Step 3: Minimized expressions:
  Z = D'
  Y = C'D + CD' = C ⊕ D
  X = B'C + B'D + BC'D' = B'(C+D) + BCD' = B ⊕ (C+D)
  W = A + BC + BD = A + B(C+D)
```

### Binary to Gray Code Converter

```
CONVERSION RULES:
  G₃ = B₃
  G₂ = B₃ ⊕ B₂
  G₁ = B₂ ⊕ B₁
  G₀ = B₁ ⊕ B₀

CIRCUIT:
    B₃ ──────────────────────────────── G₃
        │
        └────┐
             │)────────────────────────── G₂
    B₂ ──────┤
        │    └────┐
        └─────────│)───────────────────── G₁
    B₁ ──────────┤
        │        └────┐
        └─────────────│)───────────────── G₀
    B₀ ───────────────┘
```

### Gray to Binary Converter

```
CONVERSION RULES:
  B₃ = G₃
  B₂ = G₃ ⊕ G₂ = B₃ ⊕ G₂
  B₁ = B₂ ⊕ G₁
  B₀ = B₁ ⊕ G₀

CIRCUIT:
    G₃ ──────────────────────────────── B₃
         │
         └────┐
              │)───────────────────────── B₂
    G₂ ───────┤        │
              │        └────┐
              │             │)─────────── B₁
    G₁ ───────│─────────────┤      │
              │             │      └────┐
              │             │           │)── B₀
    G₀ ───────│─────────────│───────────┘
```

---

## 9. Comparators

### What is a Comparator?

**Comparator**: Compares two binary numbers and indicates relationship

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMPARATOR OUTPUTS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Three possible relationships:                                 │
│     1. A > B  (Greater than)                                   │
│     2. A = B  (Equal)                                          │
│     3. A < B  (Less than)                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1-Bit Comparator

```
TRUTH TABLE:
┌─────┬─────┬───────┬───────┬───────┐
│  A  │  B  │ A > B │ A = B │ A < B │
├─────┼─────┼───────┼───────┼───────┤
│  0  │  0  │   0   │   1   │   0   │
│  0  │  1  │   0   │   0   │   1   │
│  1  │  0  │   1   │   0   │   0   │
│  1  │  1  │   0   │   1   │   0   │
└─────┴─────┴───────┴───────┴───────┘

Boolean Expressions:
  (A > B) = AB'
  (A = B) = A'B' + AB = (A ⊕ B)'
  (A < B) = A'B
```

### 4-Bit Magnitude Comparator

```
COMPARISON LOGIC:

For 4-bit numbers A = A₃A₂A₁A₀ and B = B₃B₂B₁B₀:

First, compare bit-by-bit (xᵢ = 1 if Aᵢ = Bᵢ):
  x₃ = A₃ ⊙ B₃
  x₂ = A₂ ⊙ B₂
  x₁ = A₁ ⊙ B₁
  x₀ = A₀ ⊙ B₀

Then:
  (A = B) = x₃ · x₂ · x₁ · x₀
  
  (A > B) = A₃B₃' + x₃A₂B₂' + x₃x₂A₁B₁' + x₃x₂x₁A₀B₀'
  
  (A < B) = A₃'B₃ + x₃A₂'B₂ + x₃x₂A₁'B₁ + x₃x₂x₁A₀'B₀

BLOCK SYMBOL (74HC85):
              ┌─────────────────┐
         A₀ ──┤                 │
         A₁ ──┤                 ├── A > B
         A₂ ──┤   4-BIT         ├── A = B
         A₃ ──┤  COMPARATOR     ├── A < B
         B₀ ──┤                 │
         B₁ ──┤                 │
         B₂ ──┤   (74HC85)      │
         B₃ ──┤                 │
              │                 │
    (A>B)ᵢₙ ──┤  Cascade        │
    (A=B)ᵢₙ ──┤  inputs         │
    (A<B)ᵢₙ ──┤                 │
              └─────────────────┘
```

---

## 10. Parity Generators and Checkers

### Parity Generator

```
EVEN PARITY GENERATOR (for 3-bit data):

Data bits: D₂, D₁, D₀
Parity bit P makes total 1s even

P = D₂ ⊕ D₁ ⊕ D₀

TRUTH TABLE:
┌─────┬─────┬─────┬─────┐
│ D₂  │ D₁  │ D₀  │  P  │
├─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  0  │  (0 ones, even)
│  0  │  0  │  1  │  1  │  (1 one, add 1)
│  0  │  1  │  0  │  1  │  (1 one, add 1)
│  0  │  1  │  1  │  0  │  (2 ones, even)
│  1  │  0  │  0  │  1  │  (1 one, add 1)
│  1  │  0  │  1  │  0  │  (2 ones, even)
│  1  │  1  │  0  │  0  │  (2 ones, even)
│  1  │  1  │  1  │  1  │  (3 ones, add 1)
└─────┴─────┴─────┴─────┘

CIRCUIT:
    D₀ ────┐
           │)────┐
    D₁ ────┘     │)──── P (even parity)
                 │
    D₂ ──────────┘
```

### Parity Checker

```
EVEN PARITY CHECKER (for 4-bit received data including parity):

Received: D₂, D₁, D₀, P
Error check: E = D₂ ⊕ D₁ ⊕ D₀ ⊕ P

If E = 0: No error (even number of 1s)
If E = 1: Error detected (odd number of 1s)

CIRCUIT:
    D₀ ────┐
           │)────┐
    D₁ ────┘     │
                 │)────┐
    D₂ ──────────┘     │)──── E (Error flag)
                       │
    P  ────────────────┘
    
E = 0: Valid (no error or even # of errors)
E = 1: Error (odd # of bit errors)
```

---

## 11. ALU Design

### What is an ALU?

```
┌─────────────────────────────────────────────────────────────────┐
│            ARITHMETIC LOGIC UNIT (ALU)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Central component of CPU                                      │
│                                                                 │
│   Performs:                                                     │
│     - Arithmetic operations (ADD, SUB, INC, DEC)               │
│     - Logic operations (AND, OR, NOT, XOR)                     │
│     - Comparison operations                                     │
│     - Shift operations (in some ALUs)                          │
│                                                                 │
│   Control inputs (opcode) select the operation                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Simple 1-Bit ALU Slice

```
1-BIT ALU SLICE:

Inputs: A, B, Cᵢₙ, S₁, S₀ (operation select)
Outputs: Result, Cₒᵤₜ

OPERATION TABLE:
┌─────┬─────┬─────────────────┐
│ S₁  │ S₀  │   Operation     │
├─────┼─────┼─────────────────┤
│  0  │  0  │   A AND B       │
│  0  │  1  │   A OR B        │
│  1  │  0  │   A + B (add)   │
│  1  │  1  │   A XOR B       │
└─────┴─────┴─────────────────┘

1-BIT ALU CIRCUIT:

         A ───┬───────────────────────────────────────┐
              │                                       │
         B ───┼───┬───────────────────────────────────┼───┐
              │   │                                   │   │
              │   │  ┌─────┐                          │   │
              ├───┼──┤ AND ├───────────────────────── I₀  │
              │   │  └─────┘                          │   │
              │   │                                   │   │
              │   │  ┌─────┐                          │   │
              ├───┼──┤ OR  ├───────────────────────── I₁  │
              │   │  └─────┘                          │   │
              │   │                                   │   │
              │   │  ┌─────────┐                 ┌────┴───┴────┐
              └───┴──┤         │       Sum ───── I₂           │
         Cᵢₙ ────────┤   FA    │                 │    4:1     ├── Result
                     │         ├── Cₒᵤₜ          │    MUX     │
                     └─────────┘                 │            │
                                          ┌───── I₃           │
              A ──────┬────┐               │     └─────┬──────┘
                      │    │)──────────────┘           │
              B ──────┘                             S₁ S₀
```

### 4-Bit ALU (74181-style)

```
4-BIT ALU BLOCK DIAGRAM:

                    ┌─────────────────────────────┐
           A₀-A₃ ───┤                             │
                    │                             ├─── F₀-F₃ (Result)
           B₀-B₃ ───┤         4-BIT ALU           │
                    │                             ├─── Cₒᵤₜ
           Cᵢₙ   ───┤                             │
                    │                             ├─── A=B (Compare)
           S₀-S₃ ───┤   (Operation Select)        │
                    │                             ├─── G (Generate)
              M  ───┤   (Mode: 0=Arith, 1=Logic)  │
                    │                             ├─── P (Propagate)
                    └─────────────────────────────┘

74181 OPERATION TABLE (selected operations):
┌─────┬───────────────┬─────────────────────┬────────────────────┐
│  M  │    S₃S₂S₁S₀   │  Logic (M=1)        │  Arithmetic (M=0)  │
├─────┼───────────────┼─────────────────────┼────────────────────┤
│  0  │     0000      │      A'             │    A               │
│  0  │     0001      │     (AB)'           │    A + B           │
│  0  │     0011      │      0              │    A + B + 1       │
│  0  │     0110      │    A ⊕ B            │    A - B - 1       │
│  0  │     1001      │   (A ⊕ B)'          │    A + B           │
│  0  │     1100      │      1              │    A + A           │
│  1  │     XXXX      │   Logic only        │       -            │
└─────┴───────────────┴─────────────────────┴────────────────────┘
```

### ALU Status Flags

```
┌─────────────────────────────────────────────────────────────────┐
│                      ALU STATUS FLAGS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Z (Zero):      Set if result = 0                              │
│                 Z = (F₃ + F₂ + F₁ + F₀)'                       │
│                                                                 │
│  N (Negative):  Set if result is negative (MSB = 1)            │
│                 N = F₃ (in 2's complement)                     │
│                                                                 │
│  C (Carry):     Set if carry out of MSB                        │
│                 C = Cₒᵤₜ                                        │
│                                                                 │
│  V (Overflow):  Set if signed overflow occurred                │
│                 V = Cₒᵤₜ ⊕ C₃ (carry into MSB ≠ carry out)     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

FLAGS GENERATION CIRCUIT:

    F₃ ─────────────────────────────────────── N (Negative)
        │
        ├───┐
    F₂ ─┼───┤
        │   │
    F₁ ─┼───┼──)──────────────────|>o───────── Z (Zero)
        │   │
    F₀ ─┼───┘
        │
   Cₒᵤₜ ────────────────────────────────────── C (Carry)
        │
        └───┐
            │)────────────────────────────────── V (Overflow)
   C₃  ─────┘
```

---

## 12. Summary Tables

### Adder Summary

| Type | Function | Delay | Complexity |
|------|----------|-------|------------|
| Half Adder | A + B (2 inputs) | 2 gates | Low |
| Full Adder | A + B + Cᵢₙ (3 inputs) | 2-3 gates | Low |
| Ripple Carry | n-bit addition | 2n gates | O(n) |
| Carry Look-ahead | n-bit addition | 4 gates | O(n²) |

### MUX/DEMUX Summary

| Device | Inputs | Outputs | Select Lines | Function |
|--------|--------|---------|--------------|----------|
| 2:1 MUX | 2 data, 1 select | 1 | 1 | Select 1 of 2 |
| 4:1 MUX | 4 data, 2 select | 1 | 2 | Select 1 of 4 |
| 8:1 MUX | 8 data, 3 select | 1 | 3 | Select 1 of 8 |
| 1:2 DEMUX | 1 data, 1 select | 2 | 1 | Route to 1 of 2 |
| 1:4 DEMUX | 1 data, 2 select | 4 | 2 | Route to 1 of 4 |

### Encoder/Decoder Summary

| Device | Inputs | Outputs | Function |
|--------|--------|---------|----------|
| 4:2 Encoder | 4 | 2 | One-hot to binary |
| 8:3 Priority Encoder | 8 | 3 + Valid | Priority encoding |
| 2:4 Decoder | 2 + Enable | 4 | Binary to one-hot |
| 3:8 Decoder | 3 + Enable | 8 | Binary to one-hot |

### IC Numbers Reference

| Function | TTL IC | CMOS IC |
|----------|--------|---------|
| Quad 2-input AND | 7408 | 4081 |
| Quad 2-input OR | 7432 | 4071 |
| Hex Inverter | 7404 | 4069 |
| Quad 2-input NAND | 7400 | 4011 |
| 4-bit Full Adder | 7483 | 4008 |
| 8:1 MUX | 74151 | 4512 |
| 3:8 Decoder | 74138 | 4028 |
| 4-bit Comparator | 7485 | 4585 |
| 4-bit ALU | 74181 | - |

---

## 13. Quick Revision Questions

### Question 1
Design a half subtractor and derive its Boolean expressions.

<details>
<summary>Click for Answer</summary>

```
Truth Table:
┌───┬───┬──────┬────────┐
│ A │ B │ Diff │ Borrow │
├───┼───┼──────┼────────┤
│ 0 │ 0 │  0   │   0    │
│ 0 │ 1 │  1   │   1    │
│ 1 │ 0 │  1   │   0    │
│ 1 │ 1 │  0   │   0    │
└───┴───┴──────┴────────┘

Diff = A'B + AB' = A ⊕ B
Borrow = A'B

Circuit: XOR gate for Diff, AND with A inverted for Borrow
```
</details>

### Question 2
Implement F(A,B,C) = Σm(0,2,5,7) using an 8:1 MUX.

<details>
<summary>Click for Answer</summary>

```
Use A, B, C as select lines S₂, S₁, S₀

Connect inputs:
  I₀ = 1 (m₀ = 1)
  I₁ = 0 (m₁ = 0)
  I₂ = 1 (m₂ = 1)
  I₃ = 0 (m₃ = 0)
  I₄ = 0 (m₄ = 0)
  I₅ = 1 (m₅ = 1)
  I₆ = 0 (m₆ = 0)
  I₇ = 1 (m₇ = 1)

              ┌─────────────┐
       1  ────┤I₀           │
       0  ────┤I₁   8:1     │
       1  ────┤I₂   MUX     ├── F
       0  ────┤I₃           │
       0  ────┤I₄           │
       1  ────┤I₅           │
       0  ────┤I₆           │
       1  ────┤I₇           │
              └──────┬──────┘
                  A  B  C
```
</details>

### Question 3
What is the propagation delay of an 8-bit ripple carry adder if each full adder has a carry propagation delay of 10ns?

<details>
<summary>Click for Answer</summary>

```
In ripple carry adder:
  - Carry must propagate through all stages
  - Each stage adds one carry propagation delay
  
For n-bit RCA: Delay = n × (carry delay)

For 8-bit with 10ns per stage:
Delay = 8 × 10ns = 80ns

Note: This is worst case (all carries propagate)
```
</details>

### Question 4
Design a circuit to implement F(A,B,C) = Σm(1,3,5,7) using a 3-to-8 decoder and OR gate.

<details>
<summary>Click for Answer</summary>

```
F = Σm(1,3,5,7)

              ┌─────────────┐
              │             ├── Y₀
         A ───┤             ├── Y₁ ──┐
         B ───┤    3:8      ├── Y₂   │
         C ───┤  DECODER    ├── Y₃ ──┼──┐
         1 ───┤     E       ├── Y₄   │  │
              │             ├── Y₅ ──┼──┼──┐
              │             ├── Y₆   │  │  │
              │             ├── Y₇ ──┼──┼──┼──┐
              └─────────────┘        │  │  │  │
                                     └──┴──┴──┴──)── F

F = Y₁ + Y₃ + Y₅ + Y₇

Notice: All minterms where C = 1
Simplified: F = C
```
</details>

### Question 5
What is the difference between a decoder and a demultiplexer?

<details>
<summary>Click for Answer</summary>

```
DECODER:
- Converts n-bit input to 2ⁿ outputs
- Outputs are minterms of select inputs
- Used for: Address decoding, instruction decoding
- Enable can be used to cascade or control

DEMULTIPLEXER:
- Routes 1 input to one of 2ⁿ outputs
- Has data input D plus select lines
- Selected output = D, others = 0
- Used for: Data routing, serial-to-parallel

Relationship:
- DEMUX = Decoder with data input as Enable
- Decoder with E input can work as DEMUX
```
</details>

### Question 6
In a 4-bit CLA adder, write the expression for C₂ in terms of G, P, and C₀.

<details>
<summary>Click for Answer</summary>

```
Carry Look-Ahead equations:

Where:
  Gᵢ = AᵢBᵢ (Generate)
  Pᵢ = Aᵢ ⊕ Bᵢ (Propagate)

C₁ = G₀ + P₀C₀

C₂ = G₁ + P₁C₁
   = G₁ + P₁(G₀ + P₀C₀)
   = G₁ + P₁G₀ + P₁P₀C₀

Therefore:
C₂ = G₁ + P₁G₀ + P₁P₀C₀

Interpretation:
- C₂ = 1 if:
  - G₁ generates carry, OR
  - G₀ generates and P₁ propagates, OR
  - C₀ exists and both P₀ and P₁ propagate
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 3: Minimization](Unit-03-Minimization-Techniques.md) | Return to Index | [Unit 5: Flip-Flops](Unit-05-Flip-Flops.md) |

---

*End of Unit 4*
