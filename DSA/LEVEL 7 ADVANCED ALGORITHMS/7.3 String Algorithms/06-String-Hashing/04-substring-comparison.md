# Chapter 6.4 — O(1) Substring Comparison with Hashing

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

With prefix hashes precomputed, any two substrings can be compared in O(1).
This technique is foundational for binary search on strings, lexicographic
ordering, and many competitive programming problems.

---

## 1. The Problem

```
  Given string S of length n, answer Q queries:
    "Is S[l₁..r₁] equal to S[l₂..r₂]?"

  Naive: O(m) per query where m = substring length
  With hashing: O(1) per query after O(n) preprocessing!
```

---

## 2. Substring Hash Formula

```
  Prefix hashes (little-endian form):
  h[i] = S[0]×p⁰ + S[1]×p¹ + ... + S[i-1]×p^(i-1)

  Hash of S[l..r]:
  raw(l, r) = h[r+1] - h[l]
            = S[l]×p^l + S[l+1]×p^(l+1) + ... + S[r]×p^r

  Notice: raw(l, r) has a factor of p^l.
  
  To compare raw(l₁, r₁) with raw(l₂, r₂):
  Multiply each by the other's power to normalize:
  
  raw(l₁, r₁) × p^l₂  ==  raw(l₂, r₂) × p^l₁  (mod M)

  ┌───────────────────────────────────────────────────┐
  │  No modular inverse needed!                      │
  │  Just cross-multiply by the power of p.          │
  └───────────────────────────────────────────────────┘
```

---

## 3. Visual Example

```
  S = "abcabc"
       012345

  p = 31, M = 10⁹+7
  Values: a=1, b=2, c=3

  h[0] = 0
  h[1] = 1                        (a×31⁰)
  h[2] = 1 + 2×31 = 63            (+ b×31¹)
  h[3] = 63 + 3×961 = 2946        (+ c×31²)
  h[4] = 2946 + 1×29791 = 32737   (+ a×31³)
  h[5] = 32737 + 2×923521 = 1879779  (+ b×31⁴)
  h[6] = 1879779 + 3×28629151 = 87767232  (+ c×31⁵)

  Compare S[0..2] = "abc" with S[3..5] = "abc":

  raw(0, 2) = h[3] - h[0] = 2946
  raw(3, 5) = h[6] - h[3] = 87767232 - 2946 = 87764286

  Check: 2946 × p³  ==  87764286 × p⁰  ?
         2946 × 29791  ==  87764286 × 1  ?
         87764286  ==  87764286  ✓  They're equal!
```

---

## 4. Lexicographic Comparison in O(log n)

```
  To compare S[l₁..r₁] with S[l₂..r₂] LEXICOGRAPHICALLY:

  Step 1: Binary search for the LCP (Longest Common Prefix)
          Find the largest k such that S[l₁..l₁+k-1] = S[l₂..l₂+k-1]
          Each check is O(1) with hashing.
          Binary search: O(log n)

  Step 2: Compare the first differing character:
          S[l₁+k] vs S[l₂+k]

  Total: O(log n) per lexicographic comparison!

  ┌─────────────────────────────────────────────────┐
  │  This is much better than O(n) naive comparison │
  │  and enables O(n log² n) suffix array building  │
  └─────────────────────────────────────────────────┘
```

```python
def lcp_length(sh, l1, l2, max_len):
    """Find LCP length between s[l1..] and s[l2..] using binary search + hash."""
    lo, hi = 0, max_len
    while lo < hi:
        mid = (lo + hi + 1) // 2
        if sh.compare(l1, l1 + mid - 1, l2, l2 + mid - 1):
            lo = mid
        else:
            hi = mid - 1
    return lo

def lexicographic_compare(sh, s, l1, r1, l2, r2):
    """Compare s[l1..r1] with s[l2..r2] lexicographically.
    Returns -1, 0, or 1."""
    len1 = r1 - l1 + 1
    len2 = r2 - l2 + 1
    max_len = min(len1, len2)
    
    lcp = lcp_length(sh, l1, l2, max_len)
    
    if lcp == max_len:
        # One is prefix of the other
        return (len1 > len2) - (len1 < len2)
    else:
        # Compare first differing character
        return (s[l1 + lcp] > s[l2 + lcp]) - (s[l1 + lcp] < s[l2 + lcp])
```

