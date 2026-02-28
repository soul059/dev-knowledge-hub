# Unit 3: Minimization Techniques

---

## Table of Contents
1. [Introduction to Minimization](#1-introduction-to-minimization)
2. [Karnaugh Maps - Basics](#2-karnaugh-maps---basics)
3. [2-Variable K-Maps](#3-2-variable-k-maps)
4. [3-Variable K-Maps](#4-3-variable-k-maps)
5. [4-Variable K-Maps](#5-4-variable-k-maps)
6. [5-Variable K-Maps](#6-5-variable-k-maps)
7. [Don't Care Conditions](#7-dont-care-conditions)
8. [Quine-McCluskey Method](#8-quine-mccluskey-method)
9. [Prime Implicants](#9-prime-implicants)
10. [Summary Tables](#10-summary-tables)
11. [Quick Revision Questions](#11-quick-revision-questions)

---

## 1. Introduction to Minimization

### Why Minimize?

```
┌─────────────────────────────────────────────────────────────────┐
│                  BENEFITS OF MINIMIZATION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Fewer Gates        → Lower cost                            │
│  2. Fewer Inputs       → Simpler wiring                        │
│  3. Fewer Levels       → Less propagation delay                │
│  4. Less Power         → Lower power consumption               │
│  5. Higher Reliability → Fewer components to fail              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Minimization Methods

| Method | Variables | Type | Best For |
|--------|-----------|------|----------|
| Boolean Algebra | Any | Manual | Simple expressions |
| Karnaugh Map (K-map) | 2-6 | Visual | Quick hand solving |
| Quine-McCluskey | Any | Systematic | Computer implementation |
| Espresso | Any | Heuristic | Very large functions |

### Goals of Minimization

```
For SOP:
  - Minimum number of product terms
  - Each product term has minimum literals

For POS:
  - Minimum number of sum terms
  - Each sum term has minimum literals

Trade-offs:
  - SOP minimum ≠ POS minimum (choose based on implementation)
  - Two-level vs multi-level optimization
```

---

## 2. Karnaugh Maps - Basics

### What is a K-Map?

A **Karnaugh Map** is a visual tool for simplifying Boolean expressions. It's a rearranged truth table where adjacent cells differ by only ONE variable (Gray code ordering).

```
┌─────────────────────────────────────────────────────────────────┐
│                    K-MAP FUNDAMENTALS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Key Property: Adjacent cells differ by ONE variable           │
│                                                                 │
│  Adjacency includes:                                           │
│    - Horizontal neighbors                                      │
│    - Vertical neighbors                                        │
│    - Edge wrapping (left-right, top-bottom)                   │
│    - Corner wrapping (for 4+ variables)                       │
│                                                                 │
│  Grouping Rules:                                               │
│    - Group sizes must be powers of 2: 1, 2, 4, 8, 16...       │
│    - Groups must be rectangular                                │
│    - Larger groups = simpler terms                            │
│    - Each 1 must be covered at least once                     │
│    - Overlap is allowed and often beneficial                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Grouping and Term Reduction

```
Group Size → Variables Eliminated → Term Complexity
─────────────────────────────────────────────────────
    1      →        0             → n-variable term
    2      →        1             → (n-1)-variable term
    4      →        2             → (n-2)-variable term
    8      →        3             → (n-3)-variable term
   16      →        4             → (n-4)-variable term
   2^k     →        k             → (n-k)-variable term

For n-variable K-map:
  - A group of 2^k cells eliminates k variables
  - Remaining (n-k) variables form the product term
```

---

## 3. 2-Variable K-Maps

### Structure

```
        ┌─────────────────┐
        │    2-VAR K-MAP  │
        └─────────────────┘
        
              B
           0     1
         ┌─────┬─────┐
     0   │ m₀  │ m₁  │   A'
   A     ├─────┼─────┤
     1   │ m₂  │ m₃  │   A
         └─────┴─────┘
           B'    B

    Minterms:
    m₀ = A'B' (cell 0,0)
    m₁ = A'B  (cell 0,1)
    m₂ = AB'  (cell 1,0)
    m₃ = AB   (cell 1,1)
```

### Example: F = Σm(0, 1, 3)

```
Step 1: Fill the K-map
              B
           0     1
         ┌─────┬─────┐
     0   │  1  │  1  │   ← m₀, m₁
   A     ├─────┼─────┤
     1   │  0  │  1  │   ← m₃
         └─────┴─────┘

Step 2: Form groups

              B
           0     1
         ┌─────┬─────┐
     0   │  1  │  1  │   Group 1: m₀, m₁ (horizontal pair)
   A     ├─────┼─────┤
     1   │  0  │  1  │   Group 2: m₁, m₃ (vertical pair)
         └─────┴─────┘
         
    ┌───────────┐
    │  1  │  1  │  ← Group 1: A' (B changes, A=0)
    ├─────┼─────┤
    │     │  1  │  ← Part of Group 2
    └─────┴─────┘
              │
              └─── Group 2: B (A changes, B=1)

Step 3: Read the result
  Group 1: A' (covers m₀, m₁)
  Group 2: B  (covers m₁, m₃)
  
  F = A' + B
```

---

## 4. 3-Variable K-Maps

### Structure

```
        ┌─────────────────────────────────────┐
        │           3-VAR K-MAP               │
        └─────────────────────────────────────┘
        
                      BC
              00    01    11    10     ← Gray code order!
            ┌─────┬─────┬─────┬─────┐
        0   │ m₀  │ m₁  │ m₃  │ m₂  │   A'
   A        ├─────┼─────┼─────┼─────┤
        1   │ m₄  │ m₅  │ m₇  │ m₆  │   A
            └─────┴─────┴─────┴─────┘
              B'C'  B'C   BC   BC'

    Note: Column order is 00, 01, 11, 10 (NOT 00, 01, 10, 11)
          This ensures adjacent cells differ by 1 bit
          
    Edge Wrapping: Column 00 and column 10 are adjacent!
```

### Example 1: F = Σm(0, 2, 4, 5, 6)

```
Step 1: Fill the K-map
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  1  │  0  │  0  │  1  │   m₀, m₂
   A        ├─────┼─────┼─────┼─────┤
        1   │  1  │  1  │  0  │  1  │   m₄, m₅, m₆
            └─────┴─────┴─────┴─────┘

Step 2: Identify groups

    Group 1: m₀, m₂, m₄, m₆ (corners - they wrap!)
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │ [1] │  0  │  0  │ [1] │
   A        ├─────┼─────┼─────┼─────┤
        1   │ [1] │  1  │  0  │ [1] │
            └─────┴─────┴─────┴─────┘
            
    These 4 cells form a group! (edges wrap)
    Result: C' (A and B both change, C=0 for all)

    Group 2: m₄, m₅ (pair)
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  1  │  0  │  0  │  1  │
   A        ├─────┼─────┼─────┼─────┤
        1   │ [1] │ [1] │  0  │  1  │
            └─────┴─────┴─────┴─────┘
            
    Result: AB' (A=1, B=0, C changes)

Step 3: Final result
    F = C' + AB'
    
Verification: C' covers 0,2,4,6; AB' covers 4,5
              Together: 0,2,4,5,6 ✓
```

### Example 2: F = Σm(1, 2, 3, 5, 7)

```
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  0  │  1  │  1  │  1  │
   A        ├─────┼─────┼─────┼─────┤
        1   │  0  │  1  │  1  │  0  │
            └─────┴─────┴─────┴─────┘

Groups:
    Group 1: m₁, m₃, m₅, m₇ (4 cells - middle columns)
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │     │ [1] │ [1] │     │
   A        ├─────┼─────┼─────┼─────┤
        1   │     │ [1] │ [1] │     │
            └─────┴─────┴─────┴─────┘
    Result: C (A changes, B changes, C=1 for all)

    Group 2: m₂, m₃ (pair)
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │     │     │ [1] │ [1] │
   A        ├─────┼─────┼─────┼─────┤
        1   │     │     │     │     │
            └─────┴─────┴─────┴─────┘
    Result: A'B (A=0, B=1, C changes)

Final result: F = C + A'B
```

---

## 5. 4-Variable K-Maps

### Structure

```
        ┌─────────────────────────────────────────────────┐
        │                4-VAR K-MAP                      │
        └─────────────────────────────────────────────────┘
        
                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │ m₀  │ m₁  │ m₃  │ m₂  │   A'B'
                ├─────┼─────┼─────┼─────┤
            01  │ m₄  │ m₅  │ m₇  │ m₆  │   A'B
      AB        ├─────┼─────┼─────┼─────┤
            11  │ m₁₂ │ m₁₃ │ m₁₅ │ m₁₄ │   AB
                ├─────┼─────┼─────┼─────┤
            10  │ m₈  │ m₉  │ m₁₁ │ m₁₀ │   AB'
                └─────┴─────┴─────┴─────┘
                  C'D'  C'D   CD   CD'

    Wrapping:
    - Left edge (00) wraps to right edge (10)
    - Top edge (00) wraps to bottom edge (10)
    - All four corners are adjacent!
```

### Adjacency in 4-Variable K-Map

```
    ADJACENT CELLS EXAMPLES:
    
    ┌─────┬─────┬─────┬─────┐
    │  0  │  1  │  3  │  2  │
    ├─────┼─────┼─────┼─────┤
    │  4  │  5  │  7  │  6  │
    ├─────┼─────┼─────┼─────┤
    │ 12  │ 13  │ 15  │ 14  │
    ├─────┼─────┼─────┼─────┤
    │  8  │  9  │ 11  │ 10  │
    └─────┴─────┴─────┴─────┘
    
    Cell 0 adjacent to: 1, 2(wrap), 4, 8(wrap)
    Cell 5 adjacent to: 1, 4, 7, 13
    Cell 15 adjacent to: 7, 11, 13, 14
    
    CORNER GROUP (all 4 corners):
    ┌─────┬─────┬─────┬─────┐
    │ [0] │     │     │ [2] │
    ├─────┼─────┼─────┼─────┤
    │     │     │     │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │     │     │
    ├─────┼─────┼─────┼─────┤
    │ [8] │     │     │[10] │
    └─────┴─────┴─────┴─────┘
    These form a valid group of 4! → B'D'
```

### Example 1: F(A,B,C,D) = Σm(0, 2, 5, 7, 8, 10, 13, 15)

```
Step 1: Fill the K-map

                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │  1  │  0  │  0  │  1  │
                ├─────┼─────┼─────┼─────┤
            01  │  0  │  1  │  1  │  0  │
      AB        ├─────┼─────┼─────┼─────┤
            11  │  0  │  1  │  1  │  0  │
                ├─────┼─────┼─────┼─────┤
            10  │  1  │  0  │  0  │  1  │
                └─────┴─────┴─────┴─────┘

Step 2: Identify groups

    Group 1: m₀, m₂, m₈, m₁₀ (4 corners)
    ┌─────┬─────┬─────┬─────┐
    │ [1] │     │     │ [1] │
    ├─────┼─────┼─────┼─────┤
    │     │     │     │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │     │     │
    ├─────┼─────┼─────┼─────┤
    │ [1] │     │     │ [1] │
    └─────┴─────┴─────┴─────┘
    Result: B'D' (B=0, D=0 for all cells)

    Group 2: m₅, m₇, m₁₃, m₁₅ (4 cells in center columns)
    ┌─────┬─────┬─────┬─────┐
    │     │     │     │     │
    ├─────┼─────┼─────┼─────┤
    │     │ [1] │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │ [1] │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │     │     │
    └─────┴─────┴─────┴─────┘
    Result: BD (B=1, D=1 for all cells)

Step 3: Final result
    F = B'D' + BD
    
    Simplified: F = B ⊙ D (XNOR)
```

### Example 2: F(A,B,C,D) = Σm(0, 1, 2, 4, 5, 6, 8, 9, 10, 12, 13, 14)

```
Step 1: Fill the K-map

                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │  1  │  1  │  0  │  1  │   m₀,m₁,m₂
                ├─────┼─────┼─────┼─────┤
            01  │  1  │  1  │  0  │  1  │   m₄,m₅,m₆
      AB        ├─────┼─────┼─────┼─────┤
            11  │  1  │  1  │  0  │  1  │   m₁₂,m₁₃,m₁₄
                ├─────┼─────┼─────┼─────┤
            10  │  1  │  1  │  0  │  1  │   m₈,m₉,m₁₀
                └─────┴─────┴─────┴─────┘

Step 2: Identify the largest group

    All 1s form a pattern - entire K-map except column CD=11
    
    ┌─────┬─────┬─────┬─────┐
    │ [1] │ [1] │  0  │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │  0  │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │  0  │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │  0  │ [1] │
    └─────┴─────┴─────┴─────┘

    This can be covered by:
    - Columns 00 and 01 (8 cells): C'
    - Columns 10 and 00 wrap (8 cells): D'

Step 3: Alternative grouping view
    
    Left half (columns 00, 01): C' (8 cells)
    ┌─────┬─────┬─────┬─────┐
    │ [1] │ [1] │     │     │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │     │     │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │     │     │
    ├─────┼─────┼─────┼─────┤
    │ [1] │ [1] │     │     │
    └─────┴─────┴─────┴─────┘

    Edge columns (00 and 10) wrap: D' (8 cells)
    ┌─────┬─────┬─────┬─────┐
    │ [1] │     │     │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │     │     │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │     │     │ [1] │
    ├─────┼─────┼─────┼─────┤
    │ [1] │     │     │ [1] │
    └─────┴─────┴─────┴─────┘

Step 4: Final result
    F = C' + D'
    
    Or equivalently (De Morgan): F = (CD)'
```

### Group Size Quick Reference (4 Variables)

```
┌─────────────────────────────────────────────────────────────────┐
│              4-VARIABLE K-MAP GROUP SIZES                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Group of 1:   4 variables   (e.g., A'B'C'D')                  │
│  Group of 2:   3 variables   (e.g., A'B'C')                    │
│  Group of 4:   2 variables   (e.g., A'B')                      │
│  Group of 8:   1 variable    (e.g., A')                        │
│  Group of 16:  0 variables   (constant 1)                      │
│                                                                 │
│  Valid group shapes for 4 cells:                               │
│    □□     □      □□□□    □□                                    │
│    □□     □               □□  (wrapping)                       │
│           □                                                     │
│           □                                                     │
│                                                                 │
│  Valid group shapes for 8 cells:                               │
│    □□□□    □□                                                   │
│    □□□□    □□                                                   │
│            □□                                                   │
│            □□                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 5-Variable K-Maps

### Structure

For 5 variables, we use two 4-variable K-maps (one for A=0, one for A=1):

```
        ┌────────────────────────────────────────────────────────────────┐
        │                      5-VAR K-MAP                               │
        └────────────────────────────────────────────────────────────────┘

                    A = 0                               A = 1
                            CD                                  CD
                  00    01    11    10            00    01    11    10
                ┌─────┬─────┬─────┬─────┐       ┌─────┬─────┬─────┬─────┐
            00  │ m₀  │ m₁  │ m₃  │ m₂  │       │ m₁₆ │ m₁₇ │ m₁₉ │ m₁₈ │
                ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
            01  │ m₄  │ m₅  │ m₇  │ m₆  │       │ m₂₀ │ m₂₁ │ m₂₃ │ m₂₂ │
      BC        ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
            11  │ m₁₂ │ m₁₃ │ m₁₅ │ m₁₄ │       │ m₂₈ │ m₂₉ │ m₃₁ │ m₃₀ │
                ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
            10  │ m₈  │ m₉  │ m₁₁ │ m₁₀ │       │ m₂₄ │ m₂₅ │ m₂₇ │ m₂₆ │
                └─────┴─────┴─────┴─────┘       └─────┴─────┴─────┴─────┘
                
    Adjacency: Corresponding cells in both maps are adjacent!
               (m₀ and m₁₆, m₅ and m₂₁, etc.)
```

### Example: F = Σm(0, 2, 4, 6, 16, 18, 20, 22, 24, 26, 28, 30)

```
Fill both K-maps:

            A = 0                               A = 1
                  CD                                  CD
          00    01    11    10            00    01    11    10
        ┌─────┬─────┬─────┬─────┐       ┌─────┬─────┬─────┬─────┐
    00  │  1  │  0  │  0  │  1  │       │  1  │  0  │  0  │  1  │
        ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
    01  │  1  │  0  │  0  │  1  │       │  1  │  0  │  0  │  1  │
 BC     ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
    11  │  0  │  0  │  0  │  0  │       │  1  │  0  │  0  │  1  │
        ├─────┼─────┼─────┼─────┤       ├─────┼─────┼─────┼─────┤
    10  │  0  │  0  │  0  │  0  │       │  1  │  0  │  0  │  1  │
        └─────┴─────┴─────┴─────┘       └─────┴─────┴─────┴─────┘

Analysis:
    In A=1 map: All cells in columns 00 and 10 are 1
                This is D' (8 cells) → AD'
    
    In A=0 map: Upper half of columns 00 and 10
                Cells: 0,2,4,6 → A'B'D' + A'BD' = A'D' (but only when B'C' or B'C)
                Actually: These 4 cells in A=0 + corresponding 4 in A=1 (0,2,4,6,16,18,20,22)
                Forms group of 8: B'D' (spans both maps, B=0 in both)

Wait, let me reanalyze:
    m₀,m₂,m₄,m₆ in A=0: D'(B=0,1 but C must vary)
    Actually: {0,2,4,6} = A'B'D' not quite...
    
    Let me identify correctly:
    Cells with 1: 
    A=0: 0,2,4,6 (D'=1, CD in {00,10}, BC in {00,01})
    A=1: 16,18,20,22,24,26,28,30 (all D'=1 cells)

Group 1: All D'=1 in A=1 map (8 cells): AD'
Group 2: Corresponding cells 0,2,16,18 + 4,6,20,22 (spanning both maps)
         Cells 0,2,4,6,16,18,20,22 → B'D' (8 cells spanning both maps)

Final: F = AD' + B'D' = D'(A + B')
```

---

## 7. Don't Care Conditions

### What are Don't Cares?

**Don't care conditions** occur when:
1. Certain input combinations never occur
2. Output doesn't matter for certain inputs

Symbol: X or d (can be treated as 0 or 1)

```
┌─────────────────────────────────────────────────────────────────┐
│                    DON'T CARE CONDITIONS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Notation:                                                      │
│    F = Σm(1,3,5) + d(2,4)   or   F = Σm(1,3,5) + Σd(2,4)       │
│                                                                 │
│  Strategy:                                                      │
│    - Treat don't cares as 1 if it helps form larger groups    │
│    - Treat don't cares as 0 if they don't help               │
│    - Goal: Minimize the final expression                       │
│                                                                 │
│  Common sources:                                                │
│    - BCD codes (values 10-15 never occur)                     │
│    - Invalid states in state machines                         │
│    - Physically impossible input combinations                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Example: F(A,B,C,D) = Σm(1, 3, 7, 11, 15) + d(0, 2, 5)

```
Step 1: Fill K-map (X = don't care)

                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │  X  │  1  │  1  │  X  │   d₀, m₁, m₃, d₂
                ├─────┼─────┼─────┼─────┤
            01  │  0  │  X  │  1  │  0  │   d₅, m₇
      AB        ├─────┼─────┼─────┼─────┤
            11  │  0  │  0  │  1  │  0  │   m₁₅
                ├─────┼─────┼─────┼─────┤
            10  │  0  │  0  │  1  │  0  │   m₁₁
                └─────┴─────┴─────┴─────┘

Step 2: Form groups using don't cares strategically

    Option 1: Treat d₀ and d₂ as 1
    ┌─────┬─────┬─────┬─────┐
    │ [X] │  1  │  1  │ [X] │   Row becomes all 1s → A'B'
    ├─────┼─────┼─────┼─────┤
    │     │ [X] │  1  │     │   Group with m₇ → A'D
    ├─────┼─────┼─────┼─────┤
    │     │     │  1  │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │  1  │     │
    └─────┴─────┴─────┴─────┘

    Better approach - look at column CD=11:
    ┌─────┬─────┬─────┬─────┐
    │     │     │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │     │ [1] │     │
    └─────┴─────┴─────┴─────┘
    Entire column! Result: CD (8 cells)

    But we need to cover m₁ also:
    Use d₀ and d₂ to form pair with m₁:
    ┌─────┬─────┬─────┬─────┐
    │ [X] │ [1] │     │ [X] │   A'B'
    ├─────┼─────┼─────┼─────┤
    
    Or use d₅ with m₁, m₃:
    ┌─────┬─────┬─────┬─────┐
    │     │ [1] │ [1] │     │
    ├─────┼─────┼─────┼─────┤
    │     │ [X] │ [1] │     │
    This forms group of 4 → A'D

Step 3: Optimal solution
    Group 1: CD (column 11) - covers m₃, m₇, m₁₅, m₁₁
    Group 2: Use d₀ to pair with m₁ → A'B'C' (cells 0,1)

    F = CD + A'B'C'
    
    OR with different don't care usage:
    F = CD + A'B'D (if we use d₀,d₂ to extend m₁,m₃)

Step 4: Compare solutions
    F = CD + A'B'C' → 2 terms, 5 literals
    F = CD + A'B'D  → 2 terms, 5 literals (using d₂)
    
    Both are minimal!
```

### BCD Don't Care Example

```
BCD codes only use 0-9, so minterms 10-15 are don't cares.

F = Σm(1, 2, 3, 5, 7, 9) in BCD (values 10-15 are don't cares)
d = {10, 11, 12, 13, 14, 15}

                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │  0  │  1  │  1  │  1  │   m₁, m₃, m₂
                ├─────┼─────┼─────┼─────┤
            01  │  0  │  1  │  1  │  0  │   m₅, m₇
      AB        ├─────┼─────┼─────┼─────┤
            11  │  X  │  X  │  X  │  X  │   d₁₂-d₁₅
                ├─────┼─────┼─────┼─────┤
            10  │  0  │  1  │  X  │  X  │   m₉, d₁₁, d₁₀
                └─────┴─────┴─────┴─────┘

Using don't cares:
    Group 1: m₁,m₃,m₅,m₇,d₁₃,d₁₅,m₉,d₁₁ (column 01,11) → D
    Group 2: m₂,m₃,d₁₀,d₁₁ (row/column intersection) → C
    
    But let's verify: m₂ is at BC=00, CD=10
    
    Optimal: F = D + BC'D' (need to cover m₂)
    
    Actually: Using don't cares fully:
    F = D (covers 1,3,5,7,9 using d₁₁,d₁₃,d₁₅)
    But m₂ is not covered by D!
    
    Group for m₂: m₂,m₃,d₁₀,d₁₁ → B'C
    
    F = D + B'C
```

---

## 8. Quine-McCluskey Method

### Overview

The **Quine-McCluskey** method is a tabular method for minimizing Boolean functions. It's systematic and suitable for computer implementation.

```
┌─────────────────────────────────────────────────────────────────┐
│              QUINE-McCLUSKEY ALGORITHM                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: List all minterms in binary                           │
│  Step 2: Group by number of 1s                                 │
│  Step 3: Compare adjacent groups, combine differing by 1 bit  │
│  Step 4: Repeat until no more combinations                     │
│  Step 5: Find prime implicants                                 │
│  Step 6: Create prime implicant chart                          │
│  Step 7: Find essential prime implicants                       │
│  Step 8: Cover remaining minterms                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Example: F(A,B,C,D) = Σm(0, 1, 2, 5, 6, 7, 8, 9, 10, 14)

```
STEP 1 & 2: List minterms grouped by number of 1s

┌───────────────────────────────────────────────────────────────┐
│  Group  │ Minterm │  A B C D  │  # of 1s                     │
├─────────┼─────────┼───────────┼──────────────────────────────┤
│    0    │    0    │  0 0 0 0  │     0                        │
├─────────┼─────────┼───────────┼──────────────────────────────┤
│    1    │    1    │  0 0 0 1  │     1                        │
│         │    2    │  0 0 1 0  │     1                        │
│         │    8    │  1 0 0 0  │     1                        │
├─────────┼─────────┼───────────┼──────────────────────────────┤
│    2    │    5    │  0 1 0 1  │     2                        │
│         │    6    │  0 1 1 0  │     2                        │
│         │    9    │  1 0 0 1  │     2                        │
│         │   10    │  1 0 1 0  │     2                        │
├─────────┼─────────┼───────────┼──────────────────────────────┤
│    3    │    7    │  0 1 1 1  │     3                        │
│         │   14    │  1 1 1 0  │     3                        │
└─────────┴─────────┴───────────┴──────────────────────────────┘

STEP 3: First pass - combine adjacent groups

Compare Group 0 with Group 1:
  (0,1):   0000 + 0001 = 000-  ✓
  (0,2):   0000 + 0010 = 00-0  ✓
  (0,8):   0000 + 1000 = -000  ✓

Compare Group 1 with Group 2:
  (1,5):   0001 + 0101 = 0-01  ✓
  (1,9):   0001 + 1001 = -001  ✓
  (2,6):   0010 + 0110 = 0-10  ✓
  (2,10):  0010 + 1010 = -010  ✓
  (8,9):   1000 + 1001 = 100-  ✓
  (8,10):  1000 + 1010 = 10-0  ✓

Compare Group 2 with Group 3:
  (5,7):   0101 + 0111 = 01-1  ✓
  (6,7):   0110 + 0111 = 011-  ✓
  (6,14):  0110 + 1110 = -110  ✓
  (10,14): 1010 + 1110 = -010 → already have, = 1-10 ✓

First pass results:
┌─────────────────────────────────────────────────────────────────┐
│  Pair   │  Pattern  │  Meaning                                  │
├─────────┼───────────┼───────────────────────────────────────────┤
│  (0,1)  │   000-    │  A'B'C'                                   │
│  (0,2)  │   00-0    │  A'B'D'                                   │
│  (0,8)  │   -000    │  B'C'D'                                   │
│  (1,5)  │   0-01    │  A'C'D                                    │
│  (1,9)  │   -001    │  B'C'D                                    │
│  (2,6)  │   0-10    │  A'CD'                                    │
│  (2,10) │   -010    │  B'CD'                                    │
│  (8,9)  │   100-    │  AB'C'                                    │
│  (8,10) │   10-0    │  AB'D'                                    │
│  (5,7)  │   01-1    │  A'BD                                     │
│  (6,7)  │   011-    │  A'BC                                     │
│  (6,14) │   -110    │  BCD'                                     │
│ (10,14) │   1-10    │  ACD'                                     │
└─────────┴───────────┴───────────────────────────────────────────┘

STEP 4: Second pass - combine again

Compare patterns with one dash:
  (0,1) with (8,9): 000- + 100- = -00-  ✓  (0,1,8,9)
  (0,2) with (8,10): 00-0 + 10-0 = -0-0  ✓  (0,2,8,10)
  (2,6) with (10,14): 0-10 + 1-10 = --10  ✓  (2,6,10,14)

Second pass results:
┌─────────────────────────────────────────────────────────────────┐
│   Combo    │  Pattern  │  Meaning                               │
├────────────┼───────────┼────────────────────────────────────────┤
│ (0,1,8,9)  │   -00-    │  B'C'                                  │
│ (0,2,8,10) │   -0-0    │  B'D'                                  │
│(2,6,10,14) │   --10    │  CD'                                   │
└────────────┴───────────┴────────────────────────────────────────┘

STEP 5: Identify Prime Implicants (uncombined terms)

Prime Implicants:
  P1: B'C'     (covers 0,1,8,9)
  P2: B'D'     (covers 0,2,8,10)
  P3: CD'      (covers 2,6,10,14)
  P4: A'BD     (covers 5,7) - couldn't combine further
  P5: A'BC     (covers 6,7) - couldn't combine further

STEP 6: Prime Implicant Chart

           │  0 │  1 │  2 │  5 │  6 │  7 │  8 │  9 │ 10 │ 14 │
     ──────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
     B'C'  │  ✓ │  ✓ │    │    │    │    │  ✓ │  ✓ │    │    │
     B'D'  │  ✓ │    │  ✓ │    │    │    │  ✓ │    │  ✓ │    │
     CD'   │    │    │  ✓ │    │  ✓ │    │    │    │  ✓ │  ✓ │
     A'BD  │    │    │    │  ✓ │    │  ✓ │    │    │    │    │
     A'BC  │    │    │    │    │  ✓ │  ✓ │    │    │    │    │

STEP 7: Find Essential Prime Implicants

Essential: A prime implicant that is the ONLY one covering a minterm

  - Minterm 1: Only covered by B'C' → B'C' is ESSENTIAL
  - Minterm 5: Only covered by A'BD → A'BD is ESSENTIAL
  - Minterm 9: Only covered by B'C' → (already selected)
  - Minterm 14: Only covered by CD' → CD' is ESSENTIAL

Essential PIs: B'C', A'BD, CD'

STEP 8: Cover remaining minterms

After selecting essential PIs:
  B'C' covers: 0, 1, 8, 9
  A'BD covers: 5, 7
  CD' covers: 2, 6, 10, 14

All minterms covered!

FINAL ANSWER: F = B'C' + A'BD + CD'
```

---

## 9. Prime Implicants

### Definitions

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRIME IMPLICANT TERMS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  IMPLICANT:                                                     │
│    Any product term that implies the function                  │
│    (outputs 1 only when function outputs 1)                    │
│                                                                 │
│  PRIME IMPLICANT (PI):                                         │
│    An implicant that cannot be combined with another          │
│    implicant to eliminate a variable                          │
│    (Maximal implicant - can't be made larger)                 │
│                                                                 │
│  ESSENTIAL PRIME IMPLICANT (EPI):                              │
│    A prime implicant that covers at least one minterm         │
│    not covered by any other prime implicant                   │
│    (MUST be in final answer)                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Identifying in K-Maps

```
Example: F = Σm(0, 1, 2, 5, 6, 7)

                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  1  │  1  │  0  │  1  │   m₀, m₁, m₂
   A        ├─────┼─────┼─────┼─────┤
        1   │  0  │  1  │  1  │  1  │   m₅, m₇, m₆
            └─────┴─────┴─────┴─────┘

Prime Implicants (largest possible groups):
  PI₁: m₀, m₁ → A'B' (2 cells)
  PI₂: m₁, m₅ → B'C  (2 cells, wrapping columns)
  PI₃: m₅, m₇ → AC   (2 cells)
  PI₄: m₆, m₇ → AB   (2 cells)
  PI₅: m₂, m₆ → BC'  (2 cells)

Essential Prime Implicants:
  - m₀ only in PI₁ → PI₁ is essential
  - m₂ only in PI₅ → PI₅ is essential
  - m₇ in PI₃ and PI₄ → neither essential for m₇
  
After essentials: PI₁ (covers 0,1) + PI₅ (covers 2,6)
Remaining: 5, 7

For m₅: PI₂ or PI₃
For m₇: PI₃ or PI₄

If choose PI₃ (AC): covers both 5 and 7!

Minimal: F = A'B' + BC' + AC
```

### Selection Process

```
┌─────────────────────────────────────────────────────────────────┐
│           PRIME IMPLICANT SELECTION ALGORITHM                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Find ALL prime implicants                                  │
│                                                                 │
│  2. Identify ESSENTIAL prime implicants                        │
│     - Find minterms covered by only one PI                     │
│     - These PIs MUST be in the solution                        │
│                                                                 │
│  3. Remove covered minterms                                    │
│     - Mark minterms covered by essential PIs                   │
│                                                                 │
│  4. Cover remaining minterms                                   │
│     - Use minimum number of remaining PIs                      │
│     - Prefer PIs that cover more uncovered minterms           │
│                                                                 │
│  5. Check for multiple minimal solutions                       │
│     - Different PI combinations may give same cost            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Petrick's Method (for selecting from remaining PIs)

When essential PIs don't cover all minterms:

```
Example: Remaining minterms 4, 5, 6 with PIs:
  P₁ covers 4, 5
  P₂ covers 5, 6
  P₃ covers 4, 6

Create Boolean expression for coverage:
  Minterm 4: P₁ + P₃
  Minterm 5: P₁ + P₂
  Minterm 6: P₂ + P₃

All must be covered:
  (P₁ + P₃)(P₁ + P₂)(P₂ + P₃)
  
Expand:
  = (P₁ + P₂P₃)(P₂ + P₃)
  = P₁P₂ + P₁P₃ + P₂P₃ + P₂P₃
  = P₁P₂ + P₁P₃ + P₂P₃

Minimal solutions (fewest PIs):
  - P₁ and P₂ (2 PIs)
  - P₁ and P₃ (2 PIs)
  - P₂ and P₃ (2 PIs)

Choose any based on literal count or preference.
```

---

## 10. Summary Tables

### K-Map Quick Reference

| Variables | K-Map Size | Max Group Size | Cell Adjacency |
|-----------|------------|----------------|----------------|
| 2 | 2×2 = 4 | 4 | Edge wrap |
| 3 | 2×4 = 8 | 8 | Column wrap |
| 4 | 4×4 = 16 | 16 | All edge wrap |
| 5 | 2×(4×4) = 32 | 32 | 3D wrap |
| 6 | 4×(4×4) = 64 | 64 | 3D wrap |

### Group Size to Variables Eliminated

| Group Size | Variables Eliminated | Example (4-var) |
|------------|---------------------|-----------------|
| 1 | 0 | ABCD |
| 2 | 1 | ABC |
| 4 | 2 | AB |
| 8 | 3 | A |
| 16 | 4 | 1 (constant) |

### K-Map Grouping Rules Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    K-MAP GROUPING RULES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✓ Groups must contain 1, 2, 4, 8, 16... cells (powers of 2)  │
│  ✓ Groups must be rectangular (including wrap-around)          │
│  ✓ Each group should be as large as possible                  │
│  ✓ Every 1 must be covered by at least one group             │
│  ✓ Overlapping groups are allowed                             │
│  ✓ Fewer groups = fewer terms in expression                   │
│                                                                 │
│  ✗ Don't use more groups than necessary                       │
│  ✗ Don't make groups smaller than possible                    │
│  ✗ Don't forget edge wrapping                                 │
│  ✗ Don't forget corner adjacency (4-var maps)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Prime Implicant Terms

| Term | Definition | Required in Solution? |
|------|------------|----------------------|
| Implicant | Any product term → F | Not necessarily |
| Prime Implicant | Maximal implicant (can't enlarge) | Maybe |
| Essential PI | Only PI covering some minterm | Always |
| Redundant PI | PI whose minterms all covered by other PIs | Never |

### Minimization Method Comparison

| Aspect | K-Map | Quine-McCluskey |
|--------|-------|-----------------|
| Variables | Up to 6 | Any number |
| Method | Visual | Tabular |
| Speed (small) | Fast | Slower |
| Speed (large) | Impractical | Systematic |
| Computer | Difficult | Easy to program |
| Error-prone | For large maps | Less so |
| Finding all PIs | May miss some | Guaranteed |

---

## 11. Quick Revision Questions

### Question 1
Minimize using K-map: F(A,B,C) = Σm(0, 1, 2, 4, 6)

<details>
<summary>Click for Answer</summary>

```
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  1  │  1  │  0  │  1  │
   A        ├─────┼─────┼─────┼─────┤
        1   │  1  │  0  │  0  │  1  │
            └─────┴─────┴─────┴─────┘

Groups:
  - m₀, m₁ (pair): A'B'
  - m₀, m₂, m₄, m₆ (corners - wrap): C'

F = A'B' + C'

Verification: 
  A'B' covers 0,1
  C' covers 0,2,4,6
  Together: 0,1,2,4,6 ✓
```
</details>

### Question 2
Find all prime implicants: F(A,B,C,D) = Σm(0, 4, 5, 8, 9, 12, 13)

<details>
<summary>Click for Answer</summary>

```
                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │  1  │  0  │  0  │  0  │  m₀
                ├─────┼─────┼─────┼─────┤
            01  │  1  │  1  │  0  │  0  │  m₄, m₅
      AB        ├─────┼─────┼─────┼─────┤
            11  │  1  │  1  │  0  │  0  │  m₁₂, m₁₃
                ├─────┼─────┼─────┼─────┤
            10  │  1  │  1  │  0  │  0  │  m₈, m₉
                └─────┴─────┴─────┴─────┘

Prime Implicants:
  - PI₁: m₄, m₅, m₁₂, m₁₃ (square): BD'C'... 
    Actually: BC'D' + BC'D = BC' (need to check)
    Cells: (01,00), (01,01), (11,00), (11,01) = BC'
    
  - PI₂: m₀, m₄, m₈, m₁₂ (left column): C'D'
  
  - PI₃: m₄, m₅, m₈, m₉ (middle rows): AB'... check
    (01,00), (01,01), (10,00), (10,01) - not adjacent!
    
  Let me redo:
  Column 00: m₀, m₄, m₁₂, m₈ → C'D' (4 cells)
  Column 01: m₅, m₁₃, m₉ but not m₁ → need pairs
  
  Pairs in column 01:
  - m₄, m₅ + m₁₂, m₁₃ (middle two rows): BC' 
  - m₅, m₁₃: A'BD + ABD = BD (C'D) - no, check binary
    m₅ = 0101, m₁₃ = 1101, differ in A → B'C'D
  - m₈, m₉: AB'C' (pair)
  - m₄, m₅: A'BC' (pair)
  - m₁₂, m₁₃: ABC' (pair)

PIs: C'D', B'C'D, BC'

Essential: C'D' (only covers m₀)
Remaining: 5, 9, 13 all need C'D

F = C'D' + BC'
```
</details>

### Question 3
Use don't cares to minimize: F = Σm(1, 5, 6, 7) + d(2, 3, 4)

<details>
<summary>Click for Answer</summary>

```
3-variable K-map:
                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  0  │  1  │  X  │  X  │  m₁, d₃, d₂
   A        ├─────┼─────┼─────┼─────┤
        1   │  X  │  1  │  1  │  1  │  d₄, m₅, m₇, m₆
            └─────┴─────┴─────┴─────┘

Using don't cares:
  - Treat d₃ as 1: Group m₁, d₃, m₅, m₇ → C
  - Bottom row (using d₄): d₄, m₅, m₇, m₆ → A

F = C + A (or just F = A + C)

Verify: A covers 4,5,6,7; C covers 1,3,5,7
Actual minterms 1,5,6,7 all covered ✓
```
</details>

### Question 4
Apply Quine-McCluskey to: F = Σm(0, 1, 5, 7)

<details>
<summary>Click for Answer</summary>

```
Step 1: Group by 1s count
  Group 0: m₀ = 000
  Group 1: m₁ = 001
  Group 2: m₅ = 101
  Group 3: m₇ = 111

Step 2: Combine
  (0,1): 000 + 001 = 00- → A'B'
  (1,5): 001 + 101 = -01 → B'C
  (5,7): 101 + 111 = 1-1 → AC

Step 3: Check for further combinations
  00- and -01 cannot combine
  
Prime Implicants: A'B', B'C, AC

Step 4: Essential PIs
  m₀: only A'B' → essential
  m₇: only AC → essential
  
Remaining: m₅ covered by B'C or AC
           m₁ covered by A'B' or B'C

A'B' covers 0,1
AC covers 5,7
All covered!

F = A'B' + AC
```
</details>

### Question 5
What is the maximum number of prime implicants for a 3-variable function?

<details>
<summary>Click for Answer</summary>

```
For n variables, maximum PIs occur when function is an XOR-like pattern.

For 3 variables:
  - 8 minterms possible
  - Each single minterm is a PI (if isolated)
  - Maximum PIs = 8 (when no adjacent 1s)

Example: F = Σm(0, 3, 5, 6) - checkerboard pattern

                      BC
              00    01    11    10
            ┌─────┬─────┬─────┬─────┐
        0   │  1  │  0  │  1  │  0  │
   A        ├─────┼─────┼─────┼─────┤
        1   │  0  │  1  │  0  │  1  │
            └─────┴─────┴─────┴─────┘

No adjacent 1s → 4 single-cell PIs
Each minterm is its own PI.

Answer: Maximum = 2^(n-1) for XOR pattern, but 
        theoretically 2^n if all minterms isolated.
        For 3 variables: Maximum practical = 4 PIs
```
</details>

### Question 6
For a 4-variable K-map, how many cells are adjacent to a corner cell?

<details>
<summary>Click for Answer</summary>

```
Consider corner cell m₀ (position 00, 00):

                            CD
                  00    01    11    10
                ┌─────┬─────┬─────┬─────┐
            00  │ [0] │  1  │     │  2  │
                ├─────┼─────┼─────┼─────┤
            01  │  4  │     │     │     │
      AB        ├─────┼─────┼─────┼─────┤
            11  │     │     │     │     │
                ├─────┼─────┼─────┼─────┤
            10  │  8  │     │     │     │
                └─────┴─────┴─────┴─────┘

Adjacent to m₀:
  - m₁ (right)
  - m₂ (wrap right edge)
  - m₄ (down)
  - m₈ (wrap bottom edge)

Answer: 4 cells adjacent to each corner
        (same as any other cell in 4-var K-map)
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 2: Boolean Algebra](Unit-02-Boolean-Algebra-and-Logic-Gates.md) | Return to Index | [Unit 4: Combinational Circuits](Unit-04-Combinational-Circuits.md) |

---

*End of Unit 3*
