# Chapter 10.3: Minimization with Karnaugh Maps (K-Maps)

[← Previous: Boolean Functions](02-boolean-functions.md) | [Back to README](../README.md) | [Next: Logic Gates →](04-logic-gates.md)

---

## 📋 Chapter Overview

**Karnaugh Maps** (K-maps) provide a visual, systematic method for simplifying Boolean expressions to their **minimal SOP or POS form**. By arranging truth table values in a grid using **Gray code** ordering, adjacent cells differ by exactly one variable — making it easy to spot simplifications. This chapter covers 2, 3, and 4-variable K-maps with extensive worked examples.

---

## 1. Why Minimize?

```
  Canonical SOP for majority function:
  f = ā b c + a b̄ c + a b c̄ + a b c
  
  Literals: 12    Gates: 4 AND (3-input) + 1 OR (4-input)
  
  Minimized:
  f = ab + ac + bc
  
  Literals: 6     Gates: 3 AND (2-input) + 1 OR (3-input)
  
  Result: 50% fewer literals, smaller/cheaper circuit!
```

---

## 2. Two-Variable K-Map

### Structure

```
             b=0    b=1
          ┌───────┬───────┐
  a=0     │  m₀   │  m₁   │
          ├───────┼───────┤
  a=1     │  m₂   │  m₃   │
          └───────┴───────┘
```

### Example: $f(a,b) = \sum m(1, 2, 3)$

```
             b=0    b=1
          ┌───────┬───────┐
  a=0     │   0   │   1   │
          ├───────┼───────┤
  a=1     │   1   │   1   │
          └───────┴───────┘
  
  Groups:
  ┌──────────────────────────┐
  │ Group 1: m₁, m₃ (column b=1)  → b      │
  │ Group 2: m₂, m₃ (row a=1)     → a      │
  └──────────────────────────┘
  
  f = a + b
```

---

## 3. Three-Variable K-Map

### Structure (Gray Code column ordering!)

```
              bc
           00    01    11    10
        ┌──────┬──────┬──────┬──────┐
  a=0   │  m₀  │  m₁  │  m₃  │  m₂  │
        ├──────┼──────┼──────┼──────┤
  a=1   │  m₄  │  m₅  │  m₇  │  m₆  │
        └──────┴──────┴──────┴──────┘
  
  Note: Column order is 00, 01, 11, 10 (Gray code)
        NOT 00, 01, 10, 11!
```

### Example 1: $f(a,b,c) = \sum m(0, 2, 4, 6)$

```
              bc
           00    01    11    10
        ┌──────┬──────┬──────┬──────┐
  a=0   │  1   │  0   │  0   │  1   │
        ├──────┼──────┼──────┼──────┤
  a=1   │  1   │  0   │  0   │  1   │
        └──────┴──────┴──────┴──────┘
  
  Group: All four 1s form a group (m₀, m₂, m₄, m₆)
  
       bc
           00    01    11    10
        ┌──────┬──────┬──────┬──────┐
  a=0   │ [1]  │  0   │  0   │ [1]  │  ← wraps around!
        ├──────┼──────┼──────┼──────┤
  a=1   │ [1]  │  0   │  0   │ [1]  │
        └──────┴──────┴──────┴──────┘
  
  Group of 4: a varies, b varies, c=0 always
  
  f = c̄
```

### Example 2: $f(a,b,c) = \sum m(3, 5, 6, 7)$

```
              bc
           00    01    11    10
        ┌──────┬──────┬──────┬──────┐
  a=0   │  0   │  0   │  1   │  0   │
        ├──────┼──────┼──────┼──────┤
  a=1   │  0   │  1   │  1   │  1   │
        └──────┴──────┴──────┴──────┘
  
  Group 1 (pair): m₃, m₇ → bc      (column 11)
  Group 2 (pair): m₅, m₇ → ac      (row a=1, c=1)
  Group 3 (pair): m₆, m₇ → ab      (row a=1, b=1)
  
  f = bc + ac + ab
```

---

## 4. Four-Variable K-Map

