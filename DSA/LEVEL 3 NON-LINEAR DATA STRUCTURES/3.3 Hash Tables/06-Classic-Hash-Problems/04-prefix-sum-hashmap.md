# 6.4 Prefix Sum with HashMap

## Unit 6: Classic Hash Problems

---

## The Pattern

Combine **prefix sums** with a **hash map** to efficiently find subarrays with a target sum. This pattern converts a range-sum problem into a lookup problem.

```
╔══════════════════════════════════════════════════════════════╗
║               CORE IDEA                                     ║
║                                                              ║
║  prefixSum[j] - prefixSum[i] = sum of elements from i to j ║
║                                                              ║
║  If prefixSum[j] - prefixSum[i] = target                   ║
║  Then prefixSum[i] = prefixSum[j] - target                 ║
║                                                              ║
║  → At each position j, look up (currentSum - target)       ║
║    in a hash map of previous prefix sums!                   ║
║                                                              ║
║  arr = [1, 2, 3, -2, 5]                                    ║
║  prefix = [0, 1, 3, 6, 4, 9]                               ║
║           ↑                                                  ║
║           prefix[0]=0 (empty prefix, important!)            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Problem: Subarray Sum Equals K

> Given an array of integers and a target K, find the **number of contiguous subarrays** whose sum equals K.

```
Input:  nums = [1, 1, 1], k = 2
Output: 2   (subarrays [1,1] at indices 0-1 and 1-2)
```

### Algorithm

```
FUNCTION subarraySum(nums, k):
    prefixCount = new HashMap()
    prefixCount.put(0, 1)    // Empty prefix has sum 0
    currentSum = 0
    count = 0

    FOR each num in nums:
        currentSum += num
        target = currentSum - k

        IF prefixCount.containsKey(target):
            count += prefixCount.get(target)

        prefixCount.put(currentSum, 
            prefixCount.getOrDefault(currentSum, 0) + 1)

    RETURN count

Time: O(n)  |  Space: O(n)
```

### Step-by-Step Trace

```
nums = [1, 2, 3, -2, 5], k = 3

  i │ num │ currSum │ target=currSum-k │ map lookup        │ count │ map after
 ───┼─────┼─────────┼──────────────────┼───────────────────┼───────┼──────────────
  - │  -  │    0    │        -         │        -          │   0   │ {0:1}
  0 │  1  │    1    │    1-3 = -2      │ -2 not found      │   0   │ {0:1, 1:1}
  1 │  2  │    3    │    3-3 = 0       │ 0 found! count=1  │   1   │ {0:1, 1:1, 3:1}
  2 │  3  │    6    │    6-3 = 3       │ 3 found! count=1  │   2   │ {0:1, 1:1, 3:1, 6:1}
  3 │ -2  │    4    │    4-3 = 1       │ 1 found! count=1  │   3   │ {0:1, 1:1, 3:1, 6:1, 4:1}
  4 │  5  │    9    │    9-3 = 6       │ 6 found! count=1  │   4   │ {0:1, 1:1, 3:1, 6:1, 4:1, 9:1}

Answer: 4 subarrays sum to 3

Subarrays: [1,2], [3], [3,-2,2→wait...], [2,3,-2]
  Actually: indices 0-1 (sum=3), 2 (sum=3), 1-3 (sum=3), 3-4 (sum=3)
  Verify: [1,2]=3 ✓, [3]=3 ✓, [2,3,-2]=3 ✓, [-2,5]=3 ✓
```

---

## Why Initialize with {0: 1}?

```
╔══════════════════════════════════════════════════════════════╗
║  The entry {0: 1} represents the "empty prefix"            ║
║                                                              ║
║  If currentSum == k at some point:                          ║
║    target = currentSum - k = 0                              ║
║    We need to find 0 in the map!                            ║
║                                                              ║
║  Example: nums = [3], k = 3                                 ║
║    currentSum = 3, target = 0                               ║
║    Without {0:1}: map = {} → 0 not found → returns 0 ✗    ║
║    With {0:1}: map = {0:1} → 0 found! → returns 1 ✓       ║
║                                                              ║
║  The subarray from index 0 to current position              ║
║  has sum = currentSum = k.                                  ║
║  The "previous prefix sum" for this is 0 (before index 0). ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Visual Explanation

