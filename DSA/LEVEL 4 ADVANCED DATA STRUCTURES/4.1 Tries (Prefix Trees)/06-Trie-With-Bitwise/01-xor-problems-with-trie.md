# XOR Problems with Trie

## Overview
A **bitwise trie** (or binary trie) stores integers bit by bit from the most significant bit (MSB) to the least significant bit (LSB). This structure enables solving XOR problems in **O(B)** time per query (B = number of bits), making it one of the most powerful patterns in competitive programming.

---

## Why XOR + Trie?

```
  XOR Property: To maximize a ⊕ b, at each bit position
  we want opposite bits.
  
  Brute force: Try all pairs → O(N²)
  Trie: For each number, greedily pick opposite bit → O(N × B)
  
  Since B = 30 (or 31 for 32-bit integers), this is effectively O(N).
```

---

## Binary Trie Structure

```
  Store numbers 5 (101), 2 (010), 3 (011) in 3-bit trie:
  
              root
             /    \
           [0]    [1]
           /        \
         [1]        [0]
        /   \         \
      [0]   [1]      [1]
      (2)   (3)      (5)
  
  Each edge represents a bit: left = 0, right = 1
  Traverse from MSB (bit 2) to LSB (bit 0)
```

---

## Core Implementation

```python
class BitTrieNode:
    def __init__(self):
        self.children = [None, None]  # [0-child, 1-child]
        self.count = 0  # Numbers passing through this node

class BitTrie:
    def __init__(self, max_bits=30):
        self.root = BitTrieNode()
        self.max_bits = max_bits  # For non-negative ints up to ~10^9
    
    def insert(self, num):
        node = self.root
        for i in range(self.max_bits, -1, -1):  # MSB to LSB
            bit = (num >> i) & 1
            if not node.children[bit]:
                node.children[bit] = BitTrieNode()
            node = node.children[bit]
            node.count += 1
    
    def remove(self, num):
        node = self.root
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            node = node.children[bit]
            node.count -= 1
    
    def max_xor(self, num):
        """Find maximum XOR of num with any number in trie."""
        node = self.root
        result = 0
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            toggled = 1 - bit  # Opposite bit
            
            if node.children[toggled] and node.children[toggled].count > 0:
                result |= (1 << i)
                node = node.children[toggled]
            elif node.children[bit] and node.children[bit].count > 0:
                node = node.children[bit]
            else:
                break  # Trie is empty for this path
        
        return result
```

### Java Implementation

```java
class BitTrie {
    int[][] children;
    int[] count;
    int idx = 0;
    int maxBits;
    
    BitTrie(int maxNodes, int maxBits) {
        children = new int[maxNodes][2];
        count = new int[maxNodes];
        this.maxBits = maxBits;
        Arrays.fill(children[0], -1);
    }
    
    void insert(int num) {
        int node = 0;
        for (int i = maxBits; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (children[node][bit] == -1) {
                children[node][bit] = ++idx;
                Arrays.fill(children[idx], -1);
            }
            node = children[node][bit];
            count[node]++;
        }
    }
    
    int maxXor(int num) {
        int node = 0, result = 0;
        for (int i = maxBits; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int toggled = 1 - bit;
            if (children[node][toggled] != -1 && count[children[node][toggled]] > 0) {
                result |= (1 << i);
                node = children[node][toggled];
            } else {
                node = children[node][bit];
            }
        }
        return result;
    }
}
```

---

## Greedy Algorithm Trace

```
  Trie has: 3 (011), 10 (1010), 5 (0101)
  Query: max_xor(6)  → 6 = 0110
  
  Using 4 bits (max_bits = 3):
  
  Bit 3: num bit = 0, want 1
    → 1-child exists (leads to 10)? YES → result |= 8, go right
  Bit 2: num bit = 1, want 0
    → 0-child exists? YES (10 = 1010) → result |= 4, go left
  Bit 1: num bit = 1, want 0
    → 0-child exists? YES → result |= 2, go left
  Bit 0: num bit = 0, want 1
    → 1-child exists? YES (10 ends here) → result |= 1, go right
  
  result = 8 + 4 + 2 + 1 = 15
  Verify: 6 ⊕ 10 = 0110 ⊕ 1010 = 1100 ≠ wait...
  
  Let me re-trace with 10 = 1010 (4 bits):
  6 = 0110, 10 = 1010
  6 ⊕ 10 = 1100 = 12
  
  But greedy got 15? Let's check more carefully:
  The trie with {3,10,5}:
    3 = 0011, 10 = 1010, 5 = 0101
  
  Bit 3: 6's bit=0, want 1 → 1-child? YES (10) → go, result=8
  Bit 2: 6's bit=1, want 0 → 0-child from 10's path? 10=1010
         at bit2: 10 has 0 → YES! → result=12
  Bit 1: 6's bit=1, want 0 → 10's bit1=1, so 1-child only → take 1, result stays 12
  Bit 0: 6's bit=0, want 1 → 10's bit0=0, so 0-child only → take 0, result stays 12
  
  result = 12, which is 6 ⊕ 10 = 12 ✓
```

---

## Common XOR Problem Patterns

| Problem | Technique |
|---------|-----------|
| Max XOR of two numbers in array | Insert all, query each |
| Max XOR with constraint | Sort + insert conditionally |
| XOR subarray max | Prefix XOR + trie |
| Count pairs with XOR > K | Trie with count field |
| Persistent XOR queries | Persistent trie (versioned) |

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|:----:|:-----:|
| Insert | O(B) | O(B) per number |
| Delete | O(B) | — |
| Max XOR query | O(B) | — |
| Build trie for N numbers | O(N × B) | O(N × B) |
| All-pairs max XOR | O(N × B) vs O(N²) brute | — |

Where B = number of bits (typically 30 or 31).

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Structure | Binary trie: each node has [0-child, 1-child] |
| Insert | Decompose number MSB→LSB, create path |
| Max XOR | Greedily choose opposite bit at each level |
| Bit count | 30 for values up to 10⁹, 31 for signed 32-bit |
| Key insight | XOR maximized by choosing opposite bits |
| vs Brute Force | O(N×B) vs O(N²) |

---

## Quick Revision Questions

1. **Why do we traverse from MSB to LSB in a binary trie?**
   > Higher-order bits contribute more to the final value. By making greedy decisions from MSB, we maximize the overall XOR value.

2. **What does the `count` field in each node enable?**
   > It supports deletion — decrementing count marks a number as removed without restructuring the trie. Queries skip nodes with count = 0.

3. **How many children does each node in a binary trie have?**
   > Exactly 2: index 0 for bit 0 and index 1 for bit 1.

4. **What is the maximum depth of a binary trie for 32-bit integers?**
   > 31 levels for non-negative integers (bits 30 down to 0), or 32 levels if including a sign bit.

5. **Why is the greedy approach optimal for maximizing XOR?**
   > XOR at each bit position is independent — choosing the opposite bit at position i always adds 2ⁱ to the result. Since we go MSB→LSB, each greedy choice is globally optimal.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Replace Words](../05-Word-Search-Problems/06-replace-words.md) | [Next: Maximum XOR of Two Numbers ▶](02-maximum-xor-of-two-numbers.md) |