---

## 5. Application: Suffix Array via Hashing

```
  Suffix array = sorted order of all suffixes.

  Sort suffixes using hash-based lexicographic comparison:
  
  Comparator: O(log n) per comparison (binary search + hash)
  Sorting: O(n log n) comparisons
  Total: O(n log² n)

  Not the fastest suffix array construction, but very simple!
```

```python
def suffix_array_hashing(s):
    """Build suffix array using hashing + sorting."""
    sh = StringHash(s)
    n = len(s)
    
    # Sort indices by suffix
    indices = list(range(n))
    indices.sort(key=lambda i: i, 
                 cmp_to_key=lambda i, j: lexicographic_compare(sh, s, i, n-1, j, n-1)))
    
    return indices
```

---

## 6. Application: Number of Distinct Substrings

```
  Count distinct substrings of S.

  Approach:
  1. For each length L from 1 to n:
     - Hash all substrings of length L
     - Count distinct hashes
  2. Sum up counts.

  Time: O(n²) with hashing (each hash is O(1))
  Better than O(n³) naive approach.

  Even better with suffix array: O(n log n) or O(n).

  With double hashing, collision probability is negligible.
```

```python
def count_distinct_substrings(s):
    """Count distinct substrings using hashing."""
    sh = StringHash(s)
    n = len(s)
    total = 0
    
    for length in range(1, n + 1):
        seen = set()
        for i in range(n - length + 1):
            h = sh.get_hash(i, i + length - 1)
            seen.add(h)
        total += len(seen)
    
    return total
```

---

## 7. Application: Longest Common Substring

```
  Given S₁ and S₂, find the longest common substring.

  Binary search on length L:
  - Hash all substrings of length L in S₁ → set H₁
  - Hash all substrings of length L in S₂ → set H₂
  - If H₁ ∩ H₂ ≠ ∅ → common substring of length L exists

  Binary search O(log n) × hashing O(n) = O(n log n)
```

---

## 📝 Summary Table

| Operation | Preprocessing | Query Time |
|-----------|--------------|------------|
| Substring equality | O(n) | O(1) |
| Lexicographic compare | O(n) | O(log n) |
| Suffix array construction | O(n) | O(n log² n) total |
| Count distinct substrings | O(n) | O(n²) total |
| Longest common substring | O(n) | O(n log n) total |

---

## ❓ Quick Revision Questions

1. **How do you compare two substrings in O(1) without modular inverse?**
   <details><summary>Answer</summary>Cross-multiply: raw(l₁,r₁) × p^l₂ == raw(l₂,r₂) × p^l₁ (mod M). This avoids division by normalizing both sides to the same power of p.</details>

2. **How does binary search + hashing give O(log n) lexicographic comparison?**
   <details><summary>Answer</summary>Binary search for the longest common prefix (LCP) — each step checks substring equality in O(1) using hashing. O(log n) steps to find LCP, then compare one character.</details>

3. **What is the time complexity of counting all distinct substrings using hashing?**
   <details><summary>Answer</summary>O(n²): for each of n possible lengths, hash all O(n) substrings of that length in O(1) each, inserting into a hash set.</details>

4. **How does binary search help find the longest common substring?**
   <details><summary>Answer</summary>Binary search on length L. For each L, hash all substrings of length L in both strings and check if hash sets intersect. If yes, try longer; if no, try shorter. O(n log n) total.</details>

5. **Why is hash-based suffix array construction O(n log² n)?**
   <details><summary>Answer</summary>Sorting n suffixes with O(n log n) comparisons, each comparison taking O(log n) for binary-search-based lexicographic comparison using hashing.</details>

---

| [⬅️ Previous: Hash Collisions](03-hash-collisions.md) | [Next: Longest Duplicate Substring ➡️](05-longest-duplicate-substring.md) |
|:---|---:|