### Structure

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  m₀  │  m₁  │  m₃  │  m₂  │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  m₄  │  m₅  │  m₇  │  m₆  │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  m₁₂ │  m₁₃ │  m₁₅ │  m₁₄ │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  m₈  │  m₉  │  m₁₁ │  m₁₀ │
         └──────┴──────┴──────┴──────┘
  
  Both rows AND columns use Gray code ordering:
  ab: 00, 01, 11, 10
  cd: 00, 01, 11, 10
```

### Example 1: $f(a,b,c,d) = \sum m(0, 1, 2, 4, 5, 6, 8, 9, 10, 12, 13, 14)$

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  1   │  1   │  0   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  1   │  1   │  0   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  1   │  1   │  0   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  1   │  1   │  0   │  1   │
         └──────┴──────┴──────┴──────┘
  
  All 1s are where d=0 or c=0 (i.e., NOT (c=1 AND d=1))
  
  Group of 8 (col 00 + col 10): d̄
  Group of 8 (col 00 + col 01): c̄
  
  Wait — let's re-examine. The 0s are only in column cd=11.
  The 1s cover all columns EXCEPT cd=11.
  
  f = c̄ + d̄     ← covers all 12 minterms
  
  Verify: c̄ + d̄ = (cd)' by De Morgan's = NOT(c AND d)
  This is 0 only when c=1 AND d=1 ✓
```

### Example 2: $f(a,b,c,d) = \sum m(0, 2, 5, 7, 8, 10, 13, 15)$

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  1   │  0   │  0   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  0   │  1   │  1   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  0   │  1   │  1   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  1   │  0   │  0   │  1   │
         └──────┴──────┴──────┴──────┘
  
  Pattern: 1s appear where b⊕d = 0 (b and d same)
  
  Group 1 (4 corners): m₀, m₂, m₈, m₁₀ → b̄ d̄
  Group 2 (center block): m₅, m₇, m₁₃, m₁₅ → b d
  
  f = b̄ d̄ + b d  =  b ⊙ d  (XNOR)
```

---

## 5. K-Map Grouping Rules

### Valid Groups

Groups must be **rectangular** and contain $2^k$ cells ($k = 0,1,2,3,\ldots$):

```
  Valid group sizes: 1, 2, 4, 8, 16
  
  ┌─────────────────────────────────┐
  │  Size 1:  ■           (0 vars eliminated)  │
  │                                              │
  │  Size 2:  ■ ■  or  ■  (1 var eliminated)   │
  │                     ■                        │
  │                                              │
  │  Size 4:  ■ ■  or  ■ ■ ■ ■   (2 vars)     │
  │           ■ ■                                │
  │                                              │
  │  Size 8:  ■ ■ ■ ■                           │
  │           ■ ■ ■ ■    (3 vars eliminated)    │
  │                                              │
  │  Size 16: Entire map (f = 1)                │
  └─────────────────────────────────┘
```

### Rules

1. **Only 1s** can be grouped (for SOP minimization)
2. Groups must be **powers of 2** in size
3. Groups must be **rectangular** (rows × columns)
4. **Wrapping** is allowed (left↔right, top↔bottom)
5. Every 1 must be in **at least one** group
6. Make groups as **large as possible**
7. Use as **few groups** as possible
8. **Overlapping** groups are allowed

### Wraparound Examples

```
  4-variable K-map wrapping:
  
  Left-Right wrap:           Top-Bottom wrap:
  ┌──┬──┬──┬──┐             ┌──┬──┬──┬──┐
  │[1]│  │  │[1]│            │[1]│[1]│  │  │
  ├──┼──┼──┼──┤             ├──┼──┼──┼──┤
  │[1]│  │  │[1]│            │  │  │  │  │
  ├──┼──┼──┼──┤             ├──┼──┼──┼──┤
  │  │  │  │  │             │  │  │  │  │
  ├──┼──┼──┼──┤             ├──┼──┼──┼──┤
  │  │  │  │  │             │[1]│[1]│  │  │
  └──┴──┴──┴──┘             └──┴──┴──┴──┘
  Group of 4                 Group of 4
  (wraps columns)            (wraps rows)
  
  Four corners:
  ┌──┬──┬──┬──┐
  │[1]│  │  │[1]│
  ├──┼──┼──┼──┤
  │  │  │  │  │
  ├──┼──┼──┼──┤
  │  │  │  │  │
  ├──┼──┼──┼──┤
  │[1]│  │  │[1]│
  └──┴──┴──┴──┘
  Valid group of 4! (wraps both ways)
