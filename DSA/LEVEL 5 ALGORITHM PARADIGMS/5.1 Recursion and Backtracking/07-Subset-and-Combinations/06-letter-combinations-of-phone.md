# Chapter 6: Letter Combinations of a Phone Number

## Overview
This classic problem maps **digits to letters** (like a phone keypad) and generates all possible letter combinations. It's a beautiful example of **k-ary branching** backtracking where each level has a different set of choices.

---

## Problem Statement

```
Phone keypad mapping:
  2 → "abc"    5 → "jkl"    8 → "tuv"
  3 → "def"    6 → "mno"    9 → "wxyz"
  4 → "ghi"    7 → "pqrs"

Input:  digits = "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]

Each digit maps to 3-4 letters.
Generate ALL possible letter strings.
```

---

## The State Space Tree

```
Input: "23"

Digit 2 → {a, b, c}     Digit 3 → {d, e, f}

                    ""
               /    |    \
             "a"   "b"   "c"        ← choices for digit '2'
            /|\   /|\    /|\
          ad ae af bd be bf cd ce cf  ← choices for digit '3'

9 combinations = 3 × 3 ✓

For "234": 3 × 3 × 3 = 27 combinations
For "79":  4 × 4 = 16 combinations
```

---

## Pseudocode

```
MAPPING = {
    '2': "abc", '3': "def", '4': "ghi",
    '5': "jkl", '6': "mno", '7': "pqrs",
    '8': "tuv", '9': "wxyz"
}

FUNCTION letterCombinations(digits):
    IF digits is empty: RETURN []
    result = []
    backtrack(digits, 0, [], result)
    RETURN result

FUNCTION backtrack(digits, index, current, result):
    IF index == len(digits):            ← Processed all digits
        result.add(join(current))
        RETURN
    
    letters = MAPPING[digits[index]]    ← Get letters for this digit
    
    FOR each letter in letters:
        current.add(letter)             ← MAKE choice
        backtrack(digits, index + 1, current, result)  ← EXPLORE
        current.removeLast()            ← UNDO choice
```

---

## Complete Trace: digits = "23"

```
backtrack(index=0, current=[])
│  digit='2' → letters="abc"
├── letter='a': backtrack(index=1, ['a'])
│   │  digit='3' → letters="def"
│   ├── letter='d': backtrack(index=2, ['a','d'])
│   │   → index==len → RECORD "ad" ✓
│   ├── letter='e': backtrack(index=2, ['a','e'])
│   │   → RECORD "ae" ✓
│   └── letter='f': backtrack(index=2, ['a','f'])
│       → RECORD "af" ✓
├── letter='b': backtrack(index=1, ['b'])
│   ├── letter='d': → RECORD "bd" ✓
│   ├── letter='e': → RECORD "be" ✓
│   └── letter='f': → RECORD "bf" ✓
└── letter='c': backtrack(index=1, ['c'])
    ├── letter='d': → RECORD "cd" ✓
    ├── letter='e': → RECORD "ce" ✓
    └── letter='f': → RECORD "cf" ✓

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

## Why No Start Index Needed?

```
SUBSETS/COMBINATIONS:          PHONE LETTERS:
  Choose FROM the same pool      Each level has DIFFERENT choices
  Need start to avoid repeats    No overlap between levels
  "Which elements to include?"   "Which letter for THIS digit?"

Subsets: [1,2,3] → choose which to include
  start prevents picking 2 before 1 AND 1 before 2

Phone: "23" → digit 2 has {a,b,c}, digit 3 has {d,e,f}
  Completely separate choice sets! No start needed.
```

---

## Alternative: Iterative (BFS) Approach

```
FUNCTION letterCombinationsIterative(digits):
    IF digits is empty: RETURN []
    result = [""]
    
    FOR each digit in digits:
        letters = MAPPING[digit]
        newResult = []
        FOR each existing in result:
            FOR each letter in letters:
                newResult.add(existing + letter)
        result = newResult
    
    RETURN result

Step-by-step for "23":
  Start:     [""]
  After '2': ["a", "b", "c"]
  After '3': ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

## Complexity Analysis

```
Let n = number of digits
Let k = max letters per digit (k=4 for digits 7 and 9)

┌────────────────────────────────────────────────┐
│ Total combinations: product of letter counts    │
│   For all 3-letter digits: 3^n                 │
│   For all 4-letter digits: 4^n                 │
│   General: O(4^n) upper bound                  │
│                                                │
│ Time: O(n × 4^n)                               │
│   4^n combinations, each of length n            │
│                                                │
│ Space: O(n) recursion depth + current string    │
└────────────────────────────────────────────────┘
```

---

## Pattern Recognition: Variable Branching

```
This problem demonstrates VARIABLE BRANCHING:

Level 0 (digit '2'): 3 branches (a, b, c)
Level 1 (digit '7'): 4 branches (p, q, r, s)
Level 2 (digit '3'): 3 branches (d, e, f)

Total = 3 × 4 × 3 = 36 combinations

The template adapts naturally:
  The "candidates" change at each level
  candidates = MAPPING[digits[currentLevel]]
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Branching factor | Variable (3 or 4 per digit) |
| Depth | Number of digits |
| Total solutions | Product of letter counts per digit |
| Pruning needed? | No (all combinations valid) |
| Start index needed? | No (different choices per level) |
| Time complexity | O(n × 4^n) |
| Space complexity | O(n) |

---

## Quick Revision Questions

1. **Why doesn't this problem** need a start index?
2. **What is the maximum branching factor** and for which digits?
3. **How many combinations** does "79" produce?
4. **Convert the recursive solution** to an iterative one.
5. **What if the digit '1'** mapped to no letters — how would you handle it?
6. **How is this problem different** from generating permutations?

---

[← Previous: Combination Sum II](05-combination-sum-ii.md) | [Next Unit: Permutations →](../08-Permutations/01-generate-permutations.md)

[← Back to README](../README.md)
