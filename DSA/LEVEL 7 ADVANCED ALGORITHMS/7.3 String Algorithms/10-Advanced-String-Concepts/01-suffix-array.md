# Chapter 10.1 — Suffix Array

> **Unit 10: Advanced String Concepts** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **suffix array** is a sorted array of all suffixes of a string,
represented by their starting indices. It provides many of the same
functionalities as a suffix tree but with much less memory.

---

## 1. What is a Suffix Array?

```
  String S = "banana$"

  All suffixes:
  Index  Suffix
    0    banana$
    1    anana$
    2    nana$
    3    ana$
    4    na$
    5    a$
    6    $

  Sorted suffixes (lexicographic):
  Index  Suffix
    6    $
    5    a$
    3    ana$
    1    anana$
    0    banana$
    4    na$
    2    nana$

  Suffix Array SA = [6, 5, 3, 1, 0, 4, 2]

  ┌──────────────────────────────────────────────────┐
  │  SA[i] = starting index of the i-th smallest     │
  │  suffix in lexicographic order.                   │
  │                                                    │
  │  '$' is a sentinel character smaller than all.    │
  └──────────────────────────────────────────────────┘
```

---

## 2. Naive Construction: O(n² log n)

```python
def suffix_array_naive(s):
    """O(n² log n): sort all suffixes."""
    n = len(s)
    suffixes = [(s[i:], i) for i in range(n)]
    suffixes.sort()
    return [idx for _, idx in suffixes]

# Note: Each comparison takes O(n), and sorting does
# O(n log n) comparisons → O(n² log n) total.
```

---

## 3. O(n log²n) Construction (Prefix Doubling)

```
  Key idea: Sort by first k characters, then 2k, then 4k, ...
  Use RANKS from previous round as comparison keys.

  S = "banana$"  (indices 0-6)

  Round k=1: Sort by first 1 character
    Ranks: b=2, a=1, n=3, a=1, n=3, a=1, $=0
    SA: [6, 1, 3, 5, 0, 2, 4]

  Round k=2: Sort by (rank[i], rank[i+1])
    Pair at 0: (2, 1) → "ba"
    Pair at 1: (1, 3) → "an"
    Pair at 3: (1, 3) → "an"
    Pair at 5: (1, 0) → "a$"
    ...
    New ranks assigned based on sorted pairs.

  Round k=4: Sort by (rank[i], rank[i+2])
    Uses 4-character prefixes.

  Round k=8 ≥ n=7: Done! All suffixes uniquely ranked.

  ┌─────────────────────────────────────────────────┐
  │  Rounds: O(log n)                               │
  │  Each round: O(n log n) for sorting              │
  │  Total: O(n log²n)                              │
  │  Can be reduced to O(n log n) with radix sort.   │
  └─────────────────────────────────────────────────┘
```

---

## 4. Implementation (O(n log²n))

```python
def build_suffix_array(s):
    """O(n log²n) suffix array construction."""
    n = len(s)
    # Initial ranks based on character values
    rank = [ord(c) for c in s]
    sa = list(range(n))
    k = 1
    
    while k < n:
        # Sort by (rank[i], rank[i+k])
        def compare_key(i):
            return (rank[i], rank[i + k] if i + k < n else -1)
        
        sa.sort(key=compare_key)
        
        # Compute new ranks
        new_rank = [0] * n
        new_rank[sa[0]] = 0
        for i in range(1, n):
            new_rank[sa[i]] = new_rank[sa[i-1]]
            if compare_key(sa[i]) != compare_key(sa[i-1]):
                new_rank[sa[i]] += 1
        
        rank = new_rank
        if rank[sa[-1]] == n - 1:
            break  # all ranks unique
        k *= 2
    
    return sa
```

---

## 5. LCP Array (Longest Common Prefix)

```
  LCP[i] = length of longest common prefix between
            SA[i] and SA[i-1] (adjacent suffixes in sorted order)

  S = "banana$"   SA = [6, 5, 3, 1, 0, 4, 2]

  SA[0]=6: $
  SA[1]=5: a$         LCP[1] = 0 ($ vs a$)
  SA[2]=3: ana$       LCP[2] = 1 (a$ vs ana$, share "a")
  SA[3]=1: anana$     LCP[3] = 3 (ana$ vs anana$, share "ana")
  SA[4]=0: banana$    LCP[4] = 0 (anana$ vs banana$)
  SA[5]=4: na$        LCP[5] = 0 (banana$ vs na$)
  SA[6]=2: nana$      LCP[6] = 2 (na$ vs nana$, share "na")

  LCP = [-, 0, 1, 3, 0, 0, 2]

  ┌────────────────────────────────────────────────────┐
  │  Kasai's algorithm computes LCP in O(n)            │
  │  using the observation that LCP values decrease    │
  │  by at most 1 when scanning in text order.         │
  └────────────────────────────────────────────────────┘
```

