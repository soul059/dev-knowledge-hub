# Maximum XOR of Two Numbers in an Array (LeetCode 421)

## Overview
Given an array of non-negative integers, find the **maximum XOR** of any two numbers in the array. The binary trie approach solves this in **O(N × B)** time, far better than the O(N²) brute-force.

---

## Problem Statement

```
  Input: nums = [3, 10, 5, 25, 2, 8]
  Output: 28
  Explanation: 5 ⊕ 25 = 00101 ⊕ 11001 = 11100 = 28
```

---

## Approach: Binary Trie

```
  1. Insert all numbers into binary trie (MSB → LSB)
  2. For each number, query trie for max XOR
  3. Track global maximum
  
  Key insight: For each number, try to pick the OPPOSITE
  bit at each position to maximize XOR.
```

---

## Step-by-Step Trace

```
  nums = [3, 10, 5, 25, 2, 8]
  Using 5 bits (max value 25 < 32):
  
  3  = 00011
  10 = 01010
  5  = 00101
  25 = 11001
  2  = 00010
  8  = 01000
  
  Binary Trie after inserting all:
  
              root
             /    \
           [0]    [1]
          / \       \
        [0] [1]    [1]
        / \   \      \
      [0] [1] [0]   [0]
      /\   |    |     |
    [1][0] [0] [1]   [0]
    /    \   \   \     \
  [1]   [0] [0] [0]   [1]
  (3)   (2) (8)(10)  (25)
  
  Query max_xor(5 = 00101):
  Bit4: 5's bit=0, want 1 → YES (path to 25) → result=10000
  Bit3: 5's bit=0, want 1 → YES → result=11000
  Bit2: 5's bit=1, want 0 → YES → result=11100
  Bit1: 5's bit=0, want 1 → NO, take 0 → result=11100
  Bit0: 5's bit=1, want 0 → NO, take 1 → result=11100
  
  result = 28 = 5 ⊕ 25 ✓
```

---

## Implementation

### Python

```python
class Solution:
    def findMaximumXOR(self, nums):
        MAX_BITS = 30  # Sufficient for nums[i] <= 10^9
        
        # Build trie
        root = [None, None]  # Compact: [0-child, 1-child]
        
        for num in nums:
            node = root
            for i in range(MAX_BITS, -1, -1):
                bit = (num >> i) & 1
                if node[bit] is None:
                    node[bit] = [None, None]
                node = node[bit]
        
        # Query each number
        max_xor = 0
        for num in nums:
            node = root
            curr_xor = 0
            for i in range(MAX_BITS, -1, -1):
                bit = (num >> i) & 1
                toggled = 1 - bit
                
                if node[toggled] is not None:
                    curr_xor |= (1 << i)
                    node = node[toggled]
                else:
                    node = node[bit]
            
            max_xor = max(max_xor, curr_xor)
        
        return max_xor
```

### Java

```java
class Solution {
    public int findMaximumXOR(int[] nums) {
        int MAX_BITS = 30;
        int[][] trie = new int[nums.length * (MAX_BITS + 1) + 1][2];
        for (int[] row : trie) Arrays.fill(row, -1);
        int idx = 0;
        
        // Build
        for (int num : nums) {
            int node = 0;
            for (int i = MAX_BITS; i >= 0; i--) {
                int bit = (num >> i) & 1;
                if (trie[node][bit] == -1) {
                    trie[node][bit] = ++idx;
                }
                node = trie[node][bit];
            }
        }
        
        // Query
        int maxXor = 0;
        for (int num : nums) {
            int node = 0, currXor = 0;
            for (int i = MAX_BITS; i >= 0; i--) {
                int bit = (num >> i) & 1;
                int toggled = 1 - bit;
                if (trie[node][toggled] != -1) {
                    currXor |= (1 << i);
                    node = trie[node][toggled];
                } else {
                    node = trie[node][bit];
                }
            }
            maxXor = Math.max(maxXor, currXor);
        }
        return maxXor;
    }
}
```

### C++

