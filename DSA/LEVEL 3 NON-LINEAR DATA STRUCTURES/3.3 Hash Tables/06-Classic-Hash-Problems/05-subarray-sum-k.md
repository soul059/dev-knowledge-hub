# 6.5 Subarray Sum Equals K

## Unit 6: Classic Hash Problems

---

## Problem Statement

> Given an integer array `nums` and an integer `k`, return the **total number of subarrays** whose sum equals `k`.

```
Input:  nums = [1, 2, 3], k = 3
Output: 2
Explanation: [1,2] and [3]
```

This is the canonical application of the prefix-sum + hashmap pattern.

---

## From Brute Force to Optimal

### Approach 1: Brute Force — O(n³)

```
FUNCTION subarraySum_v1(nums, k):
    count = 0
    FOR i = 0 TO n-1:
        FOR j = i TO n-1:
            sum = 0
            FOR x = i TO j:          // Compute sum of subarray
                sum += nums[x]
            IF sum == k:
                count++
    RETURN count
```

### Approach 2: Prefix Sum Array — O(n²)

```
FUNCTION subarraySum_v2(nums, k):
    count = 0
    FOR i = 0 TO n-1:
        sum = 0
        FOR j = i TO n-1:
            sum += nums[j]          // Running sum
            IF sum == k:
                count++
    RETURN count
```

### Approach 3: HashMap — O(n) ✓

```
FUNCTION subarraySum_v3(nums, k):
    prefixCount = {0: 1}
    currentSum = 0
    count = 0

    FOR each num in nums:
        currentSum += num
        count += prefixCount.getOrDefault(currentSum - k, 0)
        prefixCount[currentSum] = prefixCount.getOrDefault(currentSum, 0) + 1

    RETURN count
```

---

## Complete Trace

```
nums = [1, -1, 1, 1, 1, -1], k = 2

Step │ num │ currSum │ target │ map                              │ found │ count
─────┼─────┼─────────┼────────┼──────────────────────────────────┼───────┼──────
init │  -  │    0    │   -    │ {0:1}                            │   -   │  0
  0  │  1  │    1    │   -1   │ {0:1, 1:1}                      │   0   │  0
  1  │ -1  │    0    │   -2   │ {0:2, 1:1}                      │   0   │  0
  2  │  1  │    1    │   -1   │ {0:2, 1:2}                      │   0   │  0
  3  │  1  │    2    │    0   │ {0:2, 1:2, 2:1}                 │   2   │  2
  4  │  1  │    3    │    1   │ {0:2, 1:2, 2:1, 3:1}            │   2   │  4
  5  │ -1  │    2    │    0   │ {0:2, 1:2, 2:2, 3:1}            │   2   │  6

Answer: 6

Verify all 6 subarrays with sum=2:
  [1,-1,1,1]     indices 0-3  sum = 2 ✓
  [1,1]          indices 2-3  sum = 2 ✓
  [-1,1,1,1]     indices 1-4  sum = 2 ✓ (wait: sum = 2 ✓)
  [1,1]          indices 3-4  sum = 2 ✓
  [1,-1,1,1,1,-1] indices 0-5 sum = 2 ✓
  [1,1,-1]       indices 2-4... Let me re-verify:

  Prefix sums: [0, 1, 0, 1, 2, 3, 2]
  Pairs where p[j]-p[i]=2:
    p[4]-p[0]=2-0=2 ✓  → [1,-1,1,1]
    p[4]-p[2]=2-0=2 ✓  → [1,1]
    p[5]-p[1]=3-1=2 ✓  → [-1,1,1,1]
    p[5]-p[3]=3-1=2 ✓  → [1,1]
    p[6]-p[0]=2-0=2 ✓  → [1,-1,1,1,1,-1]
    p[6]-p[2]=2-0=2 ✓  → [1,1,1,-1]
  All 6 verified ✓
```

---

## Handling Negative Numbers

```
╔══════════════════════════════════════════════════════════════╗
║  Unlike sliding window, this works with NEGATIVE numbers!  ║
║                                                              ║
║  Sliding window requires all-positive (monotone prefix)     ║
║  HashMap approach works with ANY integers                   ║
║                                                              ║
║  nums = [3, -1, -1, 2], k = 3                              ║
║  prefix = [0, 3, 2, 1, 3]                                  ║
║                                                              ║
║  Subarrays with sum 3:                                      ║
║  p[1]-p[0] = 3-0 = 3 ✓  → [3]                             ║
║  p[4]-p[0] = 3-0 = 3 ✓  → [3,-1,-1,2]                     ║
║  p[4]-p[1] = 3-3 = 0 ✗                                    ║
║                                                              ║
║  Answer: 2                                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Edge Cases

```
Case 1: k = 0
  nums = [1, -1, 1, -1]
  prefix = [0, 1, 0, 1, 0]
  Pairs where p[j]-p[i]=0 means p[j]==p[i]:
    p[2]=p[0]=0 → [1,-1]
    p[4]=p[0]=0 → [1,-1,1,-1]
    p[4]=p[2]=0 → [1,-1]
    p[3]=p[1]=1 → [-1,1]
  Answer: 4

