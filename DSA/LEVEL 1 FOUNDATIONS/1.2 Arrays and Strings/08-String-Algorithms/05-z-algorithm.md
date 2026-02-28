# Chapter 5: Z-Algorithm

[← Previous: Rabin-Karp Algorithm](04-rabin-karp.md) | [Back to README](../README.md) | [Next: Algorithm Comparison →](06-algorithm-comparison.md)

---

## Overview

The **Z-Algorithm** builds a **Z-array** in O(n) time, where Z[i] tells us the length of the longest substring starting at position i that matches a prefix of the string. It provides an elegant O(n + m) pattern matching solution and has applications beyond basic search.

---

## Z-Array Definition

```
  For string S of length n:

  Z[i] = length of the longest substring starting at S[i]
         that matches a prefix of S.

  S =  a a b x a a b x a a
  i:   0 1 2 3 4 5 6 7 8 9

  Z[0] = undefined (or n by convention)
  Z[1] = 1  "a" matches prefix "a"        (but "ab"≠"aa")
  Z[2] = 0  "b" ≠ "a"
  Z[3] = 0  "x" ≠ "a"
  Z[4] = 6  "aabxaa" matches prefix "aabxaa"  ✓
  Z[5] = 1  "a" matches prefix "a"
  Z[6] = 0  "b" ≠ "a"
  Z[7] = 0  "x" ≠ "a"
  Z[8] = 2  "aa" matches prefix "aa"
  Z[9] = 1  "a" matches prefix "a"

  Z = [-, 1, 0, 0, 6, 1, 0, 0, 2, 1]
```

### Visual

```
  S = "aabxaabxaa"
       0123456789

  Position 4: S[4..9] = "aabxaa"
              S[0..5] = "aabxaa"  ← prefix
              Match length = 6
              Z[4] = 6

  Position 8: S[8..9] = "aa"
              S[0..1] = "aa"      ← prefix
              Match length = 2
              Z[8] = 2
```

---

## Pattern Matching with Z-Algorithm

```
  Key Trick: concatenate pattern + "$" + text
  
  P = "aab"      (length m = 3)
  T = "aabxaab"  (length n = 7)
  
  S = "aab$aabxaab"
       0123456789 10
  
  Build Z-array for S:
  Z = [-, 1, 0, 0, 3, 1, 0, 0, 3, 1, 0]
                    ↑              ↑
                    Z[4]=3=m       Z[8]=3=m
                    Match at T[0]! Match at T[4]!

  Wherever Z[i] = m (for i > m), pattern occurs at text position i-m-1.
  
  Position 4 in S → position 4-3-1 = 0 in T  ✓
  Position 8 in S → position 8-3-1 = 4 in T  ✓
```

### Why "$"?

```
  The separator "$" (any char NOT in P or T) ensures:
  - No Z-value can extend across the boundary
  - Z[i] ≤ m for all i > m
  - Clean separation between pattern and text
```

---

## Building the Z-Array: O(n) Algorithm

```
  Maintain a "Z-box" [L, R] = the interval with the 
  rightmost endpoint that matches a prefix of S.

  For each i:
  - If i > R: compute Z[i] by naive comparison, update [L,R]
  - If i ≤ R: 
    k = i - L
    If Z[k] < R - i + 1: Z[i] = Z[k]  (fits inside Z-box)
    Else: start from R+1 and extend, update [L,R]
```

### Algorithm

```
FUNCTION buildZArray(s, n):
    Z = array of size n
    Z[0] = n    // by convention
    L = 0, R = 0

    FOR i = 1 TO n-1:
        IF i > R:
            // Outside Z-box: naive scan
            L = i
            R = i
            WHILE R < n AND s[R - L] == s[R]:
                R = R + 1
            Z[i] = R - L
            R = R - 1
        ELSE:
            // Inside Z-box
            k = i - L
            IF Z[k] < R - i + 1:
                Z[i] = Z[k]    // within Z-box, copy
            ELSE:
                // Need to check beyond R
                L = i
                WHILE R < n AND s[R - L] == s[R]:
                    R = R + 1
                Z[i] = R - L
                R = R - 1

    RETURN Z
```

