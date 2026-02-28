# Maximum XOR With an Element From Array (LeetCode 1707)

## Overview
Given an array of non-negative integers `nums` and queries `[xi, mi]`, for each query find the maximum XOR of `xi` with any element ≤ `mi` from `nums`. This extends the basic max-XOR problem with a **value constraint**, solved using **offline processing + sorted insertion**.

---

## Problem Statement

```
  nums = [0, 1, 2, 3, 4]
  queries = [[3,1], [1,3], [5,6]]
  
  Query [3,1]: elements ≤ 1 are {0,1}. max(3⊕0, 3⊕1) = max(3,2) = 3
  Query [1,3]: elements ≤ 3 are {0,1,2,3}. max(1⊕0,...,1⊕3) = max(1,0,3,2) = 3
  Query [5,6]: elements ≤ 6 are {0,1,2,3,4}. max(5⊕0,...,5⊕4) = max(5,4,7,6,1) = 7
  
  Output: [3, 3, 7]
```

---

## Strategy: Offline Queries

```
  Key Insight: Sort queries by mi (ascending), sort nums.
  Process queries in order of increasing mi.
  For each query, insert all nums ≤ mi into trie, then query.
  
  Steps:
  1. Sort nums ascending
  2. Sort queries by mi, keeping original indices
  3. Pointer j tracks how far we've inserted into trie
  4. For each query (xi, mi):
     - Insert all nums[j] ≤ mi into trie
     - Query max_xor(xi)
     - Store answer at original index
```

---

## Trace

```
  nums = [0, 1, 2, 3, 4] (already sorted)
  queries with indices:
    [3,1,0], [1,3,1], [5,6,2]
  
  Sort queries by mi:
    [3,1,0], [1,3,1], [5,6,2]  (already sorted)
  
  j = 0
  
  Query [3, 1, idx=0]:
    Insert nums while nums[j] ≤ 1:
      Insert 0 (00), Insert 1 (01) → j=2
    Trie: {0, 1}
    max_xor(3 = 11):
      Bit1: want 0, have 0 ✓ → result = 10
      Bit0: want 0, have 0 ✓ → result = 11
    result = 3 (3 ⊕ 0 = 3) → ans[0] = 3
  
  Query [1, 3, idx=1]:
    Insert nums while nums[j] ≤ 3:
      Insert 2 (10), Insert 3 (11) → j=4
    Trie: {0, 1, 2, 3}
    max_xor(1 = 01):
      Bit1: want 1, have 1 (path to 2,3) → result = 10
      Bit0: want 0, have 0 (path to 2) → result = 11
    result = 3 (1 ⊕ 2 = 3) → ans[1] = 3
  
  Query [5, 6, idx=2]:
    Insert nums while nums[j] ≤ 6:
      Insert 4 (100) → j=5
    Trie: {0, 1, 2, 3, 4}
    max_xor(5 = 101):
      Bit2: want 0, have 0 (path to 0,1,2,3) → result = 100
      Bit1: want 1, have 1 (path to 2,3) → result = 110
      Bit0: want 0, have 0 (path to 2) → result = 111
    result = 7 (5 ⊕ 2 = 7) → ans[2] = 7
```

---

## Implementation

### Python

```python
class Solution:
    def maximizeXor(self, nums, queries):
        MAX_BITS = 30
        nums.sort()
        
        # Attach original index to each query
        indexed_queries = sorted(
            [(mi, xi, i) for i, (xi, mi) in enumerate(queries)]
        )
        
        # Binary trie (compact list form)
        trie = [None, None]
        
        def insert(num):
            node = trie
            for bit_pos in range(MAX_BITS, -1, -1):
                bit = (num >> bit_pos) & 1
                if node[bit] is None:
                    node[bit] = [None, None]
                node = node[bit]
        
        def max_xor(num):
            node = trie
            result = 0
            for bit_pos in range(MAX_BITS, -1, -1):
                bit = (num >> bit_pos) & 1
                toggled = 1 - bit
                if node[toggled] is not None:
                    result |= (1 << bit_pos)
                    node = node[toggled]
                elif node[bit] is not None:
                    node = node[bit]
                else:
                    return -1  # Should not happen if trie non-empty
            return result
        
        ans = [0] * len(queries)
        j = 0  # Pointer into sorted nums
        
        for mi, xi, orig_idx in indexed_queries:
            # Insert all numbers ≤ mi
            while j < len(nums) and nums[j] <= mi:
                insert(nums[j])
                j += 1
            
            if j == 0:
                ans[orig_idx] = -1  # No valid element
            else:
                ans[orig_idx] = max_xor(xi)
        
        return ans
```

