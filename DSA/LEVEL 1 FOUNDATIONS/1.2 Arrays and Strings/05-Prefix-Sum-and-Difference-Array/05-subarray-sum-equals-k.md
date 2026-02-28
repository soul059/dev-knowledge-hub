# Chapter 5: Subarray Sum Equals K

[← Previous: Difference Array](04-difference-array.md) | [Back to README](../README.md) | [Next: Product of Array Except Self →](06-product-except-self.md)

---

## Overview

**Problem (LeetCode 560):** Given an integer array `nums` and an integer `k`, return the total number of subarrays whose sum equals `k`.

This is a classic application of prefix sums combined with a hash map. It handles **negative numbers** (where sliding window fails) and runs in O(n) time.

---

## Problem Understanding

```
  nums = [1, 1, 1], k = 2
  
  All subarrays:
  [1]         sum=1
  [1,1]       sum=2  ✓
  [1]         sum=1
  [1,1]       sum=2  ✓
  [1]         sum=1
  [1,1,1]     sum=3

  Answer: 2
```

---

## Why Sliding Window Fails Here

```
  nums = [1, -1, 1, 1, -1], k = 1

  The array has NEGATIVE numbers → sum is NOT monotonic.
  Adding an element can DECREASE the sum.
  Sliding window's contraction logic breaks.

  ┌──────────────────────────────────────────────────┐
  │  Sliding window needs monotonicity:              │
  │  "If window sum > k, shrinking always helps"     │
  │                                                  │
  │  With negatives, shrinking might INCREASE sum!   │
  │  → Cannot use sliding window                     │
  │  → Use prefix sum + hash map instead             │
  └──────────────────────────────────────────────────┘
```

---

## Approach 1: Brute Force — O(n²)

```
FUNCTION subarraySum(nums, n, k):
    count = 0

    FOR i = 0 TO n-1:
        sum = 0
        FOR j = i TO n-1:
            sum = sum + nums[j]
            IF sum == k:
                count = count + 1

    RETURN count
```

```
  Time: O(n²), Space: O(1)
  Works but too slow for large inputs.
```

---

## Approach 2: Prefix Sum + HashMap — O(n)

### Key Insight

```
  sum(i, j) = prefix[j] - prefix[i-1]

  If sum(i, j) = k, then:
    prefix[j] - prefix[i-1] = k
    prefix[i-1] = prefix[j] - k

  So for each j, we need to count how many previous
  prefix values equal (prefix[j] - k).
```

### Visualization

```
  nums =  [a, b, c, d, e, f, g]
  prefix = 0  p₁ p₂ p₃ p₄ p₅ p₆ p₇
           ↑
           initial (before any element)

  At position j with prefix[j] = P:
  We need prefix[i] = P - k for some i < j
  
  ┌─────────────┬───────────────────┐
  │  prefix[i]  │     subarray      │
  │  = P - k    │     sum = k       │
  └─────┬───────┴───────────┬───────┘
        │                   │
        ▼                   ▼
  [  ...  i  |  i+1  ...  j  ]
  ├─prefix[i]─┤├──── k ────┤
  ├──────── prefix[j] ─────────┤
```

---

## Algorithm

```
FUNCTION subarraySum(nums, n, k):
    prefixCount = HashMap   // prefix_value → number of times seen
    prefixCount[0] = 1      // empty prefix (sum = 0) seen once
    
    currentSum = 0
    count = 0

    FOR i = 0 TO n-1:
        currentSum = currentSum + nums[i]
        
        // How many previous prefix sums equal currentSum - k?
        target = currentSum - k
        IF target IN prefixCount:
            count = count + prefixCount[target]

        // Record current prefix sum
        prefixCount[currentSum] = prefixCount.getOrDefault(currentSum, 0) + 1

    RETURN count
```

---

## Detailed Trace: nums = [1, 2, 3], k = 3

```
  ┌─────┬──────────┬────────┬────────────────┬───────┬──────────────────┐
  │  i  │ nums[i]  │currSum │ target=sum-k   │ count │ prefixCount      │
  ├─────┼──────────┼────────┼────────────────┼───────┼──────────────────┤
  │ init│    -     │   0    │       -        │   0   │ {0:1}            │
  │  0  │    1     │   1    │  1-3 = -2      │   0   │ {0:1, 1:1}      │
  │  1  │    2     │   3    │  3-3 = 0       │   1   │ {0:1, 1:1, 3:1} │
  │  2  │    3     │   6    │  6-3 = 3       │   2   │ {0:1,1:1,3:1,6:1}│
  └─────┴──────────┴────────┴────────────────┴───────┴──────────────────┘

  At i=1: target=0, found in map with count 1
    → subarray from index 0 to 1: [1,2]? No, sum=3? 
    Actually: prefix[2]=3, prefix[0]=0, so subarray(0,1)=3 ✓

  At i=2: target=3, found in map with count 1
    → prefix[3]=6, prefix[1]=3, so subarray(1,2)=[2,3]=5? 
    Wait: target is prefix value 3, which was at index 1.
    subarray from index 2 to 2: sum(2,2)=3 ← this works via prefix[3]-prefix[1]
    BUT prefix indices are shifted. Let me re-verify:
    
    prefix = [0, 1, 3, 6]  (0-indexed, with leading 0)
    At j=2 (prefix[3]=6): target = 6-3 = 3
    prefix[2] = 3 → subarray from index 2 to 2: [3], sum=3 ✓

  Answer: 2  (subarrays [1,2] and [3])  ✓
```

---

## Trace with Negatives: nums = [1, -1, 0], k = 0