---

## 6. Kasai's Algorithm for LCP

```python
def build_lcp(s, sa):
    """Kasai's algorithm: O(n) LCP array construction."""
    n = len(s)
    rank = [0] * n
    for i in range(n):
        rank[sa[i]] = i
    
    lcp = [0] * n
    k = 0
    
    for i in range(n):
        if rank[i] == 0:
            k = 0
            continue
        j = sa[rank[i] - 1]  # previous suffix in sorted order
        while i + k < n and j + k < n and s[i+k] == s[j+k]:
            k += 1
        lcp[rank[i]] = k
        if k > 0:
            k -= 1
    
    return lcp
```

---

## 7. Applications

```
  1. Pattern Search: Binary search in SA → O(m log n)
     Find pattern P by binary searching sorted suffixes.

  2. Longest Repeated Substring: max(LCP array)
     The maximum LCP value gives the longest repeated substring.

  3. Number of Distinct Substrings:
     n(n+1)/2 - Σ LCP[i] = count of distinct substrings

  4. Longest Common Substring of 2 strings:
     Concatenate A + "$" + B + "#", build SA + LCP.
     Max LCP where adjacent suffixes come from different strings.

  5. Suffix Array vs Suffix Tree:
  ┌──────────────────┬───────────────┬───────────────┐
  │ Feature          │ Suffix Array  │ Suffix Tree   │
  ├──────────────────┼───────────────┼───────────────┤
  │ Space            │ O(n)          │ O(n) but 20×  │
  │ Build time       │ O(n log n)    │ O(n)          │
  │ Pattern search   │ O(m log n)    │ O(m)          │
  │ Implementation   │ Simpler       │ Complex       │
  │ Cache friendly   │ Yes (array)   │ No (pointers) │
  └──────────────────┴───────────────┴───────────────┘
```

---

## 📝 Summary Table

| Component | Details |
|-----------|---------|
| SA definition | SA[i] = start index of i-th smallest suffix |
| Naive construction | O(n² log n) |
| Prefix doubling | O(n log²n) or O(n log n) with radix sort |
| SA-IS / DC3 | O(n) construction |
| LCP array | Kasai's algorithm O(n) |
| Pattern search | O(m log n) via binary search |
| Space | O(n) integers |

---

## ❓ Quick Revision Questions

1. **What does SA[i] represent?**
   <details><summary>Answer</summary>SA[i] is the starting index in the original string of the i-th lexicographically smallest suffix. For example, in "banana$", SA[0]=6 because "$" is the smallest suffix.</details>

2. **How does prefix doubling work?**
   <details><summary>Answer</summary>In round k, sort suffixes by their first 2^k characters using pairs (rank[i], rank[i+2^(k-1)]) from the previous round. After O(log n) rounds, all suffixes are uniquely ranked.</details>

3. **What is the LCP array and how is it computed?**
   <details><summary>Answer</summary>LCP[i] = length of the longest common prefix between the i-th and (i-1)-th suffixes in sorted order. Kasai's algorithm computes it in O(n) by exploiting that LCP decreases by at most 1 when scanning in text order.</details>

4. **How to count distinct substrings using SA + LCP?**
   <details><summary>Answer</summary>Total substrings = n(n+1)/2. Subtract duplicates counted by LCP: distinct = n(n+1)/2 - Σ LCP[i]. Each LCP[i] represents substrings shared with the previous suffix in sorted order.</details>

5. **When would you prefer suffix array over suffix tree?**
   <details><summary>Answer</summary>When memory is limited (SA uses ~4× less space), when cache performance matters (arrays vs pointers), or when the simpler implementation is preferred. Suffix tree is better when O(m) pattern matching (vs O(m log n)) is critical.</details>

---

| [⬅️ Previous: Interleaving Strings](../09-String-DP/06-interleaving-strings.md) | [Next: Suffix Tree ➡️](02-suffix-tree.md) |
|:---|---:|
