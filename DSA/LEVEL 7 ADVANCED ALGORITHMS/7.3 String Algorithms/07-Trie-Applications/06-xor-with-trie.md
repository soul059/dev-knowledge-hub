# Chapter 7.6 — Maximum XOR with Trie

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Finding the **maximum XOR** of a number with any element in an array
is elegantly solved using a **binary trie**. This chapter explains the
bit-level trie construction and greedy XOR maximization.

---

## 1. Problem Statement

```
  Given an array of non-negative integers, find two elements
  whose XOR is maximum.

  Example: arr = [3, 10, 5, 25, 2, 8]

  All XOR pairs (selected):
    3 ^ 10  = 9       10 ^ 25 = 19
    3 ^ 25  = 26      5 ^ 25  = 28  ← maximum!
    3 ^ 5   = 6       25 ^ 2  = 27

  Answer: 28

  Brute force: O(n²) — try all pairs
  Binary trie: O(n × L) where L = max bits (e.g., 30 or 32)
```

---

## 2. XOR Property & Greedy Insight

```
  XOR truth table:          Maximize XOR strategy:
  ┌───┬───┬─────┐          ┌─────────────────────────────┐
  │ A │ B │ A^B │          │ To maximize XOR, at each    │
  ├───┼───┼─────┤          │ bit position (MSB to LSB),  │
  │ 0 │ 0 │  0  │          │ choose the OPPOSITE bit.    │
  │ 0 │ 1 │  1  │          │                             │
  │ 1 │ 0 │  1  │          │ If current bit = 1,         │
  │ 1 │ 1 │  0  │          │   try to go to 0 branch.    │
  └───┴───┴─────┘          │ If current bit = 0,         │
                           │   try to go to 1 branch.    │
  XOR = 1 when bits differ │                             │
  → want DIFFERENT bits!   │ Greedy: opposite = larger   │
                           └─────────────────────────────┘
```

---

## 3. Binary Trie Structure

```
  Numbers stored as binary strings (MSB first, fixed width).

  arr = [3, 10, 5, 25]   (using 5 bits)
    3  = 00011
    10 = 01010
    5  = 00101
    25 = 11001

  Binary Trie:
                    (root)
                   /      \
                  0        1
                /   \       \
              0      1       1
             / \      \       \
            0    1     0       0
           / \    \     \       \
          1    0   0     1       0
          |    |   |     |       |
          1    1   1     0       1
          ↓    ↓   ↓     ↓       ↓
         (3)  (5) (5*)  (10)    (25)

  * Note: simplified — actual paths:
    3:  root→0→0→0→1→1
    5:  root→0→0→1→0→1
    10: root→0→1→0→1→0
    25: root→1→1→0→0→1
```

---

## 4. Algorithm: Find Max XOR for a Number

```
function MaxXOR(root, num, L):
    // L = number of bits
    node = root
    xor_val = 0

    for bit = L-1 down to 0:
        current_bit = (num >> bit) & 1
        opposite = 1 - current_bit

        if node.children[opposite] exists:
            // Take opposite path for max XOR
            xor_val = xor_val | (1 << bit)   // this bit contributes 1
            node = node.children[opposite]
        else:
            // Forced to take same path (bit contributes 0)
            node = node.children[current_bit]

    return xor_val
```

---

## 5. Complete Trace

```
  arr = [3, 10, 5, 25]    L = 5 bits

  Step 1: Insert all numbers into binary trie.

  Step 2: For each number, find max XOR using greedy.

  Finding max XOR for num = 5 (00101):

  Bit 4 (MSB): current=0, want opposite=1
    1-child exists? YES (path to 25) → go to 1, XOR |= 10000
  Bit 3: current=0, want opposite=1
    1-child exists? YES → go to 1, XOR |= 01000
  Bit 2: current=1, want opposite=0
    0-child exists? YES → go to 0, XOR |= 00100
  Bit 1: current=0, want opposite=1
    1-child exists? NO → go to 0, XOR |= 00000
  Bit 0: current=1, want opposite=0
    0-child exists? NO → go to 1, XOR |= 00000

  XOR = 10000 + 01000 + 00100 = 11100 = 28

  This matched 5 ^ 25:
    00101
  ^ 11001
  -------
    11100 = 28  ✓

  Overall maximum: 28
```

---

## 6. Implementation

