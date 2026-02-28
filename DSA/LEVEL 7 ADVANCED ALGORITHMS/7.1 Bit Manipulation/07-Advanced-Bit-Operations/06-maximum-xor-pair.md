# Chapter 7.6: Maximum XOR Pair

> **Unit 7 — Advanced Bit Operations**
> Find two elements in an array whose XOR is maximum — using a binary trie.

---

## Overview

Given an array, find the maximum value of `arr[i] ^ arr[j]` for any pair. Unlike AND (where agreement is good), XOR is maximized when bits **differ**. A binary trie achieves O(n × 32) = O(n).

---

## Key Insight

```
  ┌──────────────────────────────────────────────────────────┐
  │  To maximize XOR, at each bit from MSB to LSB, we want  │
  │  the two numbers to DISAGREE (one has 0, other has 1).  │
  │                                                          │
  │  Build a trie of all numbers' binary representations.    │
  │  For each number, traverse the trie greedily choosing    │
  │  the OPPOSITE bit at each level.                         │
  └──────────────────────────────────────────────────────────┘
```

---

## Binary Trie Structure

```
  Insert: 5 (101), 2 (010), 7 (111), 3 (011)

  Root
  ├── 0
  │   ├── 0
  │   │   └── 0 → (not used)
  │   └── 1
  │       ├── 0 → 2 (010)
  │       └── 1 → 3 (011)
  └── 1
      ├── 0
      │   └── 1 → 5 (101)
      └── 1
          └── 1 → 7 (111)
```

---

## Algorithm

```
FUNCTION maxXorPair(arr[], n):
    // Build trie
    trie = new TrieNode()
    FOR each num in arr:
        insert(trie, num)
    
    // For each number, find best XOR partner
    maxXor = 0
    FOR each num in arr:
        partner = findMaxXorPartner(trie, num)
        maxXor = MAX(maxXor, num ^ partner)
    RETURN maxXor

FUNCTION insert(trie, num):
    node = trie
    FOR k = 30 DOWNTO 0:
        bit = (num >> k) & 1
        IF node.children[bit] == NULL:
            node.children[bit] = new TrieNode()
        node = node.children[bit]

FUNCTION findMaxXorPartner(trie, num):
    node = trie
    partner = 0
    FOR k = 30 DOWNTO 0:
        bit = (num >> k) & 1
        opposite = 1 - bit
        IF node.children[opposite] != NULL:
            partner = partner | (opposite << k)
            node = node.children[opposite]
        ELSE:
            partner = partner | (bit << k)
            node = node.children[bit]
    RETURN partner
```

---

## Step-by-Step Trace

### Example: arr = [3, 10, 5, 25, 2, 8]

```
  Binary (5 bits): 3=00011, 10=01010, 5=00101, 25=11001, 2=00010, 8=01000

  Build trie with all numbers...

  Query: Find max XOR partner for 25 (11001)

  Bit 4: 25 has 1, want 0 → take 0 branch ✓ (exists: 3,10,5,2,8)
  Bit 3: 25 has 1, want 0 → take 0 branch ✓ (exists: 3,5,2)
  Bit 2: 25 has 0, want 1 → take 1 branch ✓ (exists: 5)
  Bit 1: 25 has 0, want 1 → take 1 branch? No → take 0 branch
  Bit 0: 25 has 1, want 0 → take 0 branch? No → take 1 branch

  Partner = 00101 = 5
  XOR = 25 ^ 5 = 11001 ^ 00101 = 11100 = 28

  Check all pairs:
    25^3=26, 25^10=19, 25^5=28, 25^2=27, 25^8=17
    10^3=9,  10^5=15,  10^2=8,  10^8=2
    
  Maximum = 28 (25 ^ 5) ✓
```

---

## Alternative: Greedy Bit-by-Bit with Sets

```
FUNCTION maxXor(arr[], n):
    maxXor = 0
    mask = 0
    FOR k = 30 DOWNTO 0:
        mask = mask | (1 << k)
        
        // Collect all prefixes up to bit k
        prefixes = SET()
        FOR each num in arr:
            prefixes.ADD(num & mask)
        
        // Try to set bit k in the answer
        candidate = maxXor | (1 << k)
        FOR each p in prefixes:
            IF (candidate ^ p) IN prefixes:
                maxXor = candidate
                BREAK
    
    RETURN maxXor
```

> This uses the property: if `a ^ b = c`, then `a ^ c = b`. So for each prefix `p`, check if `candidate ^ p` exists.

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Trie approach | O(n × 31) = **O(n)** | O(n × 31) for trie |
| Set/greedy approach | O(n × 31) | O(n) for set |
| Brute force | O(n²) | O(1) |

---

## Applications

- **Maximum subarray XOR** — extend with linear basis
- **K-th maximum XOR** — persistent trie
- **XOR in range queries** — offline with trie + sorting
- **Networking** — find most different IP prefix pair

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Objective | Maximize disagreement at each bit position |
| Trie approach | Insert all, query each number for opposite bits |
| Greedy set approach | Try setting each bit, verify with hash set |
| Key property | a^b=c implies a^c=b (check existence) |
| Time | O(n × 31) for both approaches |

---

## Quick Revision Questions

1. **What is the max XOR pair in [1, 2, 3, 4]?**
   <details><summary>Answer</summary>Check: 1^2=3, 1^3=2, 1^4=5, 2^3=1, 2^4=6, 3^4=7. Maximum = 7 (3^4). The trie would greedily pick the most different bits.</details>

2. **Why do we process from MSB to LSB in the trie?**
   <details><summary>Answer</summary>Higher bits contribute more value. Disagreeing at bit 30 (2^30) is worth more than agreeing at all bits 0-29 combined (2^30 - 1).</details>

3. **What if all array elements are the same?**
   <details><summary>Answer</summary>Max XOR = 0. Every pair gives a^a = 0.</details>

4. **How is a binary trie different from a regular trie?**
   <details><summary>Answer</summary>A binary trie has exactly 2 children per node (0 and 1) and represents binary numbers. A regular trie can have up to 26 children (for alphabet) and represents strings.</details>

5. **Can this find the max XOR of any subset (not just pairs)?**
   <details><summary>Answer</summary>No, this only finds max XOR of a pair. Max XOR of any subset is different and requires linear algebra over GF(2) (Gaussian elimination on bit vectors).</details>

---

[← Previous: Maximum AND Pair](05-maximum-and-pair.md) | [Next: Unit 8 — Traveling Salesman →](../08-Bit-Manipulation-In-DP/01-traveling-salesman.md)
