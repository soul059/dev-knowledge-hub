# Chapter 7.5: Maximum AND Pair

> **Unit 7 — Advanced Bit Operations**
> Find two elements in an array whose AND is maximum — using greedy bit-by-bit selection.

---

## Overview

Given an array, find the maximum value of `arr[i] & arr[j]` for any pair (i, j) where i ≠ j. The greedy approach processes bits from the most significant to least significant.

---

## Key Insight

```
  ┌──────────────────────────────────────────────────────────┐
  │  To maximize AND, we want BOTH elements to have 1 at    │
  │  the highest possible bit positions.                     │
  │                                                          │
  │  Strategy: Process from MSB to LSB.                      │
  │  At each bit k: try to keep only elements with bit k    │
  │  set. If at least 2 remain, commit to this filtering.   │
  │  Otherwise, skip this bit.                               │
  └──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION maxAndPair(arr[], n):
    result = 0
    candidates = arr    // all elements are candidates
    
    FOR k = 30 DOWNTO 0:
        // Try keeping only candidates with bit k set
        newCandidates = [x for x in candidates if x & (1 << k)]
        
        IF LENGTH(newCandidates) >= 2:
            result = result | (1 << k)   // this bit will be in the answer
            candidates = newCandidates    // filter down
    
    RETURN result
```

---

## Step-by-Step Trace

### Example: arr = [4, 8, 6, 2]

```
  Binary: 4=0100, 8=1000, 6=0110, 2=0010

  Bit 3 (value 8):
    Candidates with bit 3: [8]
    Only 1 element → skip. Candidates stay [4, 8, 6, 2].

  Bit 2 (value 4):
    Candidates with bit 2: [4, 6]
    2 elements → commit! result = 0100, candidates = [4, 6]

  Bit 1 (value 2):
    Candidates with bit 1: [6]  (from [4, 6])
    Only 1 → skip. Candidates stay [4, 6].

  Bit 0 (value 1):
    Candidates with bit 0: []  (neither 4=100 nor 6=110 has bit 0)
    0 elements → skip.

  Answer: result = 0100 = 4
  Verify: 4 & 6 = 0100 & 0110 = 0100 = 4  ✓
```

### Example: arr = [12, 10, 14, 7]

```
  Binary: 12=1100, 10=1010, 14=1110, 7=0111

  Bit 3: Candidates with bit 3: [12, 10, 14] (3 elements ≥ 2)
    → commit! result = 1000, candidates = [12, 10, 14]

  Bit 2: Candidates with bit 2: [12, 14] (from [12, 10, 14])
    → commit! result = 1100, candidates = [12, 14]

  Bit 1: Candidates with bit 1: [14] (only 1)
    → skip. Candidates stay [12, 14].

  Bit 0: Candidates with bit 0: [] (neither has bit 0)
    → skip.

  Answer: result = 1100 = 12
  Verify: 12 & 14 = 1100 & 1110 = 1100 = 12  ✓
```

---

## Why Greedy Works

```
  AND produces 1 only when BOTH operands have 1.
  
  Higher bits contribute more to the value:
    Bit k contributes 2^k.
    Setting bit 30 (2^30 ≈ 10^9) is worth more than
    all lower bits combined (2^0 + ... + 2^29 = 2^30 - 1).
  
  So we greedily prioritize higher bits.
  Once we commit to bit k being set, we only keep elements
  that support this — ensuring both selected elements have
  all committed bits set.
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Greedy bit-by-bit | O(n × 30) = **O(n)** | O(n) for candidates |
| Brute force all pairs | O(n²) | O(1) |

---

## Variation: Maximum AND of K Elements

```
  Modify: keep candidates with bit k only if ≥ K remain.

  FUNCTION maxAndOfK(arr[], n, K):
      result = 0
      candidates = arr
      FOR k = 30 DOWNTO 0:
          filtered = [x for x in candidates if x & (1 << k)]
          IF LENGTH(filtered) >= K:
              result = result | (1 << k)
              candidates = filtered
      RETURN result
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Strategy | Greedy from MSB to LSB |
| At each bit | Keep elements with that bit set if ≥ 2 remain |
| Why greedy | Higher bits dominate lower bits in value |
| Time | O(n × 30) = O(n) |
| Result | Maximum possible AND of any pair |

---

## Quick Revision Questions

1. **What is the max AND pair in [3, 5, 6]?**
   <details><summary>Answer</summary>3=011, 5=101, 6=110. Bit 2: {5,6} keep. Bit 1: {6} only 1 → skip. Bit 0: {5} only 1 → skip. Result = 100 = 4. Verify: 5&6 = 4 ✓.</details>

2. **Can the max AND pair be 0?**
   <details><summary>Answer</summary>Yes, if no two elements share any bit. E.g., [1, 2, 4] → 1&2=0, 1&4=0, 2&4=0. Max AND = 0.</details>

3. **Why O(n × 30) and not O(n × log(max))?**
   <details><summary>Answer</summary>They're effectively the same. 30 comes from 32-bit integers (bit 30 is the maximum for positive values). For arbitrary precision, it'd be O(n × log(max)).</details>

4. **Does this algorithm identify WHICH pair achieves the maximum AND?**
   <details><summary>Answer</summary>The final candidates list contains all elements that achieve the max AND with each other. Any pair from this list works.</details>

5. **How is this different from Maximum XOR Pair?**
   <details><summary>Answer</summary>Max XOR wants bits to DIFFER (use trie). Max AND wants bits to AGREE (use greedy filtering). Opposite objectives, completely different approaches.</details>

---

[← Previous: Sum of XOR of All Subsets](04-sum-of-xor-of-all-subsets.md) | [Next: Maximum XOR Pair →](06-maximum-xor-pair.md)
