# Chapter 4: Subsequences of a String

## Overview
Generating all subsequences is one of the most important recursive problems. It introduces the **include/exclude pattern** — for each element, you either include it or skip it. This pattern is the foundation for subset generation and many backtracking problems.

---

## What is a Subsequence?

```
A subsequence is formed by deleting zero or more characters
from a string WITHOUT changing the relative order.

String: "abc"
Subsequences: "", "a", "b", "c", "ab", "ac", "bc", "abc"
Total: 2^n = 2^3 = 8 subsequences

[!] "ba" is NOT a subsequence of "abc" (order changed!)
[!] "ac" IS a subsequence (skipped 'b', order preserved)
```

---

## The Include/Exclude Pattern

```
┌──────────────────────────────────────────────────┐
│         INCLUDE / EXCLUDE PATTERN                 │
│                                                  │
│  For each character, make a CHOICE:              │
│                                                  │
│        current character                         │
│           /          \                           │
│      INCLUDE it    EXCLUDE it                    │
│         /              \                         │
│   recurse on rest   recurse on rest              │
│                                                  │
│  This creates a BINARY TREE of choices           │
│  with 2^n leaves = 2^n subsequences              │
└──────────────────────────────────────────────────┘
```

---

## Recursive Solution

### Pseudocode
```
function subsequences(s, index, current, result):
    if index >= length(s):
        result.add(current)          // Base: add what we've built
        return
    
    // Choice 1: INCLUDE s[index]
    subsequences(s, index + 1, current + s[index], result)
    
    // Choice 2: EXCLUDE s[index]
    subsequences(s, index + 1, current, result)
```

### Trace: subsequences("abc", 0, "", [])

```
                          ""
                    /            \
               "a"                ""
             /      \          /      \
          "ab"      "a"      "b"       ""
         /    \    /   \    /   \    /    \
      "abc" "ab" "ac" "a" "bc" "b" "c"   ""
        ↓    ↓    ↓    ↓    ↓    ↓   ↓    ↓
      Leaves (all 8 subsequences)

Decision tree:
  Index 0 ('a'): Include 'a' or Skip 'a'?
  Index 1 ('b'): Include 'b' or Skip 'b'?
  Index 2 ('c'): Include 'c' or Skip 'c'?

Each path from root to leaf = one subsequence
Left branch = INCLUDE, Right branch = EXCLUDE
```

### Detailed Step-by-Step

```
Call: subseq("abc", 0, "")
├── INCLUDE 'a': subseq("abc", 1, "a")
│   ├── INCLUDE 'b': subseq("abc", 2, "ab")
│   │   ├── INCLUDE 'c': subseq("abc", 3, "abc") → add "abc"
│   │   └── EXCLUDE 'c': subseq("abc", 3, "ab")  → add "ab"
│   └── EXCLUDE 'b': subseq("abc", 2, "a")
│       ├── INCLUDE 'c': subseq("abc", 3, "ac")  → add "ac"
│       └── EXCLUDE 'c': subseq("abc", 3, "a")   → add "a"
└── EXCLUDE 'a': subseq("abc", 1, "")
    ├── INCLUDE 'b': subseq("abc", 2, "b")
    │   ├── INCLUDE 'c': subseq("abc", 3, "bc")  → add "bc"
    │   └── EXCLUDE 'c': subseq("abc", 3, "b")   → add "b"
    └── EXCLUDE 'b': subseq("abc", 2, "")
        ├── INCLUDE 'c': subseq("abc", 3, "c")   → add "c"
        └── EXCLUDE 'c': subseq("abc", 3, "")    → add ""

Result: ["abc", "ab", "ac", "a", "bc", "b", "c", ""]
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────────┐
│         COMPLEXITY ANALYSIS                       │
│                                                  │
│  Total subsequences: 2^n                          │
│  (each character has 2 choices: in or out)        │
│                                                  │
│  Time:  O(n × 2^n)                               │
│    - 2^n subsequences generated                   │
│    - Each up to length n (string building)        │
│                                                  │
│  Space: O(n) for recursion depth                  │
│    + O(n × 2^n) to store all results              │
│                                                  │
│  Recursion tree:                                  │
│    Depth = n                                     │
│    Total nodes = 2^(n+1) - 1                     │
│    Leaves = 2^n (each = one subsequence)          │
└──────────────────────────────────────────────────┘
```

---

## Key Insight: Binary Representation

```
Each subsequence maps to a BINARY NUMBER:
  For "abc" (n=3), use 3-bit numbers:

  000 → ""      (skip all)
  001 → "c"     (include only c)
  010 → "b"     (include only b)
  011 → "bc"    (include b and c)
  100 → "a"     (include only a)
  101 → "ac"    (include a and c)
  110 → "ab"    (include a and b)
  111 → "abc"   (include all)

  Total: 2^3 = 8 subsequences

  This connection to binary → iterative approach using bit masks!
```

---

## Real-World Applications

- **Bioinformatics**: DNA sequence matching
- **Combinatorics**: Generating all possible selections
- **Dynamic programming**: LCS (Longest Common Subsequence)
- **String matching**: Pattern matching with gaps

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Pattern | Include/Exclude for each character |
| Total count | 2^n subsequences |
| Base case | Reached end of string → add current |
| Time | O(n × 2^n) |
| Space | O(n) depth + O(n × 2^n) output |
| Key connection | Binary representation of choices |
| Foundation for | Subset generation, combination problems |

---

## Quick Revision Questions

1. **What is the difference** between a subsequence and a substring?
2. **Why are there exactly 2^n** subsequences of a string of length n?
3. **Draw the decision tree** for subsequences of "ab".
4. **How does the binary representation** relate to subsequences?
5. **What is the include/exclude pattern** and where else is it used?
6. **What would be the total subsequences** of a string of length 5?

---

[← Previous: Remove Character](03-remove-character.md) | [Next: Permutations Basics →](05-permutations-basics.md)

[← Back to README](../README.md)
