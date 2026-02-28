# Chapter 5: Minimum Window Substring

[← Previous: Longest Substring Without Repeating Characters](04-longest-substring-no-repeat.md) | [Back to README](../README.md) | [Next: Window Expansion and Contraction →](06-window-expansion-contraction.md)

---

## Overview

**Problem:** Given two strings `s` and `t`, find the minimum window in `s` which contains **all characters** of `t` (including duplicates).

This is LeetCode #76, rated **Hard**, and is considered one of the most important sliding window problems. It combines variable-size window with frequency counting.

---

## Problem Understanding

```
  s = "ADOBECODEBANC"
  t = "ABC"

  Output: "BANC"

  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │ A │ D │ O │ B │ E │ C │ O │ D │ E │ B │ A │ N │ C │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5   6   7   8   9  10  11  12

  Valid windows containing A, B, C:
  "ADOBEC"     (0-5)   length 6
  "DOBECODEBA" (1-10)  length 10
  "BECODEBA"   (3-10)  length 8
  "CODEBA"     (5-10)  length 6
  "CODEBANC"   (5-12)  length 8
  "BANC"       (9-12)  length 4  ← MINIMUM!
```

---

## Key Insight: Frequency Matching

```
  t = "ABC"
  Need: {A:1, B:1, C:1}

  t = "AABC"
  Need: {A:2, B:1, C:1}    ← duplicates matter!

  Window is VALID when every character in t appears
  in the window with frequency ≥ required frequency.
```

---

## Algorithm

```
FUNCTION minWindow(s, t):
    IF t is empty: RETURN ""

    // Step 1: Build frequency map of t
    need = frequency map of t        // chars we need
    have = 0                         // chars satisfied
    required = number of UNIQUE chars in t

    // Step 2: Window state
    windowFreq = empty map
    left = 0
    bestLen = INFINITY
    bestLeft = 0

    // Step 3: Expand window
    FOR right = 0 TO len(s)-1:
        char = s[right]
        windowFreq[char] = windowFreq[char] + 1

        // Check if this char's requirement is now met
        IF char IN need AND windowFreq[char] == need[char]:
            have = have + 1

        // Step 4: Contract while valid
        WHILE have == required:
            // Update best
            IF (right - left + 1) < bestLen:
                bestLen = right - left + 1
                bestLeft = left

            // Remove leftmost
            leftChar = s[left]
            windowFreq[leftChar] = windowFreq[leftChar] - 1
            IF leftChar IN need AND windowFreq[leftChar] < need[leftChar]:
                have = have - 1
            left = left + 1

    IF bestLen == INFINITY:
        RETURN ""
    RETURN s[bestLeft .. bestLeft + bestLen - 1]
```

---

## Detailed Trace

```
  s = "ADOBECODEBANC",  t = "ABC"
  need = {A:1, B:1, C:1},  required = 3

  right=0, char='A':
    windowFreq={A:1}, A:1==need 1 → have=1
    have(1) ≠ required(3)

  right=1, char='D':
    windowFreq={A:1,D:1}, D not in need
    have(1) ≠ required(3)

  right=2, char='O':
    windowFreq={A:1,D:1,O:1}
    have(1) ≠ required(3)

  right=3, char='B':
    windowFreq={A:1,D:1,O:1,B:1}, B:1==need 1 → have=2
    have(2) ≠ required(3)

  right=4, char='E':
    windowFreq={A:1,D:1,O:1,B:1,E:1}
    have(2) ≠ required(3)

  right=5, char='C':
    windowFreq={...,C:1}, C:1==need 1 → have=3
    have(3) == required(3)! → VALID WINDOW

    ┌───────────────────────────┐
    │ Window: "ADOBEC" len=6    │
    │ best = 6, bestLeft = 0    │
    └───────────────────────────┘

    Contract: remove 'A', windowFreq[A]=0, 0<1 → have=2
    left=1, have(2) ≠ required(3) → stop contracting

  right=6, char='O':
    windowFreq={...,O:2}
    have(2) ≠ required(3)

  right=7, char='D':
    windowFreq={...,D:2}
    have(2) ≠ required(3)

  right=8, char='E':
    windowFreq={...,E:2}
    have(2) ≠ required(3)

  right=9, char='B':
    windowFreq={...,B:2}
    have(2) ≠ required(3)  (still missing A)

  right=10, char='A':
    windowFreq={...,A:1}, A:1==need 1 → have=3
    have(3) == required(3)! → VALID WINDOW

    ┌────────────────────────────────┐
    │ Window: "DOBECODEBA" len=10    │
    │ best stays 6                   │
    └────────────────────────────────┘

    Contract: remove 'D', D not in need → have still 3
    left=2, window "OBECODEBA" len=9, best stays 6

    Contract: remove 'O', O not in need → have still 3
    left=3, window "BECODEBA" len=8, best stays 6

    Contract: remove 'B', windowFreq[B]=1, 1>=1 → have still 3
    left=4, window "ECODEBA" len=7, best stays 6

    Contract: remove 'E', E not in need → have still 3
    left=5, window "CODEBA" len=6, best stays 6 (tie)

    Contract: remove 'C', windowFreq[C]=0, 0<1 → have=2
    left=6, have(2) ≠ required(3) → stop

  right=11, char='N':
    have(2) ≠ required(3)

  right=12, char='C':
    windowFreq={...,C:1}, C:1==need 1 → have=3
    have(3) == required(3)! → VALID WINDOW

    ┌─────────────────────────────────┐
    │ Window: "ODEBANC" len=7         │
    │ best stays 6                    │
    └─────────────────────────────────┘

    Contract: remove 'O', not in need → have still 3
    left=7, "DEBANC" len=6, best stays 6

    Contract: remove 'D', not in need → have still 3
    left=8, "EBANC" len=5, best=5, bestLeft=8

    Contract: remove 'E', not in need → have still 3
    left=9, "BANC" len=4, best=4, bestLeft=9  ← NEW BEST!

    Contract: remove 'B', windowFreq[B]=0, 0<1 → have=2
    left=10, stop

  Final: bestLen=4, bestLeft=9 → s[9..12] = "BANC"  ✓
```

