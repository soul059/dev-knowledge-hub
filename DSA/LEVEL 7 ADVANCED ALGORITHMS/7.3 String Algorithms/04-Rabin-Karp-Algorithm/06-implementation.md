# Chapter 4.6 — Rabin-Karp: Complete Implementation & Worked Example

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

This chapter brings together all the pieces — rolling hash, hash function
design, spurious hit handling — into a complete, well-documented Rabin-Karp
implementation with a detailed step-by-step trace.

---

## 1. Complete Algorithm Summary

```
  ┌─────────────────────────────────────────────────────────┐
  │              RABIN-KARP ALGORITHM                       │
  ├─────────────────────────────────────────────────────────┤
  │  Input:  Text T[0..n-1], Pattern P[0..m-1]             │
  │  Output: All positions where P occurs in T             │
  │                                                         │
  │  1. Choose base d and prime modulus q                   │
  │  2. Compute hp = hash(P)                               │
  │  3. Compute ht = hash(T[0..m-1])                       │
  │  4. Precompute h = d^(m-1) mod q                       │
  │  5. For each position i from 0 to n-m:                 │
  │       a. If ht == hp → verify T[i..i+m-1] == P        │
  │       b. If verified → report match at i               │
  │       c. Roll hash: ht = (d×(ht - T[i]×h) + T[i+m]) % q│
  └─────────────────────────────────────────────────────────┘
```

---

## 2. Full Implementation — Python

```python
def rabin_karp(text, pattern, d=256, q=101):
    """
    Rabin-Karp string matching algorithm.
    
    Args:
        text: The text to search in
        pattern: The pattern to search for
        d: Base for hash function (typically alphabet size)
        q: A prime number for modular hashing
    
    Returns:
        List of starting indices where pattern is found
    """
    n = len(text)
    m = len(pattern)
    matches = []
    
    if m > n:
        return matches
    
    # Step 1: Precompute h = d^(m-1) mod q
    h = 1
    for _ in range(m - 1):
        h = (h * d) % q
    
    # Step 2: Compute initial hashes
    hp = 0  # hash of pattern
    ht = 0  # hash of first window of text
    for i in range(m):
        hp = (d * hp + ord(pattern[i])) % q
        ht = (d * ht + ord(text[i])) % q
    
    # Step 3: Slide the pattern over text
    for i in range(n - m + 1):
        # Check if hashes match
        if ht == hp:
            # Verify character by character (avoid spurious hit)
            if text[i:i + m] == pattern:
                matches.append(i)
        
        # Compute hash for next window
        if i < n - m:
            ht = (d * (ht - ord(text[i]) * h) + ord(text[i + m])) % q
            # Ensure non-negative
            if ht < 0:
                ht += q
    
    return matches
```

---

## 3. Full Implementation — C++

```cpp
#include <vector>
#include <string>
using namespace std;

vector<int> rabinKarp(const string& text, const string& pattern,
                      int d = 256, int q = 101) {
    int n = text.size();
    int m = pattern.size();
    vector<int> matches;
    
    if (m > n) return matches;
    
    // Precompute h = d^(m-1) % q
    long long h = 1;
    for (int i = 0; i < m - 1; i++)
        h = (h * d) % q;
    
    // Compute initial hashes
    long long hp = 0, ht = 0;
    for (int i = 0; i < m; i++) {
        hp = (d * hp + pattern[i]) % q;
        ht = (d * ht + text[i]) % q;
    }
    
    // Slide pattern over text
    for (int i = 0; i <= n - m; i++) {
        if (ht == hp) {
            // Verify
            bool match = true;
            for (int j = 0; j < m; j++) {
                if (text[i + j] != pattern[j]) {
                    match = false;
                    break;
                }
            }
            if (match) matches.push_back(i);
        }
        
        if (i < n - m) {
            ht = (d * (ht - text[i] * h) + text[i + m]) % q;
            if (ht < 0) ht += q;
        }
    }
    
    return matches;
}
```

