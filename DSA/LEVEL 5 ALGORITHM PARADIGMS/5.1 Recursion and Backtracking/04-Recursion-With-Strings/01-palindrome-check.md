# Chapter 1: Palindrome Check

## Overview
Checking if a string is a palindrome recursively uses the **two-pointer shrink** pattern — compare outermost characters and recurse on the inner substring. This is an elegant recursive solution.

---

## Problem Statement
A palindrome reads the same forwards and backwards: "racecar", "madam", "level".

---

## Recursive Solution

### Pseudocode
```
function isPalindrome(s, left, right):
    if left >= right: return true          // Base: 0 or 1 chars left
    if s[left] != s[right]: return false   // Mismatch found
    return isPalindrome(s, left + 1, right - 1)  // Check inner
```

### Trace: isPalindrome("racecar", 0, 6)

```
Step 1: s[0]='r' == s[6]='r' ✓ → recurse(1, 5)
Step 2: s[1]='a' == s[5]='a' ✓ → recurse(2, 4)
Step 3: s[2]='c' == s[4]='c' ✓ → recurse(3, 3)
Step 4: left=3 >= right=3 → return true ← BASE

  r  a  c  e  c  a  r
  ↕                 ↕   Step 1: r==r ✓
     ↕           ↕      Step 2: a==a ✓
        ↕     ↕         Step 3: c==c ✓
           ↕            Step 4: single char, done!
```

### Trace: NOT a palindrome — "hello"

```
Step 1: s[0]='h' != s[4]='o' ✗ → return false

  h  e  l  l  o
  ↕            ↕   h ≠ o → NOT palindrome (immediate!)
```

---

## Complexity

```
Time:  O(n/2) = O(n) — check at most n/2 pairs
Space: O(n/2) = O(n) — stack depth
Best:  O(1) — first pair mismatch
```

---

## Variations

| Variation | Approach |
|-----------|----------|
| Case-insensitive | Convert to lowercase first |
| Ignore spaces/punctuation | Skip non-alphanumeric characters |
| Numeric palindrome | Convert number to string or extract digits |
| Longest palindromic substring | Expand from center (different approach) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Pattern | Two-pointer shrink from outside in |
| Base case | left >= right → true |
| Early exit | Mismatch → false |
| Time | O(n) |
| Space | O(n) stack |

---

## Quick Revision Questions

1. **Why is "left >= right" the base case** instead of "left == right"?
2. **What is the best case** time complexity and when?
3. **How would you handle** case-insensitive palindrome checking?
4. **Trace isPalindrome("abcba", 0, 4)**.
5. **Is an empty string** a palindrome? What about a single character?
6. **How many recursive calls** for a palindrome of length n?

---

[← Previous: First and Last Occurrence](../03-Recursion-With-Arrays/06-first-and-last-occurrence.md) | [Next: String Reversal →](02-string-reversal.md)

[← Back to README](../README.md)