Case 2: All same values
  nums = [1, 1, 1], k = 2
  prefix = [0, 1, 2, 3]
  p[2]-p[0]=2 ✓, p[3]-p[1]=2 ✓
  Answer: 2

Case 3: Single element
  nums = [5], k = 5
  prefix = [0, 5]
  p[1]-p[0]=5 ✓
  Answer: 1
```

---

## Common Mistakes

```
╔══════════════════════════════════════════════════════════════╗
║                  COMMON MISTAKES                            ║
║                                                              ║
║  1. Forgetting {0: 1} initialization                       ║
║     → Misses subarrays starting at index 0                 ║
║                                                              ║
║  2. Using sliding window instead                            ║
║     → Doesn't work with negative numbers                   ║
║                                                              ║
║  3. Storing index instead of count                          ║
║     → Same prefix sum can appear multiple times             ║
║     → Must count all occurrences                           ║
║                                                              ║
║  4. Updating map before the lookup                          ║
║     → Current prefix might match itself                    ║
║     → Always lookup FIRST, then insert                     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Variants of This Problem

| Problem | Modification |
|---------|-------------|
| **Subarray sum = 0** | Same algorithm with k=0 |
| **Subarray divisible by K** | Store `prefixSum mod K` as key |
| **Binary array equal 0s and 1s** | Replace 0→-1, then subarray sum=0 |
| **Longest subarray sum K** | Store first index of each prefix sum |
| **Count of subarrays with at most sum K** | Can't use this pattern directly |
| **Subarray sum in range [lo, hi]** | Two lookups: count(≥lo) - count(≥hi+1) |

---

## Complexity Comparison

| Approach | Time | Space |
|----------|:---:|:---:|
| Brute force (3 loops) | $O(n^3)$ | $O(1)$ |
| Running sum (2 loops) | $O(n^2)$ | $O(1)$ |
| Prefix sum + HashMap | $O(n)$ | $O(n)$ |

For $n = 100{,}000$: brute force ≈ $10^{15}$ ops (impossible), HashMap ≈ $10^5$ ops (instant).

---

## Quick Revision Question

**Q: Given `nums = [2, 4, -2, 4, -4, 2]` and `k = 4`, trace through the prefix sum HashMap algorithm step by step and find the count.**

<details>
<summary>Click to reveal answer</summary>

```
prefix sums: [0, 2, 6, 4, 8, 4, 6]

Step │ num │ curr │ target(curr-4) │ map                     │ found │ count
─────┼─────┼──────┼────────────────┼─────────────────────────┼───────┼──────
init │  -  │  0   │       -        │ {0:1}                   │   -   │  0
  0  │  2  │  2   │      -2        │ {0:1, 2:1}             │   0   │  0
  1  │  4  │  6   │       2        │ {0:1, 2:1, 6:1}        │   1   │  1
  2  │ -2  │  4   │       0        │ {0:1, 2:1, 6:1, 4:1}   │   1   │  2
  3  │  4  │  8   │       4        │ {0:1,2:1,6:1,4:1,8:1}  │   1   │  3
  4  │ -4  │  4   │       0        │ {0:1,2:1,6:1,4:2,8:1}  │   1   │  4
  5  │  2  │  6   │       2        │ {0:1,2:1,6:2,4:2,8:1}  │   1   │  5

Answer: **5** subarrays with sum = 4
```

The 5 subarrays: `[4]` (idx 1), `[2,4,-2]` (idx 0-2), `[4,-2,4,-4,2]` (idx 1-5), `[4]` (idx 3), and `[4,-4,2,... ]`... 

Verification via prefix pairs where `p[j]-p[i]=4`: (6-2), (4-0), (8-4), (4-0 at position 4), (6-2 at index 5 using 2 at index 0) = 5 pairs ✓

</details>

---

| [← Previous: Prefix Sum with HashMap](04-prefix-sum-hashmap.md) | [Next: Longest Substring Without Repeating →](06-longest-substring-no-repeat.md) |
|:---|---:|