---

## 4. Step-by-Step Worked Example

```
  Text:    "ABCCDABCDABCD"      n = 13
  Pattern: "ABCD"                m = 4
  d = 256, q = 101

  ═══════════════════════════════════════════

  Step 1: Precompute h = 256³ mod 101
  ─────────────────────────────────────
  256¹ mod 101 = 54
  256² mod 101 = (54 × 256) mod 101 = 13824 mod 101 = 91
  256³ mod 101 = (91 × 256) mod 101 = 23296 mod 101 = 74
  h = 74

  Step 2: Compute initial hashes
  ──────────────────────────────
  Pattern "ABCD": A=65, B=66, C=67, D=68

  hp = 0
  hp = (256×0 + 65) % 101 = 65
  hp = (256×65 + 66) % 101 = 16706 % 101 = 42
  hp = (256×42 + 67) % 101 = 10819 % 101 = 12
  hp = (256×12 + 68) % 101 = 3140 % 101 = 8
  hp = 8

  First window "ABCC": A=65, B=66, C=67, C=67
  ht = 0
  ht = (256×0 + 65) % 101 = 65
  ht = (256×65 + 66) % 101 = 42
  ht = (256×42 + 67) % 101 = 12
  ht = (256×12 + 67) % 101 = 3139 % 101 = 7
  ht = 7

  Step 3: Sliding
  ───────────────

  i=0: Window "ABCC"  ht=7   hp=8   7≠8 → skip
       Roll: remove A(65), add D(68)
       ht = (256×(7 - 65×74) + 68) % 101
       ht = (256×(7 - 4810) + 68) % 101
       ht = (256×(-4803) + 68) % 101
       ht = (-1229568 + 68) % 101
       ht = -1229500 % 101 = ... → (compute) = 46

  i=1: Window "BCCD"  ht=46  hp=8   46≠8 → skip
       Roll: remove B(66), add A(65)
       ... → ht = 55

  i=2: Window "CCDA"  ht=55  hp=8   55≠8 → skip
       Roll: remove C(67), add B(66)
       ... → ht = 31

  i=3: Window "CDAB"  ht=31  hp=8   31≠8 → skip
       Roll: remove C(67), add C(67)
       ... → ht = 98

  i=4: Window "DABC"  ht=98  hp=8   98≠8 → skip
       Roll: remove D(68), add D(68)
       ... → ht = 8

  i=5: Window "ABCD"  ht=8   hp=8   8==8 → VERIFY!
       Compare: A==A, B==B, C==C, D==D → MATCH at position 5 ✅
       Roll: remove A(65), add A(65)
       ... → ht = 8

  i=6: Window "BCDA"  ht=... hp=8   → skip
       ...

  i=7: Window "CDAB"  → skip

  i=8: Window "DABC"  → skip

  i=9: Window "ABCD"  ht=8   hp=8   → VERIFY! → MATCH at position 9 ✅

  Results: Pattern found at positions [5, 9]
```

---

## 5. Visualization of Rolling

```
  Text: A B C C D A B C D A B C D
        ─ ─ ─ ─
  i=0:  [A B C C]           ht=7   ≠ 8
        
          ─ ─ ─ ─
  i=1:    [B C C D]         ht=46  ≠ 8

              ─ ─ ─ ─
  i=2:        [C C D A]     ht=55  ≠ 8

                  ─ ─ ─ ─
  i=3:            [C D A B]   ht=31  ≠ 8

                      ─ ─ ─ ─
  i=4:                [D A B C]  ht=98  ≠ 8

                          ─ ─ ─ ─
  i=5:                    [A B C D]  ht=8 == 8 → ✅ MATCH

  ... continues ...

                                      ─ ─ ─ ─
  i=9:                                [A B C D]  ht=8 == 8 → ✅ MATCH
```

---

## 6. Comparison with Other Algorithms

