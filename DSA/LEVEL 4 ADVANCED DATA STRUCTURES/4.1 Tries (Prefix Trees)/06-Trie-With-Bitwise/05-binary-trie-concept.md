# Binary Trie Concept

## Overview
A **binary trie** is a trie where each node has exactly two children (0 and 1), used to store and query **integers** in their binary representation. This chapter consolidates the concept, implementation patterns, and comparison with standard tries.

---

## Structure

```
  A standard trie has up to 26 children (a-z).
  A binary trie has exactly 2 children (0 and 1).
  
  Each level represents one bit position.
  Depth = number of bits (e.g., 31 for 32-bit integers).
  
  Numbers are stored MSB (most significant bit) first.
```

```
  Numbers: 5 (101), 3 (011), 7 (111)  — using 3 bits
  
              root
             /    \
           [0]    [1]
             \   /   \
            [1] [0]  [1]
            /     \    \
          [1]    [1]   [1]
          (3)    (5)   (7)
  
  Path root→0→1→1 = 011 = 3
  Path root→1→0→1 = 101 = 5
  Path root→1→1→1 = 111 = 7
```

---

## Complete Implementation Template

```python
class BinaryTrie:
    """Full-featured binary trie for integers."""
    
    def __init__(self, max_bits=30):
        self.max_bits = max_bits
        # Using flat arrays for performance
        self.MAXN = 1  # Node count
        self.ch = [[0, 0]]  # Children: ch[node][bit]
        self.cnt = [0]       # Count of numbers passing through
        self.val = [0]       # Count of numbers ending here
    
    def _new_node(self):
        self.ch.append([0, 0])
        self.cnt.append(0)
        self.val.append(0)
        node_id = self.MAXN
        self.MAXN += 1
        return node_id
    
    def insert(self, num):
        node = 0
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            if not self.ch[node][bit]:
                self.ch[node][bit] = self._new_node()
            node = self.ch[node][bit]
            self.cnt[node] += 1
        self.val[node] += 1
    
    def remove(self, num):
        """Remove one occurrence of num. Assumes num exists."""
        node = 0
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            node = self.ch[node][bit]
            self.cnt[node] -= 1
        self.val[node] -= 1
    
    def max_xor(self, num):
        """Find max XOR of num with any number in trie."""
        if self.cnt[self.ch[0][0]] + self.cnt[self.ch[0][1]] == 0:
            return -1  # Trie empty
        
        node = 0
        result = 0
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            toggled = 1 - bit
            
            if self.ch[node][toggled] and self.cnt[self.ch[node][toggled]] > 0:
                result |= (1 << i)
                node = self.ch[node][toggled]
            else:
                node = self.ch[node][bit]
        return result
    
    def min_xor(self, num):
        """Find min XOR of num with any number in trie."""
        node = 0
        result = 0
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            
            # Try to follow SAME bit (XOR = 0, minimize)
            if self.ch[node][bit] and self.cnt[self.ch[node][bit]] > 0:
                node = self.ch[node][bit]
            else:
                result |= (1 << i)
                node = self.ch[node][1 - bit]
        return result
    
    def count_less_than(self, num, limit):
        """Count numbers x in trie where num ⊕ x < limit."""
        node = 0
        count = 0
        
        for i in range(self.max_bits, -1, -1):
            if not node and i < self.max_bits:
                break
            
            num_bit = (num >> i) & 1
            limit_bit = (limit >> i) & 1
            
            if limit_bit == 1:
                # All nums in "same" branch have XOR bit=0 < 1
                same = num_bit
                if self.ch[node][same]:
                    count += self.cnt[self.ch[node][same]]
                # Continue on "diff" branch
                diff = 1 - num_bit
                if not self.ch[node][diff]:
                    break
                node = self.ch[node][diff]
            else:
                # Must have XOR bit=0, follow same branch
                same = num_bit
                if not self.ch[node][same]:
                    break
                node = self.ch[node][same]
        
        return count
```

---

## Binary Trie vs Standard Trie

| Feature | Standard Trie | Binary Trie |
|---------|:---:|:---:|
| Children per node | Up to 26 (or more) | Exactly 2 |
| Key type | Strings | Integers |
| Depth | Key length | Number of bits (fixed) |
| Branching | Character-based | Bit-based (0 or 1) |
| Primary use | String operations | Bitwise operations |
| Space per node | 26 pointers + flags | 2 pointers + count |
| Typical depth | Variable (5-20) | Fixed (30-31) |