```

---

## 6. Reading the Simplified Expression

For each group, write the product term by including only variables that are **constant** throughout the group:

```
  Variable = 1 throughout → include uncomplemented
  Variable = 0 throughout → include complemented
  Variable varies (0 and 1) → omit from term
```

### Example

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  0   │  0   │  0   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  0   │  1   │  1   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  0   │  1   │  1   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  0   │  0   │  1   │  0   │
         └──────┴──────┴──────┴──────┘
  
  Group 1 (green): m₅, m₇, m₁₃, m₁₅ → {ab=01,11; cd=01,11}
    a: varies (0,1) → omit
    b: always 1 → b
    c: varies → omit  
    d: always 1 → d
    Term: b d
  
  Group 2 (blue): m₇, m₁₅, m₁₄, m₆... wait, let me re-read.
  
  Actually: m₁₅ and m₁₄ are in ab=11. m₇ is ab=01,cd=11. m₁₁ is ab=10,cd=11.
  
  Group 2: m₇, m₁₅, m₁₁ ... not a valid rectangle. Let me re-do.
  
  Correct groupings:
  Group A: {m₅, m₇, m₁₃, m₁₅} → bd          (quad)
  Group B: {m₁₃, m₁₅, m₁₄} → need 4th: m₁₂=0, invalid
  Group B: {m₇, m₁₅, m₁₁} → column cd=11, rows 01,11,10 → not power of 2
  
  Let me redo with proper minterms:
  1s at: m₅(0101), m₇(0111), m₁₃(1101), m₁₅(1111), m₁₄(1110), m₁₁(1011)
  
  Group A: m₅, m₇, m₁₃, m₁₅ → bd           (4 cells)
  Group B: m₁₄, m₁₅ → abd                    (pair → abc̄d... wait)
  
  m₁₄ = ab=11, cd=10 → a=1,b=1,c=1,d=0
  m₁₅ = ab=11, cd=11 → a=1,b=1,c=1,d=1
  Group B: a=1, b=1, c=1, d varies → abc     (pair)
  
  m₁₁ = ab=10, cd=11 → a=1,b=0,c=1,d=1
  Group C: m₇, m₁₅, m₁₁ → not a power of 2 rectangle
  
  m₁₁ alone → ab̄cd                            (single)
  
  f = bd + abc + ab̄cd
  
  But we can simplify: m₁₁ can join m₁₅ as pair:
  m₁₁(1011), m₁₅(1111) → a=1, c=1, d=1, b varies → acd
  
  f = bd + abc + acd
    = bd + ac(b + d)  ... or just leave as 3 terms.
```

---

## 7. Don't Care Conditions

Sometimes certain input combinations **never occur** or their output **doesn't matter**. These are marked as **X** (don't care) and can be treated as either 0 or 1 to help form larger groups.

### Example: BCD to 7-Segment (simplified)

In BCD (Binary-Coded Decimal), inputs 1010 through 1111 (10-15) never occur:

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  1   │  0   │  1   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  0   │  1   │  1   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  X   │  X   │  X   │  X   │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  1   │  1   │  X   │  X   │
         └──────┴──────┴──────┴──────┘
  
  X = don't care (inputs 10-15 in BCD)
  
  Without don't cares: smaller groups, more terms
  With don't cares: treat X as 1 where helpful
  
  Group 1: m₀, m₂, m₈, m₁₀(X) → wrap col 00 & 10, rows 00 & 10 → c̄
  Wait, that includes m₂, but m₀=1, m₂=1, m₈=1, m₁₀=X→1
  a varies, b=0 for rows 00,10? No, ab=00 has b=0, ab=10 has b=0. Yes!
  b=0, d=0 → b̄ d̄
  
  Treating some X's as 1 gives larger groups → simpler expression!
