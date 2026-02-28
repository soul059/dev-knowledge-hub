# Chapter 10.4: Logic Gates

[← Previous: Minimization with K-Maps](03-minimization-kmaps.md) | [Back to README](../README.md) | [Next: Circuit Design →](05-circuit-design.md)

---

## 📋 Chapter Overview

**Logic gates** are the physical building blocks of digital circuits, implementing Boolean operations in hardware. This chapter covers all fundamental gates with their **symbols (ASCII art)**, truth tables, Boolean expressions, and the concept of **universal gates** (NAND and NOR).

---

## 1. Basic Logic Gates

### AND Gate

Outputs 1 only when **all** inputs are 1.

```
  Symbol:                    Truth Table:
                             ┌───┬───┬───────┐
  A ───┐                     │ A │ B │ A·B   │
       │  D  \               ├───┼───┼───────┤
       │      )───── Y       │ 0 │ 0 │   0   │
       │  D  /               │ 0 │ 1 │   0   │
  B ───┘                     │ 1 │ 0 │   0   │
                             │ 1 │ 1 │   1   │
  Y = A · B                  └───┴───┴───────┘
```

```
  ASCII Circuit Diagram:
  
  A ──────┐
          │ AND ├──── Y = A·B
  B ──────┘
```

### OR Gate

Outputs 1 when **at least one** input is 1.

```
  Symbol:                    Truth Table:
                             ┌───┬───┬───────┐
  A ───\                     │ A │ B │ A+B   │
        \                    ├───┼───┼───────┤
         )────── Y           │ 0 │ 0 │   0   │
        /                    │ 0 │ 1 │   1   │
  B ───/                     │ 1 │ 0 │   1   │
                             │ 1 │ 1 │   1   │
  Y = A + B                  └───┴───┴───────┘
```

```
  ASCII Circuit Diagram:
  
  A ──────┐
          │ OR  ├──── Y = A+B
  B ──────┘
```

### NOT Gate (Inverter)

Outputs the **complement** of the input.

```
  Symbol:                    Truth Table:
                             ┌───┬──────┐
  A ────▷○──── Y             │ A │  Ā   │
                             ├───┼──────┤
  Y = Ā                      │ 0 │  1   │
                             │ 1 │  0   │
                             └───┴──────┘
```

```
  ASCII Circuit Diagram:
  
  A ──── NOT ──── Y = Ā
         (○)
```

---

## 2. Derived Gates

### NAND Gate (NOT-AND)

```
  Symbol:                    Truth Table:
                             ┌───┬───┬──────────┐
  A ───┐                     │ A │ B │ (A·B)'   │
       │  D  \               ├───┼───┼──────────┤
       │      )○──── Y       │ 0 │ 0 │    1     │
       │  D  /               │ 0 │ 1 │    1     │
  B ───┘                     │ 1 │ 0 │    1     │
                             │ 1 │ 1 │    0     │
  Y = (A · B)' = Ā + B̄       └───┴───┴──────────┘
  (○ = bubble = inversion)
```

```
  ASCII Circuit:
  
  A ──────┐
          │NAND ├○──── Y = (A·B)'
  B ──────┘
```

### NOR Gate (NOT-OR)

```
  Symbol:                    Truth Table:
                             ┌───┬───┬──────────┐
  A ───\                     │ A │ B │ (A+B)'   │
        \                    ├───┼───┼──────────┤
         )○───── Y           │ 0 │ 0 │    1     │
        /                    │ 0 │ 1 │    0     │
  B ───/                     │ 1 │ 0 │    0     │
                             │ 1 │ 1 │    0     │
  Y = (A + B)' = Ā · B̄       └───┴───┴──────────┘
```

```
  ASCII Circuit:
  
  A ──────┐
          │ NOR ├○──── Y = (A+B)'
  B ──────┘
```

### XOR Gate (Exclusive OR)

```
  Symbol:                    Truth Table:
                             ┌───┬───┬───────┐
  A ───\                     │ A │ B │ A⊕B   │
       \\                    ├───┼───┼───────┤
        ))────── Y           │ 0 │ 0 │   0   │
       //                    │ 0 │ 1 │   1   │
  B ───/                     │ 1 │ 0 │   1   │
                             │ 1 │ 1 │   0   │
  Y = A ⊕ B = ĀB + AB̄       └───┴───┴───────┘
```

### XNOR Gate (Equivalence)

```
  Symbol:                    Truth Table:
                             ┌───┬───┬───────┐
  A ───\                     │ A │ B │ A⊙B   │
       \\                    ├───┼───┼───────┤
        ))○───── Y           │ 0 │ 0 │   1   │
       //                    │ 0 │ 1 │   0   │
  B ───/                     │ 1 │ 0 │   0   │
                             │ 1 │ 1 │   1   │
  Y = A ⊙ B = AB + ĀB̄       └───┴───┴───────┘
```