---

## Choosing max_bits

```
  Value Range          max_bits
  ─────────────────────────────
  0 to 255             7
  0 to 65535           15
  0 to 10^6            19
  0 to 10^9            29
  0 to 2^31 - 1        30
  0 to 10^18           59
  
  Formula: max_bits = floor(log2(max_value))
  
  In Python: max_value.bit_length() - 1
  In Java/C++: 31 - Integer.numberOfLeadingZeros(maxValue)
```

---

## Memory Layout: Array-Based (Competitive Programming Style)

```cpp
const int MAXN = 3e6 + 5;  // Max nodes
const int BITS = 30;

int ch[MAXN][2], cnt[MAXN];
int tot = 0;

void init() { 
    tot = 0;
    memset(ch[0], 0, sizeof(ch[0]));
    cnt[0] = 0;
}

int newNode() {
    ++tot;
    ch[tot][0] = ch[tot][1] = 0;
    cnt[tot] = 0;
    return tot;
}

void insert(int x) {
    int u = 0;
    for (int i = BITS; i >= 0; i--) {
        int b = (x >> i) & 1;
        if (!ch[u][b]) ch[u][b] = newNode();
        u = ch[u][b];
        cnt[u]++;
    }
}

int queryMax(int x) {
    int u = 0, ans = 0;
    for (int i = BITS; i >= 0; i--) {
        int b = (x >> i) & 1;
        if (ch[u][b ^ 1] && cnt[ch[u][b ^ 1]] > 0) {
            ans |= (1 << i);
            u = ch[u][b ^ 1];
        } else {
            u = ch[u][b];
        }
    }
    return ans;
}
```

```
  This style uses global arrays — fastest possible.
  Estimated nodes: N × (BITS + 1) in worst case.
  For N = 10^5, BITS = 30: ~3 × 10^6 nodes.
```

---

## Applications Summary

```
  1. Maximum XOR of two numbers → LC 421
  2. Maximum XOR with constraint → LC 1707
  3. Maximum subarray XOR → prefix XOR
  4. Count pairs with XOR < K → count_less_than
  5. K-th smallest XOR → guided walk with counts
  6. Minimum XOR pair → min_xor query
  7. XOR basis / linear independence → related concept
```

---

## Common Pitfalls

```
  ❌ Using LSB → MSB order
     → Greedy doesn't work; MSB has highest weight
  
  ❌ Wrong max_bits (too small)
     → Numbers collide; wrong answers
  
  ❌ Forgetting to handle empty trie
     → Null pointer when querying before any insert
  
  ❌ Not using count for deletion
     → Removing requires count decrement, not node deletion
  
  ❌ Mixing signed and unsigned
     → Negative numbers need special handling (sign bit)
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Structure | Binary tree with 0/1 edges per bit |
| Depth | Fixed: number of bits in integer representation |
| Insert/Query | O(B) per operation, B ≈ 30 |
| Space | O(N × B) nodes |
| Core operation | Greedy walk choosing opposite bit |
| Supported ops | Max XOR, Min XOR, Count, K-th, Range |

---

## Quick Revision Questions

1. **Why must we traverse MSB to LSB in a binary trie?**
   > MSB has the highest positional value. Greedy choices at higher bits dominate the result. Going LSB→MSB would not allow correct greedy optimization.

2. **How does the count field enable deletion without removing nodes?**
   > When deleting, we decrement count along the path. During queries, we check `count > 0` to skip logically deleted paths. Nodes remain for other numbers sharing the same prefix.

3. **What's the relationship between max_bits and the value range?**
   > `max_bits = floor(log₂(max_value))`. All numbers are padded with leading zeros to this width, ensuring consistent trie depth.

4. **How does min_xor differ from max_xor in the trie walk?**
   > max_xor follows the opposite bit (to set XOR bit to 1). min_xor follows the same bit (to set XOR bit to 0). Both are greedy from MSB to LSB.

5. **Why are global arrays preferred over class-based nodes in competitive programming?**
   > Global arrays avoid memory allocation overhead, have better cache locality, and are significantly faster — critical when N × B can reach 3×10⁶ nodes.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Bit Manipulation in Trie](04-bit-manipulation-in-trie.md) | [Next: Palindrome Pairs ▶](../07-Advanced-Trie-Problems/01-palindrome-pairs.md) |
