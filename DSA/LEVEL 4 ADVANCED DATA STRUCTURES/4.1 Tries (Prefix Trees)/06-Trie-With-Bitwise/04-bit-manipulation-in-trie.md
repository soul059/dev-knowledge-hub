# Bit Manipulation in Trie

## Overview
This chapter covers how **bit-level operations** integrate with trie structures beyond XOR maximization — including counting pairs, range queries, k-th smallest XOR, and subarray XOR problems.

---

## Pattern 1: Count Pairs with XOR in Range

**Problem**: Count pairs (i, j) where `L ≤ nums[i] ⊕ nums[j] ≤ R`.

```
  Strategy: count(≤ R) - count(≤ L-1)
  
  For count(≤ limit):
    Insert numbers one by one.
    Before inserting nums[i], count how many existing
    numbers j give nums[i] ⊕ nums[j] ≤ limit.
```

### Implementation

```python
class BitTrieWithCount:
    def __init__(self, max_bits=15):
        self.root = {}
        self.max_bits = max_bits
    
    def insert(self, num):
        node = self.root
        for i in range(self.max_bits, -1, -1):
            bit = (num >> i) & 1
            if bit not in node:
                node[bit] = {'count': 0}
            node[bit]['count'] += 1
            node = node[bit]
    
    def count_less_equal(self, num, limit):
        """Count numbers in trie where num ⊕ x ≤ limit."""
        node = self.root
        count = 0
        
        for i in range(self.max_bits, -1, -1):
            if not node:
                break
            
            num_bit = (num >> i) & 1
            limit_bit = (limit >> i) & 1
            
            if limit_bit == 1:
                # XOR bit = 0 path: all numbers on this path give XOR < limit at this bit
                # So ALL of them satisfy ≤ limit → add their count
                same = num_bit  # Following same bit gives XOR bit = 0
                if same in node and 'count' in node[same]:
                    count += node[same]['count']
                
                # Continue with XOR bit = 1 path (might still be ≤ limit)
                diff = 1 - num_bit
                node = node.get(diff, None)
            else:
                # limit_bit = 0: XOR bit MUST be 0, follow same bit
                same = num_bit
                node = node.get(same, None)
        
        # If we reached the end, the remaining number equals limit exactly
        if node and 'count' in node:
            count += node['count']
        
        return count
```

### Trace

```
  nums = [1, 4, 2, 7], count pairs with XOR ≤ 5
  
  Binary (3 bits): 1=001, 4=100, 2=010, 7=111
  
  Process num=1: trie empty → 0 pairs
  Insert 1 (001)
  
  Process num=4 (100): query count_less_equal(4, 5)
    bit2: num=1, limit=1
      → limit_bit=1: count += trie[1].count (same-bit path)
        trie[1] at bit2? 1's bit2=0, not 1. count += 0
      → follow diff path (bit=0): 1's bit2=0, go there
    bit1: num=0, limit=0
      → limit_bit=0: follow same (bit=0) path
        1's bit1=0, go there
    bit0: num=0, limit=1
      → limit_bit=1: count += same-bit path count
        1's bit0=1, same=0... 
  Result: 1⊕4 = 5 ≤ 5 → counted ✓
```

---

## Pattern 2: Maximum Subarray XOR

**Problem**: Find maximum XOR of any subarray.

```
  Key Insight: XOR of subarray [l..r] = prefix[r+1] ⊕ prefix[l]
  
  prefix[i] = nums[0] ⊕ nums[1] ⊕ ... ⊕ nums[i-1]
  prefix[0] = 0
  
  So max subarray XOR = max of prefix[i] ⊕ prefix[j] for all i > j
  This reduces to "Max XOR of two numbers" on prefix array!
```

### Implementation

```python
def max_subarray_xor(nums):
    MAX_BITS = 30
    trie = [None, None]
    
    def insert(num):
        node = trie
        for i in range(MAX_BITS, -1, -1):
            bit = (num >> i) & 1
            if node[bit] is None:
                node[bit] = [None, None]
            node = node[bit]
    
    def max_xor(num):
        node = trie
        result = 0
        for i in range(MAX_BITS, -1, -1):
            bit = (num >> i) & 1
            toggled = 1 - bit
            if node[toggled] is not None:
                result |= (1 << i)
                node = node[toggled]
            else:
                node = node[bit]
        return result
    
    prefix = 0
    insert(0)  # Empty prefix
    max_val = 0
    
    for num in nums:
        prefix ^= num
        max_val = max(max_val, max_xor(prefix))
        insert(prefix)
    
    return max_val
```

```
  Example: nums = [3, 10, 5, 25, 2, 8]
  
  prefix:  0, 3, 9, 12, 21, 23, 31
  
  Best pair: prefix[6]=31 ⊕ prefix[3]=12 = 19
  Or: prefix[6]=31 ⊕ prefix[0]=0 = 31
  
  Maximum subarray XOR = 31 (entire array: 3⊕10⊕5⊕25⊕2⊕8 = 31)
```

---

## Pattern 3: K-th Smallest XOR

