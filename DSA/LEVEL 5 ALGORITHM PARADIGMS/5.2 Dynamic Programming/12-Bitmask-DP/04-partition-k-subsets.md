# Chapter 4: Partition into K Equal Sum Subsets — Bitmask DP

## 📋 Overview
Given an array of integers, determine if it can be partitioned into **K subsets** each with **equal sum**. Bitmask DP checks all possible subset assignments in $O(n \cdot 2^n)$, feasible for $n \leq 20$.

---

## 🧠 Core Idea

### Problem Setup
```
Input:  nums = [4, 3, 2, 3, 5, 2, 1], k = 4
Total sum = 20, target per subset = 20/4 = 5

Find 4 subsets each summing to 5:
  {4,1}, {3,2}, {3,2}, {5}  ✓

Early termination:
  If total % k != 0 → impossible
  If max(nums) > target → impossible
```

### State Definition
```
dp[mask] = true if the elements selected by 'mask'
           can be perfectly partitioned into
           complete groups of sum = target

Alternatively, track running sum within current bucket:
  dp[mask] = remaining capacity in current bucket
             when elements in mask are already assigned
           = -1 if impossible
```

### Efficient Approach: Track Current Bucket Sum
```
dp[mask] = the sum used in the current (incomplete) bucket
           when elements in 'mask' have been placed

When sum reaches target → bucket complete, start new bucket

Transition:
  For each element i NOT in mask:
    if dp[mask] + nums[i] <= target:
      new_mask = mask | (1 << i)
      dp[new_mask] = (dp[mask] + nums[i]) % target
```

---

## 🔍 Step-by-Step Trace

```
nums = [2, 2, 2, 2, 3, 3], k = 3
total = 14 → 14 % 3 ≠ 0 → IMPOSSIBLE

nums = [2, 2, 2, 2, 3, 3], k = 2
total = 14 → target = 7

dp[000000] = 0  (nothing placed, current bucket sum = 0)

Place nums[0]=2: dp[000001] = (0+2)%7 = 2
Place nums[1]=2: dp[000010] = 2
Place nums[4]=3: dp[010000] = 3

From dp[000001]=2, add nums[4]=3:
  dp[010001] = (2+3)%7 = 5

From dp[010001]=5, add nums[1]=2:
  dp[010011] = (5+2)%7 = 0  ← bucket complete! (2+3+2=7)

From dp[010011]=0, add nums[2]=2:
  dp[010111] = 2

From dp[010111]=2, add nums[3]=2:
  dp[011111] = 4

From dp[011111]=4, add nums[5]=3:
  dp[111111] = (4+3)%7 = 0  ← second bucket complete!

dp[111111] = 0 → all elements placed in complete buckets ✓
Answer: TRUE
```

---

## 💻 Pseudocode

```
function canPartitionKSubsets(nums, k):
    total = sum(nums)
    if total % k != 0: return false
    target = total / k
    if max(nums) > target: return false
    
    n = len(nums)
    FULL = (1 << n) - 1
    
    // dp[mask] = current bucket sum, -1 if impossible
    dp = array [FULL+1] = -1
    dp[0] = 0
    
    // Sort descending for better pruning
    sort nums descending
    
    for mask = 0 to FULL - 1:
        if dp[mask] == -1: continue
        
        for i = 0 to n-1:
            if mask & (1 << i): continue  // already used
            
            if dp[mask] + nums[i] <= target:
                new_mask = mask | (1 << i)
                dp[new_mask] = (dp[mask] + nums[i]) % target
                // Note: % target resets to 0 when bucket completes
            
            // Optimization: only try first unused element
            // at each bucket start (symmetry breaking)
            if dp[mask] == 0: break
    
    return dp[FULL] == 0
```

---

## 🔧 Key Optimization: Symmetry Breaking

```
When current bucket sum = 0 (starting new bucket):
  Only try the first unused element.
  
Why? The order of filling buckets doesn't matter.
  {A,B} then {C,D} = {C,D} then {A,B}

Without: O(n · 2ⁿ) states, each O(n) transitions
With:    Significantly pruned in practice

  dp[mask] == 0: try only first unset bit
  dp[mask] != 0: try all remaining elements
```

```
Without symmetry breaking:
  Bucket 1: {1,4}, Bucket 2: {2,3}
  Bucket 1: {2,3}, Bucket 2: {1,4}  ← redundant

With symmetry breaking (always start with lowest index):
  Only: Bucket 1: {1,...}, Bucket 2: {next,...}
```

---

## 🆚 Alternative Approaches

```
1. Backtracking with pruning
   Time: O(k^n) worst case
   Good for small k, works well with pruning
   
2. Bitmask DP
   Time: O(n · 2ⁿ)
   Guaranteed polynomial in 2ⁿ, works for n ≤ 20-23

3. Meet in the middle
   Split array in half, enumerate subsets of each half
   Time: O(3^(n/2))
   For n up to ~40

Recommendation:
  k small → backtracking
  k large, n ≤ 20 → bitmask DP
```

---

## ⚡ Complexity Analysis

```
┌──────────────────────────────────────────┐
│ Time:  O(n · 2ⁿ)                        │
│   2ⁿ masks, each tries up to n elements │
│                                          │
│ Space: O(2ⁿ)                            │
│   dp array indexed by mask               │
│                                          │
│ With symmetry breaking:                  │
│   Same worst case but much faster in     │
│   practice                               │
│                                          │
│ Feasible: n ≤ 20                         │
└──────────────────────────────────────────┘
```

---

## 🧩 Related Problems

| Problem | Modification |
|---------|-------------|
| Partition into 2 subsets (equal sum) | Subset sum with target = total/2. O(n·sum) DP |
| Minimum difference partition | Standard knapsack, no bitmask needed |
| Fair distribution of cookies | k children, minimize max sum → bitmask + backtrack |
| Matchsticks to square | k=4, partition into 4 equal subsets |
| Split array with equal sum | Finding split points, prefix sums |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Partition array into K equal-sum subsets |
| **State** | dp[mask] = current bucket partial sum |
| **Key trick** | Modulo target resets on bucket completion |
| **Symmetry** | At bucket start, fix first unused element |
| **Complexity** | O(n · 2ⁿ) time, O(2ⁿ) space |
| **Prerequisite** | total % k == 0 and max ≤ target |

---

## ❓ Quick Revision Questions

1. **What early termination checks should you do before starting DP?**
2. **What does dp[mask] store and how does modulo trick work?**
3. **Why is symmetry breaking important and how does it work?**
4. **When would you prefer backtracking over bitmask DP?**
5. **How do you verify a full partition is valid at the end?**
6. **What is the key difference from standard subset sum partition?**

---

[← Previous: Hamiltonian Path](03-hamiltonian-path.md) | [Next: Parallel Courses →](05-parallel-courses.md)

[← Back to README](../README.md)