---

## 3. Complete Gate Summary

```
  ┌────────┬──────────────┬──────────────────────────┐
  │  Gate  │   Expression │  Output = 1 when...      │
  ├────────┼──────────────┼──────────────────────────┤
  │  AND   │   A · B      │  Both inputs are 1       │
  │  OR    │   A + B      │  At least one is 1       │
  │  NOT   │   Ā          │  Input is 0              │
  │  NAND  │   (A·B)'     │  NOT both 1              │
  │  NOR   │   (A+B)'     │  Both inputs are 0       │
  │  XOR   │   A ⊕ B      │  Inputs differ           │
  │  XNOR  │   A ⊙ B      │  Inputs are same         │
  │ Buffer │   A          │  Input is 1 (no change)  │
  └────────┴──────────────┴──────────────────────────┘
```

---

## 4. Universal Gates: NAND

**NAND is a universal gate** — any Boolean function can be built using **only NAND gates**.

### Implementing Basic Gates with NAND

```
  NOT from NAND:
  
  A ──┐
      │NAND├○──── Ā
  A ──┘
  
  A NAND A = (A·A)' = A' = Ā  ✓
  
  ─────────────────────────────────────
  
  AND from NAND (2 gates):
  
  A ──┐                    ┌──┐
      │NAND├○───┤          │
  B ──┘         ├NAND├○──── A·B
                │          │
           ─────┘          └──┘
  
  Step 1: A NAND B = (AB)'
  Step 2: (AB)' NAND (AB)' = ((AB)')' = AB  ✓
  
  Simplified:
  A ──┐              ┌──┐
      │NAND├○──────┤  │
  B ──┘             ├NAND├○──── A·B
               ┌────┘  │
               └───────┘
  
  ─────────────────────────────────────
  
  OR from NAND (3 gates):
  
  A ──┐                          ┌──┐
      │NAND├○──── Ā ────────┤    │
  A ──┘                     ├NAND├○──── A+B
                             │    │
  B ──┐                ┌────┘    └──┘
      │NAND├○──── B̄ ──┘
  B ──┘
  
  Ā NAND B̄ = (Ā · B̄)' = A + B  (De Morgan's) ✓
```

---

## 5. Universal Gates: NOR

**NOR is also a universal gate.**

### Implementing Basic Gates with NOR

```
  NOT from NOR:
  
  A ──┐
      │ NOR├○──── Ā
  A ──┘
  
  A NOR A = (A+A)' = A' = Ā  ✓
  
  ─────────────────────────────────────
  
  OR from NOR (2 gates):
  
  A ──┐               ┌──┐
      │NOR ├○────────┤  │
  B ──┘              ├NOR ├○──── A+B
                ┌────┘  │
                └───────┘
  
  (A+B)' NOR (A+B)' = ((A+B)')' = A+B  ✓
  
  ─────────────────────────────────────
  
  AND from NOR (3 gates):
  
  A ──┐                          ┌──┐
      │NOR ├○──── Ā ───────┤    │
  A ──┘                    ├NOR ├○──── A·B
                            │    │
  B ──┐               ┌────┘    └──┘
      │NOR ├○──── B̄ ──┘
  B ──┘
  
  Ā NOR B̄ = (Ā + B̄)' = A · B  (De Morgan's) ✓
```

---

## 6. Multi-Input Gates

Most gates extend naturally to more than 2 inputs:

```
  3-input AND:          3-input OR:
  A ──┐                 A ──\
  B ──┤ AND ├── Y       B ──  OR  ├── Y
  C ──┘                 C ──/
  
  Y = A · B · C         Y = A + B + C
  
  3-input NAND:          3-input NOR:
  A ──┐                  A ──\
  B ──┤NAND├○── Y        B ──  NOR ├○── Y
  C ──┘                  C ──/
  
  Y = (ABC)'             Y = (A+B+C)'
  
  3-input XOR:
  A ──\
  B ── XOR ├── Y
  C ──/
  
  Y = A ⊕ B ⊕ C  (odd parity: 1 when odd number of inputs are 1)
```

---

## 7. Gate-Level Implementation

### Example: Implement $f = AB + \overline{C}$

```
  Step 1: Identify operations
    - AND: A · B
    - NOT: C̄
    - OR:  (A·B) + C̄
  
  Circuit:
  
  A ──────┐
          │ AND ├──────┐
  B ──────┘            │
                       │ OR ├──── f = AB + C̄
  C ──── NOT ──────────┘
          (○)
  
  Gate count: 3 (1 AND + 1 NOT + 1 OR)
  Gate inputs: 5 (2 + 1 + 2)
```

### Example: Implement $f = \overline{A}B + A\overline{B}$ (XOR)

