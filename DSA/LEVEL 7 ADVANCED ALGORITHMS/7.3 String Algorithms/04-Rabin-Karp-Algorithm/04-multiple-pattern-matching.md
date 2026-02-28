# Chapter 4.4 — Multiple Pattern Matching with Rabin-Karp

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Rabin-Karp's true power shines when searching for **multiple patterns**
simultaneously. While KMP handles one pattern at a time, Rabin-Karp can
check all patterns in one pass using a hash set.

---

## 1. The Multi-Pattern Problem

```
  Given: Text T of length n
         Set of k patterns: P₁, P₂, ..., Pₖ  (each of length m)

  Find: All occurrences of any pattern in T

  ┌────────────────────────────────────────────────────┐
  │  Approach 1: Run KMP k times → O(k(n + m))        │
  │  Approach 2: Rabin-Karp with hash set → O(n + km)  │
  │              (average case)                        │
  └────────────────────────────────────────────────────┘
```

---

## 2. Algorithm Idea

```
  Step 1: Compute hash of each pattern → store in a hash set
  Step 2: Slide window over text, compute rolling hash
  Step 3: Check if window hash exists in hash set → O(1) lookup!
  Step 4: On hash match → verify character by character

  ┌───────────────────────────────────────────────┐
  │   Patterns:   "abc"  "xyz"  "pqr"  "mno"    │
  │                ↓       ↓      ↓       ↓       │
  │   Hashes:     28     104     73      56      │
  │                ↓                               │
  │   Hash Set:  {28, 104, 73, 56}               │
  │                                                │
  │   For each window in text:                    │
  │     hash(window) → check if in hash set      │
  │     YES → verify string match                 │
  │     NO  → move to next window                 │
  └───────────────────────────────────────────────┘
```

---

## 3. Step-by-Step Trace

```
  Text:     "xyzabcmnopqr"
  Patterns: {"abc", "pqr", "mno"}
  Pattern length m = 3

  Step 1: Hash patterns
    hash("abc") = 28
    hash("pqr") = 73
    hash("mno") = 56
    Hash Set = {28, 56, 73}

  Step 2: Slide window
  ┌─────┬────────┬──────┬──────────┬─────────────┐
  │  i  │ Window │ Hash │ In Set?  │ Action      │
  ├─────┼────────┼──────┼──────────┼─────────────┤
  │  0  │ xyz    │ 104  │ No       │ Skip        │
  │  1  │ yza    │ 87   │ No       │ Skip        │
  │  2  │ zab    │ 51   │ No       │ Skip        │
  │  3  │ abc    │ 28   │ YES      │ Verify ✅   │
  │  4  │ bcm    │ 41   │ No       │ Skip        │
  │  5  │ cmn    │ 64   │ No       │ Skip        │
  │  6  │ mno    │ 56   │ YES      │ Verify ✅   │
  │  7  │ nop    │ 69   │ No       │ Skip        │
  │  8  │ opq    │ 82   │ No       │ Skip        │
  │  9  │ pqr    │ 73   │ YES      │ Verify ✅   │
  └─────┴────────┴──────┴──────────┴─────────────┘

  Found: "abc" at 3, "mno" at 6, "pqr" at 9
```

---

## 4. Pseudocode

```
function MultiPatternRabinKarp(T, patterns[], d, q):
    n = len(T)
    m = len(patterns[0])         // assume all same length
    k = len(patterns)

    // Step 1: Hash all patterns
    pattern_hashes = new HashSet()
    pattern_map = new HashMap()   // hash → list of patterns
    for each P in patterns:
        hp = polynomial_hash(P, d, q)
        pattern_hashes.add(hp)
        pattern_map[hp].append(P)

    // Step 2: Precompute
    h = d^(m-1) mod q
    ht = polynomial_hash(T[0..m-1], d, q)

    // Step 3: Slide
    for i = 0 to n - m:
        if ht in pattern_hashes:           // O(1) set lookup
            // Step 4: Verify against all patterns with this hash
            for P in pattern_map[ht]:
                if T[i..i+m-1] == P:
                    report match at position i

        if i < n - m:
            ht = (d * (ht - T[i] * h) + T[i+m]) mod q
            ht = (ht + q) mod q            // ensure non-negative
```

---

## 5. Time Complexity Analysis

```
  ┌─────────────────────────────────────────────────┐
  │  Phase              │  Cost                     │
  ├─────────────────────┼───────────────────────────┤
  │  Hash all patterns  │  O(k × m)                │
  │  Build hash set     │  O(k)                    │
  │  Slide window       │  O(n)  (rolling hash)    │
  │  Set lookups        │  O(n)  (O(1) each)       │
  │  Verifications      │  O(m × matches)          │
  ├─────────────────────┼───────────────────────────┤
  │  TOTAL (average)    │  O(n + k × m)            │
  │  TOTAL (worst)      │  O(n × m × k)            │
  └─────────────────────┴───────────────────────────┘

  Compare:
  ┌──────────────────────────────────────────────────┐
  │  Method              │  Time                     │
  ├──────────────────────┼───────────────────────────┤
  │  Naive × k patterns  │  O(k × n × m)            │
  │  KMP × k patterns    │  O(k × (n + m))          │
  │  Rabin-Karp multi    │  O(n + k × m)  average   │
  │  Aho-Corasick        │  O(n + k × m + matches)  │
  └──────────────────────┴───────────────────────────┘

  Rabin-Karp is nearly as fast as Aho-Corasick in practice
  and MUCH simpler to implement!
```