### Java

```java
class Solution {
    int[][] trie;
    int idx = 0;
    int MAX_BITS = 30;
    
    public int[] maximizeXor(int[] nums, int[][] queries) {
        Arrays.sort(nums);
        int n = queries.length;
        int[][] sortedQueries = new int[n][3];
        for (int i = 0; i < n; i++) {
            sortedQueries[i] = new int[]{queries[i][1], queries[i][0], i};
        }
        Arrays.sort(sortedQueries, (a, b) -> a[0] - b[0]);
        
        trie = new int[nums.length * (MAX_BITS + 1) + 2][2];
        for (int[] row : trie) Arrays.fill(row, -1);
        
        int[] ans = new int[n];
        int j = 0;
        
        for (int[] q : sortedQueries) {
            int mi = q[0], xi = q[1], origIdx = q[2];
            while (j < nums.length && nums[j] <= mi) {
                insert(nums[j++]);
            }
            ans[origIdx] = j == 0 ? -1 : maxXor(xi);
        }
        return ans;
    }
    
    void insert(int num) {
        int node = 0;
        for (int i = MAX_BITS; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (trie[node][bit] == -1) {
                trie[node][bit] = ++idx;
                Arrays.fill(trie[idx], -1);
            }
            node = trie[node][bit];
        }
    }
    
    int maxXor(int num) {
        int node = 0, result = 0;
        for (int i = MAX_BITS; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int toggled = 1 - bit;
            if (trie[node][toggled] != -1) {
                result |= (1 << i);
                node = trie[node][toggled];
            } else {
                node = trie[node][bit];
            }
        }
        return result;
    }
}
```

---

## Online Alternative: Persistent Trie

For online queries (cannot sort queries), use a **persistent trie** — version the trie so query at version `v` only sees elements inserted up to version `v`.

```
  Idea:
  - Sort nums. Insert one by one.
  - version[i] = trie state after inserting first i elements
  - For query (xi, mi), binary search for largest i where nums[i] ≤ mi
  - Query max_xor(xi) on version[i]
  
  Time: O((N + Q) × B)
  Space: O(N × B) for persistent nodes (path copying)
```

This is advanced and rarely needed in interviews — the offline approach is almost always sufficient.

---

## Complexity Analysis

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force per query | O(Q × N) | O(1) |
| Offline + Binary Trie | O((N + Q) × B + N log N + Q log Q) | O(N × B) |
| Online + Persistent Trie | O((N + Q) × B) | O(N × B) |

Sorting: O(N log N + Q log Q), Trie ops: O((N+Q) × 31) ≈ O(N+Q).

---

## Edge Cases

```
  1. No element ≤ mi: answer is -1
  2. mi = 0 and 0 is in nums: answer = xi ⊕ 0 = xi
  3. All elements equal: max XOR = xi ⊕ nums[0]
  4. xi = 0: answer = largest element ≤ mi
  5. Single element in nums: check if ≤ mi, then answer is xi ⊕ nums[0] or -1
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Problem | Max XOR of xi with element ≤ mi |
| Key technique | Offline: sort queries by mi, sort nums |
| Two-pointer | Insert nums[j] while nums[j] ≤ mi |
| Trie query | Standard greedy max XOR |
| Handle no valid element | Return -1 when trie empty |
| LeetCode | 1707 |

---

## Quick Revision Questions

1. **Why do we sort queries by mi?**
   > So we can incrementally insert numbers into the trie. Each query only needs numbers ≤ mi, and by processing in ascending mi order, we never need to remove elements from the trie.

2. **What makes this "offline" processing?**
   > We reorder queries before processing them. We don't answer them in the given order — we sort by mi, process, then map answers back to original indices.

3. **How do we handle the case when no number ≤ mi exists?**
   > Before querying max_xor, check if j == 0 (no elements inserted yet). If so, the answer is -1.

4. **Could we use binary search instead of two-pointer for insertion?**
   > Yes — for each query, binary search for the count of nums ≤ mi. But sorting queries + two-pointer is simpler and avoids redundant insertions.

5. **What's the advantage of persistent trie over offline approach?**
   > Persistent trie supports online queries (answering in original order without sorting). But it's much more complex to implement and rarely needed.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Maximum XOR of Two Numbers](02-maximum-xor-of-two-numbers.md) | [Next: Bit Manipulation in Trie ▶](04-bit-manipulation-in-trie.md) |