```cpp
class Solution {
public:
    int findMaximumXOR(vector<int>& nums) {
        const int MAX_BITS = 30;
        vector<array<int,2>> trie(1, {-1, -1});
        
        // Build
        for (int num : nums) {
            int node = 0;
            for (int i = MAX_BITS; i >= 0; i--) {
                int bit = (num >> i) & 1;
                if (trie[node][bit] == -1) {
                    trie[node][bit] = trie.size();
                    trie.push_back({-1, -1});
                }
                node = trie[node][bit];
            }
        }
        
        // Query
        int maxXor = 0;
        for (int num : nums) {
            int node = 0, currXor = 0;
            for (int i = MAX_BITS; i >= 0; i--) {
                int bit = (num >> i) & 1;
                int toggled = 1 - bit;
                if (trie[node][toggled] != -1) {
                    currXor |= (1 << i);
                    node = trie[node][toggled];
                } else {
                    node = trie[node][bit];
                }
            }
            maxXor = max(maxXor, currXor);
        }
        return maxXor;
    }
};
```

---

## Alternative: Bit-by-Bit with HashSet

```python
def findMaximumXOR(self, nums):
    max_xor = 0
    mask = 0
    
    for i in range(30, -1, -1):
        mask |= (1 << i)
        prefixes = set(num & mask for num in nums)
        
        candidate = max_xor | (1 << i)
        
        # Check if candidate is achievable:
        # If a ^ b = candidate, then a ^ candidate = b
        for prefix in prefixes:
            if prefix ^ candidate in prefixes:
                max_xor = candidate
                break
    
    return max_xor
```

```
  This approach works bit by bit from MSB.
  At each level, it tries to set the next bit of the answer to 1.
  
  Time: O(N × B), Space: O(N)
  
  Comparison:
  - HashSet: Simpler code, same complexity
  - Trie: More flexible for extensions (constraints, ranges)
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute Force (all pairs) | O(N²) | O(1) |
| Binary Trie | O(N × B) | O(N × B) |
| HashSet (bit-by-bit) | O(N × B) | O(N) |

Where B = 31 (number of bits), so O(N × B) ≈ O(31N) ≈ O(N).

---

## Edge Cases

```
  1. Single element: [5] → 5 ⊕ 5 = 0 (pair with itself? 
     Problem says "two numbers" — check constraints)
  
  2. All same: [7,7,7] → 7 ⊕ 7 = 0
  
  3. Powers of 2: [1,2,4,8] → max is 8⊕4=12 or 8⊕2=10... 
     Actually 4⊕8 = 1100 = 12 ✓
  
  4. Contains 0: [0,5] → 0 ⊕ 5 = 5
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Find max a ⊕ b for a, b in nums |
| Approach | Binary trie + greedy opposite-bit selection |
| Time | O(N × 31) ≈ O(N) |
| Space | O(N × 31) for trie |
| Key insight | At each bit, choosing opposite maximizes XOR |
| LeetCode | 421 |

---

## Quick Revision Questions

1. **Why do we insert ALL numbers before querying?**
   > We need all potential partners available in the trie. If we query before inserting all, we might miss the optimal pair. (Alternative: insert first, then query = same result.)

2. **Can we query while inserting to avoid a second pass?**
   > Yes. Insert num, then query max_xor(num) considering all previously inserted numbers. Track global max. This works because if (a,b) is the optimal pair, we'll find it when the later-inserted one queries against the earlier one.

3. **What is the compact list-based trie representation `[None, None]`?**
   > Each node is a list of two elements: index 0 = child for bit 0, index 1 = child for bit 1. Avoids class overhead for maximum performance in Python.

4. **How does the HashSet alternative work?**
   > For each bit position (MSB→LSB), check if the answer can have that bit set to 1 using the property: if a⊕b = target, then a⊕target = b. Use prefix masks to work bit by bit.

5. **What value of MAX_BITS should you use?**
   > For non-negative integers up to 10⁹: 30 (since 2³⁰ > 10⁹). For values up to 2³¹-1: 31. For problems with negative numbers or 64-bit: adjust accordingly.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: XOR Problems with Trie](01-xor-problems-with-trie.md) | [Next: Maximum XOR With Element ▶](03-maximum-xor-with-element.md) |