---

## Visual Summary of Window Movement

```
  A D O B E C O D E B A N C
  [───────────]                  "ADOBEC" len=6  ✓
            [───────────────]    "CODEBA" len=6  ✓
                    [─────────]  "EBANC"  len=5  ✓
                      [───────]  "BANC"   len=4  ✓ BEST
```

---

## The `have` / `required` Optimization

```
  NAIVE CHECK: Iterate all chars in 'need' to verify window is valid
  → O(unique chars in t) per check

  OPTIMIZED: Track 'have' counter
  → O(1) per check

  ┌─────────────────────────────────────────────────┐
  │  'have' increments when a char's window count   │
  │  REACHES its required count.                    │
  │                                                 │
  │  'have' decrements when a char's window count   │
  │  DROPS BELOW its required count.                │
  │                                                 │
  │  Window is valid when have == required.          │
  └─────────────────────────────────────────────────┘

  For t = "AABC":
    need = {A:2, B:1, C:1}
    required = 3  (three unique chars: A, B, C)
    
    have increments to 1 when window has 1 B
    have increments to 2 when window has 1 C
    have increments to 3 when window has 2 A's (not just 1!)
```

---

## Edge Cases

```
  1. t longer than s     → return ""
  2. t == s              → return s
  3. No valid window     → return ""
  4. Multiple same-size  → return first one found
  5. t has duplicates    → must have required frequency
  6. s or t is empty     → return ""
```

---

## Complexity Analysis

| Metric | Value |
|--------|-------|
| Time | O(|s| + |t|) |
| Space | O(|s| + |t|) in worst case |
| Left pointer moves | At most |s| times total |
| Right pointer moves | Exactly |s| times |
| Frequency updates | O(1) per character |

```
  Why O(|s| + |t|)?
  • O(|t|) to build the 'need' map
  • O(|s|) for the sliding window (each char entered/exited once)
  • Total: O(|s| + |t|)
```

---

## Pattern: Find Shortest Subarray/Substring with Property

```
  ┌─────────────────────────────────────────────────────┐
  │  SHORTEST VALID WINDOW TEMPLATE:                    │
  │                                                     │
  │  FOR right = 0 TO n-1:                              │
  │      expand window (add right element)              │
  │                                                     │
  │      WHILE window is VALID:         ← shrink while  │
  │          update best answer         ← valid!        │ 
  │          contract window (remove left element)      │
  │                                                     │
  │  Note: Answer is updated INSIDE the while loop      │
  │  (before contraction), unlike "longest" problems.   │
  └─────────────────────────────────────────────────────┘
```

---

## Related Problems

| Problem | Relationship |
|---------|-------------|
| Find All Anagrams | Fixed window (size = |t|) + frequency match |
| Permutation in String | Same as anagrams, return boolean |
| Smallest Window containing all chars of another string | Same as this problem |
| Substring with Concatenation of All Words | Extension with words instead of chars |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem type | Variable-size sliding window (find shortest) |
| Window state | Frequency map + have/required counters |
| Valid when | All chars of t present with required frequency |
| Expand | Add `s[right]` to window frequency map |
| Contract | While valid — update best, remove `s[left]` |
| Time | O(\|s\| + \|t\|) |
| Space | O(\|s\| + \|t\|) |
| Difficulty | Hard — multiple interacting components |

---

## Quick Revision Questions

1. **What does the `have` variable track?** When does it increment and decrement?

2. **Why do we contract while the window is VALID** (unlike "longest" problems where we contract while INVALID)?

3. **How do we handle duplicate characters in t?** E.g., t = "AABC".

4. **What is the time complexity?** Why is it O(|s| + |t|) and not O(|s|²)?

5. **Trace through s = "aa", t = "aa".** What is the answer?

6. **Why can't we use a simple set** instead of a frequency map for this problem?

---

[← Previous: Longest Substring Without Repeating Characters](04-longest-substring-no-repeat.md) | [Back to README](../README.md) | [Next: Window Expansion and Contraction →](06-window-expansion-contraction.md)
