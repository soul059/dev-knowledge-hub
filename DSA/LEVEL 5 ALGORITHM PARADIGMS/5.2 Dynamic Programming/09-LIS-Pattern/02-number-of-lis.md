# Chapter 2: Number of Longest Increasing Subsequences

## 📋 Overview
Given an integer array, return the **number of longest increasing subsequences**. This extends LIS by tracking not just length but also the **count** of subsequences achieving that length at each position.

---

## 🧠 Core Concept

```
nums = [1, 3, 5, 4, 7]

LIS length = 4

LIS sequences:
  [1, 3, 5, 7]  ✓
  [1, 3, 4, 7]  ✓

Answer: 2

Key insight: maintain TWO arrays:
  dp[i]   = length of LIS ending at i
  count[i] = number of LIS of that length ending at i
```

---

## 🔨 Algorithm

```
function numberOfLIS(nums):
    n = len(nums)
    dp    = array of size n, all 1     // length
    count = array of size n, all 1     // count of that length
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i]:
                if dp[j] + 1 > dp[i]:
                    dp[i] = dp[j] + 1
                    count[i] = count[j]       // new longer → inherit count
                elif dp[j] + 1 == dp[i]:
                    count[i] += count[j]      // same length → add count
    
    maxLen = max(dp)
    result = 0
    for i = 0 to n-1:
        if dp[i] == maxLen:
            result += count[i]
    
    return result
```

---

## 🔬 Trace: nums = [1, 3, 5, 4, 7]

```
Initial:
  dp    = [1, 1, 1, 1, 1]
  count = [1, 1, 1, 1, 1]

i=1 (num=3):
  j=0: nums[0]=1 < 3, dp[0]+1=2 > dp[1]=1
       dp[1]=2, count[1]=count[0]=1
  dp=[1,2,1,1,1], count=[1,1,1,1,1]

i=2 (num=5):
  j=0: 1<5, dp[0]+1=2 > 1 → dp[2]=2, count[2]=1
  j=1: 3<5, dp[1]+1=3 > 2 → dp[2]=3, count[2]=count[1]=1
  dp=[1,2,3,1,1], count=[1,1,1,1,1]

i=3 (num=4):
  j=0: 1<4, dp[0]+1=2 > 1 → dp[3]=2, count[3]=1
  j=1: 3<4, dp[1]+1=3 > 2 → dp[3]=3, count[3]=count[1]=1
  j=2: 5>4, skip
  dp=[1,2,3,3,1], count=[1,1,1,1,1]

i=4 (num=7):
  j=0: 1<7, dp[0]+1=2 > 1 → dp[4]=2, count[4]=1
  j=1: 3<7, dp[1]+1=3 > 2 → dp[4]=3, count[4]=1
  j=2: 5<7, dp[2]+1=4 > 3 → dp[4]=4, count[4]=count[2]=1
  j=3: 4<7, dp[3]+1=4 == 4 → count[4]+=count[3]=1 → count[4]=2
  dp=[1,2,3,3,4], count=[1,1,1,1,2]

maxLen = 4
result = count[4] = 2 ✓
```

---

## 🔑 Understanding Count Propagation

```
Visualization of count flow:

  Index:  0    1    2    3    4
  Num:    1    3    5    4    7
  Len:    1    2    3    3    4
  Cnt:    1    1    1    1    2

  Flow:
    1 ─→ 3 ─→ 5 ─→ 7   (one path, count=1)
    1 ─→ 3 ─→ 4 ─→ 7   (another path, count=1)
                         Total at 7: 1+1 = 2

  Three cases when nums[j] < nums[i]:
  ┌────────────────────────────────────────────┐
  │ dp[j]+1 > dp[i]  → found longer LIS       │
  │   dp[i] = dp[j]+1                         │
  │   count[i] = count[j]  (reset to new cnt) │
  ├────────────────────────────────────────────┤
  │ dp[j]+1 == dp[i] → found another of same  │
  │   count[i] += count[j]  (accumulate)      │
  ├────────────────────────────────────────────┤
  │ dp[j]+1 < dp[i]  → shorter, ignore        │
  └────────────────────────────────────────────┘
```

---

## 🌐 Example with Duplicates

```
nums = [2, 2, 2, 2, 2]

Every element is equal → no increasing pair.
LIS length = 1 (each element alone)

dp    = [1, 1, 1, 1, 1]
count = [1, 1, 1, 1, 1]

maxLen = 1
result = 5

nums = [1, 2, 4, 3, 5, 4, 7, 2]

                    ┌──4──┐
              1──2──┤     ├──5──┐
                    └──3──┘     ├──7
                          4─────┘

  dp    = [1, 2, 3, 3, 4, 4, 5, 2]
  count = [1, 1, 1, 1, 2, 1, 3, 1]

  Paths to 7 (len 5):
    1,2,4,5,7  (via index 2→4)
    1,2,3,5,7  (via index 3→4)
    1,2,3,4,7  (via index 3→5)

  Answer: 3
```

---

## ⚡ O(n log n) Approach with Segment Tree

```
For large n, use a segment tree over values:

1. Coordinate compress values
2. Process elements left to right
3. For each num, query segment tree for (max_len, total_count)
   among all values < num
4. Update segment tree at position num with (max_len+1, count)

Each node stores (length, count):
  merge: if len1 > len2 → (len1, cnt1)
         if len1 < len2 → (len2, cnt2)
         if len1 == len2 → (len1, cnt1 + cnt2)

Time: O(n log n), Space: O(n)

(Advanced — the O(n²) approach is sufficient for interviews)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i] = LIS length at i, count[i] = number of LIS at i |
| **New longer** | dp[i] = dp[j]+1, count[i] = count[j] |
| **Same length** | count[i] += count[j] |
| **Answer** | Sum of count[i] where dp[i] == max(dp) |
| **Complexity** | O(n²) time, O(n) space |
| **Optimized** | O(n log n) with segment tree |

---

## ❓ Quick Revision Questions

1. **What two arrays do you maintain?**
2. **When do you reset count[i] vs. accumulate into it?**
3. **How do you compute the final answer from dp and count arrays?**
4. **What happens when all elements are equal?**
5. **How would a segment tree improve the time complexity?**
6. **Can you extend this to also reconstruct all LIS sequences?**

---

[← Previous: Longest Increasing Subsequence](01-longest-increasing-subsequence.md) | [Next: Russian Doll Envelopes →](03-russian-doll-envelopes.md)

[← Back to README](../README.md)