```python
class BinaryTrieNode:
    def __init__(self):
        self.children = [None, None]  # [0-child, 1-child]

class BinaryTrie:
    def __init__(self, max_bits=30):
        self.root = BinaryTrieNode()
        self.L = max_bits  # enough for values up to ~10^9

    def insert(self, num):
        node = self.root
        for i in range(self.L - 1, -1, -1):
            bit = (num >> i) & 1
            if node.children[bit] is None:
                node.children[bit] = BinaryTrieNode()
            node = node.children[bit]

    def max_xor(self, num):
        node = self.root
        result = 0
        for i in range(self.L - 1, -1, -1):
            bit = (num >> i) & 1
            opposite = 1 - bit
            if node.children[opposite] is not None:
                result |= (1 << i)
                node = node.children[opposite]
            else:
                node = node.children[bit]
        return result

def find_max_xor_pair(arr):
    """Find maximum XOR of any two elements in arr."""
    trie = BinaryTrie()
    for num in arr:
        trie.insert(num)

    max_val = 0
    for num in arr:
        max_val = max(max_val, trie.max_xor(num))
    return max_val

# Example:
print(find_max_xor_pair([3, 10, 5, 25, 2, 8]))  # 28
```

---

## 7. Variations

```
  ┌────────────────────────────────────────────────────────┐
  │  Variation 1: Max XOR with value ≤ K                  │
  │  → Add bounds checking during trie traversal          │
  │                                                        │
  │  Variation 2: Max XOR subarray                        │
  │  → Use prefix XOR: xor(l,r) = prefix[r] ^ prefix[l-1]│
  │  → Insert prefix XORs into trie, query each           │
  │                                                        │
  │  Variation 3: Persistent trie for range queries       │
  │  → Supports "max XOR in subarray [l..r]"              │
  │                                                        │
  │  Variation 4: Offline with sorted queries             │
  │  → Sort by constraint, insert elements incrementally  │
  └────────────────────────────────────────────────────────┘
```

---

## 8. Complexity

```
  L = number of bits (30 for values ≤ 10^9, 20 for ≤ 10^6)

  Insert:      O(L)          per number
  Query:       O(L)          per number
  Build trie:  O(n × L)      for n numbers
  Max XOR pair: O(n × L)     insert all + query all

  Space: O(n × L)            worst case (all bits differ)

  vs Brute force O(n²):
  ┌────────────────────────────────────┐
  │  n = 10^5, L = 30:                │
  │  Trie:   10^5 × 30 = 3 × 10^6    │
  │  Brute:  10^10 / 2  = 5 × 10^9   │
  │  Speedup: ~1600×                  │
  └────────────────────────────────────┘
```

---

## 📝 Summary Table

| Aspect | Details |
|--------|---------|
| Data structure | Binary trie (each node has 0/1 children) |
| Key insight | Greedily pick opposite bit at each level |
| Insert | O(L) — traverse MSB to LSB |
| Query | O(L) — follow opposite bit when possible |
| Space | O(n × L) |
| Typical L | 30 (for ≤ 10^9), 20 (for ≤ 10^6) |
| Common problems | Max XOR pair, max XOR subarray, XOR queries |

---

## ❓ Quick Revision Questions

1. **Why do we process bits from MSB to LSB?**
   <details><summary>Answer</summary>Higher-order bits contribute more to the final value. By greedily setting the highest bits to 1 first (via choosing opposite paths), we maximize the XOR result.</details>

2. **Why choose the opposite bit at each level?**
   <details><summary>Answer</summary>XOR produces 1 when bits differ (0^1=1, 1^0=1). To maximize XOR, we want each bit position to produce 1, which requires following the path with the opposite bit value.</details>

3. **What is the space complexity and why?**
   <details><summary>Answer</summary>O(n × L) where n is the number of elements and L is the bit width. Each number creates at most L new nodes, and in the worst case, no paths are shared.</details>

4. **How to find max XOR subarray using a trie?**
   <details><summary>Answer</summary>Compute prefix XOR array. XOR of subarray [l,r] = prefix[r] ^ prefix[l-1]. Insert all prefix XORs into a binary trie, then for each prefix[i], query the trie for maximum XOR.</details>

5. **What happens when the opposite bit path doesn't exist?**
   <details><summary>Answer</summary>We're forced to take the same-bit path, which contributes 0 to the XOR at that position. The result is still the maximum achievable given the numbers in the trie.</details>

---

| [⬅️ Previous: Prefix Counting](05-prefix-counting.md) | [Next: Longest Palindromic Substring ➡️](../08-Manachers-Algorithm/01-longest-palindromic-substring.md) |
|:---|---:|