```
  A ──────┬──── NOT ────┐
          │     (○)     │ AND ├────┐
  B ──────│─────────────┘          │
          │                        │ OR ├──── f = A⊕B
          │     ┌── NOT ────┐      │
          │     │    (○)    │ AND ├─┘
  B ──────┴─────┘          │
  A ────────────────────────┘
  
  Redrawn clearly:
  
          ┌── NOT ──┐
  A ──┬───┤         ├── AND₁ ──┐
      │   └─────────┘   ↑      │
      │            B ────┘      │ OR ──── f
      │                         │
      │   ┌── NOT ──┐          │
  B ──┴───┤         ├── AND₂ ──┘
          └─────────┘   ↑
                   A ───┘
  
  Gate count: 5 (2 NOT + 2 AND + 1 OR)
```

---

## 8. NAND-NAND Implementation

Any SOP expression can be implemented using **only NAND gates** in a **two-level** circuit:

```
  Theorem: SOP → NAND-NAND
  
  f = AB + CD  (SOP form)
  
  Apply double complement:
  f = ((AB + CD)')' 
    = ((AB)' · (CD)')' 
    = (AB)' NAND (CD)'
  
  Level 1: NAND each product term
  Level 2: NAND the results together
  
  Circuit:
  
  A ──┐
      │NAND├○──────┐
  B ──┘             │
                    │NAND├○──── f = AB + CD
  C ──┐             │
      │NAND├○──────┘
  D ──┘
  
  This works because NAND(NAND(A,B), NAND(C,D))
  = ((AB)' · (CD)')' = AB + CD  ✓
```

---

## 9. NOR-NOR Implementation

Any POS expression can be implemented using **only NOR gates**:

```
  Theorem: POS → NOR-NOR
  
  f = (A+B)(C+D)  (POS form)
  
  Apply double complement:
  f = (((A+B)(C+D))')'
    = ((A+B)' + (C+D)')'
    = (A+B)' NOR (C+D)'
  
  Circuit:
  
  A ──┐
      │NOR ├○──────┐
  B ──┘             │
                    │NOR ├○──── f = (A+B)(C+D)
  C ──┐             │
      │NOR ├○──────┘
  D ──┘
```

---

## 10. Positive vs. Negative Logic

```
  ┌─────────────────────────────────────────────────┐
  │  Positive Logic:  High voltage = 1, Low = 0    │
  │  Negative Logic:  High voltage = 0, Low = 1    │
  │                                                  │
  │  A positive-logic AND gate behaves as a          │
  │  negative-logic OR gate, and vice versa!         │
  │                                                  │
  │  Positive AND = Negative OR                      │
  │  Positive OR  = Negative AND                     │
  │  Positive NAND = Negative NOR                    │
  └─────────────────────────────────────────────────┘
```

---

## 11. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │          Logic Gate Applications                   │
  │                                                    │
  │  1. CPU / Processors                              │
  │     Billions of gates in modern chips             │
  │                                                    │
  │  2. Memory (SRAM)                                 │
  │     Each bit stored using 6 transistors/gates     │
  │                                                    │
  │  3. FPGA / ASIC Design                            │
  │     Custom digital circuits from gate primitives  │
  │                                                    │
  │  4. Control Systems                               │
  │     Industrial PLCs use gate logic                │
  │                                                    │
  │  5. Error Detection                               │
  │     XOR gates for parity checking                 │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Gate | Symbol | Expression | Output=1 when |
|------|:------:|:----------:|:-------------:|
| AND | D-shape | $A \cdot B$ | Both 1 |
| OR | Curved | $A + B$ | Any 1 |
| NOT | Triangle+bubble | $\overline{A}$ | Input 0 |
| NAND | AND+bubble | $(AB)'$ | Not both 1 |
| NOR | OR+bubble | $(A+B)'$ | Both 0 |
| XOR | Double-curved | $A \oplus B$ | Inputs differ |
| XNOR | XOR+bubble | $A \odot B$ | Inputs same |

| Concept | Key Point |
|---------|-----------|
| Universal gates | NAND alone or NOR alone can make any circuit |
| NAND-NAND | Two-level NAND implements any SOP |
| NOR-NOR | Two-level NOR implements any POS |
| Gate count | Minimization reduces gates → cheaper circuits |

---

## ❓ Quick Revision Questions

1. **Implement a 3-input OR gate using only 2-input NAND gates.**

2. **Draw the NAND-NAND circuit for $f = \overline{A}B + C\overline{D}$.**

3. **How many NOR gates are needed to implement a NOT gate?**

4. **Why are NAND and NOR called "universal" gates?**

5. **Implement $f = A \oplus B$ using only NAND gates. How many gates are needed?**

6. **What is the output of a 3-input XOR gate when inputs are (1, 1, 1)?**

---

[← Previous: Minimization with K-Maps](03-minimization-kmaps.md) | [Back to README](../README.md) | [Next: Circuit Design →](05-circuit-design.md)