---

## Detailed Trace

```
  S = "aabcaab"
       0123456

  Z[0] = 7 (convention)
  L=0, R=0

  i=1: i > R? 1 > 0 → yes, naive scan
       L=1, R=1
       S[0]='a' = S[1]='a' → R=2
       S[1]='a' ≠ S[2]='b' → stop
       Z[1] = 2-1 = 1, R=1
  
  i=2: i > R? 2 > 1 → yes, naive scan
       L=2, R=2
       S[0]='a' ≠ S[2]='b' → stop
       Z[2] = 0, R=1 (R stays at max of old value)
       Actually R=2-1=1 → R=1

  i=3: i > R? 3 > 1 → yes, naive scan
       L=3, R=3
       S[0]='a' ≠ S[3]='c' → stop
       Z[3] = 0, R=2

  i=4: i > R? 4 > 2 → yes, naive scan
       L=4, R=4
       S[0]='a' = S[4]='a' → R=5
       S[1]='a' = S[5]='a' → R=6
       S[2]='b' = S[6]='b' → R=7
       R=7, but 7 ≥ n=7, stop
       Z[4] = 7-4 = 3, R=6

  i=5: i ≤ R? 5 ≤ 6 → yes
       k = 5 - 4 = 1, Z[k] = Z[1] = 1
       R-i+1 = 6-5+1 = 2
       Z[1]=1 < 2 → Z[5] = Z[1] = 1 (copy, fits!)

  i=6: i ≤ R? 6 ≤ 6 → yes
       k = 6 - 4 = 2, Z[k] = Z[2] = 0
       R-i+1 = 6-6+1 = 1
       Z[2]=0 < 1 → Z[6] = Z[2] = 0

  Z = [7, 1, 0, 0, 3, 1, 0]

  Verify: S[4..6] = "aab" = S[0..2] prefix ✓ Z[4]=3
```

---

## Why O(n)?

```
  Key observation: R only increases!

  - When i > R: naive scan increases R
  - When i ≤ R and we copy: O(1), R unchanged
  - When i ≤ R and we extend: starts from R+1, increases R

  R goes from 0 to at most n.
  Each character is "touched" by naive scan at most once.
  
  Total work: O(n)
```

---

## Applications

```
  1. Pattern matching:       P + "$" + T, find Z[i]=m
  2. Number of distinct substrings: O(n²) with Z
  3. String compression:    if Z[i]=n-i and n%i==0, 
                            string has period i
  4. Longest repeated substring
  5. All periods of a string
```

### String Period

```
  S = "abcabcabc"  (length 9)
  Z = [9, 0, 0, 6, 0, 0, 3, 0, 0]
  
  Check i=3: Z[3]=6, and 6=n-i=9-3=6, and 9%3=0
  → "abc" is a period of length 3!

  S = "abc" repeated 3 times ✓
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Z-array build | O(n) |
| Pattern matching | O(n + m) |
| Space | O(n + m) for concatenated string + Z-array |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Z[i] | Length of longest substring at i matching prefix |
| Pattern match | Concatenate P + "$" + T, find Z[i] = m |
| Build time | O(n) using Z-box optimization |
| Key idea | Reuse previously computed matches via Z-box |
| Applications | Pattern search, periods, distinct substrings |

---

## Quick Revision Questions

1. **Define Z[i].** Compute the Z-array for "aabxaab".

2. **How does the Z-Algorithm solve pattern matching?** What is the concatenation trick?

3. **What is the Z-box [L, R]?** Why does it ensure O(n) total time?

4. **How do you detect if a string has a period using the Z-array?**

5. **Trace the Z-array construction** for "aaaa".

6. **Compare the Z-Algorithm with KMP.** What information does each precompute?

---

[← Previous: Rabin-Karp Algorithm](04-rabin-karp.md) | [Back to README](../README.md) | [Next: Algorithm Comparison →](06-algorithm-comparison.md)
