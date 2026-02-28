# Chapter 5.3 — Pattern Matching with Z Algorithm

> **Unit 5: Z Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The Z Algorithm can be used for exact pattern matching by concatenating the
pattern and text with a separator, then using the Z-array to find matches.
This gives a clean O(n + m) solution.

---

## 1. The Key Idea

```
  To find pattern P in text T:

  Step 1: Create S = P + "$" + T
  Step 2: Compute Z-array of S
  Step 3: Any Z[i] == len(P) means a match!

  Why "$"?
  ─────────
  "$" (or any character NOT in P or T) ensures:
  - No Z-value exceeds len(P) due to crossing the boundary
  - Pattern and text don't accidentally merge

  ┌──────────────────────────────────────────────────────┐
  │  S = P $ T                                          │
  │      ↑   ↑                                          │
  │    [0..m-1]  [m+1..m+n]                             │
  │                                                      │
  │  Z[i] == m means S[i..i+m-1] = S[0..m-1] = P       │
  │  Since i > m, this means T[i-m-1..i-m-1+m-1] = P   │
  │  → Pattern found at position (i - m - 1) in T      │
  └──────────────────────────────────────────────────────┘
```

---

## 2. Complete Algorithm

```
function Z_PatternMatch(T, P):
    S = P + "$" + T
    Z = Z_array(S)
    m = len(P)
    matches = []
    
    for i = m + 1 to len(S) - 1:
        if Z[i] == m:
            matches.append(i - m - 1)   // position in T
    
    return matches
```

---

## 3. Step-by-Step Example

```
  Pattern P = "aab"       m = 3
  Text    T = "aabxaab"   n = 7
  
  S = "aab$aabxaab"
       0123456789 10
  
  Z-array computation:
  ┌─────┬────┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │  i  │  0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │
  ├─────┼────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ S[i]│  a │ a │ b │ $ │ a │ a │ b │ x │ a │ a │ b │
  ├─────┼────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ Z[i]│ 11 │ 1 │ 0 │ 0 │ 3 │ 1 │ 0 │ 0 │ 3 │ 1 │ 0 │
  └─────┴────┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                         ↑                   ↑
                       Z[4]=3=m            Z[8]=3=m
                       Match!              Match!

  Positions in T:
    i=4: position = 4 - 3 - 1 = 0  → T[0..2] = "aab" ✅
    i=8: position = 8 - 3 - 1 = 4  → T[4..6] = "aab" ✅

  Found "aab" at positions [0, 4] in T.
```

---

## 4. Visual Trace

```
  T = a a b x a a b
      0 1 2 3 4 5 6

  P = a a b

  Match at 0:    [a a b] x a a b     ✅
  Match at 4:     a a b x [a a b]    ✅

  S = a a b $ a a b x a a b
                [===]       [===]
                Z=3         Z=3
              match!      match!
```

---

## 5. Why the Separator Is Needed

```
  Without separator, P = "aa", T = "aaaa"
  S = "aaaaaa"   (cannot distinguish P from T!)

  Z-array: [6, 5, 4, 3, 2, 1]
  Z[1] = 5 ≠ 2, Z[2] = 4 ≠ 2, etc.
  We'd incorrectly compute Z-values crossing the P|T boundary!

  With separator, P = "aa", T = "aaaa"
  S = "aa$aaaa"
  
  Z-array: [7, 1, 0, 2, 2, 2, 1]
            ↑     ↑  ↑  ↑  ↑
            Z[3]=2 ✓ Z[4]=2 ✓ Z[5]=2 ✓
  
  Correctly finds "aa" at positions 0, 1, 2 in T.
  The "$" caps all Z-values in the pattern portion at m.
```

---

## 6. Full Implementation — Python