```
  ┌─────┬──────────┬────────┬────────┬───────┬────────────────┐
  │  i  │ nums[i]  │currSum │ target │ count │ prefixCount    │
  ├─────┼──────────┼────────┼────────┼───────┼────────────────┤
  │ init│    -     │   0    │   -    │   0   │ {0:1}          │
  │  0  │    1     │   1    │  1     │   0   │ {0:1, 1:1}     │
  │  1  │   -1     │   0    │  0     │   1   │ {0:2, 1:1}     │
  │  2  │    0     │   0    │  0     │   3   │ {0:3, 1:1}     │
  └─────┴──────────┴────────┴────────┴───────┴────────────────┘

  At i=1: target=0, prefixCount[0]=1 → count=1
    subarray [1,-1] sum=0 ✓

  At i=2: target=0, prefixCount[0]=2 → count=1+2=3
    subarrays: [1,-1,0] sum=0, [-1,0] sum=-1? No...
    
    Let me re-check:
    prefix = [0, 1, 0, 0]
    At prefix[3]=0, target=0, prefixCount has two 0's (at positions 0 and 2)
    → subarray(0,2)=[1,-1,0]=0 ✓  and subarray(2,2)=[0]=0 ✓
    Plus the count=1 from earlier: subarray(0,1)=[1,-1]=0 ✓
    
  Total: 3 subarrays with sum 0  ✓
```

---

## Why Initialize prefixCount[0] = 1?

```
  If currentSum == k at some point, then:
    target = currentSum - k = 0
    We need to find prefix sum = 0 in our map.

  The prefix sum "before the array" is 0, meaning
  "the subarray from the start to current index has sum k."

  Without this initialization, we'd miss subarrays
  that start at index 0!

  Example: nums = [3], k = 3
    currentSum = 3, target = 0
    Without {0:1}: count stays 0 ← WRONG!
    With {0:1}: count = 1 ✓
```

---

## Common Variations

### Variation 1: Return All Subarrays (not just count)

```
FUNCTION findSubarrays(nums, n, k):
    prefixMap = HashMap   // prefix_value → list of indices
    prefixMap[0] = [−1]   // before first element
    currentSum = 0
    result = []

    FOR i = 0 TO n-1:
        currentSum = currentSum + nums[i]
        target = currentSum - k
        
        IF target IN prefixMap:
            FOR EACH startIdx IN prefixMap[target]:
                APPEND nums[startIdx+1 .. i] TO result

        IF currentSum NOT IN prefixMap:
            prefixMap[currentSum] = []
        APPEND i TO prefixMap[currentSum]

    RETURN result
```

### Variation 2: Check if Any Subarray Sum = k (Boolean)

Just return true as soon as `count > 0`.

### Variation 3: Longest Subarray with Sum = k

```
FUNCTION longestSubarraySum(nums, n, k):
    firstOccurrence = HashMap   // prefix_value → first index
    firstOccurrence[0] = -1
    currentSum = 0
    maxLen = 0

    FOR i = 0 TO n-1:
        currentSum = currentSum + nums[i]
        target = currentSum - k

        IF target IN firstOccurrence:
            maxLen = MAX(maxLen, i - firstOccurrence[target])

        IF currentSum NOT IN firstOccurrence:
            firstOccurrence[currentSum] = i

    RETURN maxLen
```

Note: Only store the **first** occurrence — for longest subarray, earlier start = longer subarray.

### Variation 4: Shortest Subarray with Sum = k

Store the **latest** occurrence instead.

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force (3 nested loops) | O(n³) | O(1) |
| Brute force (2 loops with running sum) | O(n²) | O(1) |
| Prefix sum + HashMap | O(n) | O(n) |

---

## Common Mistakes

```
  ❌ Forgetting to initialize {0: 1}
     → Misses subarrays starting at index 0

  ❌ Updating map BEFORE checking target
     → Would count the subarray with itself (length 0)
     → Always check THEN update

  ❌ Trying sliding window with negatives
     → Doesn't work! Use prefix sum approach

  ❌ Confusing "count" vs "existence"
     → For count: add prefixCount[target] each time
     → For existence: return true when found
```

---

## Pattern Summary

```
  ┌──────────────────────────────────────────────────┐
  │  PREFIX SUM + HASHMAP PATTERN:                   │
  │                                                  │
  │  1. Initialize map with {0: 1}                   │
  │  2. Running prefix sum                           │
  │  3. target = prefix - k                          │
  │  4. Check map for target                         │
  │  5. Add current prefix to map                    │
  │                                                  │
  │  Works for:                                      │
  │  • Count subarrays with sum = k                  │
  │  • Longest subarray with sum = k                 │
  │  • Check existence of subarray sum = k           │
  │  • Subarrays with sum divisible by k             │
  └──────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Count subarrays with sum = k |
| Key insight | prefix[j] - prefix[i] = k → prefix[i] = prefix[j] - k |
| Data structures | Running sum + HashMap |
| Map stores | prefix_value → count (or index) |
| Initialize | {0: 1} |
| Order | Check target THEN update map |
| Time | O(n) |
| Space | O(n) |
| Handles negatives | Yes |

---

## Quick Revision Questions

1. **Derive the key equation** that makes this algorithm work. Start from sum(i, j) = k.

2. **Why must we check the map BEFORE updating it?** What happens if we update first?

3. **Why does sliding window fail** when the array has negative numbers?

4. **How would you modify the algorithm** to find the LONGEST subarray with sum = k?

5. **Trace through nums = [3, 4, 7, 2, -3, 1, 4, 2], k = 7.** Count all subarrays.

6. **What changes are needed** to count subarrays with sum divisible by k instead of equal to k?

---

[← Previous: Difference Array](04-difference-array.md) | [Back to README](../README.md) | [Next: Product of Array Except Self →](06-product-except-self.md)
