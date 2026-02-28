# Chapter 6: String Permutations

## Overview
Generating permutations of **characters in a string** introduces string-specific concerns: **immutability** (in some languages), **character-level operations**, and the frequent need to handle **duplicate characters**.

---

## Problem Variants

```
1. All permutations of a string (no duplicates)
   "abc" → "abc","acb","bac","bca","cab","cba"

2. All UNIQUE permutations (with duplicates)
   "aab" → "aab","aba","baa"  (not 6, just 3)

3. Permutations of a substring (k out of n chars)
   "abcd", k=2 → "ab","ac","ad","ba","bc","bd",...
```

---

## Approach 1: Swap-Based for Strings

```
Convert string to char array (strings are often immutable)

FUNCTION permuteString(s):
    chars = s.toCharArray()
    result = []
    backtrack(chars, 0, result)
    RETURN result

FUNCTION backtrack(chars, start, result):
    IF start == len(chars):
        result.add(charsToString(chars))
        RETURN
    
    FOR i = start TO len(chars) - 1:
        swap(chars, start, i)
        backtrack(chars, start + 1, result)
        swap(chars, start, i)           ← Undo swap
```

---

## Approach 2: Frequency Map for Unique Permutations

```
Best approach when string has many duplicates.

FUNCTION permuteUnique(s):
    freq = buildFrequencyMap(s)  ← {'a':2, 'b':1} for "aab"
    result = []
    backtrack(freq, len(s), [], result)
    RETURN result

FUNCTION backtrack(freq, remaining, current, result):
    IF remaining == 0:
        result.add(join(current))
        RETURN
    
    FOR each (char, count) in freq:     ← Iterate unique chars only
        IF count == 0: CONTINUE
        
        current.add(char)
        freq[char] -= 1
        backtrack(freq, remaining - 1, current, result)
        current.removeLast()
        freq[char] += 1
```

---

## Trace: permuteUnique("aab")

```
freq = {'a': 2, 'b': 1}

backtrack(rem=3, [])
├── char='a' (count=2→1): backtrack(rem=2, ['a'])
│   ├── char='a' (count=1→0): backtrack(rem=1, ['a','a'])
│   │   ├── char='a': count=0 → SKIP
│   │   └── char='b' (count=1→0): backtrack(rem=0, ['a','a','b'])
│   │       → RECORD "aab" ✓
│   └── char='b' (count=1→0): backtrack(rem=1, ['a','b'])
│       ├── char='a' (count=1→0): backtrack(rem=0, ['a','b','a'])
│       │   → RECORD "aba" ✓
│       └── char='b': count=0 → SKIP
└── char='b' (count=1→0): backtrack(rem=2, ['b'])
    ├── char='a' (count=2→1): backtrack(rem=1, ['b','a'])
    │   ├── char='a' (count=1→0): backtrack(rem=0, ['b','a','a'])
    │   │   → RECORD "baa" ✓
    │   └── char='b': count=0 → SKIP
    └── char='b': count=0 → SKIP

Result: ["aab", "aba", "baa"]  ✓  (3 = 3!/2! )
```

---

## Handling Duplicates in Swap-Based Approach

```
For swap-based, duplicates are trickier.
Use a SET to track what's been placed at each position:

FUNCTION backtrack(chars, start, result):
    IF start == len(chars):
        result.add(charsToString(chars))
        RETURN
    
    seen = new Set()                    ← Track chars tried at this level
    FOR i = start TO len(chars) - 1:
        IF chars[i] IN seen: CONTINUE   ← Skip if already tried this char
        seen.add(chars[i])
        swap(chars, start, i)
        backtrack(chars, start + 1, result)
        swap(chars, start, i)

This prevents placing the SAME character at the same position twice.
```

---

## Partial Permutations (k out of n)

```
Generate all k-length strings from characters:

FUNCTION partialPerms(s, k):
    chars = s.toCharArray()
    result = []
    used = [false] × len(chars)
    backtrack(chars, k, used, [], result)
    RETURN result

FUNCTION backtrack(chars, k, used, current, result):
    IF len(current) == k:
        result.add(join(current))
        RETURN
    
    FOR i = 0 TO len(chars) - 1:
        IF used[i]: CONTINUE
        used[i] = true
        current.add(chars[i])
        backtrack(chars, k, used, current, result)
        current.removeLast()
        used[i] = false

"abcd", k=2 → "ab","ac","ad","ba","bc","bd","ca","cb","cd","da","db","dc"
Count: P(4,2) = 4!/2! = 12 ✓
```

---

## Applications

```
┌──────────────────────────────────────────────┐
│ • Anagram generation                         │
│ • Password cracking (all arrangements)       │
│ • Puzzle solving (word games)                │
│ • Cryptography (transposition ciphers)       │
│ • Testing (generate all input orderings)     │
│ • Combinatorial optimization                 │
└──────────────────────────────────────────────┘
```

---

## Complexity

```
┌────────────────────────────────────────────────────┐
│ No duplicates: O(n × n!) time, O(n) space          │
│                                                    │
│ With duplicates: O(n × P) where                    │
│   P = n! / (c₁! × c₂! × ... × cₖ!)                │
│                                                    │
│ Partial (k of n): O(k × P(n,k)) where              │
│   P(n,k) = n! / (n-k)!                            │
└────────────────────────────────────────────────────┘
```

---

## Summary Table

| Approach | Handles Dups? | Space | Best For |
|----------|-------------|-------|----------|
| Swap-based | With set | O(n) | Simple, in-place |
| Frequency map | Naturally | O(n) | Many duplicates |
| Used-array + sort | With skip condition | O(n) | Clear logic |

---

## Quick Revision Questions

1. **Why convert strings** to char arrays for permutations?
2. **How does the frequency map** naturally avoid duplicates?
3. **What is P(n, k)** and when do you use it?
4. **How many unique permutations** of "mississippi"?
5. **What's the difference** between the set approach and sort+skip for swap-based?
6. **Generate all permutations** of "aba" using the frequency map method.

---

[← Previous: Beautiful Arrangement](05-beautiful-arrangement.md) | [Next Unit: Grid Backtracking →](../09-Grid-Backtracking/01-rat-in-a-maze.md)

[← Back to README](../README.md)
