# Chapter 4: Longest Substring Without Repeating Characters

[← Previous: Max Sum Subarray (Fixed)](03-max-sum-subarray-fixed.md) | [Back to README](../README.md) | [Next: Minimum Window Substring →](05-minimum-window-substring.md)

---

## Overview

**Problem:** Given a string, find the length of the **longest substring without repeating characters**.

This is LeetCode #3 and one of the most classic variable-size sliding window problems. It beautifully demonstrates window expansion and contraction.

---

## Problem Understanding

```
  Input:  "abcabcbb"
  Output: 3

  Explanation: "abc" is the longest substring without repeating chars.

  All possible substrings without repeats:
  "a"(1), "ab"(2), "abc"(3), "bca"(3), "cab"(3),
  "abc"(3), "cb"(2), "b"(1), "b"(1)
  
  Maximum length = 3
```

### More Examples

```
  "bbbbb"   → 1  ("b")
  "pwwkew"  → 3  ("wke")
  ""        → 0
  "abcdef"  → 6  (entire string)
  "dvdf"    → 3  ("vdf")
```

---

## Approach 1: Brute Force - O(n³)

Check every possible substring and verify it has no duplicates.

```
FUNCTION longestBrute(s, n):
    maxLen = 0

    FOR i = 0 TO n-1:
        FOR j = i TO n-1:
            IF hasAllUnique(s, i, j):
                maxLen = MAX(maxLen, j - i + 1)

    RETURN maxLen

FUNCTION hasAllUnique(s, start, end):
    charSet = empty set
    FOR k = start TO end:
        IF s[k] IN charSet:
            RETURN false
        ADD s[k] TO charSet
    RETURN true
```

```
  Time:  O(n³) — two loops + uniqueness check
  Space: O(min(n, alphabet_size))
```

---

## Approach 2: Sliding Window with Set — O(n)

```
FUNCTION lengthOfLongestSubstring(s, n):
    charSet = empty set
    left = 0
    maxLen = 0

    FOR right = 0 TO n-1:
        // Contract: remove chars until no duplicate
        WHILE s[right] IN charSet:
            REMOVE s[left] FROM charSet
            left = left + 1

        // Expand: add current char
        ADD s[right] TO charSet

        // Update answer
        maxLen = MAX(maxLen, right - left + 1)

    RETURN maxLen
```

---

## Detailed Trace: "abcabcbb"

```
  String:  a  b  c  a  b  c  b  b
  Index:   0  1  2  3  4  5  6  7
  
  ┌─────┬──────────┬─────┬────────────────────────────────┬─────────┐
  │right│ char     │left │ set after step                 │ maxLen  │
  ├─────┼──────────┼─────┼────────────────────────────────┼─────────┤
  │  0  │ 'a'      │  0  │ {a}                            │ 1       │
  │  1  │ 'b'      │  0  │ {a,b}                          │ 2       │
  │  2  │ 'c'      │  0  │ {a,b,c}                        │ 3       │
  │  3  │ 'a' dup! │  1  │ remove 'a' → {b,c}, add 'a'   │ 3       │
  │     │          │     │ → {a,b,c}                      │         │
  │  4  │ 'b' dup! │  2  │ remove 'b' → {a,c}, add 'b'   │ 3       │
  │     │          │     │ → {a,b,c}                      │         │
  │  5  │ 'c' dup! │  3  │ remove 'c' → {a,b}, add 'c'   │ 3       │
  │     │          │     │ → {a,b,c}                      │         │
  │  6  │ 'b' dup! │  4  │ remove 'a' → {b,c}            │ 3       │
  │     │          │     │ remove 'b' → {c}, add 'b'      │         │
  │     │          │  5  │ → {b,c}                        │         │
  │  7  │ 'b' dup! │  6  │ remove 'c' → {b}              │ 3       │
  │     │          │     │ remove 'b' → {}, add 'b'       │         │
  │     │          │  7  │ → {b}                          │         │
  └─────┴──────────┴─────┴────────────────────────────────┴─────────┘

  Answer: 3  ✓
```

### Visual Window Movement

```
  a b c a b c b b
  [─────]              "abc"  len=3
    [─────]            "bca"  len=3
      [─────]          "cab"  len=3
        [─────]        "abc"  len=3
          [───]        "bc"   len=2 (shrunk extra)
            [─]        "b"    len=1

  Max = 3
```

---

## Approach 3: Sliding Window with HashMap (Optimized) — O(n)

Instead of removing characters one-by-one, **jump `left` directly** to the position after the last occurrence of the duplicate character.

```
FUNCTION lengthOfLongestSubstring(s, n):
    charIndex = empty hashmap    // char → last seen index
    left = 0
    maxLen = 0

    FOR right = 0 TO n-1:
        IF s[right] IN charIndex AND charIndex[s[right]] >= left:
            // Jump left past the previous occurrence
            left = charIndex[s[right]] + 1

        charIndex[s[right]] = right
        maxLen = MAX(maxLen, right - left + 1)

    RETURN maxLen
```

### Key Insight

