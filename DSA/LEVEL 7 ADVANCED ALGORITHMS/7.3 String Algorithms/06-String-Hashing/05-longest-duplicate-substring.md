# Chapter 6.5 — Longest Duplicate Substring

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The Longest Duplicate Substring problem asks for the longest substring that
appears at least twice. This is elegantly solved using **binary search + hashing**,
demonstrating the power of string hashing techniques.

---

## 1. Problem Statement

```
  Given: String S of length n
  Find:  The longest substring that appears at least twice in S

  Example:
  S = "banana"
  Substrings appearing ≥ 2 times: "a"(3×), "an"(2×), "ana"(2×), "n"(2×), "na"(2×)
  Longest: "ana" (length 3)

  ┌──────────────────────────────────────────────┐
  │  Naive:  Check all pairs of substrings       │
  │          O(n³) or O(n²) with suffix array    │
  │                                               │
  │  Hashing: Binary search + rolling hash       │
  │           O(n log n) average                 │
  └──────────────────────────────────────────────┘
```

---

## 2. Key Insight: Monotonicity

```
  If a duplicate substring of length L exists,
  then a duplicate of length L-1 also exists
  (just take the first L-1 characters of the duplicate).

  ┌─────────────────────────────────────────────────┐
  │  Length:  1  2  3  4  5  6                      │
  │  Has dup: ✓  ✓  ✓  ✗  ✗  ✗                    │
  │                    ↑                             │
  │                 answer = 3                       │
  │                                                  │
  │  This monotonic property enables BINARY SEARCH! │
  └─────────────────────────────────────────────────┘
```

---

## 3. Algorithm

```
  function LongestDuplicate(S):
      lo = 1, hi = n - 1
      answer = ""

      while lo ≤ hi:
          mid = (lo + hi) / 2
          dup = hasDuplicate(S, mid)   // check length mid
          if dup exists:
              answer = dup
              lo = mid + 1             // try longer
          else:
              hi = mid - 1             // try shorter

      return answer

  function hasDuplicate(S, L):
      // Hash all substrings of length L
      // Return a duplicate if hash set has collision
      hashes = new HashMap()
      h = rolling hash of S[0..L-1]
      hashes[h] = 0

      for i = 1 to n - L:
          h = roll(h to S[i..i+L-1])
          if h in hashes:
              // Verify to avoid hash collision
              if S[i..i+L-1] == S[hashes[h]..hashes[h]+L-1]:
                  return S[i..i+L-1]
          hashes[h] = i

      return null
```

---

## 4. Step-by-Step Trace

```
  S = "banana"   n = 6

  Binary search: lo=1, hi=5

  ─── Iteration 1: mid = 3 ───
  Check length 3:
  Substrings: "ban" "ana" "nan" "ana"
                                 ↑
  "ana" appears at positions 1 and 3 → FOUND!
  answer = "ana", lo = 4

  ─── Iteration 2: mid = 4 ───
  Check length 4:
  Substrings: "bana" "anan" "nana"
  All distinct → NOT FOUND
  hi = 3

  ─── lo=4 > hi=3 → STOP ───
  Answer: "ana" (length 3) ✅
```

---

## 5. Implementation — Python

```python
def longest_duplicate_substring(s):
    """Find the longest duplicate substring using binary search + hashing."""
    n = len(s)
    
    def check(length):
        """Check if a duplicate of given length exists. Return start index or -1."""
        if length == 0:
            return -1
        
        p, mod = 31, (1 << 61) - 1   # Mersenne prime for fast modulo
        
        # Compute p^(length-1)
        pm = pow(p, length - 1, mod)
        
        # Hash first window
        h = 0
        for i in range(length):
            h = (h * p + ord(s[i]) - ord('a') + 1) % mod
        
        seen = {h: 0}
        
        # Roll
        for i in range(1, n - length + 1):
            h = (h - (ord(s[i-1]) - ord('a') + 1) * pm) * p % mod
            h = (h + ord(s[i + length - 1]) - ord('a') + 1) % mod
            
            if h in seen:
                # Verify
                j = seen[h]
                if s[j:j + length] == s[i:i + length]:
                    return i
            seen[h] = i
        
        return -1
    
    lo, hi = 1, n - 1
    result = ""
    
    while lo <= hi:
        mid = (lo + hi) // 2
        idx = check(mid)
        if idx != -1:
            result = s[idx:idx + mid]
            lo = mid + 1
        else:
            hi = mid - 1
    
    return result


# Test
print(longest_duplicate_substring("banana"))   # "ana"
print(longest_duplicate_substring("abcabc"))   # "abc"
print(longest_duplicate_substring("abcd"))     # ""
```

---

## 6. Complexity Analysis

```
  Binary search: O(log n) iterations
  Each iteration: O(n) to hash all substrings of that length

  Total: O(n log n) average

  Worst case: O(n² log n) if many hash collisions require verification.
  With double hashing: essentially O(n log n).

  Space: O(n) for the hash set.

  ┌─────────────────────────────────────────────────┐
  │  Comparison:                                    │
  │  Naive:              O(n³) or O(n²)            │
  │  Binary search+hash: O(n log n)                │
  │  Suffix array+LCP:   O(n)                      │
  │  (Suffix array is optimal but more complex)    │
  └─────────────────────────────────────────────────┘
```

---

## 7. Relation to LeetCode 1044

```
  This is LeetCode Problem #1044 — "Longest Duplicate Substring"
  Difficulty: Hard
  
  The binary search + rolling hash approach passes on LeetCode.
  
  Key tips for acceptance:
  1. Use a large prime modulus (2⁶¹ - 1 is good)
  2. Or use double hashing
  3. Handle edge cases: empty result, single character
  4. Don't forget to verify on hash match (optional with good hash)
```

---

## 📝 Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Find longest substring appearing ≥ 2 times |
| Key insight | Monotonicity → binary search on length |
| Per-length check | Rolling hash + hash set → O(n) |
| Total time | O(n log n) average |
| Space | O(n) |
| Alternative | Suffix array + LCP array → O(n) |

---

## ❓ Quick Revision Questions

1. **Why can binary search be applied here?**
   <details><summary>Answer</summary>The "has duplicate" property is monotonic: if a duplicate of length L exists, duplicates of all lengths < L also exist. This monotonicity enables binary search on the answer.</details>

2. **What does each binary search iteration do?**
   <details><summary>Answer</summary>It checks whether any substring of length mid appears at least twice, by computing rolling hashes of all substrings of that length and checking for hash collisions in a hash set.</details>

3. **What is the time complexity of each binary search iteration?**
   <details><summary>Answer</summary>O(n) — rolling hash computation for n-L+1 windows, each in O(1).</details>

4. **Why might verification be needed even with hashing?**
   <details><summary>Answer</summary>Hash collisions (different strings with the same hash) can produce false positives. Verification by character comparison ensures correctness.</details>

5. **What is the optimal solution for this problem?**
   <details><summary>Answer</summary>Suffix array + LCP array in O(n). The longest duplicate substring length equals the maximum value in the LCP array. Hashing gives O(n log n), which is simpler but slightly slower.</details>

---

| [⬅️ Previous: Substring Comparison](04-substring-comparison.md) | [Next: Double Hashing ➡️](06-double-hashing.md) |
|:---|---:|
