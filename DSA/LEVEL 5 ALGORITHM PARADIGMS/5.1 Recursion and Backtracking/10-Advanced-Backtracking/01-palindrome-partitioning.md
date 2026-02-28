# Chapter 1: Palindrome Partitioning

## Overview
**Palindrome Partitioning** splits a string into substrings where every substring is a palindrome. This problem demonstrates backtracking with substring-level choices — at each position, try every possible palindromic prefix, then recurse on the remainder.

---

## Problem Statement

```
Given a string s, partition it such that every 
substring of the partition is a palindrome.
Return ALL possible palindrome partitionings.

Example: s = "aab"

Partitions:
  ["a", "a", "b"]     ← each part is a palindrome ✓
  ["aa", "b"]          ← "aa" palindrome, "b" palindrome ✓

NOT valid:
  ["aab"]              ← "aab" is NOT a palindrome ✗
  ["a", "ab"]          ← "ab" is NOT a palindrome ✗
```

---

## Algorithm Idea

```
At index i, try every possible CUT position j (i ≤ j < n):
  - If s[i..j] is a palindrome → include it, recurse from j+1
  - If not → skip this cut

                    "aab"
                   /  |   \
            "a"|"ab"  "aa"|"b"  "aab"
              ↓         ↓         ✗ not palindrome
          "a"|"b"      "b"
            ↓           ↓
           "b"        (done)
            ↓
          (done)

Results: ["a","a","b"] and ["aa","b"]
```

---

## Pseudocode

```
FUNCTION partition(s):
    result = []
    backtrack(s, 0, [], result)
    RETURN result

FUNCTION backtrack(s, start, current, result):
    IF start == length(s):
        result.add(copy(current))        ← Found valid partition!
        RETURN
    
    FOR end = start TO length(s)-1:
        IF isPalindrome(s, start, end):
            current.add(s[start..end])   ← CHOOSE palindrome
            backtrack(s, end+1, current, result)  ← EXPLORE
            current.removeLast()         ← UNDO
        // If not palindrome, skip (implicit pruning)

FUNCTION isPalindrome(s, left, right):
    WHILE left < right:
        IF s[left] != s[right]: RETURN FALSE
        left++; right--
    RETURN TRUE
```

---

## Step-by-Step Trace: s = "aab"

```
backtrack("aab", 0, [], result)
│
├─ end=0: s[0..0]="a" palindrome? YES
│  current = ["a"]
│  backtrack("aab", 1, ["a"], result)
│  │
│  ├─ end=1: s[1..1]="a" palindrome? YES
│  │  current = ["a","a"]
│  │  backtrack("aab", 2, ["a","a"], result)
│  │  │
│  │  ├─ end=2: s[2..2]="b" palindrome? YES
│  │  │  current = ["a","a","b"]
│  │  │  backtrack("aab", 3, ["a","a","b"], result)
│  │  │  │ start==3==len → ADD ["a","a","b"] ✓
│  │  │  removeLast → ["a","a"]
│  │  │
│  │  removeLast → ["a"]
│  │
│  ├─ end=2: s[1..2]="ab" palindrome? NO → skip
│  │
│  removeLast → []
│
├─ end=1: s[0..1]="aa" palindrome? YES
│  current = ["aa"]
│  backtrack("aab", 2, ["aa"], result)
│  │
│  ├─ end=2: s[2..2]="b" palindrome? YES
│  │  current = ["aa","b"]
│  │  backtrack("aab", 3, ["aa","b"], result)
│  │  │ start==3==len → ADD ["aa","b"] ✓
│  │  removeLast → ["aa"]
│  │
│  removeLast → []
│
├─ end=2: s[0..2]="aab" palindrome? NO → skip

Result: [["a","a","b"], ["aa","b"]]
```

---

## Optimization: Precompute Palindromes

```
Checking isPalindrome(s, i, j) each time costs O(n).
Precompute with DP:

dp[i][j] = TRUE if s[i..j] is a palindrome

Base cases:
  dp[i][i] = TRUE           (single char)
  dp[i][i+1] = (s[i]==s[i+1])  (two chars)

Transition:
  dp[i][j] = (s[i]==s[j]) AND dp[i+1][j-1]

Fill diagonally (length 1 → 2 → 3 → ... → n)

This reduces palindrome check from O(n) to O(1)!

Example for "aab":
       j→ 0   1   2
  i↓  ┌───┬───┬───┐
   0  │ T │ T │ F │  "a"✓ "aa"✓ "aab"✗
   1  │   │ T │ F │       "a"✓  "ab"✗
   2  │   │   │ T │             "b"✓
      └───┴───┴───┘
```

---

## Complexity

```
┌──────────────────────────────────────────────┐
│ Number of partitions: O(2^(n-1))             │
│   n-1 positions to place cuts, each yes/no   │
│                                              │
│ Time: O(n × 2^n)                             │
│   2^(n-1) partitions × O(n) to copy each     │
│   With precomputed palindromes, check is O(1) │
│                                              │
│ Space: O(n) recursion depth + O(n²) DP table │
│   + O(n × 2^n) to store all results          │
└──────────────────────────────────────────────┘
```

---

## Variants

| Variant | Description |
|---------|-------------|
| Palindrome Partitioning I | Return all partitions (this problem) |
| Palindrome Partitioning II | Minimum number of cuts (pure DP) |
| Palindrome Partitioning III | Change at most k chars, min cuts (DP) |
| Palindrome Partitioning IV | Split into exactly 3 palindromes (O(n²)) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Input | String s |
| Output | All palindrome partitions |
| Choice at each step | Where to cut next (end position) |
| Constraint | Substring must be palindrome |
| Pruning | Skip non-palindrome substrings |
| Optimization | Precompute palindrome table O(n²) |
| Time | O(n × 2^n) |
| Space | O(n²) for DP table |

---

## Quick Revision Questions

1. **Where does the branching** happen in this algorithm?
2. **Why do we skip** non-palindrome substrings?
3. **How does precomputing** palindromes improve performance?
4. **How many possible partitions** can a string of length n have?
5. **What's the difference** between Palindrome Partitioning I and II?
6. **Why do we copy** the current list before adding to result?

---

[← Previous: Unit 9 - Knight's Tour](../09-Grid-Backtracking/06-knights-tour.md) | [Next: Expression Add Operators →](02-expression-add-operators.md)

[← Back to README](../README.md)