```
  "a b c d b e f"
   0 1 2 3 4 5 6

  When right=4 ('b'), we see 'b' was last at index 1.
  
  SET approach: remove 'a', then 'b' → left moves 0→1→2
  MAP approach: left jumps directly to 2 (index 1 + 1)

  ┌──────────────────────────────────────────┐
  │  HashMap saves unnecessary removals!     │
  │  Worst case is still O(n), but fewer     │
  │  operations in practice.                 │
  └──────────────────────────────────────────┘
```

---

## Trace with HashMap: "abba"

```
  s = "a b b a"    indices: 0 1 2 3

  right=0, char='a':
    map = {} → 'a' not found
    map['a'] = 0
    maxLen = max(0, 0-0+1) = 1
    left=0, window "a"

  right=1, char='b':
    map = {a:0} → 'b' not found
    map['b'] = 1
    maxLen = max(1, 1-0+1) = 2
    left=0, window "ab"

  right=2, char='b':
    map = {a:0, b:1} → 'b' found at 1, 1 >= left(0)
    left = 1 + 1 = 2
    map['b'] = 2
    maxLen = max(2, 2-2+1) = 2
    left=2, window "b"

  right=3, char='a':
    map = {a:0, b:2} → 'a' found at 0, but 0 < left(2)!
    → Previous 'a' is OUTSIDE window, ignore it
    map['a'] = 3
    maxLen = max(2, 3-2+1) = 2
    left=2, window "ba"

  Answer: 2  ✓
```

### Why Check `charIndex[s[right]] >= left`?

```
  "a b b a"
   0 1 2 3

  At right=3, 'a' was last at index 0.
  But left is already at 2.
  The old 'a' is OUTSIDE our current window!

  Window: [──] at indices 2,3
                0 is outside

  Without the >= left check, we'd set left = 0+1 = 1
  which MOVES LEFT BACKWARDS — WRONG!
```

---

## Approach 4: Using Array Instead of HashMap (ASCII)

For characters in a fixed alphabet (e.g., 128 ASCII), use an integer array.

```
FUNCTION lengthOfLongestSubstring(s, n):
    lastSeen = array of size 128, filled with -1
    left = 0
    maxLen = 0

    FOR right = 0 TO n-1:
        charCode = ASCII(s[right])

        IF lastSeen[charCode] >= left:
            left = lastSeen[charCode] + 1

        lastSeen[charCode] = right
        maxLen = MAX(maxLen, right - left + 1)

    RETURN maxLen
```

```
  Time:  O(n) — single pass
  Space: O(1) — fixed array of 128 (constant)
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force | O(n³) | O(min(n,m)) | Check all substrings |
| Sliding window + set | O(n) | O(min(n,m)) | Amortized - each char added/removed once |
| Sliding window + map | O(n) | O(min(n,m)) | Left jumps directly |
| Sliding window + array | O(n) | O(1) | Fixed 128 or 256 array |

Where m = size of the character set (26 for lowercase, 128 for ASCII).

---

## Common Mistakes

```
  ❌ Not checking if previous index is within window:
     Bad:   IF s[right] IN map: left = map[s[right]] + 1
     Good:  IF s[right] IN map AND map[s[right]] >= left:
                left = map[s[right]] + 1

  ❌ Moving left backwards:
     The 'left' pointer must NEVER move backwards.
     Use: left = MAX(left, map[s[right]] + 1)
     This is equivalent to the >= left check.

  ❌ Forgetting to update the map:
     Always set map[s[right]] = right, even for duplicates.

  ❌ Off-by-one in window length:
     Length of window [left..right] = right - left + 1
```

---

## Pattern Recognition

```
  ┌────────────────────────────────────────────────┐
  │  This problem has the VARIABLE WINDOW pattern: │
  │                                                │
  │  • Find LONGEST substring                      │
  │  • With a PROPERTY (all unique chars)          │
  │  • Window EXPANDS right, CONTRACTS left        │
  │  • State tracked with set/map                  │
  │                                                │
  │  Template:                                     │
  │  Expand → Check violation → Contract → Answer  │
  └────────────────────────────────────────────────┘
```

---

## Related Problems

| Problem | Variation |
|---------|-----------|
| Longest substring with at most k distinct | Replace "all unique" with "≤ k distinct" |
| Longest substring with at most k replacements | Count of non-majority char ≤ k |
| Longest repeating character replacement | Track max frequency in window |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem type | Variable-size sliding window |
| Window state | Set or HashMap of characters |
| Valid condition | No duplicate characters in window |
| Expand | Add `s[right]` to window |
| Contract | Remove `s[left]` or jump `left` |
| Answer updated | After contraction (window is valid) |
| Optimal time | O(n) |
| Optimal space | O(min(n, alphabet_size)) |

---

## Quick Revision Questions

1. **Why is brute force O(n³)?** What are the three nested operations?

2. **In the set-based approach, when do we remove elements?** Why might we remove multiple elements at once?

3. **What is the advantage of the hashmap approach** over the set approach?

4. **Why do we check `map[s[right]] >= left`?** What goes wrong without this check?

5. **Trace through "dvdf"** using the hashmap approach. What is the answer?

6. **What's the space complexity** if the input only contains lowercase English letters?

---

[← Previous: Max Sum Subarray (Fixed)](03-max-sum-subarray-fixed.md) | [Back to README](../README.md) | [Next: Minimum Window Substring →](05-minimum-window-substring.md)