```python
def z_array(s):
    """Compute Z-array in O(n)."""
    n = len(s)
    if n == 0:
        return []
    z = [0] * n
    z[0] = n
    l, r = 0, 0
    for i in range(1, n):
        if i < r:
            k = i - l
            if z[k] < r - i:
                z[i] = z[k]
            else:
                z[i] = r - i
                while i + z[i] < n and s[z[i]] == s[i + z[i]]:
                    z[i] += 1
                l, r = i, i + z[i]
        else:
            z[i] = 0
            while i + z[i] < n and s[z[i]] == s[i + z[i]]:
                z[i] += 1
            if z[i] > 0:
                l, r = i, i + z[i]
    return z


def z_pattern_match(text, pattern):
    """Find all occurrences of pattern in text using Z algorithm."""
    m = len(pattern)
    combined = pattern + "$" + text
    z = z_array(combined)
    
    matches = []
    for i in range(m + 1, len(combined)):
        if z[i] == m:
            matches.append(i - m - 1)
    
    return matches


# Example
text = "AABAACAADAABAABA"
pattern = "AABA"
result = z_pattern_match(text, pattern)
print(f"Pattern found at: {result}")
# Output: Pattern found at: [0, 9, 12]
```

---

## 7. Complexity Analysis

```
  ┌────────────────────────────────────────────────────┐
  │  Time:                                             │
  │    Z-array construction: O(n + m)                  │
  │    Scanning for matches: O(n + m)                  │
  │    Total: O(n + m)                                 │
  │                                                    │
  │  Space:                                            │
  │    Combined string: O(n + m)                       │
  │    Z-array: O(n + m)                               │
  │    Total: O(n + m)                                 │
  │                                                    │
  │  Comparison with KMP:                              │
  │    KMP:  O(n + m) time, O(m) space                │
  │    Z:    O(n + m) time, O(n + m) space            │
  │                                                    │
  │    KMP wins on space (doesn't need concatenation)  │
  │    Z is simpler to implement and understand        │
  └────────────────────────────────────────────────────┘
```

---

## 8. Finding All Occurrences (Including Overlapping)

```
  The Z algorithm naturally finds overlapping matches.

  P = "aa", T = "aaaa"
  S = "aa$aaaa"
  Z = [7, 1, 0, 2, 2, 2, 1]

  Z[3] = 2 = m → match at T[0]
  Z[4] = 2 = m → match at T[1]
  Z[5] = 2 = m → match at T[2]

  T:  a a a a
      [a a]
        [a a]
          [a a]

  All overlapping occurrences found! ✅
```

---

## 📝 Summary Table

| Step | Action | Cost |
|------|--------|------|
| Concatenate | S = P + "$" + T | O(n + m) |
| Compute Z-array | Z-algorithm on S | O(n + m) |
| Find matches | Z[i] == m for i > m | O(n) |
| Total | — | O(n + m) |
| Space | S + Z-array | O(n + m) |
| Handles overlapping | Yes, naturally | — |

---

## ❓ Quick Revision Questions

1. **Why do we concatenate P + "$" + T instead of T + "$" + P?**
   <details><summary>Answer</summary>Z-array measures matches with the PREFIX. By putting P first, Z[i] == m means position i matches the entire pattern P, which is the prefix of the concatenated string.</details>

2. **What position in T does a Z-match at index i correspond to?**
   <details><summary>Answer</summary>Position i - m - 1 in T, where m = len(P). We subtract m for the pattern length and 1 for the separator character.</details>

3. **Can Z[i] ever exceed m for i > m?**
   <details><summary>Answer</summary>No. The separator "$" doesn't appear in P or T, so any match starting after the separator will stop at the "$" boundary, capping Z[i] at m.</details>

4. **What is the space disadvantage of Z vs KMP?**
   <details><summary>Answer</summary>Z requires O(n+m) space for the concatenated string and its Z-array, while KMP only needs O(m) for the LPS array (it processes text character-by-character).</details>

5. **How would you count the total number of pattern occurrences?**
   <details><summary>Answer</summary>Count the number of indices i > m where Z[i] == m. This gives the total count including overlapping matches.</details>

---

| [⬅️ Previous: Z-Array Construction](02-z-array-construction.md) | [Next: Comparison with KMP ➡️](04-comparison-with-kmp.md) |
|:---|---:|
