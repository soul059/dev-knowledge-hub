# Chapter 5: Bitonic Subsequence

## 📋 Overview
A **bitonic subsequence** first increases then decreases (or only increases, or only decreases). Finding the **Longest Bitonic Subsequence** combines two LIS computations — one from the left and one from the right.

---

## 🧠 Core Concept

```
nums = [1, 11, 2, 10, 4, 5, 2, 1]

Bitonic pattern: ↗ peak ↘

                   11
                  /
                10
               /  \
              /    5
             /      \
            2        2
           /          \
          1            1

Bitonic subsequence: [1, 2, 10, 5, 2, 1] length 6

  Increasing part: 1, 2, 10  (LIS from left)
  Decreasing part: 10, 5, 2, 1  (LIS from right = LDS from left)
  Peak element: 10

Answer: 6
```

---

## 🔨 Algorithm

```
Step 1: Compute LIS ending at each index (left to right)
  lis[i] = length of LIS ending at index i

Step 2: Compute LDS ending at each index (right to left)
  lds[i] = length of longest decreasing subsequence starting at i
         = LIS computed from right to left

Step 3: For each index i (potential peak):
  bitonic_length[i] = lis[i] + lds[i] - 1  (peak counted once)

Answer: max(bitonic_length)

function longestBitonic(nums):
    n = len(nums)
    
    // LIS from left
    lis = array of size n, all 1
    for i = 1 to n-1:
        for j = 0 to i-1:
            if nums[j] < nums[i]:
                lis[i] = max(lis[i], lis[j] + 1)
    
    // LIS from right (= LDS from left)
    lds = array of size n, all 1
    for i = n-2 down to 0:
        for j = n-1 down to i+1:
            if nums[j] < nums[i]:
                lds[i] = max(lds[i], lds[j] + 1)
    
    // Combine
    maxLen = 0
    for i = 0 to n-1:
        maxLen = max(maxLen, lis[i] + lds[i] - 1)
    
    return maxLen
```

---

## 🔬 Trace: nums = [1, 11, 2, 10, 4, 5, 2, 1]

```
Index:  0   1   2   3   4   5   6   7
Nums:   1  11   2  10   4   5   2   1

LIS (left to right):
  lis[0] = 1 (1)
  lis[1] = 2 (1,11)
  lis[2] = 2 (1,2)
  lis[3] = 3 (1,2,10)
  lis[4] = 3 (1,2,4)
  lis[5] = 4 (1,2,4,5)
  lis[6] = 2 (1,2)
  lis[7] = 1 (1)
  lis = [1, 2, 2, 3, 3, 4, 2, 1]

LDS (right to left):
  lds[7] = 1 (1)
  lds[6] = 2 (2,1)
  lds[5] = 3 (5,2,1)
  lds[4] = 3 (4,2,1)
  lds[3] = 4 (10,5,2,1) or (10,4,2,1)
  lds[2] = 2 (2,1)
  lds[1] = 5 (11,10,5,2,1) or (11,10,4,2,1)
  lds[0] = 1 (1 — nothing less to the right? no, 1≤1)
  
  Actually lds[0]: j>0 where nums[j]<nums[0]=1? Only 1, not <1.
  lds[0] = 1
  lds = [1, 5, 2, 4, 3, 3, 2, 1]

Combine (lis + lds - 1):
  i=0: 1+1-1 = 1
  i=1: 2+5-1 = 6   ← peak at 11
  i=2: 2+2-1 = 3
  i=3: 3+4-1 = 6   ← peak at 10
  i=4: 3+3-1 = 5
  i=5: 4+3-1 = 6   ← peak at 5
  i=6: 2+2-1 = 3
  i=7: 1+1-1 = 1

Answer: 6
```

---

## 🌐 Variants

### Strictly Bitonic
```
Must have BOTH increasing and decreasing parts:
  lis[i] ≥ 2 AND lds[i] ≥ 2

function strictlyBitonic(nums):
    // Same lis, lds computation
    maxLen = 0
    for i = 0 to n-1:
        if lis[i] >= 2 and lds[i] >= 2:
            maxLen = max(maxLen, lis[i] + lds[i] - 1)
    return maxLen
```

### Mountain Array Check
```
Valid mountain: increases then decreases, length ≥ 3, peak not at ends.

function isMountain(arr):
    n = len(arr)
    if n < 3: return false
    
    i = 0
    while i < n-1 and arr[i] < arr[i+1]:
        i++
    
    if i == 0 or i == n-1: return false  // no increase or no decrease
    
    while i < n-1 and arr[i] > arr[i+1]:
        i++
    
    return i == n-1
```

### Longest Mountain in Array (Subarray)
```
Find longest contiguous mountain subarray:

function longestMountain(arr):
    n = len(arr)
    maxLen = 0
    i = 1
    while i < n-1:
        if arr[i] > arr[i-1] and arr[i] > arr[i+1]:  // peak
            left = i, right = i
            while left > 0 and arr[left-1] < arr[left]:
                left--
            while right < n-1 and arr[right] > arr[right+1]:
                right++
            maxLen = max(maxLen, right - left + 1)
            i = right + 1
        else:
            i++
    return maxLen
```

---

## 🔑 Visualization of LIS + LDS

```
nums: [1, 11, 2, 10, 4, 5, 2, 1]

LIS lengths (→):  1  2  2  3  3  4  2  1
                   ↗  ↗     ↗  ↗           (going up)

LDS lengths (←):  1  5  2  4  3  3  2  1
                      ↘     ↘  ↘  ↘  ↘     (going down)

Combined (peak candidates):
  Peak at index 1 (val=11): 2+5-1=6
  Peak at index 3 (val=10): 3+4-1=6
  Peak at index 5 (val=5):  4+3-1=6

Three equally long bitonic subsequences of length 6!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Definition** | Sequence that increases then decreases |
| **Method** | LIS forward + LIS backward |
| **Formula** | bitonic[i] = lis[i] + lds[i] - 1 |
| **Peak** | Index i where bitonic[i] is maximized |
| **Time** | O(n²) with basic LIS, O(n log n) with binary search |
| **Space** | O(n) |

---

## ❓ Quick Revision Questions

1. **What is a bitonic sequence?**
2. **Why subtract 1 in lis[i] + lds[i] - 1?**
3. **How do you compute longest decreasing subsequence from the right?**
4. **What additional condition makes a subsequence "strictly bitonic"?**
5. **What is the difference between bitonic subsequence and mountain array?**
6. **Can a purely increasing or purely decreasing sequence be bitonic?**

---

[← Previous: Max Sum Increasing Subsequence](04-max-sum-increasing-subsequence.md) | [Next: Building Bridges →](06-building-bridges.md)

[← Back to README](../README.md)