**Problem**: Given array and value x, find k-th smallest value of `x ⊕ nums[i]`.

```python
def kth_smallest_xor(nums, x, k):
    MAX_BITS = 30
    
    # Build trie with counts
    root = {'children': [None, None], 'count': 0}
    
    def insert(num):
        node = root
        for i in range(MAX_BITS, -1, -1):
            bit = (num >> i) & 1
            if node['children'][bit] is None:
                node['children'][bit] = {'children': [None, None], 'count': 0}
            node['children'][bit]['count'] += 1
            node = node['children'][bit]
    
    for num in nums:
        insert(num)
    
    # Find k-th smallest XOR with x
    node = root
    result = 0
    
    for i in range(MAX_BITS, -1, -1):
        x_bit = (x >> i) & 1
        
        # To get XOR bit = 0, follow same bit as x
        # To get XOR bit = 1, follow opposite bit
        same = x_bit      # XOR = 0 (smaller)
        diff = 1 - x_bit  # XOR = 1 (larger)
        
        left_count = 0
        if node['children'][same]:
            left_count = node['children'][same]['count']
        
        if k <= left_count:
            # k-th smallest is in the "XOR=0" subtree
            node = node['children'][same]
        else:
            # Skip all "XOR=0" numbers, go to "XOR=1" subtree
            result |= (1 << i)
            k -= left_count
            node = node['children'][diff]
    
    return result
```

```
  Similar to finding k-th element in a binary search tree,
  but the "ordering" is by XOR value with x.
  
  XOR bit = 0 → smaller numbers (left subtree)
  XOR bit = 1 → larger numbers (right subtree)
  
  At each bit, count elements in the "0-XOR" subtree.
  If k ≤ count, go there. Otherwise, k -= count, go to "1-XOR" subtree.
```

---

## Pattern 4: XOR Queries on Subarrays with L, R Constraint

**Problem**: Max XOR of `x` with any element in `nums[L..R]`.

```
  Approach: Persistent Binary Trie
  
  - Insert elements one by one: version[i] = trie after inserting nums[0..i]
  - Query(x, L, R) = max_xor(x) using version[R] minus version[L-1]
  
  Each version shares structure with previous (path copying).
  Count field at each node enables "subtraction" between versions.
```

This is an advanced technique combining persistent data structures with tries.

---

## Bit Operations Reference

```
  Extract bit i:    (num >> i) & 1
  Set bit i:        num | (1 << i)
  Clear bit i:      num & ~(1 << i)
  Toggle bit i:     num ^ (1 << i)
  MSB position:     num.bit_length() - 1
  Count set bits:   bin(num).count('1')
  
  XOR properties:
    a ⊕ a = 0
    a ⊕ 0 = a
    a ⊕ b = b ⊕ a  (commutative)
    (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c)  (associative)
    If a ⊕ b = c, then a ⊕ c = b  (self-inverse)
```

---

## Complexity Summary

| Pattern | Time | Space | Notes |
|---------|:----:|:-----:|-------|
| Count pairs XOR ≤ limit | O(N × B) | O(N × B) | Per number, walk trie |
| Max subarray XOR | O(N × B) | O(N × B) | Prefix XOR + trie |
| K-th smallest XOR | O(N × B) | O(N × B) | Count-guided walk |
| Range XOR queries | O((N+Q) × B) | O(N × B) | Persistent trie |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Prefix XOR | Converts subarray XOR to pair XOR |
| Count field | Enables pair counting and k-th element |
| Greedy walk | Choose XOR=0 or XOR=1 branch |
| Persistent trie | Handles range constraints |
| Key XOR property | a⊕b = c implies a⊕c = b |

---

## Quick Revision Questions

1. **How does prefix XOR reduce max subarray XOR to max pair XOR?**
   > `XOR(l..r) = prefix[r+1] ⊕ prefix[l]`. So maximizing subarray XOR = maximizing XOR of any two prefix values, which is the standard max-XOR problem.

2. **In count pairs with XOR ≤ limit, why do we add the entire subtree count when limit_bit = 1?**
   > When limit's bit is 1, any number following the XOR=0 branch at this position will have a smaller value at this bit, guaranteeing the overall XOR ≤ limit regardless of lower bits. So we count all of them.

3. **How does k-th smallest XOR use the count field?**
   > At each bit, the count of the "same-bit" subtree tells us how many XOR values have 0 at this position. If k exceeds this count, the answer has 1 at this position and we subtract the count from k.

4. **What is the purpose of inserting prefix[0] = 0 in max subarray XOR?**
   > It represents the empty prefix, allowing subarrays starting at index 0 to be considered. Without it, we'd miss subarrays `nums[0..r]`.

5. **When do you need a persistent trie vs. offline approach?**
   > Persistent trie is needed for range-constrained queries (elements in `nums[L..R]`) that can't be sorted. The offline approach (sort by one bound) works when the constraint is a simple threshold (elements ≤ mi).

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Maximum XOR With Element](03-maximum-xor-with-element.md) | [Next: Binary Trie Concept ▶](05-binary-trie-concept.md) |