```

---

## 8. POS Minimization with K-Maps

To find the **minimal POS**, group the **0s** instead:

```
  ┌──────────────────────────────────────────┐
  │  SOP: Group the 1s → sum of products     │
  │  POS: Group the 0s → product of sums     │
  │                                           │
  │  For POS: each group of 0s gives a       │
  │  maxterm factor. Variables constant       │
  │  at 0 → uncomplemented; at 1 → comp.     │
  └──────────────────────────────────────────┘
```

### Example: $f = \sum m(1, 3, 5, 7)$ (3 variables)

```
              bc
           00    01    11    10
        ┌──────┬──────┬──────┬──────┐
  a=0   │  0   │  1   │  1   │  0   │
        ├──────┼──────┼──────┼──────┤
  a=1   │  0   │  1   │  1   │  0   │
        └──────┴──────┴──────┴──────┘
  
  SOP (group 1s): column 01 + column 11 → c
  f = c  ✓
  
  POS (group 0s): column 00 + column 10 (wraps!) → c̄
  Group of 0s: c=0 always → maxterm factor = c
  f = c  ✓ (same result)
```

---

## 9. Essential Prime Implicants

### Definitions

- **Implicant:** Any product term that covers a subset of the function's 1s
- **Prime Implicant (PI):** An implicant that **cannot** be combined into a larger group
- **Essential Prime Implicant (EPI):** A PI that covers at least one minterm not covered by any other PI

```
  Procedure:
  1. Find ALL prime implicants (maximal groups)
  2. Identify essential PIs (any minterm covered by only one PI)
  3. Essential PIs MUST be in the final answer
  4. If uncovered minterms remain, select additional PIs
     (choose to cover remaining 1s with fewest PIs)
```

### Example: $f(a,b,c,d) = \sum m(0, 2, 3, 5, 7, 8, 10, 11, 14, 15)$

```
                cd
            00     01     11     10
         ┌──────┬──────┬──────┬──────┐
  ab=00  │  1   │  0   │  1   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=01  │  0   │  1   │  1   │  0   │
         ├──────┼──────┼──────┼──────┤
  ab=11  │  0   │  0   │  1   │  1   │
         ├──────┼──────┼──────┼──────┤
  ab=10  │  1   │  0   │  1   │  1   │
         └──────┴──────┴──────┴──────┘
  
  Prime Implicants:
  PI₁: m₀, m₂, m₈, m₁₀ → b̄ d̄    (4 corners left-right wrap)
  PI₂: m₂, m₃, m₁₀, m₁₁ → ā c    (wait... m₂=ab00,cd10; m₃=ab00,cd11)
       Actually: m₂(0010), m₃(0011), m₁₀(1010), m₁₁(1011)
       a varies, b=0, c=1, d varies → b̄ c
  PI₃: m₃, m₇ → ā b... m₃(0011), m₇(0111): a=0, d=1, c=1, b varies → ā c d
  PI₄: m₅, m₇ → ā b d... m₅(0101), m₇(0111): a=0, b=1, d=1, c varies → ā b d
  PI₅: m₁₄, m₁₅ → a b c... (1110, 1111): a=1, b=1, c=1, d varies → a b c
  PI₆: m₁₀, m₁₁, m₁₄, m₁₅ → a c (1010,1011,1110,1111): a=1, c=1 → a c
  PI₇: m₇, m₁₅ → c d (0111, 1111): c=1, d=1, a varies, b varies? 
       No, b=1 for both → b c d
  
  Let me re-identify carefully:
  PI₁: {m₀, m₂, m₈, m₁₀} → b̄ d̄
  PI₂: {m₂, m₃, m₁₀, m₁₁} → b̄ c
  PI₃: {m₃, m₇} → ā c d
  PI₄: {m₅, m₇} → ā b d
  PI₅: {m₁₀, m₁₁, m₁₄, m₁₅} → a c
  
  Essential PIs:
  - m₀ only in PI₁ → PI₁ (b̄ d̄) is essential
  - m₅ only in PI₄ → PI₄ (ā b d) is essential
  - m₁₄ only in PI₅ → PI₅ (a c) is essential
  
  After EPIs: covered = {0,2,5,7,8,10,11,14,15}, missing = {3}
  m₃ covered by PI₂ or PI₃. Choose PI₂ (covers more: also m₂,m₁₀,m₁₁ redundantly)
  Or PI₃ (covers m₃, m₇ redundantly). Both add 1 term.
  
  f = b̄ d̄ + ā b d + a c + ā c d    (or + b̄ c)