```
  ┌─────────────┬──────────┬───────────┬──────────────────────────┐
  │ Algorithm   │ Preproc. │ Matching  │ Best Use Case            │
  ├─────────────┼──────────┼───────────┼──────────────────────────┤
  │ Brute Force │ O(0)     │ O(nm)     │ Very short patterns      │
  │ KMP         │ O(m)     │ O(n)      │ Single pattern, worst OK │
  │ Rabin-Karp  │ O(m)     │ O(n) avg  │ Multiple patterns        │
  │ Z Algorithm │ O(n+m)   │ —         │ All occurrences          │
  │ Aho-Corasick│ O(Σm)    │ O(n)      │ Huge pattern set         │
  └─────────────┴──────────┴───────────┴──────────────────────────┘
```

---

## 7. Common Pitfalls

```
  ❌ Pitfall 1: Forgetting to handle negative modulo
     Fix: ht = ((ht % q) + q) % q

  ❌ Pitfall 2: Integer overflow (in C/C++/Java)
     Fix: Use long long and mod at every step

  ❌ Pitfall 3: Using d = 1 (no positional encoding)
     Fix: Use d ≥ 26 or d = 31

  ❌ Pitfall 4: Not verifying on hash match
     Fix: Always compare strings when hashes match

  ❌ Pitfall 5: Computing d^(m-1) inside the loop
     Fix: Precompute once before the loop
```

---

## 8. Practice Problems

| # | Problem | Difficulty | Key Concept |
|---|---------|------------|-------------|
| 1 | Implement Rabin-Karp from scratch | ⭐⭐ | Core algorithm |
| 2 | Count distinct substrings of length k | ⭐⭐ | Rolling hash + set |
| 3 | Find longest repeated substring | ⭐⭐⭐ | Binary search + RK |
| 4 | Check if string is a rotation | ⭐⭐ | Concatenate + search |
| 5 | Multi-pattern search | ⭐⭐⭐ | Hash set approach |

---

## 📝 Summary Table

| Aspect | Rabin-Karp |
|--------|------------|
| Preprocessing | O(m) — hash pattern + compute d^(m-1) |
| Average case | O(n + m) |
| Worst case | O(nm) — many spurious hits |
| Space | O(1) for single pattern |
| Key advantage | Multi-pattern matching in O(n + km) |
| Key disadvantage | Worst case same as brute force |
| Handles overlapping? | Yes |
| Online algorithm? | Yes — processes text left to right |

---

## ❓ Quick Revision Questions

1. **What are the three precomputation steps in Rabin-Karp?**
   <details><summary>Answer</summary>(1) Compute hash of pattern hp, (2) compute hash of first text window ht, (3) precompute h = d^(m-1) mod q for rolling.</details>

2. **Write the rolling hash formula.**
   <details><summary>Answer</summary>ht = (d × (ht - T[i] × h) + T[i + m]) mod q. This removes the leftmost character's contribution and adds the new rightmost character.</details>

3. **When is Rabin-Karp preferred over KMP?**
   <details><summary>Answer</summary>When searching for multiple patterns simultaneously, because all pattern hashes can be stored in a hash set and checked in O(1) per window.</details>

4. **What is the space complexity for multi-pattern Rabin-Karp?**
   <details><summary>Answer</summary>O(k × m) to store hashes and patterns, where k is the number of patterns and m is pattern length.</details>

5. **Can Rabin-Karp find all overlapping occurrences?**
   <details><summary>Answer</summary>Yes. The sliding window moves one position at a time, so overlapping occurrences are naturally detected.</details>

6. **Why do we use modular arithmetic in the hash function?**
   <details><summary>Answer</summary>Without modular arithmetic, hash values grow exponentially with pattern length, causing integer overflow. Modular arithmetic keeps values within a fixed range [0, q-1].</details>

---

| [⬅️ Previous: Plagiarism Detection](05-plagiarism-detection.md) | [Next Unit: Z Algorithm ➡️](../05-Z-Algorithm/01-z-array-concept.md) |
|:---|---:|
