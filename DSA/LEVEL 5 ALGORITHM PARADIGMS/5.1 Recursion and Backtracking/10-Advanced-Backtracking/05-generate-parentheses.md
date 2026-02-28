# Chapter 5: Generate Parentheses

## Overview
**Generate Parentheses** produces all valid combinations of `n` pairs of parentheses. This elegant problem demonstrates how constraints (open/close balance) naturally prune the search tree, and serves as a gateway to understanding **Catalan numbers**.

---

## Problem Statement

```
Input: n = 3
Output: ["((()))", "(()())", "(())()", "()(())", "()()()"]

Rules for valid parentheses:
  1. Total: exactly n '(' and n ')' 
  2. At every prefix, count('(') >= count(')')
     → never close more than you've opened

n=1: ["()"]                    → 1 combination
n=2: ["(())", "()()"]         → 2 combinations
n=3: 5 combinations
n=4: 14 combinations
n=5: 42 combinations
     → These are the CATALAN NUMBERS!
```

---

## Decision Tree

```
n = 2, building character by character:

                     ""
                     |
                    "("        ← Must start with (
                   /    \
              "(("       "()"
             /    \        |
          "(()"   ---     "()" 
            |           can add (
           "(())"       "()("
            ✓             |
                        "()()"
                          ✓

Key: At each step, two choices:
  Add '(' if: open < n
  Add ')' if: close < open
```

---

## Pseudocode

```
FUNCTION generateParenthesis(n):
    result = []
    backtrack("", 0, 0, n, result)
    RETURN result

FUNCTION backtrack(current, open, close, n, result):
    IF length(current) == 2*n:
        result.add(current)              ← Valid combination!
        RETURN
    
    IF open < n:
        backtrack(current + "(", open+1, close, n, result)
    
    IF close < open:
        backtrack(current + ")", open, close+1, n, result)
```

---

## Full Trace: n = 3

```
backtrack("", 0, 0)
├── "(" (open=1)
│   ├── "((" (open=2)
│   │   ├── "(((" (open=3)
│   │   │   ├── "((()" (close=1)
│   │   │   │   ├── "((())" (close=2)
│   │   │   │   │   └── "((()))" (close=3) ✓ ADD
│   │   ├── "(()": (close=1)
│   │   │   ├── "(()(": (open=3)
│   │   │   │   ├── "(()()": (close=2)
│   │   │   │   │   └── "(()())" ✓ ADD
│   │   │   ├── "(())": (close=2)
│   │   │   │   ├── "(())(": (open=3)
│   │   │   │   │   └── "(())()": ✓ ADD
│   ├── "()" (close=1)
│   │   ├── "()(" (open=2)
│   │   │   ├── "()((" (open=3)
│   │   │   │   ├── "()(()" (close=2)
│   │   │   │   │   └── "()(())" ✓ ADD
│   │   │   ├── "()()" (close=2)
│   │   │   │   ├── "()()(" (open=3)
│   │   │   │   │   └── "()()()" ✓ ADD

Result: ["((()))", "(()())", "(())()", "()(())", "()()()"]
```

---

## Why This Works (Constraint as Pruning)

```
Without constraints: 2^(2n) binary strings
  For n=3: 2^6 = 64 possible strings

With constraints:
  open < n → limits '(' count
  close < open → ensures validity
  
  For n=3: only 5 valid (vs 64 possible)

This gives the n-th CATALAN NUMBER:
  
  C(n) = (2n)! / ((n+1)! × n!)
  
  C(0)=1, C(1)=1, C(2)=2, C(3)=5, 
  C(4)=14, C(5)=42, C(6)=132

The two constraints perfectly encode "balanced" strings.
```

---

## Alternative: StringBuilder / Char Array

```
String concatenation creates new objects each time.
Use a char array or mutable string for efficiency:

FUNCTION backtrack(chars, pos, open, close, n, result):
    IF pos == 2*n:
        result.add(new String(chars))
        RETURN
    
    IF open < n:
        chars[pos] = '('
        backtrack(chars, pos+1, open+1, close, n, result)
    
    IF close < open:
        chars[pos] = ')'
        backtrack(chars, pos+1, open, close+1, n, result)

Note: No explicit UNDO needed!
  The next call overwrites chars[pos].
  This is an implicit backtrack.
```

---

## Variations

```
1. GENERATE BRACKET TYPES:
   Multiple bracket types: (), [], {}
   Constraint: each type independently balanced,
   AND proper nesting (no "([)]")

2. COUNT ONLY:
   Just return C(n) = (2n)! / ((n+1)! × n!)
   No backtracking needed! O(n) computation.

3. K-TH COMBINATION:
   Given k, find the k-th lexicographic parenthesization.
   Use Catalan number counting to skip subtrees.

4. REMOVE INVALID PARENTHESES:
   Given string with extra parens, remove minimum
   to make valid. (BFS or backtracking approach)
```

---

## Complexity

```
┌──────────────────────────────────────────┐
│ Output size: C(n) = n-th Catalan number  │
│   C(n) ≈ 4^n / (n√n)                    │
│                                          │
│ Time:  O(C(n) × n)                       │
│   C(n) valid strings × O(n) to copy each │
│   ≈ O(4^n / √n)                          │
│                                          │
│ Space: O(n) recursion depth              │
│   + O(C(n) × n) to store results         │
│                                          │
│ Tree nodes explored: O(C(n))             │
│   (no wasted exploration — each leaf is  │
│    a valid result!)                       │
└──────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Choices | Add '(' or ')' |
| Constraint 1 | open < n (don't exceed n opens) |
| Constraint 2 | close < open (never more closes than opens) |
| String length | Always 2n |
| Result count | C(n) = n-th Catalan number |
| Explicit undo? | No (char position overwritten) |
| Time | O(4^n / √n) |
| No pruning waste | Every leaf is valid |

---

## Quick Revision Questions

1. **What two conditions** control when to add '(' vs ')'?
2. **Why is `close < open`** the key validity constraint?
3. **What are Catalan numbers** and how do they relate?
4. **Why doesn't this algorithm** need an explicit undo step?
5. **How many valid strings** exist for n = 4?
6. **How would you count** without generating all strings?

---

[← Previous: N-Queens II](04-n-queens-ii.md) | [Next: Restore IP Addresses →](06-restore-ip-addresses.md)

[← Back to README](../README.md)