```

---

## 10. Five and Six Variable K-Maps

For 5 variables, use **two 4-variable maps** (one for $a=0$, one for $a=1$) placed side by side. Corresponding cells between the two maps are adjacent.

For 6 variables, use **four 4-variable maps**.

```
  5-variable K-map:
  
     a=0                         a=1
         cd                          cd
     00  01  11  10            00  01  11  10
  ┌────┬────┬────┬────┐    ┌────┬────┬────┬────┐
  │ m₀ │ m₁ │ m₃ │ m₂ │    │m₁₆│m₁₇│m₁₉│m₁₈│  bc=00
  ├────┼────┼────┼────┤    ├────┼────┼────┼────┤
  │ m₄ │ m₅ │ m₇ │ m₆ │    │m₂₀│m₂₁│m₂₃│m₂₂│  bc=01
  ├────┼────┼────┼────┤    ├────┼────┼────┼────┤
  │m₁₂│m₁₃│m₁₅│m₁₄│    │m₂₈│m₂₉│m₃₁│m₃₀│  bc=11
  ├────┼────┼────┼────┤    ├────┼────┼────┼────┤
  │ m₈ │ m₉ │m₁₁│m₁₀│    │m₂₄│m₂₅│m₂₇│m₂₆│  bc=10
  └────┴────┴────┴────┘    └────┴────┴────┴────┘
  
  Cells at same position in both maps are adjacent
  (they differ only in variable a)
```

---

## 11. Quine-McCluskey Method (Tabular)

For more than 4-5 variables, the **Quine-McCluskey** algorithm provides a systematic (computer-friendly) alternative to K-maps:

```
  Algorithm Outline:
  
  1. List all minterms in binary
  2. Group by number of 1s
  3. Compare adjacent groups, combine terms 
     differing in exactly 1 bit (replace with '-')
  4. Repeat until no more combinations possible
     → these are the Prime Implicants
  5. Create a PI chart and find essential PIs
  6. Use covering to select minimum set
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| K-map | Visual truth table grid using Gray code |
| Gray code | Adjacent cells differ by 1 variable |
| Group sizes | Must be powers of 2: 1, 2, 4, 8, 16 |
| Wraparound | Left↔right, top↔bottom adjacencies valid |
| SOP from K-map | Group 1s; product of constant variables |
| POS from K-map | Group 0s; sum of complemented constant vars |
| Don't care (X) | Can be 0 or 1 — use to make larger groups |
| Prime Implicant | Largest possible group (can't be expanded) |
| Essential PI | Covers a minterm no other PI covers |
| Quine-McCluskey | Tabular method for > 4 variables |

---

## ❓ Quick Revision Questions

1. **Minimize $f(a,b,c) = \sum m(0, 1, 2, 4, 6)$ using a 3-variable K-map.**

2. **Minimize $f(a,b,c,d) = \sum m(0, 1, 3, 5, 7, 9, 11, 13, 15)$ using a 4-variable K-map. How many essential prime implicants are there?**

3. **Minimize $f(a,b,c,d) = \sum m(1, 5, 6, 7, 11, 12, 13, 15) + \sum d(2, 4)$ with don't cares.**

4. **Why must K-map columns/rows use Gray code ordering instead of binary counting?**

5. **Given a K-map where all four corners are 1, what single product term covers them?**

6. **Find the minimal POS expression for $f(a,b,c) = \prod M(1, 3, 5)$ using a K-map.**

---

[← Previous: Boolean Functions](02-boolean-functions.md) | [Back to README](../README.md) | [Next: Logic Gates →](04-logic-gates.md)