```
Array:    [1,  2,  3, -2,  5]
Index:     0   1   2   3   4

Prefix:  0  1  3  6  4  9
         ↑
       "empty"

To find subarray with sum = 3:
Look for pairs where prefix[j] - prefix[i] = 3

  prefix[2] - prefix[0] = 3 - 0 = 3 ✓  → subarray [1,2]
  prefix[3] - prefix[1] = 6 - 3 = 3 ✓  → subarray [3]   (wait: just index 2)
  Actually: prefix[3] - prefix[1] = 6 - 3 = 3 → indices 1..2 → [2,3]
  
  Let me re-index:
  prefix[0]=0, prefix[1]=1, prefix[2]=3, prefix[3]=6, prefix[4]=4, prefix[5]=9

  prefix[2]-prefix[0] = 3-0 = 3 ✓ → nums[0..1] = [1,2]
  prefix[3]-prefix[2] = 6-3 = 3 ✓ → nums[2..2] = [3]
  prefix[4]-prefix[1] = 4-1 = 3 ✓ → nums[1..3] = [2,3,-2]
  prefix[5]-prefix[4] = 9-4 = 5 ✗
  prefix[5]-prefix[3] = 9-6 = 3 ✓ → nums[3..4] = [-2,5]
```

---

## Variation: Longest Subarray with Sum K

```
FUNCTION maxSubarrayLen(nums, k):
    prefixIndex = new HashMap()   // prefix_sum → earliest index
    prefixIndex.put(0, -1)        // sum 0 at virtual index -1
    currentSum = 0
    maxLen = 0

    FOR i = 0 TO n - 1:
        currentSum += nums[i]
        target = currentSum - k

        IF prefixIndex.containsKey(target):
            maxLen = max(maxLen, i - prefixIndex.get(target))

        // Only store FIRST occurrence (for longest subarray)
        IF not prefixIndex.containsKey(currentSum):
            prefixIndex.put(currentSum, i)

    RETURN maxLen
```

---

## Variation: Count Subarrays Divisible by K

```
FUNCTION subarraysDivByK(nums, k):
    // Key insight: if prefix[j] mod k == prefix[i] mod k
    //              then (prefix[j] - prefix[i]) mod k == 0
    
    remainderCount = new HashMap()
    remainderCount.put(0, 1)
    currentSum = 0
    count = 0

    FOR each num in nums:
        currentSum += num
        remainder = ((currentSum mod k) + k) mod k  // Handle negatives
        
        IF remainderCount.containsKey(remainder):
            count += remainderCount.get(remainder)
        
        remainderCount.put(remainder, 
            remainderCount.getOrDefault(remainder, 0) + 1)

    RETURN count
```

---

## Pattern Summary

| Problem | Map Stores | Lookup Key |
|---------|-----------|-----------|
| Subarray sum = K | prefix_sum → count | currentSum - K |
| Longest subarray sum K | prefix_sum → first index | currentSum - K |
| Subarrays divisible by K | remainder → count | currentSum mod K |
| Subarray sum = 0 | prefix_sum → count | currentSum |
| Equal 0s and 1s | prefix_sum → first index | currentSum (replace 0→-1) |

---

## Quick Revision Question

**Q: Why do we need to initialize the prefix sum map with `{0: 1}` before processing the array? Give a concrete example where omitting this initialization produces the wrong answer.**

<details>
<summary>Click to reveal answer</summary>

The entry `{0: 1}` represents the empty subarray prefix — a prefix sum of 0 exists before the first element.

**Concrete example**: `nums = [3, -1, 4]`, `k = 3`

Expected answer: 2 (subarrays `[3]` and `[3, -1, 4]`)

**Without {0: 1}:**
- i=0: currentSum=3, target=3-3=0, lookup 0 → NOT FOUND (empty map), count stays 0
- i=1: currentSum=2, target=2-3=-1, NOT FOUND, count=0
- i=2: currentSum=6, target=6-3=3, lookup 3 → FOUND (count 1), count=1

Returns 1 — **missed the subarray [3]!**

**With {0: 1}:**
- i=0: currentSum=3, target=0, lookup 0 → FOUND (count 1), count=1
- i=2: currentSum=6, target=3, lookup 3 → FOUND (count 1), count=2

Returns 2 — **correct!**

Any subarray starting at index 0 with sum exactly K requires looking up prefix sum 0 in the map.

</details>

---

| [← Previous: Group Anagrams](03-group-anagrams.md) | [Next: Subarray Sum Equals K →](05-subarray-sum-k.md) |
|:---|---:|