---

## 6. Handling Different-Length Patterns

```
  Problem: Rolling hash only works for fixed window size.

  Solution: Group patterns by length.

  Patterns: {"ab", "xyz", "abc", "pq", "wxyz"}

  Group by length:
  ┌──────────┬────────────────────┐
  │ Length 2  │ {"ab", "pq"}      │
  │ Length 3  │ {"xyz", "abc"}    │
  │ Length 4  │ {"wxyz"}          │
  └──────────┴────────────────────┘

  Run Rabin-Karp once for each distinct length.
  Number of passes = number of distinct lengths.

  Total: O(L × n + k × m_avg)
  where L = number of distinct lengths
```

---

## 7. Implementation — Python

```python
def multi_pattern_rabin_karp(text, patterns, d=31, q=10**9 + 7):
    """Find all occurrences of any pattern in text."""
    if not patterns:
        return []
    
    results = []
    n = len(text)
    
    # Group patterns by length
    from collections import defaultdict
    length_groups = defaultdict(list)
    for p in patterns:
        length_groups[len(p)].append(p)
    
    for m, pats in length_groups.items():
        if m > n:
            continue
        
        # Hash all patterns of this length
        pat_hashes = {}
        for p in pats:
            h = 0
            for c in p:
                h = (h * d + ord(c)) % q
            if h not in pat_hashes:
                pat_hashes[h] = []
            pat_hashes[h].append(p)
        
        # Precompute d^(m-1) mod q
        dm = 1
        for _ in range(m - 1):
            dm = (dm * d) % q
        
        # Hash first window
        ht = 0
        for i in range(m):
            ht = (ht * d + ord(text[i])) % q
        
        # Slide
        for i in range(n - m + 1):
            if ht in pat_hashes:
                window = text[i:i + m]
                for p in pat_hashes[ht]:
                    if window == p:
                        results.append((p, i))
            
            if i < n - m:
                ht = (d * (ht - ord(text[i]) * dm) + ord(text[i + m])) % q
    
    return results

# Example
text = "xyzabcmnopqrabcxyz"
patterns = ["abc", "pqr", "xyz"]
for pattern, pos in multi_pattern_rabin_karp(text, patterns):
    print(f"Found '{pattern}' at position {pos}")
```

---

## 8. Real-World Use Cases

```
  ┌────────────────────────────────────────────────────┐
  │  1. VIRUS SCANNING                                │
  │     Search file content against thousands of       │
  │     known malware signatures.                      │
  │                                                    │
  │  2. SPAM FILTERING                                │
  │     Check email against database of spam phrases.  │
  │                                                    │
  │  3. PLAGIARISM DETECTION                          │
  │     Look for copied phrases across many documents. │
  │                                                    │
  │  4. DNA SEQUENCE MATCHING                         │
  │     Search genome for multiple marker sequences.   │
  │                                                    │
  │  5. KEYWORD FILTERING                             │
  │     Content moderation with a banned-words list.   │
  └────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Aspect | Single Pattern | Multi-Pattern |
|--------|----------------|---------------|
| Storage | One hash value | Hash set of k values |
| Lookup time per window | — | O(1) set lookup |
| Total time (average) | O(n + m) | O(n + km) |
| Advantage over KMP | Simpler | k× faster than k×KMP |
| Different lengths | N/A | Group by length, run separately |
| Implementation complexity | Simple | Moderate |

---

## ❓ Quick Revision Questions

1. **Why is Rabin-Karp preferred over KMP for multi-pattern matching?**
   <details><summary>Answer</summary>With k patterns, KMP takes O(k(n+m)) because each pattern requires a separate pass. Rabin-Karp with a hash set checks all patterns in a single O(n) pass, giving O(n + km) on average.</details>

2. **What data structure stores the pattern hashes for O(1) lookup?**
   <details><summary>Answer</summary>A hash set (or hash map if we need to retrieve which pattern matched). Set membership testing is O(1) average.</details>

3. **How do you handle patterns of different lengths?**
   <details><summary>Answer</summary>Group patterns by length and run Rabin-Karp once per distinct length. Each pass uses a window size equal to that group's length.</details>

4. **What happens if two different patterns have the same hash?**
   <details><summary>Answer</summary>The hash map maps each hash to a list of patterns with that hash. On a hash match, we verify against all patterns in that list. This is correct but slightly slower.</details>

5. **What algorithm is even better than Rabin-Karp for multi-pattern matching?**
   <details><summary>Answer</summary>Aho-Corasick algorithm, which builds a trie+failure-function automaton over all patterns and scans text in O(n + total_pattern_length + matches). It handles different-length patterns naturally.</details>

---

| [⬅️ Previous: Spurious Hits](03-spurious-hits.md) | [Next: Plagiarism Detection ➡️](05-plagiarism-detection.md) |
|:---|---:|
